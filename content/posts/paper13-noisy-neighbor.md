---
title: "【论文精读】Noisy Neighbor: Exploiting RDMA for Resource Exhaustion Attacks in Containerized Clouds"
date: 2025-03-17
draft: false
tags: ["RDMA", "容器安全", "资源耗尽攻击", "云安全", "论文精读", "BlueField", "SR-IOV"]
categories: ["论文精读"]
description: "对 Noisy Neighbor 论文的逐段深度解析，涵盖容器化云中 RDMA 资源耗尽攻击的原理、实验验证及 HT-Verbs 防御框架。"
---

> **原文信息**
> - **标题**：Noisy Neighbor: Exploiting RDMA for Resource Exhaustion Attacks in Containerized Clouds
> - **作者**：Gunwoo Kim, Taejune Park, Jinwoo Kim
> - **机构**：Kwangwoon University; Chonnam National University（韩国）
> - **arXiv**：2510.12629v1 \[cs.CR\] 14 Oct 2025

---

## 标题解读

**Noisy Neighbor: Exploiting RDMA for Resource Exhaustion Attacks in Containerized Clouds**

标题中的重点词汇：

- **noisy neighbor** *n.* 吵闹邻居；云计算术语，指共享物理资源的某个租户因大量消耗资源而影响同机其他租户性能的现象；
  1. The noisy neighbor problem is amplified in RDMA-enabled containers because there are no per-container resource limits on the RNIC. 在启用 RDMA 的容器中，吵闹邻居问题更为严重，因为 RNIC 上没有针对每个容器的资源限制。

- **resource exhaustion** *n.* 资源耗尽；通过大量消耗系统关键资源（队列、缓存、处理管道）使合法服务无法正常运行的攻击手段；
  1. Resource exhaustion attacks on RNICs can cause a 93.9% bandwidth drop for victim containers sharing the same hardware. 针对 RNIC 的资源耗尽攻击可导致共享相同硬件的受害者容器带宽下降 93.9%。

- **containerized cloud** *n.* 容器化云；以容器（如 Docker/Kubernetes）而非虚拟机为基本部署单元的云计算基础设施；
  1. Containerized clouds rely on SR-IOV to provide RDMA capabilities to individual containers without full virtualization overhead. 容器化云依赖 SR-IOV 为各容器提供 RDMA 能力，同时避免完整虚拟化的开销。

- **RDMA** (Remote Direct Memory Access) *n.* 远程直接内存访问；
  1. RDMA bypasses the host OS networking stack, enabling microsecond-level latency for containerized applications. RDMA 绕过主机操作系统网络协议栈，为容器化应用实现微秒级延迟。

---

## Abstract（摘要）

### 段落原文

In modern containerized cloud environments, the adoption of 【RDMA】 has expanded to reduce CPU overhead and enable high-performance data exchange. Achieving this requires strong 【performance isolation】 to ensure that one container's RDMA workload does not degrade the performance of others, thereby maintaining critical security assurances. However, existing isolation techniques are difficult to apply effectively due to the complexity of 【microarchitectural resource management】 within RDMA NICs (RNICs). This paper experimentally analyzes two types of resource exhaustion attacks on NVIDIA 【BlueField-3】: (i) 【state saturation attacks】 and (ii) 【pipeline saturation attacks】. Our results show that state saturation attacks can cause up to a **93.9% loss in bandwidth**, a **1,117× increase in latency**, and a **115% rise in cache misses** for victim containers, while pipeline saturation attacks lead to severe link-level congestion and significant 【amplification】, where small verb requests result in disproportionately high resource consumption. To mitigate these threats and restore predictable security assurances, we propose 【HT-Verbs】, a threshold-driven framework based on real-time per-container RDMA verb telemetry and adaptive resource classification that partitions RNIC resources into hot, warm, and cold tiers and throttles abusive workloads without requiring hardware modifications.

### 生词解析

- **performance isolation** *n.* 性能隔离；在多租户环境中，保证一个租户的高负载工作不会影响其他租户正常性能的系统能力；
  1. Performance isolation breaks down in RDMA-enabled containers because RNICs lack per-VF resource quotas. 在启用 RDMA 的容器中，性能隔离发生故障，因为 RNIC 缺乏针对每个 VF 的资源配额。

- **microarchitectural resource management** *n.* 微架构资源管理；对处理器或 NIC 内部细粒度硬件资源（缓存、队列、流水线）的分配和调度机制；
  1. The complexity of microarchitectural resource management in RNICs makes it hard to enforce fair resource sharing across containers. RNIC 中微架构资源管理的复杂性使得跨容器的公平资源共享难以实施。

- **BlueField-3** *n.* 英伟达 BlueField-3；NVIDIA 推出的智能网卡（SmartNIC/DPU），集成了可编程 ARM 核心和 RDMA 功能；
  1. The NVIDIA BlueField-3 is widely deployed in cloud environments for both RDMA acceleration and programmable packet processing. NVIDIA BlueField-3 广泛部署于云环境中，用于 RDMA 加速和可编程数据包处理。

