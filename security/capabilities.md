
# Linux安全: 从SUID到Capability

Linux的Capability机制是一种Linux系统对特权操作的权限控制机制，在了解Linux的Capability机制之前，有必要先了解一下SUID。

## SUID

Linux中在执行某个程序时，默认情况下用户发起一个进程，进程的属主是进程的发起者，也就是说这个进程是以发起者的身份运行。而SUID则是让进程以所属者的身份运行，

下面用一个示例来说明一下这个机制

1. 使用`root`用户创建一个`----------`的文件`/tmp/testsuid.txt`
```bash
test :: ~ » whoami
root
test :: ~ » echo "testtesttest" > /tmp/test.txt
test :: ~ » chmod 000 /tmp/test.txt
test :: ~ » cat /tmp/test.txt
testtesttest
```
2. 创建普通用户`testsuiduser`,尝试读取文件`/tmp/testsuid.txt`
```bash
test :: ~ » whoami
root
test :: ~ » useradd testsuiduser
test :: ~ » su testsuiduser
test :: ~ » whoami
testsuiduser
test :: ~ » ls -lh /tmp/test.txt
4.0K ---------- 1 root root 4 10月  3 23:37 /tmp/test.txt
test :: ~ » cat /tmp/test.txt
cat: /tmp/test.txt: 权限不够
```
3. 此时我们发现普通用户是没有权限读取文件`/tmp/testsuid.txt`的，接下来给`cat`命令加上`SUID`权限
```bash
test :: ~ » whoami
root
test :: ~ » ls -lh $(which cat)
-rwxr-xr-x 1 root root 53K 11月 17 2020 /usr/bin/cat
test :: ~ » chmod u+s /usr/bin/cat
test :: ~ » ls -lh $(which cat)
-rwsr-xr-x 1 root root 53K 11月 17 2020 /usr/bin/cat
```
4. 再次使用普通用户`testsuiduser`，尝试读取文件`/tmp/testsuid.txt`
```bash
test :: ~ » whoami
root
test :: ~ » su testsuiduser
test :: ~ » whoami
testsuiduser
test :: ~ » ls -lh /tmp/test.txt
4.0K ---------- 1 root root 4 10月  3 23:37 /tmp/test.txt
test :: ~ » cat /tmp/test.txt
testtesttest
```

结果发现读出来了，这就是SUID的意义所在，在执行命令时，会以程序所属的用户来进行权限判断，而忽略当前实际调用的用户所有权。

通过以上尝试，也可以发现，在传统的Linux的权限控制机制中，SUID主要用于一些特殊命令的提权，实际会应用在比如`/bin/passwd`、`/bin/gpasswd`、`/bin/mount`、`/bin/su`等命令中。

## Capabilities简介

SUID 解决了部分权限控制的问题，但是问题在于控制太粗糙，并不能对root权限进行精细的划分授权，如此便有了Capabilities的必要。

Capabilities机制，是在Linux内核2.2之后引入的。它将root用户的权限细分为不同的领域，可以分别启用或禁用。从而，在实际进行特权操作时，如果euid不是root，便会检查是否具有该特权操作所对应的capabilities，并以此为依据，决定是否可以执行特权操作。

如下列出了大部分特权操作

