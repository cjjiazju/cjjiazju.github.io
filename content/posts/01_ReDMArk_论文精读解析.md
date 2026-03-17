---
title: "论文精读解析：ReDMArk: Bypassing RDMA Security Mechanisms"
date: 2027-03-17
description: "论文精读解析：ReDMArk: Bypassing RDMA Security Mechanisms"
tags: ["rdma", "paper", "security", "ReDMArk"]
showToc: true
TocOpen: true
draft: false
---
---
# 论文精读解析：ReDMArk: Bypassing RDMA Security Mechanisms

> **来源**：30th USENIX Security Symposium, August 2021
> **作者**：Benjamin Rothenberger, Konstantin Taranov, Adrian Perrig, Torsten Hoefler（ETH Zurich）
> **解析格式**：标题解读 → 原文（关键词标注）→ 生词解析 → 参考注文

---

## 标题解读

**ReDMArk: Bypassing RDMA Security Mechanisms**
绕过RDMA安全机制

- **bypass** v. 绕过，规避；
  1. The attacker attempted to bypass the authentication system. 攻击者试图绕过身份认证系统。

- **RDMA** (Remote Direct Memory Access) n. 远程直接内存访问；
  1. RDMA enables servers to exchange data without CPU involvement. RDMA允许服务器在无需CPU介入的情况下交换数据。

- **mechanism** n. 机制，机理；
  1. The security mechanism failed to detect the intrusion. 该安全机制未能检测到入侵。

---

## 段一：Abstract（摘要）

**原文（关键词已标注）：**

State-of-the-art 【remote direct memory access (RDMA)】 technologies such as 【InfiniBand (IB)】 or 【RDMA over Converged Ethernet (RoCE)】 are becoming widely used in data center applications and are gaining traction in cloud environments. Hence, the security of RDMA architectures is crucial, yet potential security implications of using RDMA communication remain largely 【unstudied】. ReDMArk shows that current security mechanisms of IB-based architectures are 【insufficient】 against both 【in-network attackers】 and attackers located on end hosts, thus affecting not only 【secrecy】, but also 【integrity】 of RDMA applications. We demonstrate multiple 【vulnerabilities】 in the design of IB-based architectures and implementations of RDMA-capable network interface cards (RNICs) and exploit those vulnerabilities to enable powerful attacks such as 【packet injection】 using 【impersonation】, 【unauthorized memory access】, and 【Denial-of-Service (DoS)】 attacks. To 【thwart】 the discovered attacks we propose multiple 【mitigation mechanisms】 that are 【deployable】 in current RDMA networks.

**生词解析：**

- **state-of-the-art** adj. 最先进的，代表当前最高技术水平的；
  1. This paper analyzes state-of-the-art network protocols used in modern data centers. 本文分析了现代数据中心中使用的最前沿网络协议。

- **gain traction** 短语 获得广泛采用，逐渐流行；
  1. RDMA technology is gaining traction in public cloud infrastructure. RDMA技术正在公有云基础设施中逐渐普及。

- **unstudied** adj. 尚未被研究的；
  1. The security implications of this protocol remain largely unstudied. 该协议的安全影响在很大程度上仍未被研究。

- **insufficient** adj. 不足的，不充分的；
  1. The existing protections are insufficient against advanced attackers. 现有保护措施不足以应对高级攻击者。

- **in-network attacker** n. 网络路径内部攻击者（位于数据传输路径上的攻击者）；
  1. An in-network attacker can intercept packets passing through a compromised switch. 网络内部攻击者可以拦截经过被攻陷交换机的数据包。

- **secrecy** n. 保密性，机密性；
  1. Encryption ensures the secrecy of transmitted data. 加密确保了传输数据的保密性。

- **integrity** n. （数据）完整性，未被篡改性；
  1. A checksum is used to verify the integrity of the message. 校验和用于验证消息的完整性。

- **vulnerability** n. 漏洞，脆弱性；
  1. Researchers discovered a critical vulnerability in the RNIC firmware. 研究人员发现了RNIC固件中的一个严重漏洞。

- **packet injection** n. 数据包注入（伪造并插入非法数据包的攻击手法）；
  1. The attacker used packet injection to corrupt the victim's connection state. 攻击者使用数据包注入破坏了受害者的连接状态。

- **impersonation** n. 身份冒充，伪装成合法端点；
  1. Without source authentication, impersonation attacks are trivially easy. 在没有源认证的情况下，身份冒充攻击极为容易实施。

- **unauthorized memory access** n. 未授权内存访问；
  1. The bug allowed unauthorized memory access across process boundaries. 该漏洞允许跨进程边界的未授权内存访问。

- **Denial-of-Service (DoS)** n. 拒绝服务攻击；
  1. A DoS attack can render an online service unavailable to legitimate users. DoS攻击可使合法用户无法访问在线服务。

- **thwart** v. 阻止，挫败；
  1. Ingress filtering can thwart spoofed packet injection attacks. 入口过滤可以阻止伪造数据包注入攻击。

- **mitigation mechanism** n. 缓解/防御机制；
  1. The paper proposes several mitigation mechanisms to defend against the attacks. 论文提出了几种缓解机制来防御这些攻击。

- **deployable** adj. 可部署的，可实际应用的；
  1. The proposed solution is deployable without modifying existing hardware. 所提方案无需修改现有硬件即可部署。

**参考注文：**

