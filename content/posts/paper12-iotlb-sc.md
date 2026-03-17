---
title: "【论文精读】IOTLB-SC: An Accelerator-Independent Leakage Source in Modern Cloud Systems"
date: 2025-03-17
draft: false
tags: ["IOMMU", "IOTLB", "侧信道攻击", "FPGA", "云安全", "论文精读", "PCIe"]
categories: ["论文精读"]
description: "对 IOTLB-SC 论文的逐段深度解析，涵盖 IOMMU 侧信道攻击原理、FPGA 实验验证及防御对策。"
---

> **原文信息**
> - **标题**：IOTLB-SC: An Accelerator-Independent Leakage Source in Modern Cloud Systems
> - **作者**：Thore Tiemann, Zane Weissman, Thomas Eisenbarth, Berk Sunar
> - **机构**：University of Lübeck; Worcester Polytechnic Institute
> - **发表**：ACM ASIA CCS '23, Melbourne, Australia
> - **arXiv**：2202.11623v2 \[cs.CR\] 9 Mar 2023

---

## 标题解读

**IOTLB-SC: An Accelerator-Independent Leakage Source in Modern Cloud Systems**

标题中的重点词汇：

- **IOTLB** (I/O Translation Look-aside Buffer) *n.* I/O 地址转换旁查缓冲区；IOMMU 内部用于缓存最近地址转换结果、加速重复 DMA 访问的硬件缓存结构；
  1. An IOTLB miss significantly increases DMA latency, creating a measurable timing side-channel. IOTLB 缺失会显著增加 DMA 延迟，形成可测量的时序侧信道。

- **SC** (Side Channel) *n.* 侧信道；通过测量物理实现的间接特征（如时序、功耗）而非直接读取数据来获取信息的攻击方式；
  1. A timing side-channel attack exploits latency differences to infer secret information without direct memory access. 时序侧信道攻击利用延迟差异来推断私密信息，而无需直接内存访问。

- **accelerator-independent** *adj.* 与加速器类型无关的；指该攻击面不局限于特定硬件加速器，对任何共享 IOMMU 的外设均适用；
  1. The IOTLB side-channel is accelerator-independent because it targets the IOMMU shared by all PCIe devices. IOTLB 侧信道与加速器类型无关，因为它针对的是所有 PCIe 设备共享的 IOMMU。

- **leakage source** *n.* 泄漏源；系统中存在信息泄露风险的硬件或软件组件；
  1. The IOTLB is a previously overlooked leakage source that reveals peripheral memory access patterns to co-located attackers. IOTLB 是一个此前被忽视的泄漏源，它会将外设内存访问模式暴露给共处一机的攻击者。

---

## Abstract（摘要）

### 段落原文

Hardware peripherals such as 【GPUs】 and 【FPGAs】 are commonly available in server-grade computing to accelerate specific compute tasks, from database queries to machine learning. CSPs have integrated these accelerators into their infrastructure and let tenants combine and configure these components flexibly, based on their needs. Securing I/O interfaces is critical to ensure proper 【isolation】 between tenants in these highly complex, 【heterogeneous】, yet shared server systems, especially in the cloud, where some peripherals may be under control of a malicious tenant.

In this work, we investigate the interfaces that connect peripheral hardware components to each other and the rest of the system. We show that the 【I/O memory management units (IOMMUs)】 — intended to ensure proper isolation of peripherals — are the source of a new attack surface: the 【I/O translation look-aside buffer (IOTLB)】. We show that by using an FPGA accelerator card one can gain precise information over IOTLB activity. That information can be used for 【covert communication】 between peripherals without bothering CPU or to directly extract 【leakage】 from neighboring accelerated compute jobs such as GPU-accelerated databases. We present the first qualitative and quantitative analysis of this newly uncovered attack surface before fine-grained channels become widely viable with the introduction of 【CXL】 and 【PCIe 5.0】.

### 生词解析

- **GPU** (Graphics Processing Unit) *n.* 图形处理器；原用于图形渲染，现广泛用于并行计算加速（如深度学习、数据库查询）的专用处理器；
  1. GPU-accelerated databases use DMA to transfer query results between device memory and host memory. GPU 加速数据库使用 DMA 在设备内存与主机内存之间传输查询结果。

- **FPGA** (Field-Programmable Gate Array) *n.* 现场可编程门阵列；可在硬件层面被编程和重配置的集成电路，云环境中常用于定制化加速计算；
  1. Cloud tenants can rent FPGA instances to implement custom hardware functions with DMA access to main memory. 云租户可以租用 FPGA 实例来实现具有主内存 DMA 访问权限的定制硬件功能。

