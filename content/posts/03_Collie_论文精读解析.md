---
title: "论文精读解析：Collie: Finding Performance Anomalies in RDMA Subsystems"
date: 2026-03-17
description: "论文精读解析：Collie: Finding Performance Anomalies in RDMA Subsystems"
tags: ["rdma", "paper", "security", "Collie"]
showToc: true
TocOpen: true
draft: false
---
# 论文精读解析：Collie: Finding Performance Anomalies in RDMA Subsystems

> **来源**：arXiv:2304.11467v1, April 2023（发表于 USENIX NSDI 2022）
> **作者**：Xinhao Kong (Duke University), Yibo Zhu, Huaping Zhou, Zhuo Jiang, Jianxi Ye, Chuanxiong Guo (ByteDance Inc.), Danyang Zhuo (Duke University)
> **解析格式**：标题解读 → 原文（关键词标注）→ 生词解析 → 参考注文

---

## 标题解读

**Collie: Finding Performance Anomalies in RDMA Subsystems**
Collie：发现RDMA子系统中的性能异常

- **anomaly** n. 异常（偏离预期行为的非正常现象）；
  1. Performance anomalies in RDMA can trigger PFC pause frame storms. RDMA中的性能异常可能触发PFC暂停帧风暴。

- **subsystem** n. 子系统（由多个相互协作的组件构成的复合系统，此处指RNIC及其服务器环境）；
  1. The RDMA subsystem includes the RNIC, PCIe controller, and host memory. RDMA子系统包括RNIC、PCIe控制器和主机内存。

---

## 段一：Abstract（摘要）

**原文（关键词已标注）：**

High-speed RDMA networks are getting rapidly adopted in the industry for their low latency and reduced CPU overheads. To verify that RDMA can be used in production, system administrators need to understand the set of application workloads that can potentially trigger 【abnormal performance behaviors】 (e.g., unexpected low throughput, 【PFC pause frame storm】). We design and implement Collie, a tool for users to systematically uncover performance anomalies in RDMA subsystems without the need to access hardware internal designs.

Instead of individually testing each hardware device (e.g., NIC, memory, PCIe), Collie is 【holistic】, constructing a comprehensive search space for application workloads. Collie then uses 【simulated annealing】 to drive RDMA-related performance and diagnostic counters to extreme value regions to find workloads that can trigger performance anomalies. We evaluate Collie on combinations of various RDMA NIC, CPU, and other hardware components. Collie found 15 new performance anomalies. All of them are acknowledged by the hardware vendors. 7 of them are already fixed after we reported them.

**生词解析：**

- **PFC pause frame storm** n. PFC暂停帧风暴（一块故障网卡持续发送暂停帧，导致整个网络流量被阻塞的现象）；
  1. A PFC storm from a single NIC can bring down the entire datacenter network. 单块网卡引发的PFC风暴可能导致整个数据中心网络瘫痪。

- **holistic** adj. 整体性的（将系统作为一个整体进行测试，而非单独测试各个组件）；
  1. A holistic testing approach covers interactions between different hardware components. 整体测试方法覆盖了不同硬件组件之间的交互。

- **simulated annealing** n. 模拟退火（一种概率搜索算法，通过模拟金属退火过程寻找全局最优解）；
  1. Simulated annealing helps Collie escape local optima and find more diverse anomalies. 模拟退火帮助Collie逃脱局部最优，找到更多样化的异常。

**参考注文：**

本文提出Collie工具，用于系统化地发现RDMA子系统的性能异常。其核心创新在于：从应用开发者视角（而非硬件内部视角）构建搜索空间，利用RNIC暴露的性能计数器和诊断计数器作为引导信号，通过模拟退火算法高效搜索能触发异常的工作负载。在8种不同RDMA子系统上，Collie共发现18个性能异常（3个已知+15个新发现），全部得到厂商确认。

---

## 段二：Section 1 — Introduction（引言）

**原文（关键词已标注）：**

