---
title: "论文精读解析：Understanding RDMA Microarchitecture Resources for Performance Isolation"
date: 2026-03-17
description: "论文精读解析：Understanding RDMA Microarchitecture Resources for Performance Isolation"
tags: ["rdma", "paper", "security", "Collie"]
showToc: true
TocOpen: true
draft: false
---
# 论文精读解析：Understanding RDMA Microarchitecture Resources for Performance Isolation

> **来源**：20th USENIX Symposium on Networked Systems Design and Implementation (NSDI), April 2023
> **作者**：Xinhao Kong, Jingrong Chen (Duke University); Wei Bai (Microsoft); Yechen Xu (SJTU); Mahmoud Elhaddad, Shachar Raindel, Jitendra Padhye (Microsoft); Alvin R. Lebeck, Danyang Zhuo (Duke University)
> **解析格式**：标题解读 → 原文（关键词标注）→ 生词解析 → 参考注文

---

## 标题解读

**Understanding RDMA Microarchitecture Resources for Performance Isolation**
理解面向性能隔离的RDMA微架构资源

- **microarchitecture** n. 微架构（处理器或网卡内部的硬件实现细节，对外部不可见）；
  1. CPU microarchitecture decisions include cache size and pipeline depth. CPU微架构决策包括缓存大小和流水线深度。

- **performance isolation** n. 性能隔离（确保一个租户的行为不影响另一个租户性能的机制）；
  1. Performance isolation is critical for multi-tenant cloud environments. 性能隔离对多租户云环境至关重要。

---

## 段一：Abstract（摘要）

**原文（关键词已标注）：**

Recent years have witnessed the wide adoption of RDMA in the cloud to accelerate 【first-party workloads】 and achieve cost savings by freeing up CPU cycles. Now cloud providers are working towards supporting RDMA in 【general-purpose guest VMs】 to benefit 【third-party workloads】. To this end, cloud providers must provide strong 【performance isolation】 so that the RDMA workloads of one 【tenant】 do not adversely impact the RDMA performance of another tenant. Despite many efforts on network performance isolation in the public cloud, we find that RDMA brings unique challenges due to its complex 【NIC microarchitecture resources】 (e.g., the NIC cache).

In this paper, we aim to systematically understand the impact of RNIC microarchitecture resources on performance isolation. We present a 【model】 that represents how RDMA operations use RNIC resources. Using this model, we develop a 【test suite】 to evaluate RDMA performance isolation solutions. Our test suite can break all existing solutions in various scenarios. Our results are acknowledged and reproduced by one of the largest RDMA NIC vendors. Finally, based on the test results, we summarize new insights on designing future RDMA performance isolation solutions.

**生词解析：**

- **first-party workload** n. 第一方负载（云服务商自身运行的应用，如Azure内部存储和ML服务）；
  1. First-party workloads benefit directly from RDMA by reducing internal latency. 第一方负载通过RDMA降低内部延迟直接获益。

- **general-purpose guest VM** n. 通用型客户虚拟机（面向外部用户开放的虚拟机实例）；
  1. Cloud providers want to offer RDMA to general-purpose guest VMs for customers. 云服务商希望向普通客户虚拟机开放RDMA能力。

- **third-party workload** n. 第三方负载（外部租户运行在云平台上的应用）；
  1. Third-party workloads include customer ML training and database applications. 第三方负载包括客户的机器学习训练和数据库应用。

- **tenant** n. 租户（在共享云基础设施上运行负载的用户或组织）；
  1. Multiple tenants share the same RNIC in a multi-tenant cloud environment. 在多租户云环境中，多个租户共享同一块RNIC。

- **NIC microarchitecture resource** n. 网卡微架构资源（RNIC内部的缓存、处理单元等硬件资源）；
  1. NIC microarchitecture resources are often overlooked in performance isolation design. 网卡微架构资源在性能隔离设计中常被忽视。

- **test suite** n. 测试套件（系统化地评估目标系统的一组测试用例集合）；
  1. The Husky test suite evaluates all existing performance isolation solutions. Husky测试套件对所有现有性能隔离方案进行评估。