- **state saturation attack** *n.* 状态饱和攻击；通过大量创建队列对、消耗连接缓存等方式耗尽 RNIC 状态存储资源的攻击类型；
  1. State saturation attacks exhaust RNIC on-chip caches by flooding queue pairs, evicting legitimate connection state. 状态饱和攻击通过大量创建队列对来耗尽 RNIC 片上缓存，驱逐合法连接的状态信息。

- **pipeline saturation attack** *n.* 流水线饱和攻击；通过大量 RDMA verb 请求淹没 RNIC 的 TX/RX 处理流水线，造成链路级拥塞的攻击类型；
  1. Pipeline saturation attacks flood the RNIC's TX/RX pipelines with verbs, generating over 20,000 PAUSE frames. 流水线饱和攻击用大量 verb 请求淹没 RNIC 的 TX/RX 流水线，产生超过 2 万个 PAUSE 帧。

- **amplification** *n.* 放大效应；攻击者发送的少量数据触发远超自身载荷的 RNIC 内部资源消耗的现象；
  1. The amplification effect means an 8-byte RDMA verb causes the RNIC to receive 23.1 bytes of protocol overhead. 放大效应意味着一个 8 字节的 RDMA verb 会导致 RNIC 接收到 23.1 字节的协议开销。

- **HT-Verbs** *n.* 基于阈值的 Verb 管理框架；本文提出的针对容器化 RDMA 环境的动态资源分类与节流防御系统；
  1. HT-Verbs monitors per-QP RDMA verb patterns in real time to detect and throttle malicious containers without hardware changes. HT-Verbs 实时监控每个 QP 的 RDMA verb 模式，无需硬件改动即可检测并节流恶意容器。

### 参考注文

本摘要阐明了容器化云中 RDMA 性能隔离失效的安全问题：由于 RNIC 内部微架构资源（缓存、队列、处理流水线）被所有容器共享而无细粒度配额，恶意容器可通过状态饱和或流水线饱和攻击，造成受害者容器高达 93.9% 的带宽损失和 1117 倍的延迟增加。论文提出的 HT-Verbs 框架通过实时 verb 遥测和三级资源分类（冷/温/热）来应对这一威胁。

---

## 段二 Section 1：Introduction（引言）

### 段落原文

RDMA (Remote Direct Memory Access) is a high-performance technology that enables direct memory access between systems without involving the host CPU or operating system kernel, significantly reducing latency and CPU overhead while delivering high throughput. RDMA communicates through RDMA NICs (RNICs), such as the 【NVIDIA BlueField-3】, which are designed with programmable cores integrated into the host NIC system on chip. These cores are directly connected to data center Ethernet or InfiniBand links, allowing RNICs to process packets on-path and respond without involving the host OS networking stack, thus reducing latency for certain applications.

Despite these advantages, RNICs face a significant challenge in cloud environments: the 【performance isolation problem】. This issue arises when a malicious tenant exhausts RNIC 【microarchitecture resources】 (e.g., NIC caches, processing units) by generating abusive RDMA workloads. Specifically, an attacker can overload the RNIC through carefully crafted RDMA operations, resulting in increased latency, elevated cache miss rates, and, in severe cases, 【denial of service (DoS)】 due to internal resource starvation. Such resource exhaustion disrupts critical NIC components and degrades the performance of co-located tenants by creating 【system-wide bottlenecks】, thus undermining the security assurance guarantees that multi-tenant clouds depend on to ensure availability and predictable performance.

### 生词解析

- **on-path processing** *n.* 路径上处理；RNIC 在数据包传输路径上直接进行协议处理，无需将数据包送往主机 CPU；
  1. BlueField-3's on-path processing allows RDMA operations to complete without any host CPU involvement. BlueField-3 的路径上处理使 RDMA 操作能够在不涉及任何主机 CPU 的情况下完成。

- **microarchitecture resource** *n.* 微架构资源；处理器或 NIC 内部的细粒度硬件组件（如缓存、队列缓冲区、执行单元）；
  1. RNIC microarchitecture resources such as the WQE cache and QP context tables are shared across all virtual functions. RNIC 微架构资源（如 WQE 缓存和 QP 上下文表）在所有虚拟功能之间共享。

- **resource starvation** *n.* 资源饥饿；由于某些进程/租户过度占用资源，导致其他合法进程长期无法获取所需资源的状态；
  1. Resource starvation occurs when an attacker exhausts all available RNIC queue pairs, blocking victim containers from creating new connections. 当攻击者耗尽所有可用 RNIC 队列对时，就会发生资源饥饿，阻止受害者容器创建新连接。

- **system-wide bottleneck** *n.* 系统全局瓶颈；影响整个系统而非单一进程/租户的性能制约点；
  1. A flooded WQE cache creates a system-wide bottleneck that degrades all containers sharing the RNIC simultaneously. 被淹没的 WQE 缓存造成系统全局瓶颈，同时降低所有共享 RNIC 的容器的性能。

### 参考注文

本段说明了 RDMA 在容器化云中面临的核心安全挑战：尽管 RDMA 的路径上处理特性带来了极低延迟，但这种绕过操作系统的设计也意味着 RNIC 内部资源无法受到系统级 QoS 策略的保护。当恶意容器滥用 RDMA 操作时，会在 RNIC 内部造成系统全局瓶颈，破坏多租户云赖以生存的性能隔离保证。