本段为论文摘要。作者指出，RDMA技术（InfiniBand与RoCE）已广泛应用于数据中心和云环境，但其安全性至今研究甚少。ReDMArk通过分析发现，现有IB架构的安全机制对网络内部及终端主机上的攻击者均存在明显不足，可被利用实施数据包注入、未授权内存访问及DoS等攻击。作者同时提出了可在现有网络中部署的多种缓解方案。

---

## 段二：Section 1 — Introduction（引言）

**原文（关键词已标注）：**

In recent years, numerous state-of-the-art systems started to 【leverage】 remote direct memory access (RDMA) 【primitives】 as a communication mechanism that enables high performance guarantees and resource utilization. Deployments in public clouds, such as Microsoft Azure and IBM Cloud, are becoming available and an increasing number of systems make use of RDMA for high-performance communication. However, the design of RDMA architectures is mainly focused on performance rather than security. Despite the trend of using RDMA, potential security implications and dangers that might be involved with using RDMA communication in 【upper layer protocols】 remain largely unstudied.

Current RDMA technologies include multiple 【plaintext access tokens】 to enforce 【isolation】 and prevent unauthorized access to system memory. As these tokens are transmitted in plaintext, any entity that obtains or guesses them can read and write memory locations that have been exposed by using RDMA on any machine in the network, compromising not only secrecy but also integrity of applications. To avoid compromise of these access tokens, RDMA architectures rely on isolation and the assumption that the underlying network is a well-protected resource. Otherwise, an attacker that is located on the path between two communicating parties (e.g., 【bugged wire】 or 【malicious switch】) can 【eavesdrop】 on access tokens of bypassing packets.

Unfortunately, 【encryption and authentication】 of RDMA packets is not part of current RDMA specifications. While 【IPsec】 transport recently became available for RoCE traffic, the IPsec standard does not support InfiniBand traffic. Furthermore, application-level encryption (e.g., based on TLS) is not possible since RDMA operations can be handled without involvement of the CPU. As TLS cannot support purely 【one-sided communication】 routines, the applications would need to store packets in a buffer before decryption, completely 【negating】 RDMA's performance advantages.

**生词解析：**

- **leverage** v. 利用，借助（某种优势或资源）；
  1. Modern distributed systems leverage RDMA primitives to minimize communication latency. 现代分布式系统利用RDMA原语来最小化通信延迟。

- **primitive** n. （计算机）原语，基本操作单元；
  1. Read and write are the two fundamental RDMA primitives. 读和写是RDMA的两种基本原语。

- **upper layer protocol** n. 上层协议（构建在底层通信协议之上的协议）；
  1. HTTP is an upper layer protocol that runs over TCP/IP. HTTP是运行在TCP/IP之上的上层协议。

- **plaintext access token** n. 明文访问令牌（未加密传输的身份凭证）；
  1. Transmitting access tokens in plaintext exposes them to interception. 以明文传输访问令牌会使其面临被截获的风险。

- **isolation** n. 隔离（防止不同进程/用户相互访问对方资源）；
  1. Memory isolation ensures that one process cannot read another's data. 内存隔离确保一个进程无法读取另一个进程的数据。

- **bugged wire** n. 被植入窃听设备的物理链路；
  1. A bugged wire allows an attacker to passively capture all passing traffic. 被植入窃听设备的线路使攻击者可被动捕获所有经过的流量。

- **malicious switch** n. 恶意交换机（被攻击者控制的网络交换设备）；
  1. A malicious switch can redirect traffic and expose sensitive tokens. 恶意交换机可重定向流量并暴露敏感令牌。

- **eavesdrop** v. 窃听，偷听（网络流量）；
  1. On-path attackers can eavesdrop on unencrypted RDMA packets. 路径上的攻击者可以窃听未加密的RDMA数据包。

- **encryption and authentication** n. 加密与认证（两种保护通信安全的核心手段）；
  1. Without encryption and authentication, packets are vulnerable to tampering. 没有加密和认证，数据包容易被篡改。

- **IPsec** n. 互联网安全协议（Internet Protocol Security），一种网络层加密协议；
  1. IPsec can protect RoCE traffic by encrypting packets at the network layer. IPsec可通过在网络层加密数据包来保护RoCE流量。

- **one-sided communication** n. 单边通信（仅发起方操作，目标方CPU不参与）；
  1. One-sided communication allows a node to read remote memory without interrupting the target CPU. 单边通信允许节点读取远程内存而不打扰目标CPU。

- **negate** v. 使无效，抵消；
  1. Adding a decryption buffer would negate the latency benefits of RDMA. 增加解密缓冲区将会抵消RDMA的低延迟优势。

**参考注文：**

本段阐明RDMA技术被广泛采用的背景及其核心安全隐患。RDMA架构在设计之初以性能为优先，几乎完全忽视了安全性。由于访问令牌以明文传输，且现有的加密标准（如IPsec、TLS）与RDMA的工作模式存在根本性冲突，当前RDMA系统缺乏有效的包级别加密和认证手段，使得路径上的攻击者可以轻松窃取令牌并实施攻击。

---

## 段三：Section 2 — Remote Direct Memory Access（RDMA技术背景）

**原文（关键词已标注）：**

RDMA enables direct data access on remote machines across a network. Memory accesses are 【offloaded】 to dedicated hardware and can be processed without involvement of the CPU (and 【context switches】). Using RDMA read and write requests application data is read from/written to a remote memory address and directly delivered to the network, reducing latency and enabling fast message transfer. RDMA can also enable 【one-sided operations】, where the CPU at the target node is not notified of incoming RDMA requests.

