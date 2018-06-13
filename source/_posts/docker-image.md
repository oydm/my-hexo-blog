---
title: Docker 镜像使用 - 列出镜像
date: 2018-06-12 15:00:55
categories:
- Docker
tags:
- 镜像
---

Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。

## 运行环境
- CentOS 7 VMware虚拟机
- Docker CE

## 获取镜像
从 Docker 镜像仓库获取镜像的命令是 docker pull 。其命令格式为：
``` bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```
具体的选项可以通过 docker pull --help 命令看到，这里我们说一下镜像名称的格式。
- Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号] 。默认地址是 DockerHub。
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名> 。对于 DockerHub，如果不给出用户名，则默认为 library ，也就是官方镜像。

比如：
```bash
docker pull ubuntu:16.04
16.04: Pulling from library/ubuntu
b234f539f7a1: Pull complete
55172d420b43: Pull complete
5ba5bbeb6b91: Pull complete
43ae2841ad7a: Pull complete
f6c9c6de4190: Pull complete
Digest: sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a
Status: Downloaded newer image for ubuntu:16.04

```
> 上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。而镜像名称是 ubuntu:16.04 ，因此将会获取官方镜像 library/ubuntu 仓库中标签为 16.04 的镜像。

## 列出镜像
要想列出已经下载下来的镜像，可以使用 docker image ls 命令。
``` bash
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        6 days ago          114 MB
hello-world         latest              e38bc07ac18e        2 months ago        1.85 kB
```
> 列表包含了 仓库名 、 标签 、 镜像 ID 、 创建时间 以及 所占用的空间 。

## 虚悬镜像

镜像列表中，还可以看到一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为 <none> 。：
```bash
<none>              <none>               5e8b97a2a082        6 days ago          114 MB
```
这个镜像原本是有镜像名和标签的，原来为 mongo:3.2 ，随着官方镜像维护，发布了新版本后，重新 docker pull mongo:3.2 时， mongo:3.2 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 <none> 。除了 docker pull 可能导致这种情况， docker build 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 <none> 的镜像。这类无标签镜像也被称为 虚悬镜像(dangling image) ，可以用下面的命令专门显示这类镜像：
```bash
$ docker image ls -f dangling=true
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              5e8b97a2a082        6 days ago          114 MB
```
一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。
```bash
$ docker image prune
```

## 中间层镜像
为了加速镜像构建、重复利用资源，Docker 会利用 中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的 docker image ls 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 -a 参数。
```bash
$ docker image ls -a
```
这样会看到很多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为之前说过，相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份，无论如何你也会需要它们。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

## 列出部分镜像
不加任何参数的情况下， docker image ls 会列出所有顶级镜像，但是有时候我们只希望列出部分镜像。 docker image ls 有好几个参数可以帮助做到这个事情。根据仓库名列出镜像
```bash
$ docker image ls ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        6 days ago          114 MB
```
列出特定的某个镜像，也就是说指定仓库名和标签
```bash
$ docker image ls ubuntu:16.04
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        6 days ago          114 MB
```
除此以外， docker image ls 还支持强大的过滤器参数 --filter ，或者简写 -f 。之前我们已经看到了使用过滤器来列出虚悬镜像的用法，它还有更多的用法。

##以特定格式显示
默认情况下， docker image ls 会输出一个完整的表格，但是我们并非所有时候都会需要这些内容。比如，刚才删除虚悬镜像的时候，我们需要利用 docker image ls 把所有的虚悬镜像的 ID 列出来，然后才可以交给 docker image rm 命令作为参数来删除指定的这些镜像，这个时候就用到了 -q 参数。
```bash
$ docker image ls -q
5e8b97a2a082
e38bc07ac18e
```
> --filter 配合 -q 产生出指定范围的 ID 列表，然后送给另一个 docker 命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见，不仅仅是镜像，将来我们会在各个命令中看到这类搭配以完成很强大的功能。因此每次在文档看到过滤器后，可以多注意一下它们的用法。

另外一些时候，我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等
```bash
$ docker image ls --format "{{.ID}}: {{.Repository}}"
5e8b97a2a082: ubuntu
e38bc07ac18e: hello-world
```
或者打算以表格等距显示，并且有标题行，和默认一样，不过自己定义列：
```bash
$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
5e8b97a2a082        ubuntu              16.04
e38bc07ac18e        hello-world         latest
```

## 运行
有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。以上面的 ubuntu:16.04 为例，如果我们打算启动里面的 bash 并且进行交互式操作的话，可以执行下面的命令。
```bash
$ docker run -it ubuntu:16.04 bash
	root@e7009c6ce357:/# cat /etc/os-release
	NAME="Ubuntu"
	VERSION="16.04.4 LTS, Trusty Tahr"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 16.04.4 LTS"
	VERSION_ID="16.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```
docker run 就是运行容器的命令，具体格式我们会在 容器 一节进行详细讲解，我们这里简要的说明一下上面用到的参数。
- -it ：这是两个参数，一个是 -i ：交互式操作，一个是 -t 终端。我们这里打算进入bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
- --rm ：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容 器并不会立即删除，除非手动 docker rm,不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
- ubuntu:16.04 ：这是指用 ubuntu:16.04 镜像为基础来启动容器。
- bash ：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash 。

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了 cat /etc/os-release ，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 Ubuntu 16.04.4 LTS 系统。
最后我们通过 exit 退出了这个容器。
