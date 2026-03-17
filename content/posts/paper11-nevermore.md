---
title: "【论文精读】NeVerMore: Exploiting RDMA Mistakes in NVMe-oF Storage Applications"
date: 2025-03-17
draft: false
tags: ["RDMA", "NVMe-oF", "网络安全", "论文精读", "InfiniBand", "存储安全"]
categories: ["论文精读"]
description: "对 NeVerMore 论文的逐段深度解析，涵盖 RDMA 协议漏洞、NVMe-oF 攻击向量及缓解机制。"
---

> **原文信息**
> - **标题**：NeVerMore: Exploiting RDMA Mistakes in NVMe-oF Storage Applications
> - **作者**：Konstantin Taranov, Benjamin Rothenberger, Daniele De Sensi, Adrian Perrig, Torsten Hoefler
> - **机构**：ETH Zurich, Switzerland
> - **arXiv**：2202.08080v1 \[cs.CR\] 16 Feb 2022

---

## 标题解读

**NeVerMore: Exploiting RDMA Mistakes in NVMe-oF Storage Applications**

标题中的重点词汇：

- **exploit** *v.* 利用（漏洞）、挖掘（弱点）；
  1. Attackers can exploit weak key generators to guess valid connection identifiers. 攻击者可以利用弱密钥生成器猜测有效的连接标识符。

- **RDMA** (Remote Direct Memory Access) *n.* 远程直接内存访问；绕过 CPU 直接在节点间读写内存的高性能网络技术；
  1. RDMA enables low-latency, high-bandwidth communication without involving the host CPU. RDMA 在不涉及主机 CPU 的情况下实现低延迟、高带宽通信。

- **NVMe-oF** (NVMe over Fabrics) *n.* 基于网络结构的 NVMe；通过网络协议访问远端 NVMe 固态存储的协议；
  1. NVMe-oF leverages RDMA to send storage commands over a high-speed network fabric. NVMe-oF 利用 RDMA 通过高速网络结构发送存储指令。

- **storage application** *n.* 存储应用程序；运行于数据中心中、负责管理和访问存储设备的软件；
  1. Industrial storage applications increasingly rely on NVMe-oF for disaggregated, low-latency data access. 工业存储应用日益依赖 NVMe-oF 实现解聚合的低延迟数据访问。

---

## Abstract（摘要）

### 段落原文

This paper presents a security analysis of the 【InfiniBand architecture】, a prevalent 【RDMA】 standard, and 【NVMe-over-Fabrics (NVMe-oF)】, a prominent protocol for industrial 【disaggregated storage】 that exploits RDMA protocols to achieve 【low-latency】 and high-bandwidth access to remote solid-state devices. Our work, NeVerMore, discovers new 【vulnerabilities】 in RDMA protocols that unveils several 【attack vectors】 on RDMA-enabled applications and the NVMe-oF protocol, showing that the current security mechanisms of the NVMe-oF protocol do not address the security vulnerabilities posed by the use of RDMA. In particular, we show how an 【unprivileged user】 can inject packets into any RDMA connection created on a local network controller, bypassing security mechanisms of the operating system and its kernel, and how the injection can be used to acquire unauthorized 【block access】 to NVMe-oF devices. Overall, we implement four attacks on RDMA protocols and seven attacks on the NVMe-oF protocol and verify them on the two most popular implementations of NVMe-oF: 【SPDK】 and the 【Linux kernel】. To mitigate the discovered attacks we propose multiple mechanisms that can be implemented by RDMA and NVMe-oF providers.

### 生词解析

- **InfiniBand architecture** *n.* InfiniBand 架构；一种主流高性能互连网络规范，是 RDMA 的主要实现标准之一；
  1. The InfiniBand architecture underpins most high-performance RDMA deployments in modern data centers. InfiniBand 架构是现代数据中心大多数高性能 RDMA 部署的基础。

- **disaggregated storage** *n.* 解聚合存储；将计算节点与存储资源分离，通过网络连接的架构模式；
  1. Disaggregated storage allows compute and storage nodes to scale independently over high-speed interconnects. 解聚合存储允许计算节点和存储节点通过高速互连独立扩展。

- **vulnerability** *n.* 漏洞、脆弱性；系统或协议中可被攻击者利用的安全缺陷；
  1. A firmware vulnerability in the RNIC allows adversaries to forge connection identifiers. RNIC 中的固件漏洞允许攻击者伪造连接标识符。

- **attack vector** *n.* 攻击向量；攻击者用于渗透目标系统的途径或方法；
  1. Packet injection provides a powerful attack vector against any RDMA-enabled kernel module. 数据包注入为针对任何 RDMA 内核模块的攻击提供了强大的攻击向量。

- **unprivileged user** *n.* 非特权用户；不具备管理员权限的普通系统用户；
  1. The attack can be launched by an unprivileged user without any administrative permissions. 该攻击可由不具备任何管理员权限的普通用户发起。

- **block access** *n.* 块级访问；直接对存储设备进行块级别（低于文件系统）的读写操作；
  1. Unauthorized block access allows an attacker to overwrite data on the NVMe disk without filesystem checks. 未授权的块级访问允许攻击者绕过文件系统检查，直接覆写 NVMe 磁盘上的数据。

- **SPDK** (Storage Performance Development Kit) *n.* 存储性能开发套件；英特尔开源的高性能用户态存储框架；
  1. SPDK provides user-space NVMe drivers that avoid kernel context switches for maximum throughput. SPDK 提供用户态 NVMe 驱动程序，通过避免内核上下文切换来实现最大吞吐量。

### 参考注文

本摘要说明，论文针对 InfiniBand/RDMA 架构及 NVMe-oF 协议进行了系统性安全分析。研究发现，普通非特权用户可以向本地 RNIC 上的任意 RDMA 连接注入数据包，从而绕过操作系统安全机制，甚至获得对 NVMe-oF 存储设备的未授权块级访问权限。作者在 SPDK 和 Linux 内核两大主流 NVMe-oF 实现上验证了 4 类 RDMA 攻击与 7 类 NVMe-oF 攻击，并提出了相应的缓解机制。