Even though several network architectures support RDMA, in this work we focus on the most widely used interconnects for RDMA: 【InfiniBand (IB)】 and 【RDMA over Converged Ethernet (RoCE)】. InfiniBand is a network architecture specifically designed to enable reliable RDMA and defines its own hardware and protocol specification. RoCE is an extension to Ethernet to enable RDMA over an Ethernet network and exists in two versions. 【RoCEv1】 uses the IB routing header, whereas 【RoCEv2】 uses UDP/IP for routing.

**2.1 RDMA packet format（数据包格式）**

The RDMA packet header consists of a 【routing header】 and a 【base transport header】. The routing header contains the source and destination ports, that identify link layer endpoints. All data communication in RDMA is based on 【queue pair (QP)】 connections between the two communicating parties. Endpoints in RDMA are identified by the combination of an adapter port address and a 【queue pair number (QPN)】, a unique identifier of a QP connection within destination port. For all QP endpoints at a destination port, the RNIC generates a unique QPN.

**2.2 InfiniBand Architecture Security Model（安全模型）**

Processing of incoming packets is based on the base transport header that contains the destination QPN and also a 【packet sequence number (PSN)】. The PSN is used to enforce in-order delivery and detect duplicate or lost packets. In addition to packet integrity checks, IBA defines three memory protection mechanisms: 【Memory Regions】, 【Memory Windows】, and 【Protection Domains (PD)】.

For each memory region RNIC generates two keys for local and remote access, namely 【lkey】 and 【rkey】. To remotely access a memory location using RDMA read or write operations, each packet must include a virtual address and its associated rkey. The rkeys are not used in any form of cryptographic computation, but used as 【access tokens】 that are transmitted in plaintext.

**生词解析：**

- **offload** v. 卸载，将处理任务转交给专用硬件；
  1. RDMA offloads memory access operations to the NIC, reducing CPU usage. RDMA将内存访问操作卸载到网卡，降低了CPU占用率。

- **context switch** n. 上下文切换（操作系统在不同进程间切换时的开销）；
  1. Frequent context switches increase latency in high-performance networking. 频繁的上下文切换会增加高性能网络的延迟。

- **one-sided operation** n. 单边操作（只需发起方参与，目标方无需感知）；
  1. One-sided RDMA operations bypass the remote CPU entirely. 单边RDMA操作完全绕过远端CPU。

- **InfiniBand (IB)** n. 无限带宽，一种专为高性能计算设计的网络架构；
  1. InfiniBand provides ultra-low latency communication for HPC clusters. InfiniBand为高性能计算集群提供超低延迟通信。

- **RoCE** (RDMA over Converged Ethernet) n. 以太网上的RDMA，将RDMA能力扩展到以太网；
  1. RoCEv2 uses UDP/IP headers to route packets across routed networks. RoCEv2使用UDP/IP头部在路由网络中转发数据包。

- **routing header** n. 路由头部（包含源/目的地址，用于网络路由）；
  1. The routing header identifies the source and destination of each packet. 路由头部标识了每个数据包的源和目的地。

- **base transport header** n. 基础传输头部（包含QPN和PSN等传输控制信息）；
  1. The base transport header carries the queue pair number and sequence number. 基础传输头部携带队列对编号和序列号。

- **queue pair (QP)** n. 队列对（RDMA中的双向通信端点，由发送队列和接收队列组成）；
  1. Each RDMA connection is identified by a queue pair at each endpoint. 每个RDMA连接由各端点的一个队列对来标识。

- **queue pair number (QPN)** n. 队列对编号（在目标端口内唯一标识一个QP连接）；
  1. The attacker guesses the victim's QPN to inject malicious packets. 攻击者猜测受害者的QPN来注入恶意数据包。

- **packet sequence number (PSN)** n. 数据包序列号（用于保证有序传输和检测重复/丢失包）；
  1. A PSN mismatch causes the RNIC to drop the incoming packet silently. PSN不匹配会导致RNIC静默丢弃传入的数据包。

- **Memory Region** n. 内存区域（向RDMA注册的一段主机内存）；
  1. Applications must register a memory region before exposing it to RDMA access. 应用程序在开放RDMA访问前必须注册内存区域。

- **Memory Window** n. 内存窗口（对内存区域的细粒度访问控制机制）；
  1. Type 2 memory windows bind a memory region to a specific queue pair. 类型2内存窗口将内存区域绑定到特定队列对。

- **Protection Domain (PD)** n. 保护域（将QP和内存区域分组，防止跨域访问）；
  1. Resources in different protection domains cannot access each other. 不同保护域中的资源无法相互访问。

- **lkey / rkey** n. 本地/远程访问密钥（内存区域的访问令牌）；
  1. An attacker who guesses a valid rkey can read or write the associated memory. 猜中有效rkey的攻击者可以读写对应的内存区域。

- **access token** n. 访问令牌（用于授权访问资源的凭证）；
  1. Access tokens transmitted in plaintext can be stolen by on-path attackers. 以明文传输的访问令牌可被路径上的攻击者窃取。

**参考注文：**

本节介绍RDMA的基本技术原理及InfiniBand安全模型。RDMA通过将内存访问操作直接卸载至网卡硬件，实现了无需CPU介入的高性能数据传输。IBA定义了三种内存保护机制（内存区域、内存窗口、保护域），并使用rkey作为远程访问的访问令牌。然而，这些令牌均以明文传输，且无任何密码学保护，构成了后续所有攻击的根本前提。

---

## 段四：Section 3 — Adversary Model（攻击者模型）

**原文（关键词已标注）：**