To deploy RDMA in production, we need to make sure that the RDMA network performance can meet our expectations, free of performance anomalies like low throughput and 【pause frame storm】. Some abnormal behaviors, like pause frame storms, can cause 【catastrophic consequences】 including 【deadlocking】 the entire data center network.

The authors describe encountering anomalies in RoCEv2 production environments that no individual vendor test caught:

• A particular application workload's performance of the same RDMA NIC varies substantially on servers with only a slight difference in their 【PCIe specifications】.
• A specific application workload only triggers pause frame storms with certain 【NUMA settings】 on a particular RNIC combined with particular server hardware.
• A particular application workload triggers pause frame storms with only a single connection on a particular RNIC from a particular vendor.

The fundamental reason why current approaches fail to uncover such anomalies is that they only test existing workloads and inherently are not able to capture anomalies triggered by 【unknown workloads】.

**生词解析：**

- **catastrophic consequence** n. 灾难性后果（导致系统完全崩溃或大规模服务中断的后果）；
  1. PFC deadlocks can have catastrophic consequences for the entire datacenter fabric. PFC死锁可能对整个数据中心网络结构产生灾难性后果。

- **deadlock** n. 死锁（多个设备相互等待对方释放资源，导致所有方均无法继续进行的状态）；
  1. Circular PFC dependencies between switches can form a network-wide deadlock. 交换机间的循环PFC依赖可形成全网范围的死锁。

- **PCIe specification** n. PCIe规格（PCIe接口的版本和通道数配置，如PCIe Gen3 x16）；
  1. Different PCIe slot configurations can cause the same RNIC to behave differently. 不同的PCIe插槽配置会导致同一块RNIC表现不同。

- **NUMA setting** n. NUMA（非一致内存访问）配置（决定内存访问需穿越哪个CPU插槽的系统设置）；
  1. Cross-NUMA traffic has higher latency and can trigger RNIC RX buffer accumulation. 跨NUMA流量延迟更高，可能触发RNIC接收缓冲区积压。

- **unknown workload** n. 未知工作负载（在测试阶段未曾出现但在生产中实际运行的应用流量模式）；
  1. Current tests cannot find anomalies triggered by unknown workloads not yet written. 现有测试无法发现由尚未编写的未知工作负载触发的异常。

**参考注文：**

引言通过真实生产案例说明了RDMA子系统测试的挑战：一个运行正常的机器学习框架，在开发者轻微修改代码（改变WQE批大小）后，突然触发了大量PFC暂停帧，导致性能严重下降——而该框架在部署前已通过了所有常规测试。根本原因是测试只覆盖了已知工作负载，而真实应用会不断演变，产生测试未曾覆盖的新流量模式。

---

## 段三：Section 2 — Background（背景）

**原文（关键词已标注）：**

An application process can directly communicate through an RNIC with a remote process without involving either side's CPUs. RDMA requires a 【lossless network】 to achieve high performance. The default technology to deploy RDMA for Ethernet-based data centers is RoCEv2. It relies on 【Priority-based Flow Control (PFC)】 mechanism to guarantee a lossless network: once an ingress queue length exceeds a threshold, the switch/NIC sends out a 【PFC pause frame】 to the upstream egress queue, asking the egress queue to pause for a duration to avoid ingress queue overflow.

Worse still, an anomalous RDMA subsystem can send out a large amount of PFC pause frames, which pauses the priority queue of the corresponding switch port and may threaten the entire data center network, such as causing 【head-of-line blocking】 and 【PFC deadlocks】.

An RNIC has at least 6 components: (1) a 【TX engine】 that receives doorbells, fetches and processes requests, and initiates transmission; (2) an 【MMU】 that translates the virtual address to physical address for RDMA memory regions; (3) an 【SRAM-based NIC cache】 that caches per-connection metadata and memory translation table; (4) a 【RX engine】 that processes incoming data and generates completion to notify server; (5)(6) buffers that hold packets to transmit and received packets.

**生词解析：**

