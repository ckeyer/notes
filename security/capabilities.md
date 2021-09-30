# 信息收集

* https://blog.csdn.net/alex_yangchuansheng/article/details/102796001
* https://blog.csdn.net/weixin_39219503/article/details/106888174


# Linux Capability机制

# 

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