In our adversary model we consider three parties: an RDMA service which hosts one or several RDMA applications, a client who interacts with the service through RDMA, and an 【adversary】 who can legitimately connect to the RDMA service, but tries to violate RDMA's security mechanisms. We consider four different attacker models.

**Model T1** — 【Off-path attacker】: An adversary located at a different end host than the victim. This attacker cannot conduct any network-based attacks such as packet injection, but can connect to RDMA services and issue RDMA messages over these connections.

**Model T2** — 【Active off-path attacker】: Attackers that can actively compromise end hosts and 【fabricate】 and inject messages. To successfully inject an arbitrary RDMA request the adversary must have 【root administrative access】. The adversary is required to know the host's address, QP numbers, and the PSN to forge a valid RDMA packet.

**Model T3** — 【On-path (network-based) attacker】: The attacker is located on the path between the victim and the service and can control routers or links between the victims (e.g., 【rogue cloud provider】, rogue administrator, malicious 【bump-in-the-wire】 device). A network-based attacker can passively 【eavesdrop】 on messages, but also actively tamper with the communication by injecting, dropping, delaying, replaying, or altering messages. Since RDMA communication is in plaintext, this only requires 【recalculation of packet checksums】.

**Model T4** — 【Covert channel attacker】: An adversary that makes use of RDMA as a 【covert channel】 for exfiltrating data. The adversary manipulates code or libraries executed by the victim such that it establishes an RDMA connection to an attacker machine, allowing exploitation of one-sided RDMA operations to "silently" access memory of the victim process.

**生词解析：**

- **adversary** n. 对手，攻击者（学术语境中常用于指威胁模型中的攻击方）；
  1. The adversary is assumed to have legitimate access to the RDMA network. 假设攻击者对RDMA网络拥有合法访问权限。

- **off-path attacker** n. 非路径攻击者（不在通信路径上，无法直接窃听或篡改流量）；
  1. An off-path attacker must rely on guessing or brute-forcing to inject packets. 非路径攻击者必须依赖猜测或暴力破解来注入数据包。

- **fabricate** v. 伪造，捏造（数据包）；
  1. The attacker fabricated a packet with a spoofed source address. 攻击者伪造了一个带有欺骗性源地址的数据包。

- **root administrative access** n. 根管理员权限（对系统拥有最高控制权）；
  1. Gaining root access allows an attacker to inject raw network packets. 获得根权限允许攻击者注入原始网络数据包。

- **on-path attacker** n. 路径攻击者（位于通信路径上，可拦截并修改流量）；
  1. An on-path attacker can modify RDMA packets without detection. 路径攻击者可在不被检测的情况下修改RDMA数据包。

- **rogue cloud provider** n. 流氓/恶意云服务提供商；
  1. A rogue cloud provider could intercept RDMA traffic between tenants. 恶意云服务提供商可以拦截租户之间的RDMA流量。

- **bump-in-the-wire** n. 线路中间人设备（插入通信链路中的恶意透明设备）；
  1. A bump-in-the-wire device can silently inspect and alter passing packets. 线路中间人设备可静默地检查和修改经过的数据包。

- **recalculation of packet checksums** n. 重新计算数据包校验和；
  1. Since checksum algorithms are public, an attacker can recalculate them after modifying packets. 由于校验和算法是公开的，攻击者在修改数据包后可以重新计算校验和。

- **covert channel** n. 隐蔽信道（利用非预期通信路径秘密传输数据的方法）；
  1. The malware used RDMA as a covert channel to exfiltrate sensitive data. 该恶意软件利用RDMA作为隐蔽信道来窃取敏感数据。

- **exfiltrate** v. 秘密窃取并传输（数据）；
  1. The attacker used one-sided RDMA reads to exfiltrate memory contents. 攻击者利用单边RDMA读操作秘密窃取内存内容。

**参考注文：**

本节定义了论文的攻击者模型，共分四类。T1为普通的非路径客户端，仅能使用合法连接；T2具有被攻陷主机的根权限，可伪造并注入数据包；T3控制网络路径上的设备，可完全监控和篡改流量；T4则将RDMA用作隐蔽信道，利用单边操作静默读取受害者内存。这一分类体系为后续攻击分析提供了清晰的前提条件框架。

---

## 段五：Section 4 — Security Analysis of IB Architectures（安全分析）

**原文（关键词已标注）：**

Given the aforementioned adversary model, we analyse existing security mechanisms in IB-based architectures including memory protection key generation, QP number generation, memory regions, memory windows, and protection domains. We identify 10 vulnerabilities, labeled **V1–V10**.

**V1 Memory Protection Key Randomness（内存保护密钥随机性不足）**

To protect remote memory against unauthorized memory access, IBA requires that RDMA read/write requests include a remote memory access key 【rkey】, which is negotiated between communicating peers and is checked at the remote RNIC. Packets with an invalid rkey cause a connection error leading to 【disconnection】. The requirement of including an rkey is built into the silicon and the driver code cannot be disabled by an attacker.

We analyze the randomness of the rkey generation process for different RNIC models. For all tested devices, rkey generation is independent of the address and length of the buffer to be registered. RNIC models from Broadcom always increase the rkey value by 0x100 independent of any factors — thus, predicting preceding or subsequent rkeys is trivial. Devices based on the mlx5 driver show that with more than 60% probability either the value 0x101 or 0x102 is chosen as the increment, meaning the sequence is still predictable by an adversary with moderate effort.

**V2 Static Initialization State for Key Generation（静态初始化状态）**

The RNICs based on the bnxt_re and mlx4 drivers are initialized using 【static state】 and the same set of keys persists across different physical reboots of the machine. Assuming that an adversary has observed the entire key set, the same keys will be reused even after the physical machine rebooted.