- **lossless network** n. 无损网络（通过流量控制确保数据包不因缓冲区溢出而丢失的网络）；
  1. RDMA requires a lossless network because its transport protocol cannot handle packet drops well. RDMA要求无损网络，因为其传输协议不能很好地处理丢包。

- **Priority-based Flow Control (PFC)** n. 基于优先级的流量控制（IEEE 802.1Qbb标准，通过发送暂停帧逐跳防止缓冲区溢出）；
  1. PFC prevents packet drops but can cause head-of-line blocking. PFC防止丢包但可能导致队头阻塞。

- **PFC pause frame** n. PFC暂停帧（通知上游停止发送特定优先级流量的以太网控制帧）；
  1. A PFC pause frame can propagate back through multiple switches. 一个PFC暂停帧可向后传播经过多个交换机。

- **head-of-line blocking** n. 队头阻塞（队列中排在前面的数据包阻止后续数据包处理的现象）；
  1. Head-of-line blocking caused by PFC can delay unrelated traffic flows. PFC引起的队头阻塞会延迟不相关的流量流。

- **PFC deadlock** n. PFC死锁（多个交换机相互发送暂停帧形成循环依赖，导致所有流量停止的状态）；
  1. PFC deadlocks can persist even after all servers are rebooted. PFC死锁即使在所有服务器重启后也可能持续存在。

- **TX/RX engine** n. 发送/接收引擎（RNIC内部负责处理出站和入站数据包的专用处理单元）；
  1. The TX engine fetches WQEs and segments data into packets for transmission. 发送引擎获取WQE并将数据分段成数据包进行传输。

- **MMU (Memory Management Unit)** n. 内存管理单元（将虚拟地址转换为物理地址的硬件组件）；
  1. The RNIC's MMU translates virtual addresses of memory regions for DMA access. RNIC的MMU为DMA访问转换内存区域的虚拟地址。

- **SRAM** n. 静态随机存取存储器（速度极快但成本高昂的片上存储，用作RNIC缓存）；
  1. SRAM-based NIC caches are small and can be easily exhausted by many connections. 基于SRAM的网卡缓存容量小，容易被大量连接耗尽。

**参考注文：**

本节介绍了RDMA子系统的技术背景，重点是PFC机制的工作原理及其潜在的危险性。PFC是保证无损传输的关键机制，但其暂停帧的传播性使得局部故障（如单块RNIC异常）可以扩散成全网死锁。作者还描述了RNIC内部的六个核心组件及其相互作用，说明性能异常往往源于多个组件的联合瓶颈，而非单一组件的缺陷。

---

## 段四：Section 3 — Overview（系统概述）

**原文（关键词已标注）：**

We focus on two types of performance anomalies that are of great importance in production environment and can be precisely defined: (1) no 【PFC pause frames】 if the network is not congested; (2) throughput should be bottlenecked either by 【bits/second or packets/second】 as in RNIC specification.

Collie consists of three core components: (1) a 【workload engine】 that conducts experiments on RDMA subsystems by setting up RDMA traffic; (2) an 【anomaly monitor】 that detects performance anomaly and 【MFS (Minimal Feature Set)】 to reproduce the observed anomaly; and (3) a 【workload generator】 that decides the next workload pattern to experiment based on the counters collected in the RDMA subsystem and the current search space.

**生词解析：**

- **anomaly monitor** n. 异常监视器（实时检测性能指标并判断是否触发异常的组件）；
  1. The anomaly monitor checks throughput against the RNIC specification every second. 异常监视器每秒将吞吐量与RNIC规格进行对比检查。

- **MFS (Minimal Feature Set)** n. 最小特征集（触发某个异常所必需的最少条件组合）；
  1. The MFS algorithm identifies which workload features are necessary to trigger an anomaly. MFS算法识别触发异常所必需的工作负载特征。

- **workload generator** n. 工作负载生成器（基于搜索算法决定下一个测试工作负载配置的组件）；
  1. The workload generator uses simulated annealing to navigate the search space. 工作负载生成器使用模拟退火算法在搜索空间中导航。