---

## Section 1：Introduction（引言）

### 段一原文

Resource 【disaggregation】 is becoming an important tool in data center design, splitting existing monolithic servers into a number of consolidated 【single-resource pools】 that communicate over high-speed interconnects. This approach improves the hardware resource utilization and deployment flexibility as both the compute and storage nodes can use different types of server hardware and can be 【dimensioned】 independently. Despite these merits, disaggregation opens up new attack vectors, as it is often implemented over low-latency, high-bandwidth but 【untrusted networks】.

### 生词解析

- **disaggregation** *n.* 解聚合、资源分离；将传统整机服务器中的不同硬件资源（计算、存储等）拆分并独立管理的架构方式；
  1. Disaggregation increases utilization by allowing storage pools to be shared across many compute nodes. 解聚合通过允许存储池在多个计算节点间共享，从而提高了利用率。

- **monolithic server** *n.* 单体服务器；将计算、存储、网络等所有资源集成在同一台物理机上的传统服务器形态；
  1. Monolithic servers are being replaced by disaggregated architectures to improve scalability. 单体服务器正逐渐被解聚合架构所取代，以提升可扩展性。

- **single-resource pool** *n.* 单一资源池；仅包含一类硬件资源（如纯存储或纯计算）并可被共享访问的资源集合；
  1. A single-resource storage pool can serve multiple compute nodes simultaneously over RDMA. 单一存储资源池可通过 RDMA 同时为多个计算节点提供服务。

- **dimension** *v.* 独立规划容量；根据需求单独为某类资源配置规模和数量（此处为技术领域用法）；
  1. In a disaggregated architecture, storage can be dimensioned separately from compute based on workload needs. 在解聚合架构中，存储容量可根据工作负载需求独立于计算资源进行规划。

- **untrusted network** *n.* 不可信网络；不能保证通信内容不被窃听或篡改的网络环境；
  1. RDMA over untrusted networks is inherently risky because packets lack source authentication. 在不可信网络上运行 RDMA 本身具有风险，因为数据包缺乏源认证机制。

### 参考注文

本段介绍了资源解聚合这一现代数据中心设计趋势：将传统整机服务器拆分为独立的计算池与存储池，通过高速互连通信。尽管这种方式提升了资源利用率和部署灵活性，但由于通信往往运行在不可信网络之上，也随之引入了新的安全攻击面。

---

### 段二原文

The 【NVMe over Fabrics (NVMe-oF) protocol】 is a leading protocol for storage disaggregation, and it is offered and maintained by numerous storage and network vendors (Intel, Xilinx, Mellanox, Broadcom, and Pure Storage). NVMe-oF combines two recent high-performance techniques: 【NVM Express (NVMe)】 and Remote Direct Memory Access (RDMA). NVMe-oF adopts RDMA connections to send NVMe requests, which are usually sent over PCIe to a local solid state drive (SSD), over a networking fabric with 【ultra-low latency】 of a few microseconds. Even though RDMA networks enable low-latency and high-bandwidth, they have been shown to suffer from 【security weaknesses】. Key reasons are the lack of 【secure channels】 and the 【exposure of memory access】 to remote parties. Despite these risks, the security implications and dangers of deploying NVMe-oF remain largely unstudied.

### 生词解析

- **NVM Express (NVMe)** *n.* 非易失性内存标准；一种专为高速 PCIe 固态存储设计的主机控制器接口与存储协议；
  1. NVMe reduces storage access latency to microseconds by communicating directly over the PCIe bus. NVMe 通过直接在 PCIe 总线上通信，将存储访问延迟降低至微秒级别。

- **ultra-low latency** *n.* 超低延迟；通常指微秒级别的网络或存储访问延迟；
  1. RDMA achieves ultra-low latency by bypassing the host OS network stack entirely. RDMA 通过完全绕过主机操作系统网络协议栈来实现超低延迟。

- **security weakness** *n.* 安全弱点；协议或系统设计上存在的、可能被利用的安全缺陷；
  1. The security weaknesses of RDMA stem from its lack of source authentication and encryption. RDMA 的安全弱点源于其缺乏源认证和加密机制。

- **secure channel** *n.* 安全信道；提供机密性和完整性保护的通信链路（如 TLS、IPsec）；
  1. Without a secure channel, RDMA packets can be intercepted and replayed by an attacker. 在没有安全信道的情况下，RDMA 数据包可能被攻击者拦截和重放。

- **exposure of memory access** *phr.* 内存访问暴露；RDMA 允许远端直接操作本地内存区域，从而将内存访问权暴露给远程参与方；
  1. The exposure of memory access in RDMA allows remote endpoints to read and write local memory directly. RDMA 中内存访问的暴露使得远端节点可以直接读写本地内存。

### 参考注文

本段指出 NVMe-oF 是当前存储解聚合领域的主流协议，融合了 NVMe 的高速存储接口与 RDMA 的低延迟网络传输。然而，RDMA 本身存在固有的安全弱点——缺乏安全信道机制且内存访问对远端暴露——而学界对 NVMe-oF 部署所带来的安全风险至今研究不足。

---

### 段三原文

In this work, we introduce a series of 【attack tools】 that can be employed to attack any RDMA-enabled system. We show that any system that tries to make use of RDMA opens an 【attack surface】 allowing local users to bypass the security mechanisms of the operating system and its kernel. Importantly, we show that any unprivileged user can inject packets into RDMA connections created on a local 【network controller】, even if they are created in 【kernel space】. Hence, any system that uses RDMA from the kernel space opens an attack surface allowing local users to manipulate RDMA-enabled kernel modules, such as the NVMe-oF 【block device】. In addition, we show how an adversary can conduct 【denial-of-service attacks】 by breaking, preventing, and slowing down RDMA connections through vulnerabilities in the RDMA 【connection manager】, RDMA 【resource sharing】, and RDMA 【congestion mechanisms】.