**参考注文：**

本文关注RDMA在多租户云环境中的性能隔离问题。当云服务商将RDMA开放给普通租户时，必须保证一个租户的行为不干扰其他租户。然而，RNIC内部存在缓存、处理单元等复杂微架构资源，这些资源在现有隔离方案中均被忽视。作者系统建模了RDMA操作与RNIC微架构资源的关系，构建了首个评估测试套件Husky，并证明目前所有商用隔离方案均无法通过测试。

---

## 段二：Section 1 — Introduction（引言）

**原文（关键词已标注）：**

It is well known that having different tenants' workloads share computing resources can lead to unpredictable application 【performance interference】 and 【privacy leakage】. This drives plenty of studies focusing on performance isolation in the cloud, especially for performance-critical applications that have stringent 【service-level objectives】. The state of the art in practice has also significantly advanced: CPU vendors even implement hardware mechanisms to control and isolate access to CPU caches. 【Side channels】 through shared resources are being patched over time.

RDMA 【offloads】 the network stack from OS kernel to NIC hardware to provide high throughput and ultra-low processing latency with near-zero CPU overhead. RDMA has been deployed in datacenters at scale to improve performance and free up CPU cores for first-party workloads like storage and ML. Now cloud providers are working towards supporting RDMA in general-purpose guest VMs to benefit third-party workloads. To this end, cloud providers must provide strong performance isolation for tenants sharing the same RNIC.

**生词解析：**

- **performance interference** n. 性能干扰（一个应用的行为导致同主机其他应用性能下降）；
  1. Cache contention between tenants can cause severe performance interference. 租户间的缓存争用可导致严重的性能干扰。

- **privacy leakage** n. 隐私泄漏（通过共享资源推断其他用户的敏感信息）；
  1. Shared RNIC resources can create side channels that enable privacy leakage. 共享RNIC资源可形成侧信道，导致隐私泄漏。

- **service-level objective (SLO)** n. 服务级别目标（定义服务性能保证的指标，如延迟上限、可用性）；
  1. Latency-sensitive applications have strict SLOs that must not be violated. 延迟敏感应用具有不得违反的严格SLO。

- **side channel** n. 侧信道（通过分析非预期信息（如响应时间）推断秘密信息的方式）；
  1. A side channel through RNIC cache timing can reveal access patterns of co-tenants. 通过RNIC缓存时序的侧信道可以揭示共租户的访问模式。

- **offload** v. 卸载（将计算任务从CPU/操作系统转移至专用硬件）；
  1. RDMA offloads packet processing to the NIC, reducing CPU overhead significantly. RDMA将数据包处理卸载到网卡，显著降低CPU开销。

**参考注文：**

引言阐述了多租户资源共享带来的性能干扰和隐私风险，并指出现有研究主要关注带宽和数据包处理资源，而忽略了RDMA带来的独特挑战——RNIC内部复杂的微架构资源。论文图1展示了即使启用了现有隔离机制，1Gbps的攻击流量也能将受害者的带宽从50Gbps骤降至2Gbps，直观说明了问题的严重性。

---

## 段三：Section 2 — Background and Motivation（背景与动机）

**原文（关键词已标注）：**

**2.2 RDMA Overview**

RDMA allows the NIC to directly transfer data between the wire and the application memory. The networking protocol is implemented in the NIC. RDMA classifies standard RDMA programming interface, a.k.a., 【verbs】, into two categories: control and data. An application first needs to call several 【control verbs】 to allocate necessary objects, such as queue pair (QP) and completion queue (CQ), to set up a reliable connection (RC), an unreliable connection (UC), or an unreliable datagram (UD) transmission endpoint. Then the application needs to register a 【memory region (MR)】. This registration essentially 【pins】 the memory in the host DRAM and obtains the mapping from virtual addresses to physical addresses, which enables the RNIC to directly read from or write to this memory region.

**2.3 Why RDMA Performance Isolation is Hard?**