**V3 Shared Key Generator（共享密钥生成器）**

On all tested devices the key generator is fully 【shared】 between applications using the same network interface even if they use different protection domains. Thus, if multiple RDMA applications are running on the same service, the prediction of rkeys of other applications based on own rkeys is trivial. This vulnerability can also be exploited to open a 【side-channel】 between applications sharing the same RNIC.

**V4 Consecutive Allocation of Memory Regions（连续内存分配）**

In addition to the rkey associated to a memory location, the adversary is also required to predict the corresponding memory address. Techniques such as 【address space layout randomization (ASLR)】 randomly arrange the address space positions of a process. However, subsequent objects in memory are allocated in consecutive addresses with respect to a random address base. Since RDMA-based applications run in a single process on the target host, objects in memory are allocated side-by-side, making address prediction possible.

**V5 Linearly Increasing QP Numbers（线性递增的队列对编号）**

For all tested devices and drivers the QP numbers are allocated 【sequentially】. Assuming that an adversary registers a QP himself or observes a QP registration request, predicting preceding or subsequent QP numbers is trivial.

**V6 Fixed Starting Packet Sequence Number（固定起始序列号）**

Many RDMA-based open-source applications opt for using the native connection interface and manually set the starting PSNs. In case the starting PSNs are not randomized on a per QP connection basis, predicting PSNs of established connections becomes much simpler.

**V7–V10（其他安全弱点）**

- **V7 Limited Attack Detection Capabilities**：由于内存访问由RNIC硬件直接处理，完全绕过CPU，应用层无法感知任何针对内存的攻击。
- **V8 Missing Encryption and Authentication**：RDMA协议对包头和负载均不提供任何加密或认证机制，攻击者可任意修改任何字段。
- **V9 Single Protection Domain for all QPs**：RDMA连接管理器默认对进程内所有QP和内存注册使用单一保护域，导致进程内所有QP可互访内存。
- **V10 Implicit On-Demand Paging (ODP)**：ODP允许进程将完整地址空间注册为RDMA可访问区域，若启用则单个rkey即可访问整个进程内存。

**生词解析：**

- **static state** n. 静态状态（不随系统重启而重置的固定初始值）；
  1. A static initialization state means the key sequence repeats after every reboot. 静态初始化状态意味着密钥序列在每次重启后都会重复。

- **side-channel** n. 旁路/侧信道（通过非预期的信息泄漏（如时序、功耗）推断秘密信息的攻击方式）；
  1. A side-channel attack can infer memory access patterns without reading data directly. 侧信道攻击可以在不直接读取数据的情况下推断内存访问模式。

- **address space layout randomization (ASLR)** n. 地址空间布局随机化（防止攻击者预测内存地址的安全技术）；
  1. ASLR makes it harder for attackers to predict the location of code or data in memory. ASLR使攻击者更难预测代码或数据在内存中的位置。

- **sequential** adj. 顺序的，按序递增的；
  1. Sequential QPN allocation allows an attacker to trivially predict other clients' QPNs. 顺序分配的QPN使攻击者可以轻松预测其他客户端的QPN。

- **min-entropy** n. 最小熵（衡量随机变量可预测程度的指标，值越低越容易猜中）；
  1. A low min-entropy means the key can be guessed with high probability. 低最小熵意味着密钥可以以较高概率被猜中。

- **On-Demand Paging (ODP)** n. 按需分页（允许RNIC在需要时动态注册内存页的特性）；
  1. ODP removes the need to pre-register memory regions, but exposes the entire address space. ODP消除了预先注册内存区域的需要，但会暴露整个地址空间。

**参考注文：**

本节系统地分析了IB架构中的10个安全漏洞。在密钥安全方面，各厂商的rkey生成算法随机性极低（Broadcom固定步长0x100，mlx5低熵随机），且所有进程共享一个密钥生成器；在内存地址方面，RDMA应用的内存分配连续可预测；在连接标识符方面，QPN顺序分配、PSN常被硬编码；在系统层面，缺乏加密认证、单一保护域、以及ODP特性进一步放大了安全风险。这10个漏洞共同构成了后续6类攻击的漏洞基础。

---

## 段六：Section 5 — Attacks on IBA（攻击实验）

**原文（关键词已标注）：**

Using the discovered vulnerabilities, an adversary could launch attacks in RDMA networks. In this section, we describe six potential attacks on RDMA networks, labeled **A1–A6**.

**A1: Packet Injection using Impersonation（数据包注入/身份冒充）**

As current RDMA systems enforce no 【source authentication】, an adversary can impersonate any other endpoint and inject packets that seem to belong to an established connection by another client. To inject an RDMA packet that is considered valid by the receiving endpoint, the adversary needs to know the QPN of the victim and the current PSN. Given that QP numbers are generated sequentially, an attacker can obtain expected QPNs of clients by simply connecting to the RDMA-enabled service and 【decrement】 the QPN that gets assigned to the attacker.

The attacker's spoofing tool for RoCE was able to generate 1.30 million packets per second (Mpps) for RoCEv1. Full 【enumeration】 of a random PSN thus took 13 seconds. If the victim does not use its connection for 13 seconds, the attacker can successfully inject 2²⁴ packets without disrupting the connection.

**A2: DoS Attack by Transiting QPs to an Error State（DoS攻击：强制QP进入错误状态）**

