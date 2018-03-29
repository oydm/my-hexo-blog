---
title: 基于Swoole的Process进程管理模块支付结果回调服务
date: 2018-03-15 11:21:14
categories:
- PHP
tags:
- Swoole
---

> 原先用PHP的Pthread多线程实现的支付结果回调服务，在后期运行中出现了服务停止的问题，在学习swoole的过程中，发现可以用swoole的Process进程管理模块实现多线程的功能，并且使用swoole_time_tick定时器功能实现进程监控在子进程退出的时候进行重启。

## 1、开发环境
Swoole版本：2.0.12
PHP版本：7.1
服务器版本：Ubuntu 14.04 64位

## 2、业务场景
游戏APP在支付后，在支付宝，微信等回调支付结果后，将支付结果回调给游戏服务器。回调逻辑为：25 小时以内完成 8 次通知（通知的间隔频率一般是：0s,2m,10m,10m,1h,2h,6h,15h）。第一次通知在接收到结果时同时回调。所以另外7次间隔性回调由7个进程分别操作。每个进程服务运行时间不一致，当前业务时间间隔各为1s，2s，30s，30s，60s，300s，600s，600s一次

## 3、代码实例

``` php
use Swoole\Process;

class MyProcess
{
    public $mpid = 0; // master pid, 即当前程序的进程ID
    public $works = []; // 记录子进程的 pid
    public $maxProcessNum = 7;
    public $newIndex = 1;

    public function __construct()
    {
        try {
            swoole_set_process_name(' MyProcess : master');
            $this->mpid = posix_getpid();
            $this->run();
            $this->processWait();
        } catch (\Exception $e) {
            die('Error: '. $e->getMessage());
        }
    }

    public function run()
    {
        //创建进程
        for ($i=0; $i<=$this->maxProcessNum; $i++) {
            $this->createProcess($i);
        }
    }


    public function createProcess($index = null)
    {
        if (is_null($index)) {
            $index = $this->newIndex;
            $this->newIndex++;
        }
        echo date('Y-m-d H:i:s') . '  |  createProcess index='.$index.PHP_EOL;
        $process = new swoole_process(function (swoole_process $worker) use($index) { // 子进程创建后需要执行的函数
            swoole_set_process_name(" MyProcess : worker $index");
            //根据进程启用不同时间间隔的定时器 $ms为毫秒 支付回调7次尝试 7个进程回调服务 每次回调的间隔时间不一致，实行25 小时以内完成 8 次通知（通知的间隔频率一般是：2m,10m,10m,1h,2h,6h,15h）
            switch ($index) {
                case 0;
                    $ms = 1000;
                    break;
                case 1;
                    $ms = 2000;
                    break;
                case 2;
                    $ms = 30000;
                    break;
                case 3;
                    $ms = 30000;
                    break;
                case 4;
                    $ms = 60000;
                    break;
                case 5;
                    $ms = 300000;
                    break;
                case 6;
                    $ms = 600000;
                    break;
                case 7;
                    $ms = 600000;
                    break;
            }
            //启用定时器
            $timer=swoole_timer_tick($ms,'MyProcess::deal_pay_notify', $index);

        }, false, false); // 不重定向输入输出; 不使用管道
        $pid = $process->start();
        $this->works[$index] = $pid;
        return $pid;
    }

    /*
        * 处理支付回调
        */
    function deal_pay_notify($timmerID, $params){
        echo date('Y-m-d H:i:s') . '  |  timmerID='.$timmerID." params=".$params.PHP_EOL;

        //支付结果回调操作
        //......

    }

    // 重启子进程
    public function rebootProcess($pid)
    {
        $index = array_search($pid, $this->works);
        if ($index !== false) {
            //重新创建进程
            $newPid = $this->createProcess($index);
            echo "rebootProcess: {$index}={$pid}->{$newPid} Done\n";
            return;
        }
        throw new \Exception("rebootProcess error: no pid {$pid}");
    }


    // 监控子进程
    public function processWait()
    {
        //定时器每秒监控
        swoole_timer_tick(1000,'MyProcess::monitor_process', '');
        /*while (1) {
            if (count($this->works)) {
                $ret = Process::wait(); // 子进程退出
                if ($ret) {
                    $this->rebootProcess($ret['pid']);
                }
            } else {
                break;
            }
        }*/
    }

    //检测进程
    public function monitor_process($timmerID, $params){
        foreach($this->works as $pid){
            if (!Process::kill($pid, 0)) { // 0 可以用来检测进程是否存在
                $this->rebootProcess($pid); //重启进程
                echo date('Y-m-d H:i:s') . '  |  monitor_process pid='.$pid. ' restart'.PHP_EOL;
            }
        }
    }
}

new MyProcess();
```
gitee：https://gitee.com/oydm/codes/ktc358gd1w02ufnel6zr413