- **isolation** *n.* 隔离；在多租户系统中，确保一个租户的操作不影响其他租户数据和性能的安全机制；
  1. IOMMUs are designed to enforce memory isolation between different PCIe peripherals and their respective tenants. IOMMU 旨在强制实施不同 PCIe 外设及其各自租户之间的内存隔离。

- **heterogeneous** *adj.* 异构的；由多种不同类型的计算单元（CPU、GPU、FPGA 等）混合组成的系统架构；
  1. Heterogeneous cloud systems introduce new attack surfaces because multiple peripheral types share the same IOMMU. 异构云系统引入了新的攻击面，因为多种外设类型共享同一 IOMMU。

- **IOMMU** (Input-Output Memory Management Unit) *n.* 输入输出内存管理单元；负责对外设的内存访问进行地址转换和权限控制的硬件单元；
  1. The IOMMU translates I/O virtual addresses to physical addresses, providing isolation between co-located peripherals. IOMMU 将 I/O 虚拟地址转换为物理地址，在共处同机的外设之间提供隔离。

- **covert communication** *n.* 隐蔽通信；利用非预期的信道（如硬件侧信道）在不被检测的情况下传递信息；
  1. The IOTLB covert channel enables two co-located peripherals to exchange information without any network traffic. IOTLB 隐蔽信道使两个共处同机的外设能够在不产生任何网络流量的情况下交换信息。

- **leakage** *n.* 信息泄漏；通过侧信道等非直接方式无意中泄露出的敏感信息；
  1. The SQL query workload on the GPU leaves a measurable leakage pattern in the shared IOTLB. GPU 上的 SQL 查询工作负载在共享 IOTLB 中留下可测量的泄漏模式。

- **CXL** (Compute Express Link) *n.* 计算快速链路；基于 PCIe 5.0 物理层的新型互连协议，支持 CPU 与加速器之间的缓存一致性内存共享；
  1. CXL's fine-grained memory coherency protocol will make IOTLB-based side-channels significantly more powerful. CXL 的细粒度内存一致性协议将使基于 IOTLB 的侧信道攻击显著增强。

### 参考注文

本摘要揭示了一个此前未被研究的云端攻击面：IOMMU 内部的 IOTLB（I/O 地址转换缓冲区）。研究者通过 FPGA 加速卡，利用 DMA 访问延迟差异精确感知 IOTLB 活动，从而构建外设间的隐蔽信道，并提取相邻 GPU 加速数据库的操作信息。作者特别指出，随着 PCIe 5.0 和 CXL 的普及，此类攻击将变得更加危险。

---

## 段二 Section 1：Introduction（引言）

### 段落原文

Modern server-grade computing infrastructures are becoming more 【heterogeneous】: computational needs are spread over fast and flexible CPUs as well as powerful peripherals such as smart storage, GPUs, smart NICs and FPGAs. Major cloud service providers (CSPs) have started to shift tasks such as networking, memory management and VM management into more specialized hardware peripherals, freeing up precious CPU time that is rented to more tenants who share the same hardware. These 【multi-tenant】, peripheral-heavy cloud systems rely on increasingly interlinked memory systems to provide high throughput for shared, scalable and parallelized cloud infrastructure. Technologies like 【VT-d】, 【DDIO】, and 【CXL】 allow peripherals to not only directly read and write to the memory of a virtual machine, but to also use a CPU's shared cache to speed up repeated reads and writes.

### 生词解析

- **multi-tenant** *adj.* 多租户的；多个独立用户（租户）共享同一套物理硬件基础设施的云计算部署模式；
  1. In a multi-tenant cloud, a malicious tenant can potentially exploit shared hardware to spy on co-located victims. 在多租户云中，恶意租户可能利用共享硬件对共处同机的受害者进行监控。

- **VT-d** (Virtualization Technology for Directed I/O) *n.* 英特尔面向定向 I/O 的虚拟化技术；英特尔实现 IOMMU 功能的硬件技术，通过地址重映射实现设备隔离；
  1. Intel VT-d enables the hypervisor to assign individual PCIe devices exclusively to specific virtual machines. 英特尔 VT-d 使虚拟机监控器能够将单个 PCIe 设备专属分配给特定虚拟机。

- **DDIO** (Data Direct I/O) *n.* 英特尔数据直接 I/O 技术；允许网络和存储设备直接向 CPU 末级缓存（LLC）读写数据的英特尔技术，绕过主内存；
  1. Intel DDIO allows RNICs to place incoming network data directly into the CPU's last-level cache. 英特尔 DDIO 允许 RNIC 将接收到的网络数据直接写入 CPU 末级缓存。

### 参考注文