### 生词解析

- **attack surface** *n.* 攻击面；系统中所有可能被攻击者利用的入口点或暴露接口的总集合；
  1. Every RDMA-enabled kernel module expands the attack surface accessible to local unprivileged users. 每一个启用 RDMA 的内核模块都会扩大本地非特权用户可访问的攻击面。

- **network controller** *n.* 网络控制器；此处特指 RDMA 能力网卡（RNIC），负责处理 RDMA 数据包的硬件设备；
  1. The adversary shares the same network controller as the victim and exploits its lack of sanity checks. 攻击者与受害者共享同一网络控制器，并利用其缺乏完整性校验的漏洞。

- **kernel space** *n.* 内核空间；操作系统内核运行的特权内存区域，与用户空间相隔离；
  1. Even connections created in kernel space are vulnerable to injection attacks via the RDMA verbs API. 即使是在内核空间创建的连接，也容易受到通过 RDMA verbs API 实施的注入攻击。

- **block device** *n.* 块设备；以固定大小数据块为单位进行读写的存储设备抽象（如磁盘、NVMe SSD）；
  1. An attacker can directly write forged data to the NVMe-oF block device without filesystem-level access controls. 攻击者可以绕过文件系统级访问控制，直接向 NVMe-oF 块设备写入伪造数据。

- **denial-of-service attack (DoS)** *n.* 拒绝服务攻击；通过耗尽系统资源或中断服务可用性来阻止合法用户访问的攻击；
  1. The congestion-based denial-of-service attack reduced victim bandwidth from 2713 MB/s to near zero. 基于拥塞的拒绝服务攻击将受害者带宽从 2713 MB/s 降至接近零。

- **connection manager** *n.* 连接管理器；RDMA 生态中负责建立和管理 RDMA 连接的用户态库与内核模块；
  1. Vulnerabilities in the RDMA connection manager allow an attacker to spoof disconnect requests. RDMA 连接管理器中的漏洞允许攻击者伪造断开连接请求。

- **congestion mechanism** *n.* 拥塞控制机制；RDMA 网络中用于避免因突发流量导致丢包的速率调控机制；
  1. The congestion mechanism is exploited to force remote endpoints to reduce their transmission rates. 拥塞控制机制被利用来强制远端节点降低其传输速率。

### 参考注文

本段是论文贡献的核心陈述。作者展示了一系列攻击工具，证明任何使用 RDMA 的系统（包括内核空间应用）都面临来自本地非特权用户的注入攻击风险。此外，攻击者还可通过利用连接管理器漏洞、资源共享缺陷和拥塞控制机制发动拒绝服务攻击。

---

## 段二 Section 1.1：Attack Classes Overview（攻击类型概述）

### 段落原文

**Injection.** An 【unprivileged user】 can inject packets into any RDMA connection created on a local network controller, including connections created by privileged 【kernel modules】. Therefore, any kernel application that makes use of RDMA opens an attack surface allowing the attacker to manipulate the kernel-level applications from user space by injecting RDMA requests into their connections. For NVMe-oF, the adversary can bypass security mechanisms of operating and file systems to directly manipulate NVMe disks at the 【block level】 without administrative privileges.

**Fake congestion.** A privileged user can 【forge】 congestion notification packets of RDMA protocols, forcing remote network controllers to slow down. Therefore, the attacker can disrupt the normal operations of any reachable RDMA-enabled application. For NVMe-oF, the attacker can significantly degrade the performance of accesses to remote NVMe disks.

**Disconnection attack.** An unprivileged user can forge packets of RDMA 【connection manager】, allowing it to disconnect any RDMA connection in the network, including connections created by local and remote kernel modules. For NVMe-oF, the attacker can temporally disconnect network-attached NVMe disks, preventing the operating system from accessing them.

**Resource exhaustion.** An unprivileged user can block local RDMA resources, preventing local applications from opening RDMA connections. For NVMe-oF, the attacker can disconnect network-attached NVMe disks using the previous attack and then prohibit them from being reconnected, preventing the operating system from accessing storage for an 【extended period】.

### 生词解析

- **kernel module** *n.* 内核模块；可动态加载进 Linux 内核的代码模块，运行于内核空间，拥有系统最高权限；
  1. The NVMe-oF Linux driver is implemented as a kernel module with full administrative privileges. NVMe-oF Linux 驱动以具备完整管理员权限的内核模块形式实现。

- **block level** *n.* 块级别；存储访问的最底层抽象，位于文件系统之下，直接操作存储设备的数据块；
  1. Block-level access bypasses filesystem permissions, allowing arbitrary read and write operations on the disk. 块级别访问绕过文件系统权限，允许对磁盘进行任意读写操作。

- **forge** *v.* 伪造；构造看似合法但实为虚假的网络数据包或认证信息；
  1. The attacker forges a congestion notification packet to deceive the remote RNIC into throttling its transmission rate. 攻击者伪造拥塞通知数据包，欺骗远端 RNIC 降低其发送速率。

- **resource exhaustion** *n.* 资源耗尽；通过大量消耗系统资源（如连接、内存、队列）使合法应用无法正常使用的攻击手段；
  1. Resource exhaustion attacks exploit system-wide RDMA connection limits to prevent legitimate applications from connecting. 资源耗尽攻击利用系统级 RDMA 连接上限，阻止合法应用建立连接。

- **extended period** *phr.* 较长时间段；指攻击效果持续的时间足以对业务造成显著影响；
  1. Combining disconnection and resource exhaustion attacks can prevent storage access for an extended period. 将断连攻击与资源耗尽攻击结合，可使存储访问在较长时间内无法恢复。