Figure 3 shows the hardware components of a commodity RNIC. In addition to the packet buffers (TX/RX Buffer), the RNIC also has multiple 【processing units (PU)】 and many types of internal caches. For example, in NVIDIA RNICs, the 【Interconnect Context Memory (ICM) cache】 stores QP contexts; the 【Memory Translation Table (MTT)】 and 【Memory Protection Table (MPT)】 store entries for memory address translation and protection information; and the 【Work Queue Entry (WQE) cache】 stores prefetched send WQEs and posted receive WQEs. A QP context cache miss can trigger an additional 【PCIe round trip】 for the RNIC to fetch the context from the host DRAM, thus degrading application performance.

**生词解析：**

- **verbs** n. pl. RDMA编程接口中的操作原语（如ibv_post_send、ibv_reg_mr等）；
  1. RDMA developers use verbs API to control the NIC and initiate data transfers. RDMA开发者使用verbs API控制网卡并发起数据传输。

- **control verb** n. 控制类原语（用于分配资源、建立连接的RDMA操作，如ibv_create_qp）；
  1. Control verbs like ibv_reg_mr are used to register memory before data transfer. 控制类原语如ibv_reg_mr用于在数据传输前注册内存。

- **pin** v. 固定（将内存页锁定在物理内存中，防止被操作系统换出）；
  1. Memory must be pinned before it can be directly accessed by the RNIC. 内存必须被固定后才能被RNIC直接访问。

- **processing unit (PU)** n. 处理单元（RNIC内部负责处理请求的专用计算单元）；
  1. RNIC processing units can be exhausted by complex atomic operations. RNIC处理单元可被复杂的原子操作耗尽。

- **ICM cache (Interconnect Context Memory cache)** n. 互连上下文内存缓存（存储QP上下文信息的RNIC内部缓存）；
  1. ICM cache misses increase latency by requiring additional PCIe round trips. ICM缓存缺失会因需要额外PCIe往返而增加延迟。

- **MTT/MPT (Memory Translation/Protection Table)** n. 内存转换表/内存保护表（RNIC中用于地址转换和访问权限检查的结构）；
  1. Frequent MR registrations can cause MTT cache thrashing and performance drops. 频繁的MR注册会导致MTT缓存颠簸和性能下降。

- **WQE cache (Work Queue Entry cache)** n. 工作队列条目缓存（预取发送WQE和存储接收WQE的RNIC缓存）；
  1. A deep receive queue can stress the WQE cache and trigger performance anomalies. 较深的接收队列会给WQE缓存施压，触发性能异常。

- **PCIe round trip** n. PCIe往返（RNIC向主机DRAM发出请求并等待响应的一次完整通信）；
  1. Each cache miss adds one PCIe round trip, significantly increasing operation latency. 每次缓存缺失都会增加一次PCIe往返，显著提升操作延迟。

**参考注文：**

本节详细介绍了RDMA的编程模型和RNIC内部硬件结构。RDMA verbs分为控制类（用于资源管理）和数据类（用于数据传输），两者对RNIC微架构资源的消耗模式截然不同。RNIC内部有ICM、MTT/MPT、WQE等多种缓存，以及多个处理单元，任何一种资源的缺失或耗尽都会触发PCIe往返请求，导致性能下降。这种复杂性使得从云服务商视角实现性能隔离极为困难。

---

## 段四：Section 3 — RNIC Microarchitecture Resources（微架构资源分析）

**原文（关键词已标注）：**

**Key finding #1: control verbs can cause excessive cache misses and a drastic performance reduction.**

We find that control verbs can easily trigger excessive cache misses, thus degrading bandwidth and request rate. We let a single-threaded attacker keep 【deregistering MRs】 using ibv_dereg_mr on the victim's sender side. The cache miss rate increases to 49.1%, and the bandwidth degrades to 48 Gbps (from 96.6 Gbps). It is worthwhile to note that the attacker does not need to issue any data verbs, so the attacker consumes no network bandwidth or request rate at all. We speculate that the MR deregistration may 【invalidate the entire MTT/MPT cache】 to avoid accessing outdated MRs.