本段描述了现代服务器计算基础设施日益异构化的趋势：CSP 将网络、内存管理等任务卸载至专用外设，同时通过 VT-d、DDIO 和 CXL 等技术使外设能够直接访问 CPU 缓存。这种高度互联的多租户架构，在提升性能的同时也创造了新的安全隐患。

---

### 段三原文

On a logic layer, 【input-output memory management units (IOMMUs)】 enforce memory isolation between these peripherals and guest VMs running on CPUs, making IOMMUs a key component for ensuring security of the cloud infrastructure. The IOMMU ensures that accesses to virtual memory spaces are isolated and appropriately virtualized: e.g., devices may handle only I/O-specific virtual addresses and not the CPU-side virtual addresses or the underlying system's physical addresses; in addition, devices may only access memory with the appropriate permissions set.

However, when many tenants share the same hardware, 【side effects】 in these complex shared memory systems weaken the security promises of virtualization. These side effects of shared hardware are exploited by 【microarchitectural attacks】, most prominently 【cache attacks】. Cache attacks exploit the measurable difference in access times to the many tiers of modern caches to overcome the sophisticated memory isolation mechanisms that protect tenants' data and computation from each other.

### 生词解析

- **side effect** *n.* 副作用（硬件层面）；硬件共享状态（如缓存、TLB）在多租户环境中产生的、可被测量的意外影响；
  1. The side effects of shared caches allow a malicious tenant to infer the access patterns of co-located processes. 共享缓存的副作用允许恶意租户推断共处同机进程的访问模式。

- **microarchitectural attack** *n.* 微架构攻击；利用处理器或内存子系统微架构特性（如缓存、分支预测器）进行信息窃取的攻击类别；
  1. Microarchitectural attacks like Spectre and Meltdown exploit CPU design features to extract sensitive data. Spectre 和 Meltdown 等微架构攻击利用 CPU 设计特性提取敏感数据。

- **cache attack** *n.* 缓存攻击；通过测量缓存命中/缺失产生的时序差异来推断受害者操作信息的侧信道攻击；
  1. Cache attacks have been successfully demonstrated in commercial cloud environments to extract cryptographic keys. 缓存攻击已在商业云环境中被成功演示，用于提取加密密钥。

### 参考注文

本段指出，IOMMU 是云基础设施中强制实施外设内存隔离的核心组件。然而，即使有 IOMMU 保护，多租户共享硬件所产生的微架构副作用（尤其是缓存时序差异）仍然是持续存在的攻击面——本文的核心贡献正是将这一分析扩展到了 IOMMU 内部的 IOTLB 结构。

---

## 段四 Section 2：Background（背景知识）

### 段落原文

**Caches and TLBs.** A 【cache】 stores data for faster access. A 【translation look-aside buffer (TLB)】 is technically just another cache, though rather than caching the data or instructions stored at an address, it caches an address translation. Modern TLBs and caches are typically organized into 【sets】 and 【ways】. The number of ways is the number of entries each set can contain. When a set is full, old entries may be 【evicted】 to make room for new ones. A set of addresses which reliably causes the eviction of all other entries in a set when accessed is called an 【eviction set】. A minimal eviction set contains as many addresses as there are ways in the cache/TLB and therefore fills an entire cache set when accessed.

**Side-Channel Attacks.** Two of the most common techniques:

**Flush+Reload (F+R)** requires 【shared memory】 between the attacker and the victim and has three steps: 1) The attacker 【flushes】 the cache line of interest. 2) She then waits for the victim to execute. 3) She 【reloads】 the flushed line and measures the reload latency. If the latency is low, the cache line was served from the cache hierarchy, so the cache line was accessed by the victim.

**Prime+Probe (P+P)** does not require shared memory. P+P has three steps: 1) The attacker 【primes】 the cache set under surveillance with dummy data. 2) She waits for the victim to execute. 3) She accesses the eviction set again and measures the access latency (probing). If the latency is above a certain threshold, the victim accessed cache lines belonging to the cache set under surveillance.

### 生词解析

- **TLB (Translation Look-aside Buffer)** *n.* 转换旁查缓冲区；处理器中缓存虚拟地址到物理地址映射的高速硬件缓存，避免频繁页表遍历；
  1. A TLB hit avoids a costly page-table walk and returns the physical address in a single cycle. TLB 命中避免了代价高昂的页表遍历，可在单个周期内返回物理地址。

- **set / way** *n.* 组/路；描述组相联缓存结构的两个维度：组（set）决定地址映射到哪个槽位，路（way）表示每组可容纳的条目数；
  1. A 4-way set-associative TLB can hold four different address mappings for the same set index. 一个 4 路组相联 TLB 可以为同一组索引保存四个不同的地址映射。

