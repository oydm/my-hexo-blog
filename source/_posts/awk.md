---
title: Linux awk 命令
date: 2018-03-20 14:09:08
categories:
- Linux
tags:
- Linux
---

> AWK是一种处理文本文件的语言，是一个强大的文本分析工具。之所以叫AWK是因为其取了三位创始人 Alfred Aho，Peter Weinberger, 和 Brian Kernighan 的Family Name的首字符。

### 语法
``` bash
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```
#### 选项参数说明：
* -F fs or --field-separator fs
指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。
* -v var=value or --asign var=value
赋值一个用户定义变量。
* -f scripfile or --file scriptfile
从脚本文件中读取awk命令。
* -mf nnn and -mr nnn
对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。
* -W compact or --compat, -W traditional or --traditional
在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。
* -W copyleft or --copyleft, -W copyright or --copyright
打印简短的版权信息。
* -W help or --help, -W usage or --usage
打印全部awk选项和每个选项的简短说明。
* -W lint or --lint
打印不能向传统unix平台移植的结构的警告。
* -W lint-old or --lint-old
打印关于不能向传统unix平台移植的结构的警告。
* -W posix
打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符**和**=不能代替^和^=；fflush无效。
* -W re-interval or --re-inerval
允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。
* -W source program-text or --source program-text
使用program-text作为源代码，可与-f命令混用。
* -W version or --version
打印bug报告信息的版本

### 基本用法
log.txt文本内容如下：
```
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```
用法一：
```
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
```
实例：
``` bash
# 每行按空格或TAB分割，输出文本中的1、4项
 $ awk '{print $1,$4}' log.txt
 ---------------------------------------------
 2 a
 3 like
 This's
 10 orange,apple,mongo
 # 格式化输出
 $ awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
 ---------------------------------------------
 2        a
 3        like
 This's
 10       orange,apple,mongo
```
用法二：
```
awk -F  #-F相当于内置变量FS, 指定分割字符
```
实例：
``` bash
# 使用","分割
 $  awk -F, '{print $1,$2}'   log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 或者使用内建变量
 $ awk 'BEGIN{FS=","} {print $1,$2}'     log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 使用多个分隔符.先使用空格分割，然后对分割结果再使用","分割
 $ awk -F '[ ,]'  '{print $1,$2,$5}'   log.txt
 ---------------------------------------------
 2 this test
 3 Are awk
 This's a
 10 There apple
 ```
用法三：
```
awk -v  # 设置变量
```
实例：
``` bash
 $ awk -va=1 '{print $1,$1+a}' log.txt
 ---------------------------------------------
 2 3
 3 4
 This's 1
 10 11
 $ awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt
 ---------------------------------------------
 2 3 2s
 3 4 3s
 This's 1 This'ss
 10 11 10s
```
用法四：
```
awk -f {awk脚本} {文件名}
```
实例：
``` bash
 $ awk -f cal.awk log.txt
```
