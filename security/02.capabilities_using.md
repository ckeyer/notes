# LINUX安全:  Capability 简单使用

在上一篇文章中已经讲到，Capabilities的主要思想在于分割root用户的特权，即将root的特权分割成不同的能力，每种能力代表一定的特权操作。

## 相关命令

### getcap

`getcap [-v] [-r] [-h] filename [ ... ]`

```bash
test :: ~ Â»getcap /usr/bin/ping
/usr/bin/ping = cap_net_admin,cap_net_raw+p
```

### setcap

`setcap [-q] [-v] (capabilities|-|-r) filename [ ... capabilitiesN fileN ]`

给指定的文件设置特权能力


### capsh

这是一个查看当前 shell 进程所拥有的 capabilities

### filecap

这是一个`libcap-ng`包下的命令，如果是Centos，需要先安装这个包`yum install libcap-ng-utils`

`filecap`命令可以用来管理文件的 capabilities，其不支持相对路径，只支持绝对路径，且不允许指定 capabilities 作用的集合，capabilities 只会被添加到 permitted 和 effective 集合。

如查看`filecap /full/path/to/file`

设置`filecap /full/path/to/file cap_name`

## 示例 setcap CAP_CHOWN

简单以一个`chown`的示例来实际感受一下`capabilities`的作用
先从`/usr/bin/`下把`chown`复制出来，修改所属用户
```
test :: ~ » whoami
root
test :: ~ » cd /home/testuser/
test :: ~ » cp /usr/bin/chown .
test :: ~ » ls -lh ./chown
-rwxr-xr-x  1 root         root          62K Nov  1 22:43 chown
test :: ~ » chown testuser:testuser ./chown
test :: ~ » ls -lh ./chown
-rwxr-xr-x 1 testuser testuser 62K Nov  1 22:43 ./chown
```

接下来切换到`testuser`用户来测试一下`chown`的功能

```
test :: ~ » whoami
testuser
test :: ~ » 
test :: ~ » touch aaa
test :: ~ » ls -alh ./aaa
-rw-rw-r-- 1 testsuiduser testsuiduser 0 Nov  4 22:45 aaa
test :: ~ » ./chown root:root ./aaa
./chown: changing ownership of './aaa': Operation not permitted
```

发现并没有更改文件属主的权限，接下来使用`setcap`对其进行授权后在尝试
```
test :: ~ » whoami
root
test :: ~ » setcap cap_chown=eip ./chown
test :: ~ » setcap cap_chown=eip ./chown
test :: ~ » getcap ./chown
./chown = cap_chown+eip
```
切回`testuser`，执行上次同样的命令
```
test :: ~ » whoami
testuser
test :: ~ » ./chown root:root ./aaa
test :: ~ » ls -alh ./aaa
-rw-rw-r--  1 root         root            0 Nov  1 22:45 aaa
```

linux中类似是使用场景还有很多，比如对敏感端口的授权等


## 参考连接

* https://fuckcloudnative.io/posts/linux-capabilities-why-they-exist-and-how-they-work/
* https://fuckcloudnative.io/posts/linux-capabilities-in-practice-1/
* https://fuckcloudnative.io/posts/linux-capabilities-in-practice-2/
* https://blog.csdn.net/weixin_42152531/article/details/120543324