- **evict** *v.* 驱逐、淘汰；当缓存/TLB 已满时，将旧条目从中移除以腾出空间的操作；
  1. Accessing a full eviction set evicts the target entry from the IOTLB, causing a measurable miss on the next access. 访问一个完整的驱逐集会将目标条目从 IOTLB 中驱逐，导致下次访问时出现可测量的缺失。

- **eviction set** *n.* 驱逐集；一组地址的集合，当全部访问后能可靠地将特定目标条目从缓存中驱逐出去；
  1. Constructing an eviction set for the IOTLB allows Prime+Probe attacks against co-resident peripherals. 为 IOTLB 构建驱逐集，可以对共处同机的外设发动 Prime+Probe 攻击。

- **Flush+Reload** *n.* 冲刷+重载攻击；一种需要攻击者与受害者共享内存的缓存侧信道攻击技术；
  1. Flush+Reload achieves single-cache-line granularity, making it the most precise cache side-channel technique. Flush+Reload 实现了单缓存行粒度，是最精确的缓存侧信道技术。

- **Prime+Probe** *n.* 预置+探测攻击；一种不需要共享内存的缓存侧信道攻击，通过检测驱逐集被替换来推断受害者活动；
  1. Prime+Probe is used to monitor IOTLB usage patterns of a co-located GPU from an attacker-controlled FPGA. Prime+Probe 被用于从攻击者控制的 FPGA 监控共处同机的 GPU 的 IOTLB 使用模式。

### 参考注文

本节介绍了理解 IOTLB 侧信道攻击所需的背景知识：TLB 是一种特殊缓存，其组相联结构与 CPU 缓存相似，存在因集合满载而驱逐旧条目的行为。Flush+Reload 和 Prime+Probe 是两种经典缓存侧信道攻击范式，本文将这些技术迁移应用于 IOMMU 的 IOTLB，首次在外设层面实现了同类攻击。

---

## 段五 Section 3：Identifying IOTLB Side-Channels（识别 IOTLB 侧信道）

### 段落原文

During their PCIe performance benchmarking, Neugebauer et al. found that an 【IOTLB miss】 results in a latency increase of 330 ns. Since the FPGAs in our systems are clocked at 200 MHz, the expected difference between fast and slow accesses is 66 clock cycles. With disabled IOMMU, the memory read latency for any address in main memory is distributed around 160 and 185 cycles. When the system is configured to use the IOMMU, this distribution shifts to 225 and 270 cycles for addresses that are accessed for the first time. Access times for subsequent accesses are distributed similarly to access times measured without IOMMU. Thus the measurable latency difference between accesses to addresses where the translation is present in or absent from the IOTLB lies between 65 and 85 clock cycles.

Our iotlb_pnp hardware module is designed against the Intel Acceleration Stack as would be the case in a cloud environment. The module is capable of performing memory accesses and timing the access latency. The prime instructions make the hardware module access a configured address (target) or set of addresses (eviction set). Probe instructions behave in the same way but additionally count clock cycles. The 【eviction sets】 used during priming and probing can be configured independent from each other.

### 生词解析

- **IOTLB miss** *n.* IOTLB 缺失；DMA 请求中的 I/O 虚拟地址不在 IOTLB 缓存中，需要进行耗时的页表遍历；
  1. An IOTLB miss forces the IOMMU to perform a full page-table walk, adding approximately 330 ns to the DMA access latency. IOTLB 缺失迫使 IOMMU 执行完整的页表遍历，使 DMA 访问延迟增加约 330 纳秒。

- **Intel Acceleration Stack (IAS)** *n.* 英特尔加速栈；英特尔为 FPGA 云部署提供的软件管理框架，包含 OPAE 等工具；
  1. The Intel Acceleration Stack provides the OPAE library, which our FPGA hardware module uses for MMIO-based configuration. 英特尔加速栈提供了 OPAE 库，我们的 FPGA 硬件模块使用该库进行基于 MMIO 的配置。

- **MMIO** (Memory-Mapped I/O) *n.* 内存映射 I/O；通过将设备寄存器映射到内存地址空间，使 CPU 可以用普通内存读写指令访问外设的机制；
  1. Configuration and programming of the FPGA hardware module is performed via MMIO through the OPAE library. FPGA 硬件模块的配置和编程通过 OPAE 库经由 MMIO 完成。

- **clock cycle** *n.* 时钟周期；处理器或硬件模块的基本时间单位，用于精确量化操作延迟；
  1. The measurable latency gap between IOTLB hits and misses spans 65 to 85 FPGA clock cycles at 200 MHz. 在 200 MHz 下，IOTLB 命中与缺失之间可测量的延迟差距为 65 到 85 个 FPGA 时钟周期。

### 参考注文