### 参考注文

本节概括了论文提出的四类 RDMA 攻击：注入攻击（可绕过操作系统安全机制直接操控内核模块）、伪造拥塞攻击（降低远端传输速率）、伪造断连攻击（中断任意 RDMA 连接）、资源耗尽攻击（阻止新连接建立）。每类攻击均可直接应用于 NVMe-oF 存储协议，造成未授权数据访问或存储服务中断。

---

## 段三 Section 2：Background on NVMe over RDMA（背景知识）

### 段落原文

**RDMA.** The NVMe-oF protocol uses RDMA network protocols specified in the 【InfiniBand architecture】 that includes native InfiniBand, 【RoCEv1】, and 【RoCEv2】 protocols. Regardless of the underlying RDMA protocol, developers make use of RDMA communication through the 【RDMA verbs user space library】. Each reliable RDMA connection consists of two endpoint handlers called 【queue pairs】, that allow applications to issue RDMA communication requests. Users submit 【asynchronous communication requests】 directly to the RDMA-capable network controller (RNIC) through its queue pair handler, 【bypassing】 the operating system and reducing CPU overhead. The RNIC performs all data accesses using an integrated 【DMA module】 that can directly write into and read from local memory. Once the RNIC finishes the execution of a communication request, it generates a 【completion event】 that is written to a user space 【completion queue】 indicating completion of the request.

### 生词解析

- **RoCE** (RDMA over Converged Ethernet) *n.* 基于融合以太网的 RDMA；允许在标准以太网硬件上运行 RDMA 协议的技术（v1 为纯以太网帧，v2 在 UDP/IP 上封装）；
  1. RoCEv2 adds IP and UDP headers to InfiniBand transport packets, enabling routing across subnets. RoCEv2 在 InfiniBand 传输包上添加了 IP 和 UDP 头部，支持跨子网路由。

- **RDMA verbs library** *n.* RDMA verbs 用户态库；提供标准化 RDMA 编程接口的用户空间软件库（如 libibverbs）；
  1. Applications use the RDMA verbs library to post work requests directly to the RNIC without OS involvement. 应用程序使用 RDMA verbs 库直接向 RNIC 提交工作请求，无需操作系统介入。

- **queue pair (QP)** *n.* 队列对；RDMA 连接的基本端点单元，由一个发送队列和一个接收队列组成；
  1. Each RDMA connection is identified by a unique queue pair number (QPN) at each endpoint. 每个 RDMA 连接在各端点处由唯一的队列对编号（QPN）标识。

- **asynchronous communication request** *n.* 异步通信请求；提交后不阻塞调用方、可在后台由硬件执行的通信指令；
  1. Asynchronous communication requests allow the CPU to continue other work while the RNIC transfers data. 异步通信请求允许 CPU 在 RNIC 传输数据期间继续执行其他任务。

- **bypass** *v.* 绕过；不经过某个中间层（如操作系统内核）而直接完成操作；
  1. RDMA bypasses the OS kernel entirely, placing work requests directly into hardware queues. RDMA 完全绕过操作系统内核，将工作请求直接提交至硬件队列。

- **DMA module** *n.* DMA 模块；直接内存访问模块，负责在不占用 CPU 的情况下直接在内存与外设之间搬运数据；
  1. The DMA module integrated into the RNIC can read from and write to host memory without CPU involvement. RNIC 内置的 DMA 模块可以在不占用 CPU 的情况下读写主机内存。

- **completion event** *n.* 完成事件；RNIC 在完成一个通信请求后生成的通知信号；
  1. When the RNIC finishes an RDMA write, it posts a completion event to the application's completion queue. 当 RNIC 完成 RDMA 写操作后，会向应用程序的完成队列投递一个完成事件。

- **completion queue (CQ)** *n.* 完成队列；用于存放 RNIC 生成的完成事件通知的用户态数据结构；
  1. The application polls the completion queue to determine when its RDMA requests have been executed. 应用程序轮询完成队列，以确定其 RDMA 请求何时执行完毕。

### 参考注文

本段介绍了 RDMA 的核心工作机制：应用通过 verbs 库向 RNIC 提交异步队列对请求，RNIC 的集成 DMA 模块绕过操作系统直接访问内存，完成后通过完成队列通知应用程序。这种"内核旁路"架构是 RDMA 实现超低延迟的根本原因，也是其安全脆弱性的根源所在。

---

## 段四 Section 3：Threat Model（威胁模型）

### 段落原文

We consider two threat models which we denote as 【TLU】 and 【TRA】. In both models, we consider a 【victim connection】 that connects two endpoints located on separate machines.

**Threat model Local User (TLU).** We consider an adversary that is on one of the endpoints of the victim connection (i.e., it is 【co-located】 with either the NVMe-oF target or client). The attacker is an unprivileged user and is assumed to have obtained access to the machines using 【legitimate means】. We assume that the attacker shares the same physical RNIC as the NVMe-oF entity and both can use it for communication. We assume that the attacker and the NVMe-oF entity are not separated through 【RNIC virtualization】.

**Threat model Remote Administrator (TRA).** We assume that the attacker is located on a different machine than the endpoints of the NVMe-oF connection. The attacker has 【administrative privileges】 on its machine which allows it to 【fabricate】 and inject messages into the network. These privileges allow the attacker to change the configuration of the network interface (e.g., its IP address).

**Adversary constraints.** We assume the NVMe-oF target only accepts connections from 【benign】 NVMe-oF clients. We assume that the adversary is further constrained by not being able to 【eavesdrop】 on existing connections. Instead of sniffing RDMA connection parameters, the attacker is required to 【guess】 these parameters in order to successfully 【impersonate】 one of the NVMe-oF endpoints.

### 生词解析

