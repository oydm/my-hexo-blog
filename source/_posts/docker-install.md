---
title: CentOS 安装 Docker CE
date: 2018-06-12 14:23:55
categories:
- Docker
tags:
- 安装
- CentOS
- Docker
---

近期了解到基于Docker部署的Pass云平台，公司部分项目迁移部署在Docker云平台。对Docker并不了解，准备开始熟悉一下，先成安装起。

## 安装环境
- CentOS 7 VMware虚拟机
- Docker CE

## 卸载旧版本
确保没有旧版本的Docker残留文件，旧版本的 Docker 称为 docker 或者 docker-engine ，使用以下命令卸载旧版本：
``` bash
$ sudo yum remove docker \
	docker-client \
	docker-client-latest \
	docker-common \
	docker-latest \
	docker-latest-logrotate \
	docker-logrotate \
	docker-selinux \
	docker-engine-selinux \
	docker-engine
```
## 使用 yum 安装
执行以下命令安装依赖包：
``` bash
$ sudo yum install -y yum-utils \
	device-mapper-persistent-data \
	lvm2
```
鉴于国内网络问题，强烈建议使用国内源，执行下面的命令添加 yum 软件源：
``` bash
$ sudo yum-config-manager \
	--add-repo \
	https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
```
## 安装 Docker CE
更新 yum 软件源缓存，并安装 docker-ce 。
``` bash
$ sudo yum makecache fast
$ sudo yum install docker-ce
```

## 使用脚本自动安装
在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS
系统上可以使用这套脚本安装：
``` bash
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```
执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安
装在系统中。
如果在安装过程中出现问题如下，为centos内核版本低，部分功能（如 overlay2 存储层驱动）无法使用，且部分功能可能不太稳定。：
``` bash
Error: Package: docker-ce-18.05.0.ce-3.el7.centos.x86_64 (docker-ce-edge)
           Requires: container-selinux >= 2.9
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodiges
```
请采用以下安装方法
要先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错
``` bash
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```
然后再安装 docker-ce-17.03.2.ce，就能正常安装
``` bash
sudo yum install docker-ce-17.03.2.ce-1.el7.centos
```
安装过程中出现
``` bash
Running transaction
  Installing : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                        1/2
  Installing : docker-ce-17.03.2.ce-1.el7.centos.x86_64                                                                                                                  2/2
  Verifying  : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                        1/2
  Verifying  : docker-ce-17.03.2.ce-1.el7.centos.x86_64                                                                                                                  2/2

Installed:
  docker-ce.x86_64 0:17.03.2.ce-1.el7.centos                                                                                                                                 

Dependency Installed:
  libtool-ltdl.x86_64 0:2.4.2-22.el7_3                                                                                                                                       

Complete!

```
安装成功！

## 测试 Docker 是否安装正确
``` bash

$ sudo docker run hello-world
	Unable to find image 'hello-world:latest' locally
	latest: Pulling from library/hello-world
	9bb5a5d4561a: Pull complete
	Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
	Status: Downloaded newer image for hello-world:latest

	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
	 1. The Docker client contacted the Docker daemon.
	 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
	    (amd64)
	 3. The Docker daemon created a new container from that image which runs the
	    executable that produces the output you are currently reading.
	 4. The Docker daemon streamed that output to the Docker client, which sent it
	    to your terminal.

	To try something more ambitious, you can run an Ubuntu container with:
	 $ docker run -it ubuntu bash

	Share images, automate workflows, and more with a free Docker ID:
	 https://hub.docker.com/

	For more examples and ideas, visit:
	 https://docs.docker.com/engine/userguide/

```
若能正常输出以上信息，则说明安装成功。

## 镜像加速
国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务。
请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存
在请新建该文件）
``` javascript
{
	"registry-mirrors": [
		"https://registry.docker-cn.com"
	]
}
```
> 注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

之后重新启动服务。
``` bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