本节证实了 IOTLB 缺失会产生约 65–85 个时钟周期的可测量延迟差异，这是构建侧信道的物理基础。研究者在 FPGA 上设计了专用的 `iotlb_pnp` 硬件模块，可以精确测量 DMA 访问延迟，从而可靠地区分 IOTLB 命中和缺失，为后续的驱逐集构建和 Prime+Probe 攻击奠定了工具基础。

---

### 段六 威胁模型

### 段落原文

We consider two general threat models with two variants each. All four threat models include a malicious actor that can program and control a fast and programmable PCIe device (referred to as the 【monitoring device】) with direct memory access and an IOMMU providing address translation services for that device.

**Models 1k and 1u** are adversarial threat models for a 【side-channel attack】, where a malicious user in control of the monitoring device exploits IOTLB contention to gain secret information from another user's application that triggers memory accesses in the 【sending device】.

**Models 2k and 2u** outline the requirements for a 【covert channel】 with cooperative sending and monitoring devices, where colluding malicious users in control of applications in separate security domains uses the IOTLB to transmit data covertly across the two devices.

Models ending in **k** include kernel access alongside the monitoring device, and models ending in **u** do not. Kernel access is necessary to implement an IOTLB flush through a custom kernel module.

The primary logistical challenge is **IOMMU co-location** – that is, ensuring that the monitoring device shares an IOMMU (and IOTLB) with the sending device.

### 生词解析

- **monitoring device** *n.* 监测设备；在侧信道攻击中，攻击者用于感知目标设备 IOTLB 活动的可编程外设（如 FPGA）；
  1. The FPGA serves as the monitoring device, using Prime+Probe to detect IOTLB evictions caused by the GPU. FPGA 充当监测设备，使用 Prime+Probe 检测 GPU 引起的 IOTLB 驱逐。

- **side-channel attack** *n.* 侧信道攻击；利用系统运行时物理特征（时序、功耗、电磁辐射等）获取敏感信息的攻击，而非直接破解加密算法；
  1. The IOTLB side-channel attack reveals whether a neighboring GPU is processing a database query without accessing its memory. IOTLB 侧信道攻击可以揭示相邻 GPU 是否在处理数据库查询，而无需访问其内存。

- **covert channel** *n.* 隐蔽信道；利用非预期通信媒介（如共享硬件资源）在不被系统安全策略检测的情况下传递信息的通信方式；
  1. The peripheral-to-peripheral covert channel achieves 7.58 bps throughput by encoding bits as GPU SQL query execution. 外设到外设的隐蔽信道通过将比特编码为 GPU SQL 查询执行，实现了 7.58 bps 的吞吐量。

- **IOMMU co-location** *n.* IOMMU 共置；攻击者的监测设备与目标发送设备被同一个 IOMMU 管理，从而共享 IOTLB 的状态；
  1. Achieving IOMMU co-location in the cloud requires adapting existing co-location techniques from cache attack research. 在云中实现 IOMMU 共置需要借鉴缓存攻击研究中的现有共置技术。

### 参考注文

本节定义了四种威胁模型，按攻击目的（侧信道 vs. 隐蔽信道）和内核访问权限（k 为有内核权限，u 为无内核权限）两个维度组合而成。核心前提是攻击者控制的监测设备（FPGA）与目标发送设备（如 GPU）共享同一 IOMMU，攻击者利用 IOTLB 争用感知目标设备的内存访问模式。

---

## 段七 Section 4：Constructing Eviction Sets（构建驱逐集）

### 段落原文

We developed a novel and platform-independent algorithm for finding eviction sets for any TLB or cache where the timing difference between a present entry and an evicted entry is known and measurable. Our approach is inspired by the baseline reduction algorithm, which only reduces an already existing eviction set to its minimum necessary size.

The most basic function in our algorithm tests whether or not a hypothetical eviction set evicts a given target address. We define that an eviction set 【evicts】 a target if the latency of the second access to the target is above a certain threshold. The construction of an eviction set for a fixed target address follows a 【grow-reduce】 strategy: during the "grow" step random addresses are chosen from the address pool and added to the eviction set until the eviction set contains enough addresses to evict the target. This is followed by a 【reduction step】 where each address is tested for its necessity. If an address is not needed, it is removed from the eviction set and put back in the address pool.

Enabling IOTLB 【flushes】 before the Prime+Probe step will make both algorithms return a single eviction set containing 118 addresses with a success rate of **100%**. Without IOTLB flushes, our grow-reduce algorithm produces eviction sets that are both smaller and twice as reliable than those produced by the grow-split algorithm.

### 生词解析