- **TLU (Threat model Local User)** *n.* 本地用户威胁模型；攻击者与受害连接端点共处同一物理机器的威胁场景；
  1. Under TLU, an unprivileged tenant on the same server can exploit shared RNIC resources to inject packets. 在 TLU 模型下，同一服务器上的非特权租户可利用共享 RNIC 资源注入数据包。

- **TRA (Threat model Remote Administrator)** *n.* 远程管理员威胁模型；攻击者位于不同机器但拥有本机管理员权限的威胁场景；
  1. Under TRA, the attacker can change the IP address of its NIC to impersonate a legitimate RDMA endpoint. 在 TRA 模型下，攻击者可修改自身网卡的 IP 地址来冒充合法的 RDMA 端点。

- **co-located** *adj.* 共同部署的；与目标运行在同一物理硬件或同一节点上；
  1. A co-located attacker shares the same RNIC as the victim and can exploit its shared queue pair namespace. 与受害者共同部署的攻击者与其共享同一 RNIC，可利用其共享的队列对命名空间。

- **legitimate means** *phr.* 合法手段；通过正常授权渠道（如租户账号）获得系统访问权限；
  1. The attacker is assumed to have gained access to the system through legitimate means such as a cloud tenant account. 假设攻击者通过合法手段（如云租户账户）获得了系统访问权限。

- **RNIC virtualization** *n.* RNIC 虚拟化；通过 SR-IOV 等技术将物理 RNIC 划分为多个虚拟设备以实现隔离；
  1. Without RNIC virtualization, multiple users share the same physical queue pair namespace, enabling injection attacks. 在没有 RNIC 虚拟化的情况下，多个用户共享相同的物理队列对命名空间，使注入攻击成为可能。

- **fabricate** *v.* 伪造、构造；在不拥有原始凭证的情况下构造虚假的网络数据包或消息；
  1. An administrator-level attacker can fabricate RDMA packets with arbitrary source addresses on its local NIC. 具有管理员权限的攻击者可以在其本地网卡上伪造具有任意源地址的 RDMA 数据包。

- **benign** *adj.* 善意的、合法的；指正常的、无恶意的通信方；
  1. The NVMe-oF target is configured to accept connections only from benign, pre-approved clients. NVMe-oF 目标被配置为仅接受来自善意的、预先批准的客户端的连接。

- **eavesdrop** *v.* 窃听；被动地监听他人的网络通信内容；
  1. The attacker cannot eavesdrop on existing RDMA connections and must therefore enumerate connection parameters. 攻击者无法窃听现有 RDMA 连接，因此必须枚举连接参数。

- **impersonate** *v.* 冒充；通过伪造标识信息使通信对方误认为自己是合法端点；
  1. By replicating the victim's queue pair configuration, the attacker can impersonate the victim endpoint. 通过复制受害者的队列对配置，攻击者可以冒充受害者端点。

### 参考注文

本节定义了两种威胁模型：TLU（本地非特权用户，与受害者共享同一 RNIC）和 TRA（远程管理员，拥有本机最高权限但位于不同物理机器）。攻击者被假设无法窃听现有连接，必须通过猜测或枚举来获取连接参数，这也是本文攻击工具设计的重要前提。

---

## 段五 Section 4.1：Packet Injection（数据包注入攻击）

### 段落原文

First, we analyze attacks that allow packet injection into InfiniBand-based protocols including RoCE and native InfiniBand. Compared to existing injection tools, our packet injection attack is feasible without administrative privileges assuming that the adversary is located on the same machine as the victim.

**Lack of sanity checks during connection creation.** Users can create an RDMA endpoint without relying on the RDMA connection manager, by directly using the RDMA verbs library (to which we further denote to as 【native RDMA connection establishment】). Unfortunately, none of the tested hardware providers of RNICs performs basic 【sanity checks】 during the creation of an RDMA endpoint using native RDMA connection establishment. This allows creating and using multiple RDMA endpoints that target the same remote RDMA endpoint (i.e., with the same 【destination QPN】) but have different local identifiers. Therefore, two different processes on a host can communicate to the same destination RDMA endpoint. This allows an attacker to create an almost identical copy (except for the source QPN) of a victim's existing connection and use it for communication to the same RDMA endpoint.

**Source QPN is not contained in RDMA packets.** InfiniBand-based protocols do not contain the source connection identifier (QPN) in the packet, but only include the destination identifier in the 【base transport header】. This design choice is based on the fact that RDMA connections are 【point-to-point channels】. Thus, the source QPN is communicated to the receiver when the RDMA connection is established and then stored in a connection table on the receiving endpoint. The fact that the source QPN is not included in RDMA packets makes the packets sent by an adversary using its forged connection 【indistinguishable】 from regular RDMA packets from the perspective of the remote RDMA endpoint.

### 生词解析

- **sanity check** *n.* 合理性校验；对输入数据或操作参数进行基本有效性验证的检查步骤；
  1. The absence of sanity checks in RNIC firmware allows users to create duplicate connection endpoints targeting the same destination QPN. RNIC 固件中缺乏合理性校验，使用户得以创建指向同一目标 QPN 的重复连接端点。

- **native RDMA connection establishment** *n.* 原生 RDMA 连接建立；不经由 RDMA 连接管理器、直接通过 verbs API 手动配置端点的连接建立方式；
  1. Native RDMA connection establishment does not involve any network communication, making impersonation attempts hard to detect. 原生 RDMA 连接建立不涉及任何网络通信，使得冒充尝试难以被检测。

- **destination QPN** *n.* 目标队列对编号；RDMA 数据包头部中标识接收端点身份的 24 位唯一标识符；
  1. If an attacker knows the victim's destination QPN, it can create a mirrored connection targeting the same remote endpoint. 如果攻击者知道受害者的目标 QPN，就可以创建一个以相同远端为目标的镜像连接。

- **base transport header** *n.* 基础传输头；InfiniBand 数据包中包含 QPN、PSN 等传输层信息的固定格式头部；
  1. The base transport header carries only the destination QPN, omitting the source identifier entirely. 基础传输头仅携带目标 QPN，完全省略了源标识符。