In IB-based architectures, 【memory errors】 such as incorrect operation numbers or an inconsistency between payload length and DMA length immediately lead to 【unrecoverable errors】, which will cause the RNIC to 【transit the QP to the error state】 and disconnect. A single fabricated packet injected into the connection was sufficient to effectively break the victim's connection. The attacker can sequentially break all QP connections by enumerating QPNs.

**A3: Unauthorized Memory Access（未授权内存访问）**

An attacker establishes an RDMA connection with a service and tries to access memory of other clients. RDMA applications typically share a PD for all RDMA resources and allocate private RDMA-accessible buffers for each new user in close proximity to each other, allowing an attacker to predict the virtual memory address of buffers belonging to other clients. The attacker then guesses the corresponding rkey based on its own obtained rkey. Since RDMA operations can be performed purely one-sided, the victim is unable to detect such attacks.

**A4: DoS based on Queue Pair Allocation Resource Exhaustion（QP资源耗尽）**

An attacker could open as many QP connections as possible and keep them open. The tested devices had different limits on the number of active QP connections: from 32,707 for Broadcom to 261,359 for Mellanox. By exhausting QP allocations, the attacker can prevent benign clients from opening connections.

**A5: Performance Degradation using Resource Exhaustion（性能降级攻击）**

An attacker floods the victim service with RDMA write requests to exhaust packet processing resources of the RNIC. With only two attackers flooding the victim, latency for regular RDMA requests increases by factor 3. For two or more attackers, the throughput of a victim reduces by factor 8–10 for RDMA read requests. Due to the one-sided nature of RDMA operations, resource exhaustion attacks can be executed "silently" with minor detection possibilities.

**A6: Facilitating Attacks using RDMA（利用RDMA辅助其他攻击）**

If an attacker has the privilege to preload a library to a victim's application, the attacker can inject code that establishes an RDMA connection to the attacker. This RDMA connection can then be used to read memory from the victim without involvement of the victim CPU. By continuously sweeping the readable memory, the attacker can eavesdrop on sensitive data such as passphrases.

**生词解析：**

- **source authentication** n. 源认证（验证数据包发送方身份的机制）；
  1. Without source authentication, any host can forge packets with a spoofed source address. 没有源认证，任何主机都可以伪造带有欺骗性源地址的数据包。

- **decrement** v. 递减；n. 减量；
  1. The attacker decrements the assigned QPN by one to predict the victim's QPN. 攻击者将分配到的QPN减一来预测受害者的QPN。

- **enumeration** n. 枚举，逐一遍历所有可能值；
  1. Full enumeration of the PSN space requires sending 2²⁴ packets. 对PSN空间的完整枚举需要发送2²⁴个数据包。

- **unrecoverable error** n. 不可恢复错误（导致连接强制断开的严重错误）；
  1. An unrecoverable error forces the RNIC to transition the QP to an error state. 不可恢复错误会强制RNIC将队列对转入错误状态。

- **transit to error state** 短语 转入错误状态（QP连接被强制中断）；
  1. Injecting a malformed packet causes the victim's QP to transit to an error state. 注入格式错误的数据包会导致受害者的QP进入错误状态。

- **resource exhaustion** n. 资源耗尽（通过大量请求耗尽系统资源的攻击方式）；
  1. Resource exhaustion attacks can prevent legitimate users from accessing the service. 资源耗尽攻击可阻止合法用户访问服务。

- **preload a library** 短语 预加载库（在程序启动时强制加载恶意动态链接库）；
  1. The attacker preloaded a malicious library to intercept the victim's function calls. 攻击者预加载了一个恶意库来拦截受害者的函数调用。

- **passphrase** n. 密码短语（用户输入的敏感认证信息）；
  1. The malware intercepted the victim's passphrase by reading it from memory via RDMA. 恶意软件通过RDMA从内存中读取受害者的密码短语。

**参考注文：**

本节基于第4节发现的漏洞，设计并验证了6类实际攻击。A1通过暴力枚举PSN实现数据包注入（平均13秒完成）；A2单包即可断开受害者连接；A3利用可预测的rkey和内存地址实现跨客户端内存读写；A4/A5分别通过耗尽QP数量和RNIC处理能力实现DoS；A6则展示了将RDMA用于静默内存窃取的恶意软件场景。所有攻击均通过实验得到验证，充分说明RDMA安全缺陷的实际威胁性。

---

## 段七：Section 6 — Vulnerability Assessment of Open-Source RDMA Systems（开源系统漏洞评估）

**原文（关键词已标注）：**

We analyse whether recent open-source applications and systems that use RDMA as a communication mechanism are vulnerable to the aforementioned attacks.

**Infiniswap** is a remote memory paging system that uses remote memory as a swap block device. Using A1, an attacker can inject a packet and modify the content of swapped pages. Infiniswap uses posix_memalign in a loop to allocate 1 GB buffers, allowing an attacker to predict the position and rkey of newly allocated buffers (difference of 0x40002000 bytes).

**Octopus** is an RDMA-enabled distributed persistent memory file system. It uses a hard-coded 【fixed starting PSN】, making PSN prediction trivial. Furthermore, all clients share a single buffer accessible using a single rkey — a misbehaving client can read and change RPCs of all other clients.

**HERD** implements an RDMA-enabled key-value store using a single memory buffer with a single registration for all RPC requests by clients. Systems such as Hermes and ccNUMA that are implemented using HERD 【inherit】 all its vulnerabilities.

**RamCloud** is a distributed key-value store based on two-sided RDMA. As one-sided RDMA operations are not enabled, performance degradation using A5 is not possible. However, unauthorized memory access A3 is still possible.

