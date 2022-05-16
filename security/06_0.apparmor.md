# Linux安全：AppArmor


## 简介

AppArmor是一项安全功能，可以在许多Linux发行版中找到。SLES（SUSE Linux Enterprise Server），openSUSE和Ubuntu是该产品附带的一些发行版。

同SELinux，Apparmor也是内核增强功能，旨在将程序限制在有限的资源集中。Apparmor与其他安全工具的不同之处在于，它将访问控制属性绑定到程序而不是单个用户。

Apparmor的限制是由加载到内核中的特殊配置文件提供的。这些配置文件可以在两种模式下运行：complain mode或enforce mode。

* enforce：以enforce模式加载的配置文件将导致配置文件中定义的策略的强制执行以及报告策略违规尝试（通过 syslog 或 auditd）。
* Complain：Complain模式下的配置文件不会强制执行策略，而是报告违反策略的尝试。

AppArmor 与 Linux 上的其他一些 MAC(如SELinux)系统不同：它是基于路径的，它允许混合执行和抱怨模式配置文件，它使用包含文件来简化开发，并且它的进入门槛远低于其他流行的 MAC 系统。

## 相关概念


## 使用


## 相关链接

* [Apparmor的绕过](https://zhuanlan.zhihu.com/p/457269937)
* [什么是AppArmor，以及如何保持Ubuntu安全？](https://mos86.com/82456.html)
* [在 Linux 上用 SELinux 或 AppArmor 实现强制访问控制（MAC）](https://mp.weixin.qq.com/s?src=3&timestamp=1652624506&ver=1&signature=V-NSaVpqkPFm98fJar4RlRl3EZJ5SR5jX7ZwXr0Z8m8y2dkosOUjvJkE3ng19ss5mFWQZaI2kLLkP1SIVjT7qF*UYqw5cyM-NW5XFptVOTc6KwTFLyQTVlcZuuHrAvXwOvFEK*EvKCgI0SZXyWYH0tUR2AhUae8WpJSVTdQ2w2Y=)

* https://zhuanlan.zhihu.com/p/457269937
* https://mos86.com/82456.html