| capability 名称 | 描述 |
| - | - |
| CAPAUDITCONTROL | 启用和禁用内核审计；改变审计过滤规则；检索审计状态和过滤规则 |
| CAPAUDITREAD | 允许通过 multicast netlink 套接字读取审计日志 |
| CAPAUDITWRITE | 将记录写入内核审计日志 |
| CAPBLOCKSUSPEND | 使用可以阻止系统挂起的特性 |
| CAP_CHOWN | 修改文件所有者的权限 |
| CAPDACOVERRIDE | 忽略文件的 DAC 访问限制 |
| CAPDACREAD_SEARCH | 忽略文件读及目录搜索的 DAC 访问限制 |
| CAP_FOWNER | 忽略文件属主 ID 必须和进程用户 ID 相匹配的限制 |
| CAP_FSETID | 允许设置文件的 setuid 位 |
| CAPIPCLOCK | 允许锁定共享内存片段 |
| CAPIPCOWNER | 忽略 IPC 所有权检查 |
| CAP_KILL | 允许对不属于自己的进程发送信号 |
| CAP_LEASE | 允许修改文件锁的 FL_LEASE 标志 |
| CAPLINUXIMMUTABLE | 允许修改文件的 IMMUTABLE 和 APPEND 属性标志 |
| CAPMACADMIN | 允许 MAC 配置或状态更改 |
| CAPMACOVERRIDE | 忽略文件的 DAC 访问限制 |
| CAP_MKNOD | 允许使用 mknod() 系统调用 |
| CAPNETADMIN | 允许执行网络管理任务 |
| CAPNETBIND_SERVICE	允许绑定到小于 1024 的端口 | | CAPNETBROADCAST | 允许网络广播和多播访问 |
| CAPNETRAW | 允许使用原始套接字 |
| CAP_SETGID | 允许改变进程的 GID |
| CAP_SETFCAP | 允许为文件设置任意的 capabilities |
| CAP_SETPCAP | 参考 capabilities man page |
| CAP_SETUID | 允许改变进程的 UID |
| CAPSYSADMIN | 允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等 |
| CAPSYSBOOT | 允许重新启动系统 |
| CAPSYSCHROOT | 允许使用 chroot() 系统调用 |
| CAPSYSMODULE | 允许插入和删除内核模块 |
| CAPSYSNICE | 允许提升优先级及设置其他进程的优先级 |
| CAPSYSPACCT | 允许执行进程的 BSD 式审计 |
| CAPSYSPTRACE | 允许跟踪任何进程 |
| CAPSYSRAWIO | 允许直接访问 /devport、/dev/mem、/dev/kmem 及原始块设备 |
| CAPSYSRESOURCE | 忽略资源限制 |
| CAPSYSTIME | 允许改变系统时钟 |
| CAPSYSTTY_CONFIG | 允许配置 TTY 设备 |
| CAP_SYSLOG | 允许使用 syslog() 系统调用 |
| CAPWAKEALARM | 允许触发一些能唤醒系统的东西(比如 CLOCKBOOTTIMEALARM 计时器) |


Capabilities是细分到线程的，即每个线程可以有自己的capabilities。

完整的capabilities实现，除了对线程的capabilities具有以下相关功能：

* 进行特权操作时，检查该线程是否拥有该操作的capability
* 提供系统调用，用于获取或修改线程的capability

还包含了对于文件的capabilities的支持，即：

* 文件系统支持文件附加属性，使得可执行文件具有一定的capabilities，从而在运行时确定其capabilities