---

## 段三 Section 2：Background and Motivation（背景与动机）

### 段落原文

**RDMA Workflow.** RDMA communication is facilitated through an API called 【verbs】, which are categorized into 【control verbs】 and 【data verbs】. Control verbs manage the initialization process by creating and configuring key resources, such as 【queue pairs (QPs)】 and 【completion queues (CQs)】. Subsequently, the application allocates memory in the host's DRAM, maps virtual addresses to physical ones, and registers these memory regions with the RNIC. This registration enables the RNIC to directly access memory without CPU intervention.

Data verbs, such as WRITE and SEND, are used to actually move bytes between endpoints. Data verbs fall into two categories: (i) 【two-sided verbs】 and (ii) 【one-sided verbs】. In the former, both the sender and the receiver must post work requests. In the latter, after capability negotiation, the initiator can read from, write to, or perform atomic operations on the peer's memory without further involvement of the remote CPU.

RDMA defines two transport modes: (i) 【Reliable Connected (RC)】 and (ii) 【Unreliable Connected (UC)】. RC provides reliable, one-to-one communication, similar to TCP. UC allows a single QP to communicate with multiple peers, similar to UDP, greatly reducing the number of QPs a thread must maintain. The trade-off is that UC omits support for one-sided operations and end-to-end reliability.

### 生词解析

- **verbs** *n.* （RDMA）操作原语；RDMA 编程接口中定义的标准化操作命令集合，包含控制面和数据面操作；
  1. An attacker can flood the RNIC with high-rate verb requests to saturate its TX/RX processing pipelines. 攻击者可以用高速率的 verb 请求淹没 RNIC，使其 TX/RX 处理流水线饱和。

- **control verb** *n.* 控制面 verb；用于初始化 RDMA 环境、创建队列对、注册内存等管理操作的命令；
  1. Control verbs like create_qp allocate on-chip RNIC resources and are therefore exploitable for resource exhaustion. 创建队列对等控制 verb 会分配 RNIC 片上资源，因此可被用于资源耗尽攻击。

- **data verb** *n.* 数据面 verb；用于实际执行数据传输（SEND、WRITE、READ、ATOMIC）的 RDMA 命令；
  1. Even lightweight data verbs with minimal payloads can saturate the WQE cache through sheer volume. 即使是载荷极小的轻量级数据 verb，也可以通过庞大的数量使 WQE 缓存饱和。

- **two-sided verb** *n.* 双边操作；发送方和接收方均需提交工作请求的 RDMA 操作（如 SEND/RECV）；
  1. Two-sided SEND operations require the remote CPU to pre-post a receive buffer before data can be delivered. 双边 SEND 操作要求远端 CPU 在数据投递前预先提交一个接收缓冲区。

- **one-sided verb** *n.* 单边操作；仅由发起方驱动、无需远端 CPU 参与的 RDMA 操作（如 WRITE、READ、ATOMIC）；
  1. One-sided WRITE verbs allow an attacker to saturate the RNIC pipeline without any response from the remote side. 单边 WRITE verb 允许攻击者使 RNIC 流水线饱和，而无需远端任何响应。

- **Reliable Connected (RC)** *n.* 可靠连接；提供端到端可靠性的 RDMA 传输模式，每对通信端点需独占一个队列对；
  1. RC mode's per-peer QP requirement becomes a scalability bottleneck in large clusters, but enables the most powerful attack in queue flooding. RC 模式的每对端点独占一个 QP 的要求在大型集群中成为可扩展性瓶颈，但也使队列泛洪攻击最为有效。

- **Unreliable Connected (UC)** *n.* 不可靠连接；允许单个队列对与多个对端通信的 RDMA 传输模式，类似 UDP，无需可靠性确认；
  1. UC mode's single-QP multi-peer design enables higher attack injection rates by eliminating reliability overhead. UC 模式单队列对支持多对端的设计，通过消除可靠性开销来实现更高的攻击注入速率。

### 参考注文

本节介绍了 RDMA 编程模型的核心概念：控制 verb 负责初始化资源，数据 verb 负责数据传输；双边操作需通信双方参与，单边操作仅由发起方驱动；RC 模式提供可靠传输但占用更多队列对，UC 模式更高效但不支持单边操作。这些基础概念直接对应后文攻击设计中的具体利用方式。

---

### 段四 RDMA-enabled Container Environment

### 段落原文

In modern cloud-native infrastructures, containers frequently require direct access to high-performance networking capabilities such as RDMA. To support this, a single physical RNIC can be virtualized through 【SR-IOV (Single Root I/O Virtualization)】, which provides one 【Physical Function (PF)】 for global management and multiple 【Virtual Functions (VFs)】, each maintaining its own hardware context (e.g., queue pair and memory translation state). By assigning a VF to a container's network namespace, containers can achieve near-native, low-latency access to the virtualized RNIC.

