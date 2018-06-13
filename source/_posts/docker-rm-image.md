---
title: Docker 镜像使用 - 删除镜像
date: 2018-06-12 16:14:05
categories:
- Docker
tags:
- 镜像
---

## 删除本地镜像
如果要删除本地的镜像，可以使用 docker image rm 命令，其格式为：
``` bash
docker image rm [选项] <镜像1> [<镜像2> ...]
```

## 用 ID、镜像名、摘要删除镜像
其中， <镜像> 可以是 镜像短 ID 、 镜像长 ID 、 镜像名 或者 镜像摘要 。
比如我们有这么一些镜像：
``` bash
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        6 days ago          114 MB
redis               latest              bfcb1f6df2db        5 weeks ago         107 MB
hello-world         latest              e38bc07ac18e        2 months ago        1.85 kB
```
我们可以用镜像的完整 ID，也称为 长 ID ，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用 短 ID 来删除镜像。 docker image ls 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。
比如这里，如果我们要删除 redis:alpine 镜像，可以执行：
```bash
$ docker image rm bfcb1f6df2db
Untagged: redis:latest
Untagged: redis@sha256:87275ecd3017cdacd3e93eaf07e26f4a91d7f4d7c311b2305fccb50ec3a1a8cd
Deleted: sha256:bfcb1f6df2db8a62694aaa732a3133799db59c6fec58bfeda84e34299e7270a8
Deleted: sha256:1b2300a82ed1f2881032ae2fa5b6d570297faf1906e054ef1459e356b3fcc569
Deleted: sha256:2ed7c0db59ecf786fa13c85e88489c3c0e5eadc1a24c9da38a62b9f564aac880
Deleted: sha256:faa8313d6bff25bcd501468b3ebb45e3d002454fe7d3a6d72dba77dce0a64d55
Deleted: sha256:ba4122a82d251fc6b432237079bbb0d645ef47049e800c1f9e56d61fc8e823cf
Deleted: sha256:2425077af8e28faee2557ddabb85a00938d81d13cf8c41897958d07aaeaa39d2
Deleted: sha256:ba291263b0854589e32a6fa7775c898d662ed835cd686ac9ac2d33dcefa91a39
```
我们也可以用 镜像名 ，也就是 <仓库名>:<标签> ，来删除镜像。
```bash
$ docker image rm redis:latest
Untagged: redis:latest
Untagged: redis@sha256:87275ecd3017cdacd3e93eaf07e26f4a91d7f4d7c311b2305fccb50ec3a1a8cd
Deleted: sha256:bfcb1f6df2db8a62694aaa732a3133799db59c6fec58bfeda84e34299e7270a8
Deleted: sha256:1b2300a82ed1f2881032ae2fa5b6d570297faf1906e054ef1459e356b3fcc569
Deleted: sha256:2ed7c0db59ecf786fa13c85e88489c3c0e5eadc1a24c9da38a62b9f564aac880
Deleted: sha256:faa8313d6bff25bcd501468b3ebb45e3d002454fe7d3a6d72dba77dce0a64d55
Deleted: sha256:ba4122a82d251fc6b432237079bbb0d645ef47049e800c1f9e56d61fc8e823cf
Deleted: sha256:2425077af8e28faee2557ddabb85a00938d81d13cf8c41897958d07aaeaa39d2
Deleted: sha256:ba291263b0854589e32a6fa7775c898d662ed835cd686ac9ac2d33dcefa91a39
```
当然，更精确的是使用 镜像摘要 删除镜像。
```bash
$ docker image ls --digests
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
ubuntu              16.04               sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a   5e8b97a2a082        6 days ago          114 MB
hello-world         latest              sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77   e38bc07ac18e        2 months ago        1.85 kB
$ docker image rm  ubuntu@sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a
```

## 用 docker image ls 命令来配合
像其它可以承接多个实体的命令一样，可以使用 docker image ls -q 来配合使用 docker image rm ，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。
比如，我们需要删除所有仓库名为 redis 的镜像：
```bash
$ docker image rm $(docker image ls -q redis)
```
或者删除所有在 mongo:3.2 之前的镜像：
```bash
$ docker image rm $(docker image ls -q -f before=mongo:3.2)
```
充分利用你的想象力和 Linux 命令行的强大，你可以完成很多非常赞的功能。
