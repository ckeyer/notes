<blockquote>来源:     公众号<br>
作者:     GoCN 【golangchina】<br>
原文标题: Cilium 开源 Tetragon – 基于 eBPF 的安全可观测性 & 运行时增强<br>
原文链接: https://mp.weixin.qq.com/s/jV3LlU-fcvlcNIUBpQW-nw<br>
原文链接: https://isovalent.com/blog/post/2022-05-16-tetragon<br>
译文原文链接: https://icloudnative.io/posts/tetragon/
发布时间: 2022-05-30 08:00<br>
</blockquote>



 
![](https://static2.ckeyer.com/imgs/739a7c8/2171e05.jpg)

Isovalent Cilium 企业版包含一个基于 eBPF 的实时安全可观测性和运行时增强（runtime enforcement）平台，2022 年 5 月 16 日，Isovalent 终于决定将该平台的主要功能开源，并将其命名为 Tetragon。

# 什么是 Tetragon？

Tetragon 提供了基于 eBPF 的完全透明的安全可观测性能力以及实时的运行时增强（runtime enforcement）能力。由于基于 eBPF 的内核级收集器中直接内置了智能内核过滤能力和聚合逻辑，因此 Tetragon 无需更改应用程序即可以非常低的开销实现深度的可观测性。内嵌的运行时执行层不仅能够在系统调用层面进行访问控制，而且能够检测到特权、Capabilities 和命名空间的提权逃逸，并实时自动阻止受影响进程的进一步执行。

![](https://static2.ckeyer.com/imgs/739a7c8/f4a997b.png)

### 智能可观测性

Tetragon 的基石是一个强大的可观测层，它可以观测整个系统，从低级别的内核可见性到跟踪文件访问、网络活动或能力（capability）变化，一直到应用层，涵盖了诸如对易受攻击的共享库的函数调用、跟踪进程执行或解析发出的 HTTP 请求。**Tetragon 可以提供对各种内核子系统的可观测性，涵盖了命名空间逃逸、Capabilities 和特权升级、文件系统和数据访问、HTTP、DNS、TLS 和 TCP 等协议的网络活动，以及系统调用层的事件，以审计系统调用和跟踪进程执行。**

![](https://static2.ckeyer.com/imgs/739a7c8/9be496d.png)

* **深度可观测性** ：可以观测整个系统和应用程序的几乎所有调用环节。比如检测 TCP 连接中的低级微突发（microbursts），为黄金信号监控面板提供 HTTP 可见性，或者检测特定易受攻击的共享库的使用的能力。Tetragon 提供了一个易于使用的框架，以涵盖更多的可观测性用例，因此可以探索更多的可能性。
* **完全透明**：Tetragon 所有的可观察性数据都是从内核中透明地收集的，无需更改应用程序代码，应用也无法检测到自己何时被监控，这是安全用例的理想选择。
* **低开销**：Tetragon 直接在内核中使用 eBPF 执行过滤、聚合、度量统计和直方图收集，大大减少了系统的开销。此外，Tetragon 使用高效的数据结构，如每个 CPU 的哈希表、环形缓冲区和 LRU 地图，以提供高效和快速的数据收集手段，并避免向用户空间 agent 发送大量的低信号事件。

译者注：微突发（microbursts）是指端口在非常短的时间（毫秒级别）内收到非常多的突发数据。

# 运行时增强（runtime enforcement）

基于丰富的可观测性，Tetragon 还提供了实时的运行时增强（runtime enforcement）能力。大部份运行时增强（runtime enforcement）系统都只有有限的一组强制执行点（例如仅在系统调用级别），而 Tetragon 能够以预防的方式在整个操作系统中执行安全策略，而不是对事件异步地做出反应。除了能够为多个层级的访问控制指定允许列表外，Tetragon 还能够自动检测特权和 Capabilities 升级或命名空间提权（容器逃逸），并自动终止受影响的进程。安全策略可以通过 Kubernetes（CRD）、JSON API 或 Open Policy Agent（OPA）等系统注入。

![](https://static2.ckeyer.com/imgs/739a7c8/79a602c.png)


* **预防式安全**：Tetragon 的运行时策略直接在内核中执行，并且是同步的（实时的），这样可以真正防止攻击，而不仅仅是对攻击做出异步的反应。
* **无需了解所有攻击载体**：Tetragon 无需了解单个攻击载体并对其进行阻断，它的做法是允许你定义一系列难以被攻破的隔离和特权限制保证措施，并自动执行这些保证措施。
* **可插拔策略架构**：Tetragon 允许执行多种不同来源的安全策略，包括用户自定义的策略，Open Policy Agent (OPA) 等系统定义的策略，以及通过可扩展的 Kubernetes CRD 和 JSON 接口对接来自第三方组件的安全策略。

### 运行时增强为何是实时的？

除了有更多的位置（enforcement points）可以执行策略，eBPF 还使得我们在遭受漏洞攻击时马上作出反应，实时、同步地执行策略。当然，Tetragon 也能够像其他运行时增强（runtime enforcement）系统一样允许或拒绝与特定参数相匹配的特定系统调用，但它的杀手锏是一旦观察到特权/功能升级或命名空间提权，便立即阻止进程继续运行，从而将运行时增强（runtime enforcement）功能提升到一个新的台阶。更厉害的是，Tetragon 甚至不需要了解这个攻击载体是什么。

![](https://static2.ckeyer.com/imgs/739a7c8/b96221a.jpg)

Tetragon 另辟蹊径，它不需要了解特定的漏洞或攻击载体，而是直接定义执行策略，指定哪些应用程序应在运行时可以提升特权、附加额外的 Capabilities、跨越内核命名空间的边界，而后便监视内核的提权和逃逸，并自动终止违反定义策略的进程。而且杀死进程是在内核中同步执行的，这意味着如果一个进程使用 write(2) 或 sendmsg(2) 来利用内核漏洞获得权限，那么这些系统调用将永远不会返回，该进程及其所有线程都将被终止，不会再继续执行。

# 为什么使用 Tetragon？

传统的可观测性和运行时增强（runtime enforcement）解决方案无外乎都是基于以下几个方面来实现，它们都有各自的优势和缺陷。而 Tetragon 利用 eBPF 将更多的优势进行结合，并消除了绝大多数的缺陷。

![](https://static2.ckeyer.com/imgs/739a7c8/fbb64aa.jpg)

![](https://static2.ckeyer.com/imgs/739a7c8/32aa457.png)

上述解决方案都是在应用程序和系统调用层面上执行，并且可观测性方案也各不相同。它们都有一个用户空间代理，这个代理依赖于按定义收集的可观测性数据，然后对其作出反应，且无法对内核级别的事件进行观测。

![](https://static2.ckeyer.com/imgs/739a7c8/c42e565.jpg)

![](https://static2.ckeyer.com/imgs/739a7c8/2c0d44d.png)

第二类解决方案都是直接在内核层面操作，主要针对运行时增强（runtime enforcement），观测能力较弱（甚至没有观测能力）。内置的内核系统提供了非常多的策略执行选项，但内核在构建时却只重点提供访问控制的能力，而且非常难以扩展，例如内核是无法感知到 Kubernetes 和容器的。虽然内核模块解决了可扩展性问题，但由于其产生的安全风险，在很多场景下往往不是一种明智的选择。

像 LSM-eBPF 这样年轻的内核子系统功能非常强大，也非常有前景，只是需要依赖最新的内核（≥5.7）。**Tetragon 可以使用 eBPF-LSM 作为策略执行点，而且不受 eBPF-LSM 所需的内核版本的限制。**

由此可见，在 eBPF 的加持下，Tetragon 结合了现有解决方案的绝大多数优势。

![](https://static2.ckeyer.com/imgs/739a7c8/a3e3874.jpg)

* **高效 & 透明** Tetragon 提供了对应用的可观测性和高效的应用运行时检测，并且完全透明，对应用无侵入性，无需更改应用代码。
* **实时增强** Tetragon 提供了像 seccomp、SELinux 和 LSM 一样的内核级别同步执行策略的能力，但它将策略执行从纯粹的访问控制提升到了防止对系统和组件造成伤害的级别，而不是仅仅限制对资源和数据的访问。
* **深度可扩展的可观测性** Tetragon 提供了深度的系统观测能力和自定义 Linux 内核模块的可扩展性，同时没有安全和可用性风险。

此外，Tetragon 还提供了一个 agent，可以原生集成各种现代化的可观测性系统和策略标准（例如 Kubernetes、Prometheus、fluentd、Open Telemetry、Open Policy Agent 以及传统的 SIEM 平台）。

# 自动缓解特权或容器逃逸

Linux 内核的安全漏洞缓解是一个具有挑战性的问题。解决方案通常依赖于检测和阻止已知的攻击途径、减少攻击面或限制受损组件的爆炸半径。Tetragon 通过检测和停止相关进程来阻止内核中的特权、Capabilities 和命名空间提权。Kuberenetes 的每个应用程序都可以明确声明一组特权、Capabilities 和跨特权的命名空间。

### 缓解 CVE-2021-22555 漏洞

如下是一个简单的策略，它描述了在 CAP_SYS_ADMIN 能力更改时，进程将终止：

```
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: "capability-protection"
spec:
  [...]
    selectors:
    - matchCapabilityChanges:
      - type: Effective
        operator: In
        values:
        - "CAP_SYS_ADMIN"
      matchActions:
      - action: Sigkill
```
CVE-2021-22555 漏洞是通过 Netfilter 漏洞来获取特权。在实施上面的策略情况下，运行该漏洞，让我们看看会发生什么：

```
root@ubuntu2:/# /root/cve-2021-22555
[+] Linux Privilege Escalation by theflow@ - 2021

[+] STAGE 0: Initialization
[*] Setting up namespace sandbox...
[...]
[+] STAGE 5: Post-exploitation
[*] Escaping container...
[*] Cleaning up...
[*] Popping root shell...
Killed
```
可以看到当利用该漏洞进行特权提权时，该进程直接被终止了。请注意，上述的策略不包含漏洞本身的特定元素，所以使用不同攻击途径的不同漏洞进行攻击，会获得相同的结果。

# 使用 Tetragon CLI 进行应用行为检查

在 Isovalent 中，Tetragon CLI 代号为 amazing-cli，可能是使用 Tetragon 进行可观测性的第一个切入点。该 CLI 通常是令人吃惊，因为它能以一种简单易懂的形式公开复杂系统的大量信息。

在以下示例中，我们将展示如何使用 Tetragon 来观测 Kubernetes Pod 的行为，该 Pod 运行命令` curl -L github.com`：


![](https://static2.ckeyer.com/imgs/739a7c8/505d23d.png)

1. 开始执行 curl -L github.com
2. 进行 github.com 的 DNS 解析，及端口 80 的 TCP 连接打开。
3. github.com 在 7 毫秒内返回 HTTP 码 301 ，将流量重定向到端口 443。启用 TLS，curl 重定向并执行另一个 DNS 解析。
4. TCP 连接端口 443 打开，开始 TLS 握手。使用 TLS 1.3 协议，协商的密钥是 AES-128-GCM-SHA256。
5. 进行数据交换。端口 80 上总共接收约 80 个字节，端口 443 上接收 218KiB。
6. 进程退出，错误码为 0。


# 网络和运行时可观测性的结合

Tetragon 使用 eBPF 的其中一个令人兴奋的优势是它可以结合多个方面的可观测性，目前为止，这些可观测性通常都是单独处理的。下面是一个结合网络和运行时可观测性的示例，以演示识别哪些进程涉及哪种类型的网络通信的能力。以下示例显示了使用 Tetragon 来观测一个 Kubernetes Pod，该 Pod 被入侵并受到横向移动攻击：

![](https://static2.ckeyer.com/imgs/739a7c8/3c97d05.png)

在上图中，我们看到了通过反向 Shell 进行的经典横向移动攻击：

1. Kubernetes 的 Pod crawler-c57f9778c-wtcbc 在 Kubernetes 命名空间 tenant-jobs 中运行。Pod 是通过 Containerd 运行的，Containerd 作为 PID 1 init 进程的子进程运行。在 Pod 内运行的二进制文件称为“爬虫”，它会产生一个执行server.js 的 Node 进程。
2. Node 应用的出口网络连接是 api.twitter.com 和 Kubernetes 中的 Elasticsearch 服务。
3. Pod 启动 5 分钟后，又启动了另一个子进程调用 netcat (nc)。结合运行时和网络可观测性来看，很明显这是一个正在进行的反向 Shell 攻击。
4. 然后，可以观察到攻击者正在运行 curl 访问内部 es 服务器，然后使用 curl 将检索到的数据上传到 S3 存储中。

# 监控对敏感文件的访问

Tetragon 具有监控文件和数据访问的能力。以下示例说明了 Tetragon 与 Splunk 集成，以跟踪对敏感文件的访问，同时提供访问的上下文（例如：Kubernetes 元数据、容器镜像、二进制文件和用户信息）。

![](https://static2.ckeyer.com/imgs/739a7c8/37024e0.png)

上面的示例列出了对 /etcd/passwd 的访问，包括进程、及容器镜像和 Kubernetes 命名空间。除了监视 /etc/passwd 或 /etc/shadow 之外，还可以监控系统上其他的明显文件，包括：容器运行时的 UNIX 套接字，及可能改变系统引导的 Systemd 单元或 Init 文件。

Tetragon 不仅提供了对此类敏感文件访问的监控能力，而且可以阻止访问此类文件。

# 检测 TLS 弱密钥和版本

TLS 是当今世界安全的基石，但使用较旧的 TLS 版本或不安全的 TLS 密钥可能会构成严重的安全威胁。无意识的错误使用 TLS 密钥和版本会导致故意的 TLS 降级攻击。

![](https://static2.ckeyer.com/imgs/739a7c8/6e73fcf.png)

上面的仪表板显示了 TLS 协议版本信息，并将其与 Kubernetes Pod 和命名空间上下文相关联。它还可以显示密钥信息，及更重要的是密钥长度信息。

# 运行时感知的网络策略

你可能熟悉 Kubernetes 的 NetworkPolicies，它定义了 Kubernetes 工作负载的允许和禁止的网络通信。简而言之，这些策略描述了允许 Pod A 与 Pod B 或 CIDR 10.0.0.0/8 通信，但禁止 Pod A 与 Pod C 或 CIDR 20.1.1.1/32 通信。

这些策略的粒度是在 Pod 级别。所以无论 Pod 中运行的 app.js 还是 attack.py 脚本调用 curl，这些策略都可以生效的。借助 Tetragon 技术，可以扩展 Cilium 的网络策略功能以包括运行时上下文：

```
kind: CiliumNetworkPolicy
[...]
  endpointSelector:
  - matchLabels:
    - name: Frontend
  egress:
  - toEndpoints:
      - matchLabels:             
        - name: Backend
    fromRuntime:
      - binary: app.py
        privileged: false
```
上述策略获得明显更好的最小特权策略。它允许前端 Pod 与后端 Pod 对话，但前提是：

* 源 Pod 中的二进制文件为 app.py
* 源 Pod 中的进程以非特权运行

上述示例将二进制名称和特权执行上下文考虑在内，此概念可以扩展为基于其他参数进行限制，例如：确保进程仍处于该命名空间、UID/GID 上下文，甚至考虑可执行文件的内存哈希。

# 总结

Laf 是一个完全开源的云开发平台，可以在线编写、调试函数，不用重启服务，随时随地在 Web 上查看函数的运行日志，无需连接服务器，无需折腾数据库、对象存储、Nginx 等，可以随时随地上线应用，招之即来，挥之即去，随手发布！

Laf 可以帮助开发者拥有自己的云开发平台，让写代码像写博客一样简单，支持云函数/云数据库/对象存储，让前端秒变全栈，节省 60% 研发成本，实现 0 运维！

我们非常乐意将 Tetragon 开源。额外的可观测性和安全控制将会帮助你们改善安全、平台和应用程序团队的工作，助力整体的基础架构（特别是 Kubernetes 环境）更加安全。更重要的是，有了开源社区的加持，我们非常期待接下来会发生的故事。


### 引用链接
* [1]Isovalent Cilium 企业版: https://isovalent.com/product
* [2]Tetragon: https://github.com/cilium/tetragon
* [3]关于 Tetragon 的网络研讨会: https://isovalent.com/tetragon/
* [4]Cilium Slack: https://cilium.io/slack