A physical RNIC consists of various microarchitecture resources, each dedicated to specific metadata storage and fast access. For instance, in the case of the NVIDIA BlueField-3, the 【Memory Translation Table (MTT)】 handles virtual-to-physical address translation, while the 【Memory Protection Table (MPT)】 enforces memory access permissions. The 【Interconnect Connect Memory (ICM) cache】 stores QP state and connection metadata to manage RDMA operations. Additionally, the 【WQE (Work Queue Entry) cache】 holds pre-fetched entries from transmission and reception queues, enabling rapid processing in the TX/RX pipeline.

### 生词解析

- **SR-IOV** (Single Root I/O Virtualization) *n.* 单根 I/O 虚拟化；PCIe 标准扩展，允许单个物理 PCIe 设备向多个虚拟机或容器暴露独立的虚拟功能（VF）；
  1. SR-IOV allows each container to have direct, near-native access to its own virtual RNIC function without hypervisor intervention. SR-IOV 允许每个容器直接、近原生地访问其自己的虚拟 RNIC 功能，无需虚拟机监控器介入。

- **Physical Function (PF)** *n.* 物理功能；SR-IOV 中代表整个物理设备的功能单元，拥有完整配置权限，通常由管理员使用；
  1. The Physical Function has exclusive access to global RNIC configuration and VF management operations. 物理功能拥有对全局 RNIC 配置和 VF 管理操作的专属访问权限。

- **Virtual Function (VF)** *n.* 虚拟功能；SR-IOV 中暴露给虚拟机或容器的轻量级设备实例，共享物理 RNIC 的硬件资源但拥有独立的配置空间；
  1. Each container is assigned its own VF, but all VFs share the same physical RNIC microarchitecture resources, enabling contention attacks. 每个容器被分配自己的 VF，但所有 VF 共享相同的物理 RNIC 微架构资源，使争用攻击成为可能。

- **Memory Translation Table (MTT)** *n.* 内存转换表；RNIC 内部用于记录已注册内存区域虚拟地址到物理地址映射关系的硬件表；
  1. Exhausting the MTT by registering large numbers of memory regions forces costly cache misses on every subsequent DMA operation. 通过注册大量内存区域来耗尽 MTT，迫使后续每次 DMA 操作都产生代价高昂的缓存缺失。

- **Memory Protection Table (MPT)** *n.* 内存保护表；RNIC 内部存储内存访问权限信息的硬件表，用于验证远端访问是否持有正确的内存密钥；
  1. The MPT enforces per-memory-region access permissions to prevent unauthorized RDMA reads and writes. MPT 强制实施每个内存区域的访问权限，以防止未授权的 RDMA 读写操作。

- **ICM (Interconnect Connect Memory) cache** *n.* 互连内存缓存；RNIC 上存储队列对状态和连接元数据的片上缓存，加速 RDMA 操作处理；
  1. Flooding the ICM cache with excessive QP states evicts legitimate connection metadata and causes processing delays. 用过多的 QP 状态淹没 ICM 缓存，会驱逐合法的连接元数据并导致处理延迟。

- **WQE (Work Queue Entry) cache** *n.* 工作队列项缓存；RNIC 上预取传输/接收队列条目的片上缓存，使 TX/RX 流水线能够快速处理 verb 请求；
  1. Saturating the WQE cache with attacker verbs evicts victim work requests, causing severe transmission delays. 用攻击者的 verb 饱和 WQE 缓存，会驱逐受害者的工作请求，造成严重的传输延迟。

### 参考注文

本节描述了容器化云中的 RDMA 部署架构：通过 SR-IOV，单个物理 RNIC 被划分为多个 VF 分配给各容器，但 MTT、MPT、ICM 缓存、WQE 缓存和 TX/RX 流水线等核心微架构资源仍然被所有 VF 共享，这正是本文所有攻击的根本原因所在。

---

## 段五 Section 3：Resource Exhaustion Attacks（资源耗尽攻击）

### 段落原文

**Threat Model.** We consider a multi-tenant environment in an on-premises cloud where tenants share the same physical machine. The attacker's goal is to induce performance isolation issues on the target host via malicious RDMA operations. To achieve this, the attacker requires a 【decoy container】 co-residing on the target host. Such 【co-residency】 can be achieved through brute-force attacks by repeatedly creating containers combined with co-residency checks. Both the attacker container and the decoy container are assumed to have access to their own VFs assigned via SR-IOV. They share the same L3 domain, enabling communication over 【RoCEv2】. Note that we do not consider RDMA protocol-related vulnerabilities, and thus assume that the attacker fully complies with the RDMA protocol and standard operations.

### 生词解析

- **decoy container** *n.* 诱饵容器；攻击者在目标主机上部署的、用于与攻击者容器建立 RDMA 连接以发动资源耗尽攻击的协同容器；
  1. The attacker uses a decoy container on the target host to establish QPs and launch RDMA-based flooding attacks. 攻击者在目标主机上使用诱饵容器来建立 QP 并发动基于 RDMA 的泛洪攻击。

- **co-residency** *n.* 共处同机；攻击者的容器/虚拟机与目标受害者的容器/虚拟机运行在同一台物理服务器上的状态；
  1. Achieving co-residency on containerized platforms has been demonstrated feasible through brute-force container spawning. 通过暴力创建容器的方式实现容器化平台上的共处同机已被证明是可行的。