**Key finding #2: performance interference between different data verbs depends on the complexity of verbs.**

Different data verbs have different complexities. Simple verbs, like SEND and READ, only copy data between machines. Complex verbs, such as 【fetch_and_add (FAA)】, atomically add a 64-bit value to the memory of a remote address. This operation leverages PCIe features (e.g., read-modify-write transactions), and may also acquire a lock on the target address. These complex verbs consume more PU resources. When the victim runs a READ workload alone, it can achieve 60 Mrps. If the attacker runs a CAS workload, the victim's request rate immediately drops to 3 Mrps.

**Key finding #3: error handling can stall RNIC processing units and hang all the applications.**

On NVIDIA ConnectX-5 and ConnectX-6 RNICs, we find handling 【RNR (Receive Not Ready) errors】 can completely stall the RNIC processing units. When the SEND application triggers RNR errors (e.g., the responder side does not post any receive requests), both the SEND application and the victim are stalled. The reason is that the RNIC of the RNR receiver is stalled, and the RNIC cannot even process the ACK packet.

**Key finding #4: PCIe bandwidth will only become the bottleneck when the request size is in a specific range.**

When the payload size is small, the commodity RNIC can mitigate the WQE overhead by embedding the small message in the WQE. When the request size is larger than 28B, the payload must be transferred in a separate DMA, and PCIe bandwidth may become the bottleneck only for payloads in this specific range.

**生词解析：**

- **deregister** v. 注销（撤销内存区域的RDMA注册，释放其RNIC映射）；
  1. Repeatedly deregistering MRs can invalidate RNIC cache entries and hurt performance. 反复注销MR会使RNIC缓存条目失效，损害性能。

- **invalidate** v. 使无效（清除或标记缓存条目为不可用，强制后续访问从内存重新加载）；
  1. Cache invalidation after MR deregistration causes all subsequent accesses to miss. MR注销后的缓存失效会导致所有后续访问缺失。

- **fetch_and_add (FAA)** n. 原子取加操作（原子地读取内存值并加上指定数值的RDMA原子操作）；
  1. Fetch-and-add requires a read-modify-write transaction on the remote RNIC. 原子取加操作需要在远端RNIC执行读-改-写事务。

- **CAS (Compare-and-Swap)** n. 比较并交换（原子地比较内存值并在匹配时进行替换的操作）；
  1. CAS operations are expensive for RNIC processing units compared to WRITE. CAS操作相比WRITE对RNIC处理单元消耗更大。

- **RNR error (Receive Not Ready)** n. 接收未就绪错误（接收方未提交足够的接收请求，导致发送方收到错误）；
  1. An RNR error occurs when the sender issues a SEND but the receiver has no pending RECV. 当发送方发出SEND而接收方无待处理RECV时发生RNR错误。

- **stall** v. 停顿，阻塞（处理流水线因资源冲突或错误处理而无法继续执行）；
  1. RNR error handling can stall the entire RNIC pipeline for all tenants. RNR错误处理会阻塞所有租户的整个RNIC流水线。

- **DMA (Direct Memory Access)** n. 直接内存访问（网卡不经CPU直接读写主机内存的机制）；
  1. Each DMA operation consumes PCIe bandwidth to transfer data between NIC and host. 每次DMA操作都消耗PCIe带宽在网卡和主机间传输数据。

- **MMIO (Memory-Mapped I/O)** n. 内存映射I/O（通过写入特定内存地址来向网卡发送命令的机制，如敲门铃）；
  1. Ringing the RNIC doorbell requires an MMIO write of 64 bytes. 敲RNIC门铃需要一次64字节的MMIO写操作。

**参考注文：**

本节是论文的核心贡献之一，揭示了四个关键发现。其中最出人意料的是#1和#3：控制类操作（尤其是MR注销）可以在完全不消耗网络带宽的情况下，通过清空MTT缓存使受害者带宽减半；RNR错误处理则更为危险，单个低带宽流量（<0.5Gbps）就可完全停止所有共存租户的RNIC处理流水线。这些发现说明，RDMA性能隔离需要考虑的资源维度远超现有方案所涵盖的范围。

