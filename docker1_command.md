<!--
{
"name":"201501019",
"author": "ckeyer",
"head": "http://moefq.com/images/2015/11/23/2341564017cc8b9a8e6a19963f82125b.png",
"date": "2015-10-19",
"title": "Docker学习笔记1",
"tags": ["Docker"],
"category": ["学习笔记"],
"status": "publish",
"summary": "docker 中run,commit的使用"
}
-->

首先要说的是Docker中的镜像。镜像类似于已经包含了文件、配置和安装好的程序的虚拟机镜像。同样的，也可以像启动虚拟机一样启动多个镜像实例。运行中的镜像称为容器。可以修改容器（比如删除一个文件），但这些修改不会影响到镜像。不过，使用docker commit <container-id> <image-name>命令可以把一个正在运行的容器变成一个新的镜像。

00. 先pull一个用于测试的镜像

```
$ docker pull busybox
...
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox             latest              0064fda8c45d        5 days ago          1.113 MB
...
```

1. 先来看下镜像和容器的关系

```
$ docker run -it -p 8080:8080 busybox sh
/ # wget www.ckeyer.com/d/tiger
Connecting to www.ckeyer.com (198.148.115.4:80)
tiger                100% |************************|  6485k  0:00:00 ETA
/ # ls
bin    dev    etc    home   proc   root   sys    tiger  tmp    usr    var
/ # chmod 755 /tiger
/ # /tiger &

```
以上的tiger为一个以当前目录为根目录的文件服务器，端口8080.
访问 http://192.168.59.103:8080/ 可以看到
![](http://moefq.com/images/2015/11/23/799f2024c204ff4628ea14bccbe2b745.md.jpg)

```
/ # exit
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                     PORTS                         NAMES
fdbabef038f6        busybox               "sh"                     18 minutes ago      Up 18 minutes              0.0.0.0:8080->8080/tcp        determined_kowalevski
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox             latest              0064fda8c45d        5 days ago          1.113 MB
```
此时可以看到```docker ps ```列表中多了刚刚运行的那个容器，而且是Up状态，而通过```docker images```查看镜像，发现并没有多出一个。

2. commit出一个镜像

```
$ docker commit fdbabef038f6 busybox:t1.0
7eb0a583bf0d7a20965e00d8fe6552c793fb8381702101388cde47a5f7cb014f
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox             t1.0                7eb0a583bf0d        15 seconds ago      7.754 MB
busybox             latest              0064fda8c45d        5 days ago          1.113 MB
```
可以发现已经新建立了一个，运行试试

```
docker run -it -p 8081:8080 busybox:t1.0
/ # ls
bin    dev    etc    home   proc   root   sys    tiger  tmp    usr    var
/ # ps -ef |grep tiger
    7 root       0:00 grep tiger
```
我们发现此时容器中已经有了之前下载的``` tiger```文件，但是并没有保持原来的运行状态，为什么呢？
下面我们继续试验

```
$ docker run -it -d -p 8081:8080 busybox:t1.0 /tiger
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                     PORTS                         NAMES
18bca193fd98        busybox:t1.0          "/tiger"                 13 seconds ago      Up 12 seconds              0.0.0.0:8081->8080/tcp        loving_colden
fdbabef038f6        busybox               "sh"                     33 minutes ago      Exited (0) 7 seconds ago                                 determined_kowalevski
$ docker commit 18bca193fd98 busybox:t2.0
e78094135b68d3e05e6acf3c0c1234f93ca64b5955158bc1d521b0e153f18a47
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox             t2.0                e78094135b68        7 seconds ago       7.754 MB
busybox             t1.0                7eb0a583bf0d        7 minutes ago       7.754 MB
busybox             latest              0064fda8c45d        5 days ago          1.113 MB
$ docker run -it -p 8082:8080 busybox:t2.0
```
此时发现已经可以访问  http://192.168.59.103:8082/ 了

# So.