- **RoCEv2** (RDMA over Converged Ethernet version 2) *n.* 第二版融合以太网 RDMA；在 UDP/IP 之上运行 InfiniBand 传输层协议的 RDMA 实现，支持跨子网路由；
  1. RoCEv2 is the dominant RDMA protocol in containerized clouds, enabling RDMA communication across standard data center Ethernet. RoCEv2 是容器化云中占主导地位的 RDMA 协议，支持通过标准数据中心以太网进行 RDMA 通信。

- **L3 domain** *n.* 三层网络域；共享同一 IP 子网路由域的网络区域，是 RoCEv2 通信的前提条件；
  1. Containers in the same L3 domain can establish RDMA connections over RoCEv2 without additional routing configuration. 同一三层网络域中的容器可以在无需额外路由配置的情况下通过 RoCEv2 建立 RDMA 连接。

### 参考注文

本节定义了攻击威胁模型：攻击者在目标主机上部署一个诱饵容器（通过暴力共置攻击实现共处同机），并利用两个容器各自的 VF 通过 RoCEv2 建立 RDMA 连接。值得注意的是，该攻击完全遵循 RDMA 协议规范，无需利用任何协议级漏洞——攻击的本质是合规但恶意的资源滥用。

---

### 段六 Attack Scenarios（攻击场景分类）

### 段落原文

**3.3 State Saturation Attacks.**

**Queue Flooding.** The attacker can rapidly instantiate a large number of queue pairs (QPs) and corresponding completion queues (CQs), similar to a 【TCP SYN flooding attack】. Each new QP allocates on-chip resources (QP state, work queue buffers), and the flood of posted work requests saturates the TX/RX pipelines, evicts legitimate QP state from the 【connection cache】, and exhausts WQE cache entries. In UC mode, the attacker can cycle large batches of SEND/RECV requests on a single QP without requiring collaboration from the decoy container. In RC mode, the creation of many QPs is more critical.

**Cache Depletion.** The attacker can issue one-sided READ or WRITE operations with continuously increasing remote addresses across its QPs, flushing the 【translation cache】 and 【connection cache】, and driving up 【cache miss rates】 on the shared RNIC. The resulting high miss rates introduce delays in the TX/RX pipelines and degrade the performance of co-located victim containers.

**3.4 Pipeline Saturation Attacks.**

**Verbs Flooding.** The attacker injects verbs at the maximum throughput for each QP. In RC mode, each verb invocation allocates per-QP context and triggers 【reliability handshakes】, stressing the atomic engine and TX/RX pipelines, evicting WQE entries and translation cache state, and causing 【pipeline stalls】 and context-switch thrashing. In UC mode, the absence of reliability overhead enables higher injection rates. Stress on the WQE FIFO and TX/RX pipelines increases rapidly, leading to 【PCIe back-pressure】.

**Verbs Amplification.** The attacker designs a more sophisticated amplification attack, in which a small RDMA verb request triggers significant resource consumption on the RNIC, similar to traditional 【DDoS amplification attacks】. Amplification occurs if the RNIC processes more bytes than the payload size of the issued verb, arising from additional overhead introduced by protocol headers, control traffic, and flow-control events.

### 生词解析

- **TCP SYN flooding** *n.* TCP SYN 泛洪攻击；发送大量伪造 SYN 包耗尽服务器连接状态表的经典 DoS 攻击，此处类比 QP 泛洪攻击；
  1. Queue flooding is analogous to TCP SYN flooding, where each new QP allocation consumes precious on-chip RNIC state. 队列泛洪类似于 TCP SYN 泛洪，其中每次新 QP 分配都会消耗宝贵的 RNIC 片上状态。

- **connection cache** *n.* 连接缓存（ICM 缓存）；RNIC 内部缓存当前活跃连接 QP 上下文信息的片上存储；
  1. Filling the connection cache with attacker QP contexts evicts victim QP state, causing expensive re-fetches for every victim operation. 用攻击者 QP 上下文填满连接缓存，会驱逐受害者 QP 状态，导致每次受害者操作都需要昂贵的重新获取。

- **translation cache** *n.* 地址转换缓存（MTT/MPT 缓存）；RNIC 内部缓存内存地址转换和权限信息的片上存储；
  1. Cache depletion attacks drive up translation cache miss rates by accessing new memory regions for every operation. 缓存耗尽攻击通过每次操作访问新内存区域来提高地址转换缓存的缺失率。

- **cache miss rate** *n.* 缓存缺失率；请求在缓存中未命中、需要从较慢的存储层获取数据的比例；
  1. A 115% rise in cache miss rates indicates that the attacker has successfully depleted the victim's share of the RNIC cache. 115% 的缓存缺失率上升表明攻击者已成功耗尽受害者在 RNIC 缓存中的份额。

- **reliability handshake** *n.* 可靠性握手；RC 传输模式中 RNIC 为确保数据包按序可靠投递而执行的确认/重传交互；
  1. Each RC verb triggers reliability handshakes that consume atomic engine resources and slow down co-resident victim flows. 每个 RC verb 触发可靠性握手，消耗原子引擎资源并降低共处同机受害者流的速度。

