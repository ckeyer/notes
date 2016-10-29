前些日子，自己玩一个golang项目，为了方便，写了个`Makefile`,大致内容如下：

```makefile
PWD := $(shell pwd)
PKG := github.com/ckeyer/bat
TMPDIR := bundles

build:
  go build -o $(TMPDIR)/app main.go

/// 其它忽略。。。
```

在docker容器内执行`make build`正常执行，但是在max osx下报错，类似：
```
go build -o bundles/tmp/bat cli/main.go
# github.com/ckeyer/bat/vendor/github.com/oxtoacart/bpool
can't create $WORK/github.com/ckeyer/bat/vendor/github.com/oxtoacart/bpool.a: open $WORK/github.com/ckeyer/bat/vendor/github.com/oxtoacart/bpool.a: no such file or directory
# github.com/ckeyer/bat/vendor/github.com/codegangsta/inject
can't create $WORK/github.com/ckeyer/bat/vendor/github.com/codegangsta/inject.a: open $WORK/github.com/ckeyer/bat/vendor/github.com/codegangsta/inject.a: no such file or directory
# github.com/ckeyer/bat/vendor/github.com/Sirupsen/logrus
can't create $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/logrus.a: open $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/logrus.a: no such file or directory
# github.com/ckeyer/bat/vendor/gopkg.in/urfave/cli.v2
can't create $WORK/github.com/ckeyer/bat/vendor/gopkg.in/urfave/cli.v2.a: open $WORK/github.com/ckeyer/bat/vendor/gopkg.in/urfave/cli.v2.a: no such file or directory
# net
can't create $WORK/net.a: open $WORK/net.a: no such file or directory
make: *** [local] Error 2
```
发现莫名其妙的错误，于是加上了build参数 `-x -work` 打印更多信息，即`go build -x -work -o $(TMPDIR)/app main.go`，在docker容器环境下的输出：
```
WORK=/tmp/go-build997336313
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/logrus/_obj/
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/codegangsta/inject/_obj/
.......
```
而在mac osx上的输出为：
```
WORK=bundles/tmp/go-build572081055
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/logrus/_obj/
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/Sirupsen/
mkdir -p $WORK/github.com/ckeyer/bat/vendor/github.com/codegangsta/inject/_obj/
........
```
发现问题就在这个`WORK`变量上，主要有两点：
1. osx上赋值`TMPDIR`时暂时覆盖了系统环境变量。
2. 新设置的`TMPDIR`是个相对路径，`go build`的时候，会跳转的各个包所在目录，此时的临时目录路径自然就变了，也就出现了找不到那几个文件的问题了。

由于发现`make`修改了环境变量，所以后对其进行了更多的尝试，为`Makefile`添加了一个tag:
```
env:
	env
```

首先在两种环境上分别执行`env`时，发现容器环境里面没有`TMPDIR`的环境变量，而osx是有的`TMPDIR=/var/folders/wj/j1ns0qdn47l1vv89nph5qp9c0000gn/T/`，执行`make env`后，容器环境里依然没有`TMPDIR`,但osx里的`TMPDIR`已经变成了`Makefile`的变量`TMPDIR`的值了，也就是`TMPDIR=bundles`,后查看go源码获取临时目录的部分，发现果然是先读取环境变量`TMPDIR`,如果没有再根据系统指定固定目录。

而另一方面，我在容器中执行了`export TMPDIR=/tmp`后，再执行以上`make build`，发现问题在容器里也重现了，说明make执行过程，如果有变量名与环境变量名相同，会直接修改到环境变量上，如果变量名没有出现在环境变量里面，这个变量则不会加入到环境变量里面去。