- **grow-reduce algorithm** *n.* 增长-削减算法；先向候选集中不断添加地址直至能可靠驱逐目标，再逐一测试并移除多余地址的双阶段驱逐集构建算法；
  1. The grow-reduce algorithm constructs minimal IOTLB eviction sets without any prior assumptions about cache organization. 增长-削减算法无需对缓存组织结构作任何先验假设，即可构建最小 IOTLB 驱逐集。

- **reduction step** *n.* 削减步骤；驱逐集构建算法中，测试并去除对驱逐效果非必要地址的优化阶段；
  1. The reduction step shrinks the eviction set by removing addresses that are not needed to evict the target. 削减步骤通过删除对驱逐目标非必要的地址来缩减驱逐集规模。

- **flush** *v./n.* 清空/刷新；将 IOTLB 中的全部或部分条目强制失效，使后续访问触发缺失；此操作通常需要内核权限；
  1. Flushing the IOTLB before each Prime+Probe measurement eliminates noise from concurrent peripheral activity. 在每次 Prime+Probe 测量前清空 IOTLB，可消除并发外设活动引入的噪声。

- **address pool** *n.* 地址池；用于驱逐集构建算法的候选内存页地址集合；
  1. The grow-reduce algorithm draws candidate addresses from a pool of 4096 randomly allocated pages. 增长-削减算法从 4096 个随机分配页面组成的地址池中抽取候选地址。

### 参考注文

本节提出了一种新颖的平台无关驱逐集构建算法（增长-削减法），其核心优势在于无需对 IOTLB 的组织结构（组数、路数、替换策略）进行先验假设。实验表明，在启用 IOTLB 清空（需内核权限）的条件下，该算法可以构建含 118 个地址的完美驱逐集，成功率达 100%；即使在无内核权限的场景下，其性能也优于现有的增长-分裂算法。

---

## 段八 Section 5：Side-Channel Leakage Analysis（侧信道泄漏分析）

### 段落原文

We now use the constructed eviction sets to further investigate the amount of leakage from PCIe devices observable in the IOMMU. We focus our analysis on an in-memory SQL database accelerated by a graphics card.

**GPU-Accelerated SQL Database Leakage.** After constructing an eviction set for the IOTLB, the test app primes the IOTLB. During the waiting phase, the app runs an SQL query on the GPU. The tested queries differ significantly in the size of the returned results. After the SQL result is returned to the test application, the FPGA probes the IOTLB and reports the access latency back to the application.

Clearly, the GPU leaves a footprint in the IOTLB when it computes an SQL query. But, there is no measurable difference between the queries even if their results significantly differ in size. This is caused by two facts: (a) Controlling an accelerator via 【MMIO】 rather than through DMA is a common usage model and limits the attack surface; (b) Current PCIe devices usually perform DMA as 【bulk transfers】, thereby limiting the overall PCIe protocol overhead. 

The two facts mentioned will likely change in the near future as **PCIe 5.0** is rolled-out and **CXL** is introduced. PCIe 5.0 reaches transfer speeds comparable with CPU main memory accesses. Furthermore, CXL features a 【coherency protocol】 that streamlines caching between main memory and PCIe device memory, leading device developers to change from bulk transfers to more fine-grained data-dependent DMA accesses.

### 生词解析

- **bulk transfer** *n.* 批量传输；将大量数据作为一个整体一次性搬运的 DMA 操作模式，而非逐条目的细粒度访问；
  1. Bulk DMA transfers hide per-query data access patterns, limiting the granularity of IOTLB side-channel leakage. 批量 DMA 传输隐藏了每次查询的数据访问模式，限制了 IOTLB 侧信道泄漏的粒度。

- **coherency protocol** *n.* 缓存一致性协议；确保多个缓存实体（CPU 缓存、设备缓存）对同一内存地址保持一致视图的机制；
  1. CXL's coherency protocol enables accelerators to cache main memory data, enabling finer-grained DMA patterns that amplify IOTLB leakage. CXL 的缓存一致性协议使加速器能够缓存主内存数据，产生更细粒度的 DMA 模式，从而放大 IOTLB 泄漏。

- **footprint** *n.* 访问足迹；一个程序或设备在共享资源（如 IOTLB、缓存）中留下的、可被旁观者测量的状态变化痕迹；
  1. The GPU's SQL processing leaves a distinct footprint in the IOTLB that is detectable even with Prime+Probe. GPU 的 SQL 处理在 IOTLB 中留下了独特的访问足迹，即使使用 Prime+Probe 也可以检测到。

### 参考注文