- **pipeline stall** *n.* 流水线停顿；处理流水线因资源冲突或缓存缺失而暂停执行的状态；
  1. WQE cache exhaustion causes pipeline stalls that propagate through the RNIC's TX and RX processing units. WQE 缓存耗尽导致流水线停顿，并传播到 RNIC 的 TX 和 RX 处理单元。

- **PCIe back-pressure** *n.* PCIe 背压；PCIe 总线因接收方缓冲区满而向发送方施加的流量控制压力，导致传输减速；
  1. Verb flooding in UC mode generates PCIe back-pressure that affects all devices sharing the same PCIe root port. UC 模式下的 verb 泛洪产生 PCIe 背压，影响共享同一 PCIe 根端口的所有设备。

- **DDoS amplification attack** *n.* DDoS 放大攻击；攻击者发送小请求引发目标系统或中间节点产生大量响应流量的分布式拒绝服务攻击；
  1. RDMA verb amplification resembles DNS amplification: a small 8-byte request triggers 23.1 bytes of RNIC processing overhead. RDMA verb 放大类似 DNS 放大攻击：一个 8 字节的请求触发 23.1 字节的 RNIC 处理开销。

### 参考注文

本节详细描述了四种具体攻击手段。状态饱和类攻击分为队列泛洪（快速创建大量 QP 耗尽连接状态资源）和缓存耗尽（持续访问新内存地址提高转换缓存缺失率）；流水线饱和类攻击分为 verb 泛洪（高速发送大量操作命令淹没处理流水线）和 verb 放大（利用协议头部开销产生资源消耗放大效应）。所有攻击均不依赖协议漏洞，而是利用共享 RNIC 缺乏资源隔离的设计缺陷。

---

## 段七 Section 4：Evaluation（实验评估）

### 段落原文

**Impact of Queue Flooding.** Under RC mode, the victim's bandwidth dropped from **26.61Gbps to 1.64Gbps**, marking a **93.9% reduction**. Under UC mode, a similar degradation trend was observed. In contrast, the attacker's bandwidth remained consistently below 5Gbps. These results demonstrate that even lightweight RDMA verbs with minimal data payloads can saturate critical shared RNIC resources, such as TX and RX processing pipelines and WQE cache entries, severely degrading victim performance. This highlights that **control intensive verbs**, despite their low data volume, are sufficient to monopolize RNIC resources.

To validate the generalizability in multi-victim scenarios, we conducted an additional experiment with three distinct victim containers (initial QP counts: 6, 3, 1). By the end of the **40-second test**, the bandwidths fell to **1.02Gbps for Victim A, 0.49Gbps for Victim B, and 0.15 Gbps for Victim C** — all suffering drops over 90%.

**Impact of Cache Depletion.** Under normal conditions, the victim container exhibited an average latency of **1.56 μs**. During the attack, the victim's latency sharply increased to **1,746.34 μs**, representing approximately a **1,117-fold increase**. The cache miss rate rose from **14.48% to 31.07%** over 40 seconds.

**Impact of Verbs Flooding.** In RC mode, WRITE verb flooding generates no PAUSE frames at a single QP but escalates rapidly beyond four QPs, reaching a maximum of **42,535 frames** at 24 QPs. In UC mode, even a single QP of WRITE or SEND verbs provokes over **22,000 PAUSE frames**.

**Impact of Verbs Amplification.** In RC mode, the ATOMIC verb yields the highest amplification ratio (**23.1**), meaning the RNIC receives **23.1 bytes** for every 8 bytes generated by the attacker, followed by WRITE (22.01), READ (20.1), and SEND (18.26).

### 生词解析

- **PAUSE frame** *n.* 暂停帧；IEEE 802.3x 以太网流量控制机制中，接收方发送给发送方、要求其暂停传输的控制帧；可量化链路级拥塞程度；
  1. The number of PAUSE frames generated during verb flooding is used as a metric for link-level congestion severity. verb 泛洪期间产生的 PAUSE 帧数量被用作衡量链路级拥塞严重程度的指标。

- **amplification ratio (AR)** *n.* 放大比率；攻击者每发送 1 字节有效载荷，RNIC 实际处理的字节数；
  1. An amplification ratio of 23.1 for ATOMIC verbs means the attacker imposes 2.89x more load on the RNIC than the data it actually sends. ATOMIC verb 的 23.1 放大比率意味着攻击者对 RNIC 施加的负载是其实际发送数据量的 2.89 倍。

- **ib_write_bw** *n.* InfiniBand 写带宽测试工具；perftest 工具包中用于测量 RDMA WRITE 操作带宽的标准基准测试程序；
  1. The victim container runs ib_write_bw as a baseline workload to measure bandwidth degradation during the attack. 受害者容器运行 ib_write_bw 作为基线工作负载，以测量攻击期间的带宽降级。

- **control intensive verb** *n.* 控制密集型操作；以频繁触发 RNIC 内部状态转换和控制逻辑为主要特征、而非传输大量数据的 RDMA 操作；
  1. Control intensive verbs with tiny payloads monopolize RNIC processing resources more efficiently than large data transfers. 载荷极小的控制密集型操作比大数据传输更有效地垄断 RNIC 处理资源。