- **workload engine** n. 工作负载引擎（实际生成和发送RDMA流量的执行组件）；
  1. The workload engine translates search parameters into RDMA traffic configurations. 工作负载引擎将搜索参数转换为RDMA流量配置。

**参考注文：**

Collie的异常定义简洁且可量化：在无网络拥塞时产生PFC暂停帧，或吞吐量低于规格上限的80%，均视为异常。系统三组件分工明确：引擎执行测试、监视器检测并提取最小触发条件、生成器根据反馈决定下一步测试方向。MFS是关键创新——它不仅加速了搜索（避免重复测试同一异常区域），还帮助开发者理解和规避异常触发条件。

---

## 段五：Section 4 — Search Space and Workload Engine（搜索空间）

**原文（关键词已标注）：**

We examine the RDMA programming model and extract four search dimensions that jointly describe the application workloads of the entire subsystem.

**Dimension 1. Host Topology.** Host topology describes how traffic flows to/from an RNIC to/from other server hardware components. For example, traffic can be from 【NUMA-affinitive DRAM】 or from a GPU that needs to traverse both PCIe and SMP interconnect between NUMA nodes. The latter will have a longer data path and therefore higher average 【DMA latency】.

**Dimension 2. Memory Allocation Settings.** The number of MRs affects RDMA subsystem performance because RNIC has an MMU that translates virtual addresses of memory regions to DMA-capable physical addresses. If too many MRs are registered, the RNIC encounters 【cache miss】 and needs to access memory address translation tables on server DRAM via extra PCIe operations.

**Dimension 3. Transport Setting.** We use the following factors: (1) QP type (RC, UC, UD), (2) the number of QPs, (3) the opcode type (SEND/RECV, WRITE, READ), and (4) the usage of 【SG (scatter-gather) list】 and WQE. RNICs have to consume extra PCIe bandwidth to fetch WQE from the host DRAM.

**Dimension 4. Message Pattern.** We build a request vector with n elements, where each element describes the request attribute (e.g., size of the message to send). We assume that the 1st request affects the 2nd, the 3rd, ..., the nth requests but won't affect the request after the nth. We discretize request size into multiple discrete value regions based on 【MTU (Maximum Transmission Unit)】 and the burst size of the RNIC.

**生词解析：**

- **NUMA-affinitive DRAM** n. NUMA亲和内存（与RNIC位于同一CPU插槽的内存，访问延迟低于跨NUMA访问）；
  1. Allocating buffers from NUMA-affinitive DRAM minimizes DMA latency for the RNIC. 从NUMA亲和内存分配缓冲区可以最小化RNIC的DMA延迟。

- **DMA latency** n. DMA延迟（RNIC通过PCIe读写主机内存所需的时间）；
  1. High DMA latency can slow down RNIC's receiving pipeline and trigger PFC. 高DMA延迟会减慢RNIC的接收流水线并触发PFC。

- **scatter-gather (SG) list** n. 分散-聚集列表（允许一个WQE引用多个不连续内存区域的机制）；
  1. A long SG list in a single WQE increases the RNIC's memory access overhead. 单个WQE中的长SG列表增加了RNIC的内存访问开销。

- **MTU (Maximum Transmission Unit)** n. 最大传输单元（网络层单次能传输的最大数据包大小）；
  1. Changing the MTU can affect RNIC packet processing overhead significantly. 改变MTU会显著影响RNIC的数据包处理开销。

- **opcode** n. 操作码（指定RDMA操作类型的编码，如WRITE、READ、SEND等）；
  1. Different opcodes trigger different code paths inside the RNIC. 不同操作码触发RNIC内部不同的代码路径。

**参考注文：**

Collie的搜索空间从应用层（verbs API）自底向上构建，覆盖了影响RDMA子系统性能的四个核心维度：主机拓扑（流量路径）、内存分配（MR数量和大小）、传输设置（QP类型、连接数、操作类型）和消息模式（请求大小序列）。总搜索空间规模约10³⁶，无法穷举，但通过将这四个维度组合，能够描述任何应用工作负载，覆盖性远超现有测试工具。