---

## 段五：Section 3.6 — The Resource Consumption Model（资源消耗模型）

**原文（关键词已标注）：**

We summarize our findings in an RDMA operation model shown in Figure 7. This model describes which 【microarchitecture resource】 a verb operation consumes heavily. Note that a verb operation can also use other microarchitecture resources that are not captured by our experiments. This is because the usages of these resources are low and do not lead to resource contention. This model is 【qualitative】: we do not try to understand the exact resource usage since we have no visibility into 【proprietary】 RNIC hardware.

For example, we know a certain traffic pattern can trigger a certain type of cache misses, but we do not figure out the total size of the cache or how much of the cache an operation consumes. Even so, we show that this model is sufficiently powerful for us to create the first test suite for RNIC performance isolation, and it can capture a wide range of workloads that can break existing performance isolation solutions.

**生词解析：**

- **qualitative** adj. 定性的（描述性质或方向而非精确数值的分析方式）；
  1. The resource consumption model is qualitative due to the black-box nature of RNICs. 由于RNIC的黑盒特性，资源消耗模型是定性的。

- **proprietary** adj. 专有的，私有的（由特定厂商拥有且不对外公开的技术或信息）；
  1. RNIC internal designs are proprietary and not disclosed to researchers. RNIC内部设计是专有的，不向研究人员披露。

**参考注文：**

作者总结了RDMA操作与RNIC微架构资源之间关系的定性模型。由于RNIC硬件内部设计不公开，模型无法给出精确的资源使用量，但足以指导测试套件的设计。该模型明确了哪类操作（数据verbs、控制verbs、错误处理）会重点消耗哪类资源（NIC缓存、处理单元、PCIe带宽），为后续系统化测试提供了框架基础。

---

## 段六：Section 4 — The Husky Test Suite（Husky测试套件）

**原文（关键词已标注）：**

After we understand how different RDMA operations use these microarchitecture resources, we can design a test suite to evaluate performance isolation solutions. Our goal is the following: given an RNIC hardware and a performance isolation solution, we want to find a set of workloads combinations for an attacker and a victim that can 【break the performance isolation】.

We build 【Husky】 to systematically test and evaluate RNIC performance isolation solutions. Husky targets at four types of resources: NIC bandwidth, PCIe bandwidth, NIC PU, and NIC cache. In all, Husky includes 52 attacker synthetic workloads (6 for NIC BW, 4 for PCIe BW, 14 for NIC PU, and 28 for NIC cache) and 20 synthetic victim workloads.

One key question is how to define a 【violation of performance isolation】. Husky uses a user-specified predicate to compute the expected performance results when isolation is enabled. Husky compares the actual performance with the expected performance to identify violation. Most of existing performance isolation solutions only provide bandwidth guarantee. The bandwidth of this application should be at least (1−α)min(Ba,Bg) under any attacker workload, where α is a 【tolerance level】. We use α = 25%.

**生词解析：**

- **Husky** n. 本文提出的测试套件名称（源自哈士奇犬，意在追踪猎物般系统寻找隔离漏洞）；
  1. Husky is the first systematic test suite for evaluating RNIC performance isolation. Husky是首个系统评估RNIC性能隔离的测试套件。

- **violation** n. 违规，违背（性能指标低于保证值，说明隔离机制失效）；
  1. A bandwidth drop below the guaranteed threshold constitutes an isolation violation. 带宽低于保证阈值构成隔离违规。

- **tolerance level (α)** n. 容忍度（定义性能保证下限时允许的最大偏差比例）；
  1. Setting α to 25% allows a 25% deviation from the guaranteed bandwidth. 将α设为25%允许带宽偏离保证值25%。