### 参考注文

本节通过实验定量证明了四类攻击的严重性：队列泛洪导致 93.9% 带宽损失，在多受害者场景下同样有效；缓存耗尽导致延迟增加 1117 倍、缓存缺失率翻倍；verb 泛洪在 UC 模式下仅用单个 QP 即可产生两万余个 PAUSE 帧，造成严重链路拥塞；verb 放大攻击中 ATOMIC 操作的放大比达到 23.1，攻击者以极小的流量代价造成 RNIC 的巨大处理负担。

---

## 段八 Section 5：Discussion and Mitigation（讨论与缓解措施）

### 段落原文

**Root Causes.** The feasibility of RDMA-based resource contention attacks in containerized environments stems from the fact that, under SR-IOV virtualization, all containers share the same set of low-level RNIC microarchitectural resources — such as the MTT, MPT, Connection Cache, WQE cache, and the TX/RX processing pipelines — without any fine-grained partitioning or per-container enforcement. Since SR-IOV assigns only logical VFs to containers, these VFs still contend for on-chip buffers and caches in a 【first-come, first-served】 manner. Commodity RNICs lack any fine-grained fairness or abuse detection, employing only 【best-effort or round-robin scheduling】 across QPs. Because RDMA bypasses the host OS networking stack entirely, container runtimes and OS-level QoS controls cannot observe or throttle these flows.

**Existing Countermeasures.** NVIDIA's RoCE protocol supports only four predefined 【ToS (Type of Service)】 values, limiting the flexibility of priority configurations. Moreover, since QoS mechanisms are primarily designed to regulate traffic prioritization, they cannot directly mitigate resource exhaustion attacks targeting RNIC resources such as the TX/RX processing unit or internal caches. Despite throttling the attacker at 15 seconds with QoS, Containers A and B still suffered a **73.4% performance degradation** compared to the baseline.

### 生词解析

- **first-come, first-served** *phr.* 先到先得；资源分配策略，按请求到达顺序分配，不考虑优先级或公平性；
  1. RNIC's first-come, first-served resource allocation allows aggressive attacker QPs to monopolize the WQE cache. RNIC 的先到先得资源分配策略允许激进的攻击者 QP 垄断 WQE 缓存。

- **round-robin scheduling** *n.* 轮询调度；为每个队列/进程依次分配相等时间片的调度策略，不区分优先级或行为是否正常；
  1. Round-robin QP scheduling treats malicious and legitimate containers equally, making it inherently vulnerable to abuse. 轮询 QP 调度平等对待恶意容器和合法容器，使其本质上容易受到滥用。

- **QoS (Quality of Service)** *n.* 服务质量；通过流量优先级、带宽限速等机制保障网络服务性能的技术体系；
  1. QoS can cap bandwidth for an attacker container but cannot prevent it from exhausting RNIC's internal processing caches. QoS 可以限制攻击者容器的带宽，但无法阻止其耗尽 RNIC 内部的处理缓存。

- **ToS (Type of Service)** *n.* 服务类型；IP 头部字段，用于标记数据包的优先级以便网络设备进行差异化处理；
  1. RoCEv2 maps ToS values to traffic classes, but the limited number of ToS values restricts fine-grained priority control. RoCEv2 将 ToS 值映射到流量类别，但有限的 ToS 值数量限制了细粒度的优先级控制。

### 参考注文

本节分析了攻击可行的根本原因：SR-IOV 仅在逻辑层面隔离 VF，所有片上缓存和处理资源仍按先到先得策略共享，而 RDMA 绕过操作系统的特性使得现有的系统级 QoS 机制无从介入。实验证明，即使对攻击者容器施加 QoS 带宽限制，受害者仍然遭受 73.4% 的性能下降，因为微架构层面的资源争用并未被解决。

---

## 段九 Section 5.3：HT-Verbs 防御框架

### 段落原文

To overcome the limitations of existing resource isolation mechanisms, we propose 【HT-Verbs】, a threshold-based RNIC resource management system. HT-Verbs operates on the RNIC to dynamically monitor and classify per-QP resource usage in real-time, enabling the system to detect malicious QPs. Upon detection, HT-Verbs can selectively 【throttle】 or block malicious QPs at the SmartNIC level.

**Resource Classification.** Based on the analysis of usage patterns, HT-Verbs categorizes RNIC resources into three levels:
- **Hot**: heavily utilized resources with high contention; access by lower-priority or malicious containers is restricted.
- **Warm**: moderate demand; standard isolation levels apply.
- **Cold**: underutilized resources; higher-priority containers get enhanced performance.

**Adaptive Threshold Adjustment.** The threshold criteria for classifying resources are continuously recalibrated by the 【Pattern Analyzer】 based on system-wide metrics such as overall RNIC utilization, traffic fluctuation, and per-QP statistics. These adaptive thresholds allow HT-Verbs to remain responsive to sudden demand spikes or evolving attack behavior.

**Implementation Strategy.** HT-Verbs can be implemented atop NVIDIA's BlueField-3 architecture using the 【DOCA SDK】, which provides low-level telemetry access and control APIs for RDMA-aware SmartNICs. The Verb Monitoring module leverages DOCA Telemetry and mlx5 hardware counters to collect per-VF resource metrics at fine-grained intervals (e.g., every **100 ms**).