**Dare** specifies an RDMA-accelerated consensus protocol that uses a static initial PSN and registers all control data using a single registration. A misbehaving participant can 【forge】 the votes of other participants using packet injection and manipulate the consensus decision.

**Crail** is a high-performance distributed data store that maps and registers 1 GB fixed-size files in a loop, making the memory addresses and corresponding rkeys highly predictable.

**生词解析：**

- **paging system** n. 分页系统（利用远程内存作为虚拟内存交换空间的系统）；
  1. Infiniswap implements a remote memory paging system to expand available RAM. Infiniswap实现了一个远程内存分页系统来扩展可用内存。

- **swap block device** n. 交换块设备（操作系统用于内存换页的存储设备）；
  1. The paging system uses a remote RDMA buffer as a swap block device. 该分页系统将远程RDMA缓冲区用作交换块设备。

- **fixed starting PSN** n. 固定起始序列号（不随连接变化而随机化的PSN初始值）；
  1. A fixed starting PSN allows attackers to predict the sequence number of new connections. 固定起始PSN允许攻击者预测新连接的序列号。

- **inherit** v. 继承（子系统或派生系统继承了基础系统的漏洞）；
  1. Systems built on top of HERD inherit all of its security vulnerabilities. 基于HERD构建的系统继承了其所有安全漏洞。

- **RPC (Remote Procedure Call)** n. 远程过程调用（客户端请求服务器执行函数的通信协议）；
  1. A malicious client can overwrite another client's RPC request buffer. 恶意客户端可以覆盖其他客户端的RPC请求缓冲区。

- **forge** v. 伪造（数据包、投票或其他信息）；
  1. The attacker forged consensus votes by injecting crafted RDMA packets. 攻击者通过注入精心构造的RDMA数据包来伪造共识投票。

- **consensus protocol** n. 共识协议（分布式系统中用于达成一致决策的协议）；
  1. DARE implements a consensus protocol accelerated by RDMA. DARE实现了一个由RDMA加速的共识协议。

**参考注文：**

本节对6个主流开源RDMA系统（Infiniswap、Octopus、HERD、RamCloud、DARE、Crail）进行了漏洞评估，结果令人担忧：几乎所有系统都对未授权内存访问（A3）存在漏洞，大多数对DoS攻击（A4、A5）也无防护。最严重的是Octopus和DARE，由于使用硬编码PSN和单一rkey，连基本的数据完整性保护都无法保证，攻击者可直接篡改其他客户端的RPC请求乃至共识投票结果。

---

## 段八：Section 7 — Mitigation Mechanisms（缓解机制）

**原文（关键词已标注）：**

**M1 Randomization of QPNs（QPN随机化）**

An RDMA application creates a pool of unconnected QP descriptors with random QPNs. As soon as a QP connection is required, one of the QP descriptors is fetched and registered. This measure will introduce some overhead on the RDMA host, but can be deployed without modification of existing RDMA protocols. Given that modern RNICs provide hardware counters that are accessible by applications, 【bruteforce】 attempts could be detected.

**M2 Randomization of rkeys（rkey随机化）**

The application 【preregisters】 a pool of empty memory regions with different rkeys. When a new buffer needs to be registered, the application can randomly get a memory descriptor and remap it to the specified buffer using an ibv_rereg_mr call. This mechanism works on all RDMA devices.

**M3 Hardware Counters in RNICs（硬件计数器监控）**

Recent RNICs from Mellanox support port and hardware counters accessible by RDMA applications. The counter resp_remote_access_errors could be used to monitor invalid requests that resulted in access errors, enabling detection of 【flooding】 attacks.

**M4 Type 2 Memory Windows（类型2内存窗口）**

IBA offers type 2 memory windows which bind a memory region to a specified QP and prevent unauthorized memory access by other QPs. However, since IBA has no means of 【source authentication】, an attacker can still spoof an RDMA packet which contains the memory address and its rkey.

**M5 Protection Domains（保护域隔离）**

Each new RDMA client could be assigned to a different PD. However, this would increase the resource usage per client on RNICs. Many RDMA applications in practice use only a single PD for all connections to reduce memory overhead.

**M6 Encryption and Authentication in RDMA Protocols（协议层加密认证）**

- **RDMA-over-IPsec**: RoCE can use IPsec tunnels for encryption and authentication, preventing A1 and A2 for Ethernet networks, but cannot be applied to InfiniBand.
- **Application-layer Encryption**: Not feasible for RDMA since one-sided operations bypass the CPU entirely.
- **Encryption integrated in IBA**: Proposals such as sRDMA extend IBA with symmetric-cryptography-based authentication and encryption. This prevents information leakage to on-path attackers and all attacks based on packet injection.

**M7 Per-Client Resource Constraints（单客户端资源限制）**

RDMA-capable devices should limit the number of concurrently open QP connections and allocated resources on a per-client basis to prevent resource exhaustion attacks.

**M8 In-Network Filtering（网络入口过滤）**

Datacenter operators could deploy a filtering mechanism at the ingress of the network to prevent an attacker from injecting 【spoofed packets】, similar to BCP38 network ingress filtering.

**生词解析：**

- **bruteforce** v./n. 暴力破解（穷举所有可能值来找到正确答案）；
  1. The attacker used bruteforce to enumerate all possible PSN values. 攻击者使用暴力破解穷举了所有可能的PSN值。

- **preregister** v. 预注册（提前创建资源池以备后续使用）；
  1. Preregistering a pool of memory regions allows random rkey assignment at runtime. 预注册内存区域池允许在运行时随机分配rkey。