本节实测了 GPU 加速 SQL 数据库对 IOTLB 的侧信道影响：GPU 执行查询时会在 IOTLB 中留下可测量的驱逐痕迹，但由于当前 GPU 主要通过 MMIO 而非 DMA 传输结果，泄漏信息仅限于"是否有查询在执行"的单比特信息。然而，随着 PCIe 5.0 和 CXL 引入更细粒度的数据依赖性内存访问，这一局面将发生根本性改变。

---

## 段九 Section 6：Covert Channels（隐蔽信道）

### 段落原文

**Peripheral-to-Peripheral Covert Channel.** The sender encodes a one into running an SQL query and running no query encodes a zero. The receiver uses the iotlb_pnp hardware function on the FPGA to monitor the IOTLB using the Prime+Probe technique. Each SQL query evicts 18–20 entries of the receiver's eviction set. We found that basically **no errors** occur if sender and receiver are synchronized. The channel's throughput highly depends on the number of one bits in the message. This is because the execution time of a single SQL query takes about **0.3 seconds**.

**CPU-to-Peripheral Covert Channel.** A global IOTLB flush takes **17 μs** on average. Flushing all entries from the IOTLB encodes a 1 and sleeping for 17 μs encodes a 0. As the receiver we use the iotlb_pnp hardware module. This basic covert channel without further optimizations already achieves a throughput of around **15 kBit/s**. The error rate is **30%** which can be improved significantly by applying error-correction and error-handling techniques.

The demonstrated covert channel is reliable without applying special synchronization, error-correction, or error-detection techniques. However, only peripherals can act as the receiver while the CPU is limited to the role of the sender.

### 生词解析

- **error-correction code** *n.* 纠错码；用于检测并自动纠正传输过程中产生错误比特的编码技术（如汉明码、Hadamard 码）；
  1. Applying Hadamard codes to the CPU-peripheral covert channel can reduce the 30% error rate to near zero. 将 Hadamard 码应用于 CPU 到外设的隐蔽信道，可以将 30% 的错误率降至接近零。

- **throughput** *n.* 吞吐量（信道容量）；单位时间内信道可以传输的有效数据量，此处以 bps（比特/秒）衡量；
  1. The peripheral-to-peripheral covert channel achieves 7.58 bps throughput for ASCII-encoded text messages. 外设到外设的隐蔽信道对 ASCII 编码文本消息实现了 7.58 bps 的吞吐量。

- **insertion/deletion error** *n.* 插入/删除错误；由于发送方和接收方未精确同步，导致接收端误判比特位置的错误类型；
  1. Without synchronization, the covert channel suffers from insertion and deletion errors requiring specialized coding. 在没有同步机制的情况下，隐蔽信道会出现插入和删除错误，需要专门的编码技术来克服。

### 参考注文

本节构建并验证了两种 IOTLB 隐蔽信道：外设到外设信道（GPU 通过执行/不执行 SQL 查询编码信息，FPGA 通过 Prime+Probe 接收，误码率接近零）和 CPU 到外设信道（CPU 通过触发 IOTLB 清空编码信息，FPGA 接收，吞吐量约 15 kbps，误码率 30%）。这两个信道直接证明了 IOTLB 可被用作多租户环境中的实用隐蔽通信媒介。

---

## 段十 Section 7：Countermeasures（防御对策）

### 段落原文

**Securing Existing Systems.** In cases where multiple users who do not trust each other may use the same machine, ensuring that no two users have access to peripherals on the same IOMMU hardware is sufficient to protect against IOTLB side-channel attacks. Endpoints cannot be reassigned to new IOMMUs, so ensuring full isolation may limit scaling capacity. 

A hypervisor can enable 【Address Translation Services (ATS)】 for a peripheral to remove all of its traces from its IOTLB. ATS allows a device to maintain and use a local on-device TLB for address translation and selectively bypass IOMMU translation. However, devices must specifically support ATS, and allowing ATS for untrusted devices is not advisable as it allows a device to provide any physical address as part of a DMA request and mark it as "translated", enabling 【unrestricted physical memory access】.

**Securing Future IOMMUs.** If hardware modifications are a viable option, **way-based partitioning** is another option. It needs to be supported by the IOMMU hardware so that the hypervisor can map each address of a thread to a fixed number of ways. Future IOMMUs could include support for flagging a page translation as 【uncacheable】. This would ensure that it is never stored in the IOTLB and that the use of that page would never affect the IOTLB state. However, all accesses to that page would be as slow as IOTLB misses, increasing latency and likely reducing maximum throughput.

### 生词解析

- **Address Translation Services (ATS)** *n.* 地址转换服务；PCIe 标准的可选扩展，允许设备在本地维护已转换的地址并在 DMA 请求中绕过 IOMMU 验证；
  1. ATS removes a peripheral's traces from the shared IOTLB but also allows malicious devices to bypass IOMMU isolation entirely. ATS 消除了外设在共享 IOTLB 中的踪迹，但也允许恶意设备完全绕过 IOMMU 隔离。