- **point-to-point channel** *n.* 点对点信道；每条连接仅在固定的两个端点之间传输数据的通信模式；
  1. Because RDMA uses point-to-point channels, the source QPN is considered implicit and not repeated in each packet. 由于 RDMA 使用点对点信道，源 QPN 被认为是隐含的，不会在每个数据包中重复携带。

- **indistinguishable** *adj.* 无法区分的；指伪造数据包与合法数据包在形式上完全相同，无法被接收方识别；
  1. Packets injected from the attacker's forged connection are indistinguishable from the victim's legitimate packets at the remote endpoint. 从攻击者伪造连接注入的数据包，在远端端点看来与受害者的合法数据包无法区分。

### 参考注文

本节揭示了数据包注入攻击的两个核心漏洞：其一，所有测试过的 RNIC 厂商均未对原生连接建立过程进行合理性校验，允许多个进程创建指向同一目标的连接端点；其二，InfiniBand 协议设计上不在数据包中携带源 QPN，导致攻击者的伪造数据包与合法数据包在远端完全无法区分。两者结合，使得无需任何管理员权限即可实施注入攻击。

---

## 段六 Section 4.2–4.4：其余三类 RDMA 攻击

### 段落原文

**4.2 Slow Down using Congestion Control.** InfiniBand-based protocols support 【congestion control】 to prevent packet drops because of 【bursted traffic】. Switches of an RDMA network mark packets contributing to the congestion by setting a congestion bit in the transport header. The congestion notification is carried through to the target, which generates a 【congestion notification packet】 that advises the initiator to reduce the 【injection rate】 to resolve congestion. The main vulnerability is that congestion notification packets are not protected, allowing an attacker to 【forge】 them. Forging a congestion notification packet only requires knowing the connection identifier (QPN) of the victim. As a result, anyone in the network can forge a congestion notification packet to disrupt the normal operations of any RDMA-enabled application.

**4.3 Attacking RDMA connection manager.** Applications use RDMA connection manager to send connect and disconnect requests, that contain 【secret connection keys】 for proving authenticity of requests. We found a vulnerability in their generator: the kernel module gets a random 32-bit starting seed when it is loaded, and then 【XORs】 it with sequential identifiers. Due to the nature of the algorithm, the difference between the two keys is usually in the least significant byte. Therefore, if the adversary can guess a recent key it can guess keys of other recent connections or at least enumerate them.

**4.4 RDMA Connection Exhaustion Attack.** RDMA drivers introduce limits on the number of open RDMA connections to operating systems. Unlike Linux, which limits the number of open TCP connections 【by each user】, RDMA limits are 【system-wide】. As a result, all RDMA-enabled applications, including privileged applications running in the kernel, share the same limit on the number of open connections. This vulnerability allows any local user to exhaust this limit and prevent other applications to have new connections.

### 生词解析

- **congestion control** *n.* 拥塞控制；网络协议中用于检测和缓解链路拥塞、防止数据包大量丢失的机制；
  1. RDMA's congestion control relies on unprotected notification packets, making it vulnerable to spoofing. RDMA 的拥塞控制依赖于未受保护的通知数据包，这使其容易受到欺骗攻击。

- **bursted traffic** *n.* 突发流量；短时间内大量集中到达的网络数据流，可能导致缓冲区溢出和数据包丢失；
  1. Bursted traffic from multiple RDMA senders can overwhelm switch buffers and trigger congestion notifications. 来自多个 RDMA 发送方的突发流量可能使交换机缓冲区溢出并触发拥塞通知。

- **congestion notification packet** *n.* 拥塞通知数据包；由交换机或目标节点生成、用于通知发送方降低发包速率的控制包；
  1. A forged congestion notification packet causes the victim RNIC to reduce its transmission rate to near zero. 伪造的拥塞通知数据包导致受害者 RNIC 将其传输速率降至接近零。

- **injection rate** *n.* 注包速率；RDMA 发送端向网络中发送数据包的速率；
  1. Upon receiving a congestion notification, the RNIC reduces its injection rate to relieve network pressure. 收到拥塞通知后，RNIC 降低其注包速率以缓解网络压力。

- **secret connection key** *n.* 秘密连接密钥；RDMA 连接管理器为每个连接分配的 32 位秘密值，用于验证断连请求的合法性；
  1. The weak key generator allows an attacker to deduce the secret connection keys of recently established RDMA connections. 弱密钥生成器使攻击者得以推断出最近建立的 RDMA 连接的秘密连接密钥。

- **XOR** *v./n.* 按位异或；一种常用于密钥生成和简单加密的位运算，但单独使用时熵值极低；
  1. XORing a fixed seed with a sequential counter produces predictable keys that can be enumerated within seconds. 将固定种子与顺序计数器异或会产生可预测的密钥，这些密钥可在数秒内被枚举出来。

- **system-wide** *adj.* 系统全局的；作用于整个操作系统范围内、不区分用户或进程的限制策略；
  1. System-wide RDMA connection limits can be exhausted by any unprivileged local user, blocking even kernel-level applications. 系统全局的 RDMA 连接上限可被任何本地非特权用户耗尽，甚至会阻塞内核级应用程序。

### 参考注文

本节描述了另外三类 RDMA 攻击：拥塞伪造攻击利用未受保护的拥塞通知包将受害者带宽降至接近零；连接管理器攻击利用弱密钥生成算法（XOR + 顺序计数器）枚举秘密密钥并伪造合法断连请求；连接耗尽攻击利用 RDMA 系统全局连接上限（不同于 TCP 的按用户限制），使任意本地用户均可阻止系统中所有 RDMA 应用建立新连接。

---

## 段七 Section 5：NVMe-oF Security Analysis（NVMe-oF 安全分析）

### 段落原文