- **synthetic workload** n. 合成负载（专为测试特定场景而人工构造的流量模式，而非真实应用流量）；
  1. Synthetic workloads are crafted to exhaust specific RNIC microarchitecture resources. 合成负载专为耗尽特定RNIC微架构资源而设计。

**参考注文：**

Husky通过枚举(攻击者负载, 受害者负载)组合，系统化地寻找能够破坏性能隔离的工作负载对。其创新之处在于：针对每类微架构资源（NIC带宽、PCIe带宽、处理单元、NIC缓存）分别设计了攻击性合成负载，总计52个攻击场景×20个受害者场景的穷举测试。性能隔离违规的判定标准基于带宽保证，允许25%的容忍偏差。

---

## 段七：Section 5 — Evaluation（评估）

**原文（关键词已标注）：**

We evaluate 3 different isolation solutions: (1) 【NVIDIA SR-IOV】, (2) 【NVIDIA HW TC (Hardware Traffic Class)】, and (3) 【Justitia】, a software-based performance isolation solution.

SR-IOV and HW TC only isolate the 【architectural resources】 (e.g., link bandwidth) and do not restrict the cache usage of a single tenant. Husky therefore is able to use an attacker workload that exhausts RNIC cache, such as MTT/MPT cache. Other applications would suffer from severe cache miss and hence the performance drop.

Justitia is not designed for the public cloud and requires the tenant to cooperate (e.g., using modified RDMA libraries). Husky can certainly break its isolation by bypassing the modified libraries. However, its isolation is violated when the attacker keeps posting requests that trigger 【error handling】 on the RNIC. The reason is that these errors are detected and handled by RNIC, which is out of Justitia's control. In addition, Justitia does not take cache and PCIe into consideration.

For real applications, the 【PU attack】 (which triggers RNR errors) is the most powerful — it can directly stall the allreduce application by exhausting the RNIC PUs. The 【Cache attack】 causes allreduce performance to drop more than half (71.3% for Justitia + HW TC).

**生词解析：**

- **SR-IOV (Single Root I/O Virtualization)** n. 单根I/O虚拟化（允许一块物理网卡呈现多个虚拟功能给不同虚拟机的硬件虚拟化技术）；
  1. SR-IOV creates separate virtual functions for each tenant's traffic. SR-IOV为每个租户的流量创建独立的虚拟功能。

- **HW TC (Hardware Traffic Class)** n. 硬件流量类别（RNIC上用于分离不同租户流量的硬件队列机制）；
  1. HW TC separates tenant traffic into different hardware queues to prevent bandwidth contention. HW TC将租户流量分配到不同硬件队列以防止带宽争用。

- **Justitia** n. 一种软件性能隔离方案（在用户态RDMA库中实现速率限制和节奏控制）；
  1. Justitia uses sender admission control to enforce fair sharing of bandwidth and request rate. Justitia使用发送方准入控制来实现带宽和请求率的公平共享。

- **architectural resource** n. 架构级资源（对应用可见并在规范中明确定义的资源，如链路带宽）；
  1. Existing solutions isolate architectural resources but ignore microarchitecture resources. 现有方案隔离架构级资源，但忽视了微架构资源。

- **error handling** n. 错误处理（RNIC在遇到异常情况时执行的内部恢复流程）；
  1. Error handling by the RNIC is opaque to software-based isolation mechanisms. RNIC的错误处理对软件隔离机制是不透明的。

- **PU attack** n. 处理单元攻击（通过触发RNR错误耗尽RNIC处理单元的攻击方式）；
  1. The PU attack stalls all victim applications with less than 0.5 Gbps of traffic. PU攻击以不足0.5Gbps的流量就能阻塞所有受害者应用。

- **Cache attack** n. 缓存攻击（通过大量MR/QP/CQ操作耗尽RNIC缓存的攻击方式）；
  1. The Cache attack degrades allreduce performance by over 70% using only 7 Gbps. 缓存攻击仅用7Gbps流量就使allreduce性能下降超过70%。

**参考注文：**