---

## 段六：Section 5 — Search for Performance Anomalies（搜索算法）

**原文（关键词已标注）：**

**5.1 Workloads Generation（工作负载生成）**

The high-level approach is to use an optimization algorithm to drive counters to extreme value regions by keeping mutating the test workloads. For 【performance counters】, we drive the counters to 【low-value regions】. For 【diagnostics counters】 (which map to unexpected events), we drive the counters to 【high-value regions】.

Our algorithm is based on 【simulated annealing (SA)】. SA is a probabilistic algorithm to find the global minimum of a given function. SA calls the function value 【energy】. To avoid getting stuck at a local minimum, SA maintains a 【temperature value】. At the beginning of the algorithm, the temperature is high and SA allows mutating input in the direction of increasing the energy. As temperature decreases during the search, SA is less likely to move the input in the direction of increasing the energy.

**5.2 Anomaly Monitor**

Our anomaly monitor detects performance anomalies and computes the MFS of them. If a workload's throughput (in terms of both metrics) is 20% lower than the upper bounds, it means that the performance is likely to be restricted by some other bottlenecks. After we detect an anomalous workload, we use a heuristic approach to find what features of this workload actually trigger the anomaly — the 【Minimal Feature Set (MFS)】. For each factor, we test whether removing it from the triggering conditions still causes the anomaly.

**生词解析：**

- **performance counter** n. 性能计数器（硬件或驱动暴露的指标，如吞吐量、时延，用于监控系统状态）；
  1. Performance counters like bits/second are available on all commodity RNICs. 比特/秒等性能计数器在所有商用RNIC上均可获取。

- **diagnostic counter** n. 诊断计数器（反映特定硬件内部事件（如缓存缺失）的计数器，通常需要厂商协助）；
  1. Diagnostic counters reveal internal RNIC bottlenecks not visible in end-to-end metrics. 诊断计数器揭示了端到端指标中不可见的RNIC内部瓶颈。

- **simulated annealing (SA)** n. 模拟退火（模仿金属退火过程的概率优化算法，能避免陷入局部最优）；
  1. Simulated annealing drives diagnostic counters to extreme regions to uncover anomalies. 模拟退火将诊断计数器驱动至极端区域以发现异常。

- **energy** n. （SA算法中的）能量（被优化的目标函数值，SA通过修改输入来降低能量）；
  1. In Collie's SA algorithm, energy represents counter deviation from the extreme region. 在Collie的SA算法中，能量表示计数器偏离极端区域的程度。

- **temperature** n. （SA中的）温度（控制SA算法接受更差解的概率，高温时允许探索，低温时趋向收敛）；
  1. High temperature in SA allows accepting worse solutions to escape local optima. SA中的高温允许接受更差的解以逃脱局部最优。

**参考注文：**

Collie的搜索算法基于模拟退火，核心思路是将诊断计数器（如WQE缓存缺失次数）驱动至极端值——因为计数器异常往往先于性能下降出现。MFS算法在发现新异常时逐一剔除条件进行验证，找出最少的触发条件集合。这既加速了后续搜索（已知异常区域不再重复测试），也让开发者能针对性地修改应用来规避异常。

---

## 段七：Section 7 — Evaluation and Experience（评估与经验）

**原文（关键词已标注）：**

Collie found 15 new anomalies on 8 RDMA subsystems. All are acknowledged by vendors; 7 are already fixed. Two tricky examples:

**Anomaly #4**: Bidirectional RC READ with large WQE batch size, long SG list, and a few connections causes PFC pause frames. This anomaly cannot be found by existing approaches such as using Perftest, because Perftest does not support flexible WQE and SG list batching strategies.