- **unrestricted physical memory access** *n.* 不受限制的物理内存访问；设备能够通过构造虚假的"已转换"DMA 请求来读写任意物理内存地址；
  1. ATS-enabled malicious devices can mark any DMA request as pre-translated, gaining unrestricted physical memory access. 启用了 ATS 的恶意设备可以将任意 DMA 请求标记为已预转换，从而获得不受限制的物理内存访问。

- **way-based partitioning** *n.* 基于路的分区；将缓存或 TLB 的不同路分配给不同租户，实现强制隔离的硬件机制（类似 Intel CAT）；
  1. Way-based partitioning of the IOTLB would prevent cross-tenant evictions, but requires hardware support not present in current IOMMUs. IOTLB 基于路的分区将防止跨租户驱逐，但需要当前 IOMMU 中尚不具备的硬件支持。

- **uncacheable** *adj.* 不可缓存的；被标记为不应存入缓存或 TLB 的内存区域，每次访问都需要重新转换；
  1. Marking security-critical page translations as uncacheable prevents them from leaking IOTLB state to co-resident peripherals. 将安全敏感的页转换标记为不可缓存，可防止其向共处同机的外设泄漏 IOTLB 状态。

### 参考注文

本节提出了多层次的防御对策：短期可行方案包括确保不同租户的外设不共享同一 IOMMU；ATS 可消除泄漏但引入新的安全风险（需仅对受信任设备启用）；长远来看，未来 IOMMU 应支持基于路的分区（类似 Intel CAT 对 CPU 缓存的处理）或不可缓存页转换标记，以从根本上消除 IOTLB 侧信道威胁，尽管这些方案都会以牺牲一定性能或可扩展性为代价。

---

## 段十一 Section 8：Conclusion（结论）

### 段落原文

State-of-the-art cloud environments use direct memory access managed by IOMMUs to offer high speed, low latency, and isolated memory access to an increasingly wide variety of peripherals. In this paper we demonstrated a new side-channel attack against IOTLBs in such IOMMUs that works across virtual environments and threatens cloud tenants. We developed a new 【eviction set finding algorithm】 that works without prior assumptions of cache or TLB organization and a hardware module for an FPGA that implements the fundamentals necessary to exploit the IOTLB side-channel. We used these tools to record a side-channel trace from a GPU running a database acceleration library. The results prove that the IOTLB can be used as a side-channel to spy on co-located devices. We highlight this fact by showing a very reliable covert channel from the GPU to the FPGA where we use the database application running on the GPU to encode messages into the GPU's system memory access patterns.

While we acknowledge the limitations of the IOTLB channel with current hardware and applications, we argue that with the upcoming PCIe 5.0 and CXL standards, IOMMU usage patterns will change and fine-grained IOTLB side-channel attacks will become practical. When designing security-critical peripherals or security-critical software or firmware that makes use of peripherals, 【timing leakages】 from peripheral memory accesses must be addressed with 【constant-time design practices】.

### 生词解析

- **eviction set finding algorithm** *n.* 驱逐集发现算法；用于在未知缓存组织结构下系统性地确定能驱逐特定条目的地址集合的方法；
  1. The new eviction set finding algorithm adapts dynamically to any IOTLB architecture without requiring prior knowledge of its parameters. 新的驱逐集发现算法可动态适应任何 IOTLB 架构，无需预先了解其参数。

- **timing leakage** *n.* 时序泄漏；系统中由操作耗时随数据内容变化而产生的、可被旁观者测量利用的信息泄露；
  1. Timing leakages from DMA access patterns can reveal the type and size of workloads running on co-located peripherals. DMA 访问模式的时序泄漏可能揭示共处同机外设上运行的工作负载的类型和规模。

- **constant-time design** *n.* 恒定时间设计；一种软件/硬件设计原则，确保操作耗时不依赖于所处理数据的值，从而消除时序侧信道；
  1. Applying constant-time design principles to GPU database kernels would prevent IOTLB-based leakage of query patterns. 将恒定时间设计原则应用于 GPU 数据库内核，将防止基于 IOTLB 的查询模式泄漏。

### 参考注文

结论部分总结了 IOTLB-SC 的主要贡献：首次证明了 IOMMU 的 IOTLB 可作为侧信道和隐蔽信道，构建了平台无关的驱逐集发现算法，并实测了对 GPU 加速 SQL 数据库的监控能力。作者特别强调，随着 PCIe 5.0 和 CXL 改变外设内存访问模式，这一攻击面将在未来显著增强，建议安全敏感的外设实现采用恒定时间设计实践。