**5.2.1 Spoofing of NVMe-oF Requests.** RDMA packet injection allows an attacker to inject an RDMA send request that contains an NVMe 【capsule】. This is possible because NVMe messages are not 【authenticated】 and the attacker can guess the connection identifier of the NVMe-oF target as it is usually launched during the boot process. Furthermore, the packet sequence number can be enumerated within 2 seconds using our injection tool. Consequently, our NVMe-oF capsule injection allows 【re-writing】 any NVMe block stored on a remote disk without administrative privileges, bypassing security mechanisms of operating and file systems.

**5.2.2 Spoofing of NVMe-oF Responses.** An adversary can spoof the response issued by the target to the client. The reception of a response for a client means that the affected communication buffer has been used and it can be now 【deregistered】 or reused for a new request. Thus, the injection of NVMe-oF responses to NVMe-oF clients can cause 【premature memory invalidation】 for the Linux kernel implementation and 【premature memory mutation】 for SPDK. Consequently, our NVMe-oF response injection allows corrupting data on a remote disk without administrative privileges.

**5.2.3 Memory Corruption using RDMA Write.** An attacker can inject RDMA write requests to change the RDMA-accessible memory of the NVMe client as well as the NVMe target. SPDK clients and targets 【pre-allocate】 memory and thus use a single memory key for their communication buffers. Additionally, ReDMArk has shown that memory key generators of existing RNICs are weak and often have problems with 【static initialization】. Thus, some RNICs assign the same memory key to the first memory registration after a reboot. This results in an SPDK application having static predictable keys as they are often loaded at the boot.

### 生词解析

- **capsule** *n.* 封装单元；NVMe-oF 协议中将 NVMe 命令及数据封装成适合网络传输的标准化数据结构；
  1. The NVMe-oF capsule carries the command, block address, and payload in a format suitable for RDMA transport. NVMe-oF 封装单元以适合 RDMA 传输的格式携带命令、块地址和载荷数据。

- **authenticated** *adj.* 经过认证的；通过密码学手段验证了发送方身份和消息完整性的通信内容；
  1. NVMe-oF messages over RDMA are not individually authenticated, making spoofing attacks trivial to execute. 通过 RDMA 传输的 NVMe-oF 消息没有经过逐条认证，使得欺骗攻击极易实施。

- **deregister** *v.* 注销注册；将之前注册为 RDMA 可访问的内存区域从 RNIC 中移除，使其不再可被远端访问；
  1. The Linux kernel deregisters the memory buffer immediately after the RDMA operation completes for security. Linux 内核在 RDMA 操作完成后立即注销内存缓冲区的注册以保证安全。

- **premature memory invalidation** *n.* 提前内存失效；在数据尚未完整传输完毕时，内存注册即被提前注销，导致数据损坏；
  1. A spoofed response triggers premature memory invalidation in the Linux driver, corrupting the in-flight data. 伪造的响应触发了 Linux 驱动中的提前内存失效，导致传输中的数据损坏。

- **premature memory mutation** *n.* 提前内存篡改；在数据仍被目标端使用时，提前将缓冲区重用于新数据，导致数据污染；
  1. SPDK's response to a spoofed message causes premature memory mutation, overwriting data before the target has finished reading it. SPDK 对伪造消息的响应引发了提前内存篡改，在目标端读取完毕之前就覆写了数据。

- **pre-allocate** *v.* 预分配；在程序启动时一次性分配所有需要的内存资源并持续复用，而非按需动态申请；
  1. SPDK pre-allocates a single large memory region and registers it once, leaving the same key valid for all operations. SPDK 预分配一个大型内存区域并一次性完成注册，导致同一密钥在所有操作中持续有效。

- **static initialization** *n.* 静态初始化；设备重启后始终从相同固定值开始的计数器或密钥生成器初始状态；
  1. Static initialization of memory key generators means that an attacker can predict the first key assigned after any reboot. 内存密钥生成器的静态初始化意味着攻击者可以预测重启后分配的第一个密钥。

### 参考注文

本节将第4节的 RDMA 攻击工具应用于 NVMe-oF 协议，展示了三类直接存储攻击：伪造 NVMe-oF 请求可在无管理员权限下向远端磁盘任意写入数据；伪造响应可诱使客户端提前释放内存缓冲区，导致数据损坏；利用 RDMA Write 直接篡改 SPDK 内存缓冲区（因其使用可预测的静态内存密钥）则可在不修改远端磁盘的情况下伪造客户端读取到的数据。

---

## 段八 Section 5.3：Mitigations（缓解措施）

### 段落原文

**NVMe-oF Message Authentication.** The NVMe-oF protocol could employ 【application-layer security】 to authenticate NVMe-oF messages including one-sided RDMA requests, even though source authentication for RDMA cannot be fully implemented at the application layer. The reason is that one-sided RDMA requests are executed by RNICs without CPU involvement, making it impossible to verify the authenticity of these packets before executing them. Nonetheless, since NVMe-oF read and write requests always involve 【two-sided RDMA sends】 source authentication at the application layer can be used for these requests. We propose that the sender includes a 【message authentication code (MAC)】 to each NVMe-oF message, thereby allowing the receiver to authenticate the sender and the data sent with a request. To additionally ensure data integrity of data sent over one-sided RDMA operations, we propose to authenticate this data with an additional MAC sent in a corresponding NVMe-oF message.

For 【RDMA providers】, the most robust approach is to add a four-byte header (RDMA packets must be 4-byte aligned) that would include the source QPN (24 bits) to the packet format, allowing the RNICs to distinguish packets from different connections. Alternatively, a 【secure transport】 should be introduced to the InfiniBand architecture, as proposed by sRDMA. The RDMA connection manager should process only messages arriving from other remote RDMA connection managers. Finally, administrators should be informed about 【RDMA controllers】 as a means to enforce per-user RDMA resource limits.

### 生词解析