关于以下线程和文件的capabilities的介绍主要参考了 [官方的 man page](https://man7.org/linux/man-pages/man7/capabilities.7.html)

## 线程的capabilities

Linux到目前为止，每一个线程，具有5个capabilities的集合，分别是：Permitted、Inheritable、Effective、Bounding、Ambient

### Permitted

这个集合定义了线程所能够拥有的特权的上限。换句话说，如果某个capability不在Permitted集合中，那么该线程便不能进行这个capability所对应的特权操作。Permitted集合是Inheritable和Effective集合的的超集。

### Inheritable

当执行exec()系运行其他命令时，能够被新命令继承的capabilities，被包含在Inheritable集合中。

### Effective

内核检查该线程是否可以进行特权操作时，检查的对象便是Effective集合。如之前所说，Permitted集合定义了上限。线程可以删除Effective集合中的某capability，随后在需要时，再从Permitted集合中恢复该capability，以此达到临时禁用capability的功能。

### Bounding

Bounding 集合是 Inheritable 集合的超集，如果某个 capability 不在 Bounding 集合中，即使它在 Permitted 集合中，该线程也不能将该 capability 添加到它的 Inheritable 集合中。

Bounding 集合的 capabilities 在执行 fork() 系统调用时会传递给子进程的 Bounding 集合，并且在执行 execve 系统调用后保持不变。

* 当线程运行时，不能向 Bounding 集合中添加 capabilities。
* 一旦某个 capability 被从 Bounding 集合中删除，便不能再添加回来。
* 将某个 capability 从 Bounding 集合中删除后，如果之前 Inherited 集合包含该 capability，将继续保留。但如果后续从 Inheritable 集合中删除了该 capability，便不能再添加回来。

### Ambient

Ambient 用来弥补 Inheritable 的不足。Ambient 具有如下特性：

* Permitted 和 Inheritable 未设置的 capabilities，Ambient 也不能设置。
* 当 Permitted 和 Inheritable 关闭某权限（比如 CAP_SYS_BOOT）后，Ambient 也随之关闭对应权限。这样就确保了降低权限后子进程也会降低权限。
* 非特权用户如果在 Permitted 集合中有一个 capability，那么可以添加到 Ambient 集合中，这样它的子进程便可以在 Ambient、Permitted 和 Effective 集合中获取这个 capability。现在不知道为什么也没关系，后面会通过具体的公式来告诉你。

Ambient 的好处在于，如果你将 CAP_NET_ADMIN 添加到当前进程的 Ambient 集合中，它便可以通过 fork() 和 execve() 调用 shell 脚本来执行网络管理任务，因为 CAP_NET_ADMIN 会自动继承下去。


## 文件的capabilities

文件的 capabilities 被保存在文件的扩展属性中。如果想修改这些属性，需要具有 CAP_SETFCAP 的 capability。文件与线程的 capabilities 共同决定了通过 execve() 运行该文件后的线程的 capabilities。

文件的 capabilities 功能，需要文件系统的支持。如果文件系统使用了 nouuid 选项进行挂载，那么文件的 capabilities 将会被忽略。

类似于线程的 capabilities，文件的 capabilities 包含了 3 个集合：Permitted、Inheritable、Effective

### Permitted

这个集合中包含的 capabilities，在文件被执行时，会与线程的 Bounding 集合计算交集，然后添加到线程的 Permitted 集合中。

### Inheritable

这个集合与线程的 Inheritable 集合的交集，会被添加到执行完 execve() 后的线程的 Permitted 集合中。

### Effective

这不是一个集合，仅仅是一个标志位。如果设置开启，那么在执行完 execve() 后，线程 Permitted 集合中的 capabilities 会自动添加到它的 Effective 集合中。对于一些旧的可执行文件，由于其不会调用 capabilities 相关函数设置自身的 Effective 集合，所以可以将可执行文件的 Effective bit 开启，从而可以将 Permitted 集合中的 capabilities 自动添加到 Effective 集合中。

## capability 权限查看及判断

Linux 中提供了`capsh`、`getcap`、`setcap`、`captest`等诸多命令来查看和设置 capabilities，如：

```bash
test :: ~ » getcap /bin/ping
/bin/ping = cap_net_admin,cap_net_raw+p
```

即该文件的capabilities，Permitted集合中包含了CAP_NEW_RAW，从而可以发送raw packet。

关于执行时，权限的判断这块，官方man文档主要使用了公式来描述，同时这里我从网上盗了个图，能清晰点表达出来

![capabilities.png](http://tva1.sinaimg.cn/large/002Yana6gy1gv96sk4uzaj61pe15y7gx02.jpg)


以上是对capability概念的部分介绍，接下来会逐步学习capability的具体使用。


### 参考连接

* https://man7.org/linux/man-pages/man7/capabilities.7.html
* https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work
* https://blog.csdn.net/alex_yangchuansheng/article/details/102796001
* https://blog.csdn.net/weixin_39219503/article/details/106888174