**Anomaly #10**: Bidirectional RC WRITE with large WQE batch size, particular message pattern, and a few connections causes PFC pause frames. This is not captured by existing approaches but was successfully reproduced by slightly modifying the production RDMA RPC library — when the 【timeout value】 is set high (for throughput-sensitive applications), the WQE batch size increases and meets all conditions of the anomaly.

**Key implications:**

- **Holistic performance testing over entire RDMA subsystems is important.** Root causes can be bottlenecks from RNIC internals, PCIe controllers, and host topologies (cross socket communication).
- **Opaque resource limitation of the RDMA subsystems.** Anomalies found by Collie suggest that invisible resources (e.g., WQE cache, QP connection context) can affect performance even when bandwidth and QP resources are well isolated.
- **Does Ethernet-based RDMA need end-to-end flow control?** Currently there is no end-to-end flow control mechanism (e.g., the sliding window for TCP) for production RoCEv2 deployment. Collie shows that host limitations can slow down RNIC's outbound rate, and without end-to-end flow control, the only defense is PFC — which can cause catastrophic consequences.

**生词解析：**

- **Perftest** n. RDMA领域最常用的性能测试工具（提供标准的带宽和延迟基准测试）；
  1. Perftest is prevalent but cannot reproduce anomalies that require specific WQE batching. Perftest很普遍，但无法重现需要特定WQE批处理的异常。

- **timeout value** n. 超时值（应用等待操作完成的最长时间，影响批处理策略）；
  1. A high timeout value leads to larger WQE batches, which can trigger RNIC anomalies. 高超时值导致更大的WQE批次，可能触发RNIC异常。

- **RPC library** n. RPC库（基于RDMA实现远程过程调用的软件库）；
  1. The RDMA RPC library's WQE batching strategy must consider hardware anomalies. RDMA RPC库的WQE批处理策略必须考虑硬件异常。

- **end-to-end flow control** n. 端到端流量控制（在通信双方之间协调发送速率的机制，如TCP的滑动窗口）；
  1. Without end-to-end flow control, PFC is the only mechanism to prevent packet drops in RoCEv2. 没有端到端流量控制，PFC是RoCEv2中防止丢包的唯一机制。

- **cross-socket communication** n. 跨插槽通信（数据需要穿越两个CPU插槽之间互连的内存访问）；
  1. Cross-socket NUMA traffic can cause PCIe backpressure and trigger PFC storms. 跨插槽NUMA流量可导致PCIe背压并触发PFC风暴。

**参考注文：**

评估表明Collie找到的15个新异常中，大多数无法被现有工具（Perftest、真实应用）重现——因为它们需要特定的WQE批大小、SG列表长度、消息模式和NUMA配置的组合。最深刻的发现是：许多异常的根本原因涉及多个组件（RNIC内部、PCIe控制器、跨CPU插槽互连）的联合瓶颈，而且RoCEv2缺乏端到端流量控制这一根本缺陷使得RNIC本身成为暂停帧的来源，而非仅仅是网络交换机。

---

## 附录：核心术语速查表

| 英文术语 | 中文释义 |
|---------|---------|
| performance anomaly | 性能异常 |
| RDMA subsystem | RDMA子系统 |
| PFC pause frame storm | PFC暂停帧风暴 |
| PFC deadlock | PFC死锁 |
| head-of-line blocking | 队头阻塞 |
| simulated annealing | 模拟退火 |
| Minimal Feature Set (MFS) | 最小特征集 |
| performance counter | 性能计数器 |
| diagnostic counter | 诊断计数器 |
| holistic testing | 整体性测试 |
| NUMA | 非一致内存访问 |
| host topology | 主机拓扑 |
| scatter-gather list | 分散-聚集列表 |
| WQE batch size | WQE批处理大小 |
| lossless network | 无损网络 |
| end-to-end flow control | 端到端流量控制 |
| TX/RX engine | 发送/接收引擎 |
| MMU | 内存管理单元 |
| SRAM cache | 静态随机存储缓存 |
| GPU-Direct RDMA | GPU直接RDMA（GPU与RNIC直接通信） |
