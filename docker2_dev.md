<!--
{
"name":"201501021",
"author": "ckeyer",
"head": "http://moefq.com/images/2015/11/23/2341564017cc8b9a8e6a19963f82125b.png",
"date": "2015-10-21",
"title": "Docker学习笔记2-Docker源码编译",
"tags": ["Docker", "Golang"],
"category": ["CaaS"],
"status": "publish",
"summary": "从0开始，Docker源码编译"
}
-->

为什么要从0搭一个docker的编译环境呢————折腾。

# 基本环境
Mac OSX ，boot2docker

编译docker代码之前，先大概研究了下docker的代码结构以及官方文档，通过文档和代码了解到docker官方推荐的是在docker本身的容器里面搭建环境和编译，而且在源码中给出的是一个基于ubuntu的dockerfile。好，那就可以更清晰安装流程了。

首先获取ubuntu的镜像，官方Dockerfile中使用的是14.04，我当然直接pull了最新的15.04

```
docker pull ubuntu:15.04
```
然后就是更新源了，直接改了

```
cp /etc/apt/sources.list /etc/apt/sources.listbk
echo "deb http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse"> /etc/apt/sources.list
```

然后需要更新和安装一些实用工具了，如wget,vim（这个ubuntu镜像里面连vi都木有的，所以上面用echo了），这一步不多说了，接着需要安装许多编译需要的包了，比如aufs相关的，sqlite相关的，这些都是在源码[Dockerfile](https://github.com/docker/docker/blob/master/Dockerfile)中看到的，就直接都安装了。

这些安装完后，发现[Dockerfile](https://github.com/docker/docker/blob/master/Dockerfile)紧接着还需要源码安装lvm和lxc，当然只能跟着做了。

接下来就是Go的环境了，毫无悬念，我选择1.5.1了。
先配置完环境变量，下载到[o1.5.1](https://golang.org/dl/go1.5.1.src.tar.gz)的源码，运行```make.bash```时，提示还需要一个旧版本的go，这个很好弄，直接下载官方提供的[linux_amd64](http://golangtc.com/static/go/go1.4.2.linux-amd64.tar.gz)的包,解压到用户根目录就OK了。然后重新编译go1.5，一切顺利。

```
# go version
go version go1.5.1 linux/amd64
```

剩下的当然就是Docker了，在这之前，我果断先保存了下当前镜像（Golang1.5）

```
$ docker commit 845ed53fb5f1 golang:1.5
```

关于docker的安装，按源码中的shell来看，几个依赖包都先clone后checkout到指定分支的，是为了防止依赖包更新导致问题。考虑了一下，直接```go get -v github.com/docker/docker```了，然后跳到hack目录下直接make，世界安静了...



# So.