### 生词解析

- **throttle** *v.* 节流、限速；动态降低某一进程/容器/流的资源访问速率或带宽，以防止其影响其他用户；
  1. HT-Verbs throttles malicious QPs by reducing their WQE pacing rate, restoring bandwidth for victim containers. HT-Verbs 通过降低恶意 QP 的 WQE 节奏速率来对其进行节流，从而为受害者容器恢复带宽。

- **Pattern Analyzer** *n.* 模式分析器；HT-Verbs 中负责分析每个容器 RDMA verb 使用趋势、检测异常行为的功能模块；
  1. The Pattern Analyzer identifies containers that consistently exceed normal verb usage distributions as potentially malicious. 模式分析器将持续超出正常 verb 使用分布的容器识别为潜在恶意容器。

- **DOCA SDK** (Data Center-on-a-Chip Architecture) *n.* 英伟达 DOCA 软件开发套件；为 BlueField DPU 提供统一编程接口、遥测和流量控制 API 的开发框架；
  1. DOCA SDK provides the DOCA Flow and DOCA Telemetry APIs needed to implement HT-Verbs on the BlueField-3 DPU. DOCA SDK 提供了在 BlueField-3 DPU 上实现 HT-Verbs 所需的 DOCA Flow 和 DOCA Telemetry API。

- **telemetry** *n.* 遥测；自动收集、传输和分析硬件或软件运行状态指标数据的技术；
  1. Real-time telemetry from RNIC hardware counters enables HT-Verbs to detect resource abuse within 100 ms. 来自 RNIC 硬件计数器的实时遥测使 HT-Verbs 能够在 100 毫秒内检测到资源滥用。

- **WQE pacing** *n.* WQE 节奏控制；限制 RNIC 每单位时间处理的工作队列项数量以防止流水线过载的机制；
  1. WQE pacing limits the rate at which an attacker's work queue entries are processed, preventing pipeline saturation. WQE 节奏控制限制了攻击者工作队列项的处理速率，防止流水线饱和。

### 参考注文

本节提出了 HT-Verbs 防御框架，其核心思路是将 RNIC 资源按使用强度分为热/温/冷三个等级，并通过 BlueField-3 上运行的遥测模块实时监控每个 QP 的行为，一旦检测到异常（如持续高速 verb 注入）即在 SmartNIC 层面实施节流，无需修改底层硬件。相较于现有的 QoS 机制，HT-Verbs 能够直接作用于微架构资源层面，而非仅限于网络流量层面。

---

## 段十 Section 7：Conclusion（结论）

### 段落原文

In this work, we experimentally analyzed performance isolation problems in RDMA-enabled container environments and proposed two types of resource exhaustion attacks that target RNIC resources: (i) state saturation attacks and (ii) pipeline saturation attacks. Our experiments reveal that state saturation attacks cause up to **93.9% bandwidth loss**, a **1,117× latency increase**, and a **115% rise in cache misses**, while pipeline saturation attacks induce severe link-level congestion and amplification, where small verb requests consume disproportionately large RNIC resources. To counter these threats, we propose 【HT-Verbs】, a threshold-driven framework that, using real-time RDMA verb telemetry, classifies RNIC resources into Hot, Warm, and Cold tiers, dynamically throttling abusive workloads without requiring hardware modifications. As future work, we plan to implement a prototype of HT-Verbs and empirically validate its practicality and effectiveness under real-world container workloads.

### 生词解析

- **threshold-driven** *adj.* 阈值驱动的；系统行为由预定义的度量指标阈值触发，而非依赖复杂的机器学习模型或人工规则；
  1. A threshold-driven approach makes HT-Verbs deployable with low overhead, adapting dynamically to workload changes. 阈值驱动的方法使 HT-Verbs 能够以低开销部署，并动态适应工作负载变化。

- **disproportionately** *adv.* 不成比例地；实际消耗的资源远超投入的资源或预期水平；
  1. Verb amplification allows attackers to disproportionately burden the RNIC by sending tiny but protocol-heavy requests. Verb 放大允许攻击者通过发送微小但协议开销繁重的请求，对 RNIC 施加不成比例的负担。

- **empirically validate** *v.* 实证验证；通过实际实验（而非仅理论分析）来证明系统或机制的有效性；
  1. Future work will empirically validate HT-Verbs on production container workloads including distributed deep learning and in-memory databases. 未来工作将在生产容器工作负载（包括分布式深度学习和内存数据库）上对 HT-Verbs 进行实证验证。

### 参考注文

结论总结了 Noisy Neighbor 论文的两大贡献：其一，在 NVIDIA BlueField-3 上系统验证了两类 RDMA 资源耗尽攻击，以精确量化数据证明了容器化云中 RDMA 性能隔离的脆弱性；其二，提出了 HT-Verbs 框架，通过在 SmartNIC 上实施实时遥测驱动的三级资源分类和自适应节流，在无需硬件修改的前提下提供微架构层面的防护，弥补了现有 QoS 机制只能作用于流量层的不足。