评估结果非常清晰：所有现有隔离方案（SR-IOV、HW TC及组合方案、Justitia）均无法抵御针对微架构资源的攻击。最具破坏性的是PU攻击（触发RNR错误），以不足0.5Gbps的流量即可使整个系统停滞；缓存攻击以7Gbps即可使分布式ML应用性能下降超过70%。关键洞见是：针对微架构资源的攻击比简单的带宽攻击效率高出几个数量级，而现有方案仅保护带宽这一架构级资源。

---

## 段八：Section 6 — Guidelines（设计建议）

**原文（关键词已标注）：**

**Hardware support for isolation is needed.** Software approaches like Justitia have a common problem. They only monitor architecture-level metrics, e.g., latency, bandwidth, and request rate. They cannot detect contention in microarchitecture resources, e.g., caches, let alone manage and fair share those resources. We believe future performance isolation solutions will have to leverage hardware support, similar to how modern hypervisors can use 【Intel Resource Director Technology (RDT)】 to monitor and manage access to the last-level cache and memory.

**A layer of indirection is needed.** RDMA means kernel bypass for data verbs. This enables low latency and reduced CPU overheads. Where should performance isolation be enforced? We believe that future performance isolation solutions will require a layer of indirection either in NIC or in software. Having the enforcement point in the userland RDMA library (as Justitia) does not work, because it lacks security. Instead, a software indirection can have a 【microkernel-like design】, with a set of cores running the isolation logic in a separate protection domain.

**生词解析：**

- **Intel RDT (Resource Director Technology)** n. 英特尔资源导向技术（一套允许监控和管理CPU末级缓存及内存带宽访问的硬件机制）；
  1. Intel RDT can partition last-level cache capacity among different VMs. Intel RDT可以在不同虚拟机间划分末级缓存容量。

- **microkernel-like design** n. 类微内核设计（将关键功能移至最小特权核心，其他功能在隔离的保护域中运行）；
  1. A microkernel-like enforcement point can securely control all RDMA verbs. 类微内核的执行点可以安全地控制所有RDMA verbs。

- **layer of indirection** n. 间接层（在访问资源和实际资源之间插入的中间控制层）；
  1. An indirection layer enables interception and rate-limiting of RDMA verbs. 间接层允许拦截和速率限制RDMA verbs。

- **enforcement point** n. 执行点（施加访问控制或资源限制的位置）；
  1. The enforcement point should be in a trusted domain that tenants cannot bypass. 执行点应位于租户无法绕过的可信域中。

**参考注文：**

作者提出了三条设计建议：首先需要硬件支持来监控和隔离微架构资源；其次需要间接层（在NIC硬件或可信软件中）来统一控制数据verbs和控制verbs，因为纯用户态方案缺乏安全性；第三，编程人员和编译器需要感知微架构资源限制，以开发高效的RDMA应用。作者坦言构建可用的RNIC性能隔离方案将是一场持久战。

---

## 附录：核心术语速查表

| 英文术语 | 中文释义 |
|---------|---------|
| microarchitecture resource | 微架构资源 |
| performance isolation | 性能隔离 |
| tenant | 租户 |
| control verb | 控制类操作原语 |
| data verb | 数据类操作原语 |
| verbs | RDMA编程接口操作集 |
| ICM cache | 互连上下文内存缓存 |
| MTT/MPT | 内存转换表/内存保护表 |
| WQE cache | 工作队列条目缓存 |
| processing unit (PU) | 处理单元 |
| PCIe round trip | PCIe往返 |
| MR deregistration | 内存区域注销 |
| RNR error | 接收未就绪错误 |
| SR-IOV | 单根I/O虚拟化 |
| HW TC | 硬件流量类别 |
| Justitia | 用户态性能隔离方案 |
| Husky | RNIC性能隔离测试套件 |
| cache invalidation | 缓存失效 |
| synthetic workload | 合成测试负载 |
| Intel RDT | 英特尔资源导向技术 |
| side channel | 侧信道 |
| service-level objective | 服务级别目标 |
| allreduce | 全规约（集合通信操作） |
| key-value store | 键值存储 |