- **application-layer security** *n.* 应用层安全；在协议栈应用层实施的安全保护措施，如消息认证码、端到端加密等；
  1. Application-layer security can authenticate NVMe-oF messages even when the underlying RDMA transport is insecure. 即使底层 RDMA 传输不安全，应用层安全也可以对 NVMe-oF 消息进行认证。

- **two-sided RDMA send** *n.* 双边 RDMA 发送操作；需要通信双方均主动参与（分别提交发送和接收工作请求）的 RDMA 操作类型；
  1. Two-sided RDMA sends involve the target CPU and can therefore carry and verify a MAC for source authentication. 双边 RDMA 发送操作涉及目标端 CPU，因此可以携带并验证用于源认证的 MAC。

- **message authentication code (MAC)** *n.* 消息认证码；利用共享密钥对消息内容进行计算得出的短摘要，用于验证消息完整性和发送方身份；
  1. Including a MAC in each NVMe-oF capsule allows the receiver to detect forged or tampered requests. 在每个 NVMe-oF 封装单元中包含 MAC，使接收方能够检测伪造或篡改的请求。

- **RDMA provider** *n.* RDMA 提供商；实现 RDMA 硬件（RNIC）和驱动程序的厂商（如 Mellanox/NVIDIA、Broadcom）；
  1. RDMA providers need to update the InfiniBand packet format to include the source QPN for proper source authentication. RDMA 提供商需要更新 InfiniBand 数据包格式，以包含源 QPN 用于正确的源认证。

- **secure transport** *n.* 安全传输层；在 RDMA 协议层面提供加密和认证保护的传输机制（如 sRDMA 所提出的方案）；
  1. A secure transport for InfiniBand would prevent packet injection regardless of the attacker's privileges. 适用于 InfiniBand 的安全传输层将阻止数据包注入，无论攻击者的权限级别如何。

- **RDMA controller** *n.* RDMA 控制器；Linux cgroup v1 中用于对 RDMA 资源进行按用户限制的管理组件；
  1. RDMA controllers in Linux cgroups allow administrators to enforce per-user limits on queue pairs and memory regions. Linux cgroup 中的 RDMA 控制器允许管理员对每个用户的队列对和内存区域数量进行限制。

### 参考注文

本节提出了针对已发现漏洞的缓解措施，分为应用层和协议层两个维度。应用层方面，建议在每条 NVMe-oF 消息中加入 MAC 以实现请求和响应的源认证与数据完整性保护。协议层方面，建议 RDMA 厂商在数据包格式中添加源 QPN 字段，或引入 sRDMA 等安全传输机制；同时建议连接管理器对消息来源进行过滤，并向系统管理员推广 RDMA 控制器这一现有但鲜为人知的资源隔离工具。

---

## 段九 Section 7：Conclusion（结论）

### 段落原文

We show how we can bypass security mechanisms of NVMe-oF using vulnerabilities in RDMA protocols. To perform attacks on NVMe-oF implementations we have designed several attack tools that can be used to attack other RDMA-enabled applications. Notably, we show how to 【spoof】 RDMA packets into victim connections without administrative permissions, even when 【IPsec over RoCE】 is enabled. Regarding the NVMe-oF protocol, we show how an attacker without administrative permissions can write data to a remote NVMe device, bypassing existing security mechanisms of the NVMe-oF protocol and operating systems. In addition, we show how we can 【falsify】 an NVMe data response, thereby forcing a victim client to observe the 【forged state】 of an NVMe device without actually modifying the state of the NVMe device. We anticipate that our work motivates security research on 【high-performance interconnects】 and systems utilizing them, leading to more secure high-performance networks and systems.

### 生词解析

- **spoof** *v.* 欺骗性伪造；伪装成合法来源发送数据包，使接收方误以为是真实通信；
  1. The tool can spoof RDMA packets into IPsec-protected connections because IPsec policies are shared between co-resident users. 该工具可以将 RDMA 数据包欺骗性注入受 IPsec 保护的连接，因为 IPsec 策略在共处一机的用户之间是共享的。

- **IPsec over RoCE** *n.* 基于 RoCE 的 IPsec 安全协议；将 RoCEv2 数据包封装在 IPsec 内以提供加密和完整性保护的方案；
  1. IPsec over RoCE provides encryption for RDMA packets but cannot isolate connections belonging to different users on the same NIC. IPsec over RoCE 为 RDMA 数据包提供加密，但无法隔离同一网卡上不同用户的连接。

- **falsify** *v.* 篡改、捏造；使数据或状态呈现虚假内容，而非真实值；
  1. The attacker can falsify the read response without modifying any data actually stored on the remote NVMe device. 攻击者可以篡改读取响应，而无需实际修改远端 NVMe 设备上存储的任何数据。

- **forged state** *n.* 伪造状态；受害客户端观察到的与实际存储设备真实状态不符的虚假数据视图；
  1. By injecting a crafted RDMA write into the client's receive buffer, the attacker forces it to observe a forged state. 通过向客户端接收缓冲区注入精心构造的 RDMA 写请求，攻击者迫使其观察到伪造状态。

- **high-performance interconnect** *n.* 高性能互连；专为低延迟、高带宽设计的数据中心网络技术，如 InfiniBand、RoCE 等；
  1. Security research on high-performance interconnects is lagging behind their rapid adoption in cloud and HPC environments. 针对高性能互连的安全研究远落后于其在云计算和高性能计算环境中的快速普及。

### 参考注文

结论部分总结了 NeVerMore 的核心成果：成功绕过 NVMe-oF 现有安全机制（包括 IPsec over RoCE），实现了无特权用户对远端 NVMe 设备的数据写入和响应伪造攻击。作者指出，即使不修改远端磁盘的真实存储内容，也可以通过伪造数据让客户端观察到虚假的设备状态，并呼吁学界加强对高性能互连技术安全性的研究。