- **flooding** n./v. 泛洪攻击（向目标发送大量请求以耗尽资源）；
  1. Hardware counters can detect flooding attacks by monitoring abnormal request rates. 硬件计数器可通过监测异常请求速率来检测泛洪攻击。

- **spoofed packet** n. 伪造数据包（包含虚假源地址或其他伪造字段的数据包）；
  1. In-network filtering can drop spoofed packets before they reach the target. 网络入口过滤可在伪造数据包到达目标之前将其丢弃。

- **ibv_rereg_mr** n. RDMA重新注册内存区域的API调用（可更改内存区域的地址映射而保留rkey）；
  1. Using ibv_rereg_mr allows remapping a buffer to a randomly selected rkey. 使用ibv_rereg_mr允许将缓冲区重新映射到随机选择的rkey。

- **ingress filtering** n. 入口过滤（在网络边界丢弃来源可疑或伪造的数据包）；
  1. Ingress filtering at the datacenter edge prevents spoofed packets from entering. 数据中心边缘的入口过滤防止伪造数据包进入网络。

**参考注文：**

本节提出了8种缓解机制，分为软件侧（M1-M2：QPN/rkey随机化）、硬件利用（M3-M5：硬件计数器、内存窗口、保护域）和协议层改进（M6：sRDMA加密认证）以及网络层防护（M7-M8：资源限制、入口过滤）。其中M1和M2可无需修改硬件或协议立即部署，是当前最实用的短期方案；而M6（在IBA中集成加密认证）是从根本上解决问题的长期方案，但需要修改RDMA协议规范。

---

## 段九：Section 9 — Conclusion（结论）

**原文（关键词已标注）：**

RDMA architectures such as RoCE and InfiniBand were designed for 【HPC】 and private networks, and have 【neglected】 security in their design in favor of focusing on high performance. As illustrated by ReDMArk, the design of IBA and the implementation of IB-capable NICs contain multiple vulnerabilities and design flaws. These weaknesses allow an adversary to inject packets, gain unauthorized access to memory regions of other clients connected to an RDMA-based service with potentially 【drastic】 consequences, and effectively disrupt communication in RDMA networks.

Given that InfiniBand is deployed in public infrastructure and more providers plan to adopt RDMA networking, weak RDMA security creates real-world vulnerabilities in RDMA-enabled systems. This work shows the security implications of RDMA on cloud systems and demonstrates the critical importance of security in the design of upcoming versions of InfiniBand and RoCE (e.g., by fully integrating 【header authentication】 and 【payload encryption】). In addition, developers of RDMA-enabled systems must be aware of the threats introduced by RDMA networking and should employ mitigations such as using type 2 memory windows, a separate PD for each connection, and the proposed algorithms to randomize the QPN and the rkey generation.

**生词解析：**

- **HPC (High-Performance Computing)** n. 高性能计算（科学计算、超算等需要极高计算性能的场景）；
  1. InfiniBand was originally designed for HPC clusters in isolated environments. InfiniBand最初是为隔离环境中的高性能计算集群设计的。

- **neglect** v. 忽视，忽略（在设计中未充分考虑某方面）；
  1. The protocol designers neglected security in favor of maximum performance. 协议设计者为了追求最高性能而忽视了安全性。

- **drastic** adj. 严重的，极端的，影响深远的；
  1. An unauthorized memory write can have drastic consequences for application correctness. 未经授权的内存写入可能对应用程序的正确性产生严重影响。

- **header authentication** n. 包头认证（对数据包头部字段进行密码学认证，防止伪造）；
  1. Header authentication would prevent attackers from spoofing QPN or PSN fields. 包头认证将防止攻击者伪造QPN或PSN字段。

- **payload encryption** n. 负载加密（对数据包携带的实际数据进行加密保护）；
  1. Payload encryption ensures that on-path attackers cannot read transferred data. 负载加密确保路径上的攻击者无法读取传输的数据。

**参考注文：**

结论部分强调，RDMA技术的安全缺陷并非偶然疏漏，而是源于其设计哲学对性能的极端优先。随着RDMA向公有云环境扩展，这些设计缺陷已从"私有网络中的已知风险"演变为"公共基础设施上的真实威胁"。作者呼吁在下一代InfiniBand和RoCE标准中从根本上集成包头认证与负载加密，同时建议开发者立即采用本文提出的软件层缓解措施。

---

## 附录：核心术语速查表

| 英文术语 | 中文释义 |
|---------|---------|
| RDMA | 远程直接内存访问 |
| InfiniBand (IB) | 无限带宽网络架构 |
| RoCE | 以太网上的RDMA |
| RNIC | RDMA能力网络接口卡 |
| Queue Pair (QP) | 队列对（RDMA通信端点） |
| QPN | 队列对编号 |
| PSN | 数据包序列号 |
| rkey | 远程内存访问密钥 |
| lkey | 本地内存访问密钥 |
| Protection Domain (PD) | 保护域 |
| Memory Region | 内存区域 |
| Memory Window | 内存窗口 |
| ODP | 按需分页 |
| ASLR | 地址空间布局随机化 |
| IPsec | 互联网安全协议 |
| DoS | 拒绝服务攻击 |
| covert channel | 隐蔽信道 |
| min-entropy | 最小熵（密钥可预测性指标） |
| packet injection | 数据包注入攻击 |
| impersonation | 身份冒充 |
| brute-force | 暴力破解 |
| side-channel | 侧信道攻击 |
| ingress filtering | 网络入口过滤 |
_build:
  render: never
  list: never
  publishResources: false
---

