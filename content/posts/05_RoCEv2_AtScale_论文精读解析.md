---
title: "论文精读解析：RDMA over Commodity Ethernet at Scale"
date: 2026-03-17
description: "论文精读解析：RDMA over Commodity Ethernet at Scale"
tags: ["rdma", "paper", "security", "Collie"]
showToc: true
TocOpen: true
draft: false
---
# 论文精读解析：RDMA over Commodity Ethernet at Scale

> **来源**：ACM SIGCOMM 2016, Florianopolis, Brazil
> **作者**：Chuanxiong Guo, Haitao Wu, Zhong Deng, Gaurav Soni, Jianxi Ye, Jitendra Padhye, Marina Lipshteyn（Microsoft）
> **解析格式**：标题解读 → 原文（关键词标注）→ 生词解析 → 参考注文

---

## 标题解读

**RDMA over Commodity Ethernet at Scale**
在商用以太网上大规模部署RDMA

- **commodity Ethernet** n. 商用以太网（使用标准商用交换机和网卡的以太网，而非专有InfiniBand网络）；
  1. Commodity Ethernet switches are cheaper and more widely available than InfiniBand. 商用以太网交换机比InfiniBand更便宜、更易获取。

- **at scale** 短语 大规模（指在数以万计的服务器规模下进行部署）；
  1. Deploying RDMA at scale requires solving safety and management challenges. 大规模部署RDMA需要解决安全性和管理挑战。

---

## 段一：Abstract（摘要）

**原文（关键词已标注）：**

Over the past one and half years, we have been using 【RDMA over commodity Ethernet (RoCEv2)】 to support some of Microsoft's highly-reliable, latency-sensitive services. This paper describes the challenges we encountered during the process and the solutions we devised to address them. In order to scale RoCEv2 beyond VLAN, we have designed a 【DSCP-based priority flow-control (PFC)】 mechanism to ensure large-scale deployment. We have addressed the safety challenges brought by 【PFC-induced deadlock】, 【RDMA transport livelock】, and the 【NIC PFC pause frame storm】 problem. We have also built the monitoring and management systems to make sure RDMA works as expected. Our experiences show that the safety and scalability issues of running RoCEv2 at scale can all be addressed, and RDMA can replace TCP for 【intra data center communications】 and achieve low latency, low CPU overhead, and high throughput.

**生词解析：**

- **RoCEv2 (RDMA over Converged Ethernet v2)** n. 以太网汇聚RDMA第二版（将InfiniBand传输封装在UDP/IP之上的RDMA协议）；
  1. RoCEv2 encapsulates RDMA transport inside Ethernet/IPv4/UDP packets for routing. RoCEv2将RDMA传输封装在以太网/IPv4/UDP数据包中以支持路由。

- **DSCP (Differentiated Services Code Point)** n. 差分服务代码点（IP头部中用于指定数据包优先级的6位字段）；
  1. DSCP-based PFC maps packet priority to PFC classes using IP header fields. 基于DSCP的PFC使用IP头部字段将数据包优先级映射到PFC类别。

- **PFC-induced deadlock** n. PFC引发的死锁（PFC暂停帧传播形成循环依赖，导致网络完全停止的状态）；
  1. PFC-induced deadlock can persist indefinitely without manual intervention. PFC引发的死锁如不手动干预可能无限持续。

- **RDMA transport livelock** n. RDMA传输活锁（链路看似繁忙但应用无法取得进展的状态，通常由go-back-0重传机制引起）；
  1. RDMA livelock occurs when packets keep being retransmitted from the beginning. 当数据包不断从头重传时发生RDMA活锁。

- **NIC PFC pause frame storm** n. 网卡PFC暂停帧风暴（故障网卡持续发送暂停帧，导致整个网络瘫痪的现象）；
  1. A single malfunctioning NIC can cause a PFC storm affecting thousands of servers. 单块故障网卡可导致影响数千台服务器的PFC风暴。

- **intra data center communication** n. 数据中心内部通信（同一数据中心内服务器之间的流量）；
  1. RDMA is designed for intra-DC communications where low latency is critical. RDMA专为对延迟敏感的数据中心内部通信设计。

**参考注文：**

本文是微软在大规模生产环境中部署RoCEv2的经验总结，是RDMA领域最具工程影响力的论文之一。作者描述了VLAN限制催生DSCP-PFC方案、三类严重安全故障（活锁、死锁、PFC风暴）的发现与解决过程，以及配套的监控管理系统。核心结论是：生产环境的RDMA部署需要系统性解决协议安全性、网络配置和运维监控等问题，缺一不可。

---

## 段二：Section 2 — Background（背景）

**原文（关键词已标注）：**

Our data center network is an Ethernet-based multi-layer 【Clos network】. Twenty to forty servers connect to a 【top-of-rack (ToR) switch】. Tens of ToRs connect to a layer of 【Leaf switches】. The Leaf switches in turn connect to a layer of tens to hundreds of 【Spine switches】.

RoCEv2 encapsulates an RDMA transport packet within an Ethernet/IPv4/UDP packet. The destination UDP port is always set to 4791, while the source UDP port is randomly chosen for each queue pair (QP). The intermediate switches use standard 【five-tuple hashing】. Thus, traffic belonging to the same QP follows the same path.

**PFC and buffer reservation:** PFC is a hop-by-hop protocol between two Ethernet nodes. Once the 【ingress queue length】 reaches a certain threshold (XOFF), the switch sends out a PFC pause frame to the corresponding upstream egress queue. After the egress queue receives the pause frame, it stops sending packets. Once the ingress queue length falls below another threshold (XON), the switch sends a pause with zero duration to resume transmission. The reserved buffer for the "gray period" between sending and receiving a pause frame is called 【headroom】.

**生词解析：**

- **Clos network** n. Clos网络（一种多级交叉开关网络拓扑，广泛用于数据中心以实现无阻塞通信）；
  1. Clos networks provide high bisection bandwidth and support thousands of servers. Clos网络提供高对分带宽，支持数千台服务器。

- **top-of-rack (ToR) switch** n. 架顶交换机（位于机架顶部，连接机架内所有服务器的接入层交换机）；
  1. Each ToR switch connects 20-40 servers within the same rack. 每台架顶交换机连接同一机架内的20至40台服务器。

- **Leaf/Spine switch** n. 叶层/脊层交换机（Fat-tree/Clos拓扑中的汇聚层和核心层交换机）；
  1. Leaf and Spine switches form the core of the multi-tier Clos topology. 叶层和脊层交换机构成多层Clos拓扑的核心。

- **five-tuple hashing** n. 五元组哈希（基于源/目的IP地址、端口和协议的哈希，用于ECMP多路径负载均衡）；
  1. Five-tuple hashing ensures packets of the same flow follow the same network path. 五元组哈希确保同一流量的数据包走相同的网络路径。

- **XOFF/XON threshold** n. 暂停/恢复阈值（触发发送PFC暂停帧和恢复帧的队列占用量阈值）；
  1. The XOFF threshold must be set conservatively to ensure no buffer overflow. XOFF阈值必须设置保守以确保不发生缓冲区溢出。

- **headroom** n. 余量缓冲（PFC暂停帧从发出到生效期间，用于吸收持续到达数据包的预留缓冲空间）；
  1. Headroom size is determined by link propagation delay and PFC reaction time. 余量缓冲大小由链路传播延迟和PFC响应时间决定。

- **ECMP (Equal-Cost Multi-Path)** n. 等价多路径（允许流量在多条等价路径之间分摊的路由技术）；
  1. ECMP routing achieves approximately 60% network utilization due to hash collisions. ECMP路由因哈希冲突约实现60%的网络利用率。

**参考注文：**

背景节描述了微软数据中心的三层Clos拓扑，以及RoCEv2与PFC的技术细节。关键点是PFC的逐跳特性：暂停帧会从拥塞点向上游逐跳传播，这是后续安全问题的根源。由于PFC有8个优先级类别，而浅缓冲交换机只能为无损流量预留2个类别的余量缓冲，微软实际只用2个无损类别部署RDMA。

---

## 段三：Section 3 — DSCP-Based PFC（DSCP-PFC方案）

**原文（关键词已标注）：**

VLAN-based PFC carries packet priority in the 【VLAN tag】, which also contains VLAN ID. The coupling of packet priority and VLAN ID created two serious problems. First, the switch 【trunk mode】 has an undesirable interaction with our OS provisioning service — 【PXE boot】 cannot work when server ports are in trunk mode. Second, we have moved away from a layer-2 VLAN, and all our switches are running 【layer-3 IP forwarding】. In a layer-3 network, there is no standard way to preserve the VLAN PCP value when crossing subnet boundaries.

Our key observation is that the PFC pause frames do not have a VLAN tag at all. The solution is to move the packet priority from the VLAN tag into 【DSCP】. With DSCP-based PFC, data packets no longer need to carry the VLAN tag. The server facing ports no longer need to be in trunk mode, which means that PXE boot works without any issues. Also, the packet priority information, in form of DSCP value, is correctly propagated by IP routing across subnets.

**生词解析：**

- **VLAN tag** n. VLAN标签（以太网帧头部中标识虚拟局域网和优先级的4字节字段）；
  1. VLAN tags couple packet priority with VLAN ID, causing scalability problems. VLAN标签将数据包优先级与VLAN ID耦合，导致可扩展性问题。

- **trunk mode** n. 干道模式（允许交换机端口传输带VLAN标签数据包的配置模式）；
  1. Switch trunk mode prevents PXE boot because NIC does not have VLAN configuration yet. 交换机干道模式会阻止PXE启动，因为网卡尚无VLAN配置。

- **PXE boot** n. PXE启动（通过网络从服务器安装或恢复操作系统的技术）；
  1. PXE boot is essential for automated OS provisioning in large data centers. PXE启动对于大型数据中心的自动化操作系统部署至关重要。

- **layer-3 IP forwarding** n. 三层IP转发（交换机根据IP地址而非MAC地址进行路由转发的模式）；
  1. Layer-3 networks offer better scalability and management than layer-2 VLAN networks. 三层网络比二层VLAN网络提供更好的可扩展性和管理性。

- **DSCP** n. 差分服务代码点（IP头部ToS字节中6位的优先级标记字段）；
  1. DSCP values are preserved by IP routing across subnet boundaries. DSCP值在跨子网路由时得到保留。

**参考注文：**

DSCP-based PFC是本文最直接的工程贡献。VLAN-based PFC的两个核心问题（干道模式破坏PXE启动、VLAN标签无法跨子网传递优先级）在微软的三层IP数据中心中都是致命的。DSCP方案的改动极小（仅修改数据包格式，PFC暂停帧格式不变），却优雅地解决了这两个问题，并已被所有主流厂商采纳为标准。

---

## 段四：Section 4.1 — RDMA Transport Livelock（活锁问题）

**原文（关键词已标注）：**

We found that the performance of RDMA degraded drastically even with a very low packet loss rate (1/256 = 0.4%). The system was in a state of 【livelock】 — the link was fully utilized with line rate, yet the application was not making any progress.

The root cause of this problem was the 【go-back-0】 algorithm used for loss recovery by the RDMA transport. Suppose A is sending a message to B. The message is segmented into packets 0, 1, ···, i, ···, m. Suppose packet i is dropped. B then sends an NAK(i) to A. After A receives the NAK, it will restart sending the message from packet 0. A 4MB message is segmented into 4000 packets. Since the packet drop rate is a deterministic 1/256, one packet of the first 256 packets will be dropped. Then the sender will restart from the first packet, again and again, without making any progress.

Our solution is to replace the go-back-0 with a 【go-back-N】 scheme. In go-back-N, retransmission starts from the first dropped packet and the previous received packets are not retransmitted. Go-back-N is not ideal as up to RTT×C bytes can be wasted for a single packet drop. But go-back-N is almost as simple as go-back-0, and it avoids livelock.

**生词解析：**

- **livelock** n. 活锁（进程持续运行但无法取得进展的状态，区别于死锁中进程被阻塞）；
  1. In RDMA livelock, the network is busy but the application completes zero bytes. 在RDMA活锁中，网络繁忙但应用完成了零字节的传输。

- **go-back-0** n. 回退0重传（从消息开头重新发送整个消息的重传策略，实现简单但效率极低）；
  1. Go-back-0 causes livelock when the packet drop rate exceeds 1/message_size. 当丢包率超过1/消息大小时，go-back-0会导致活锁。

- **go-back-N** n. 回退N重传（从第一个丢失包开始重传，之前已接收的包不重传的策略）；
  1. Go-back-N avoids livelock by not retransmitting correctly received packets. go-back-N通过不重传已正确接收的数据包来避免活锁。

- **NAK (Negative Acknowledgment)** n. 否定确认（通知发送方某个数据包未被正确接收的控制消息）；
  1. Upon receiving a NAK, the RDMA sender initiates retransmission. 收到NAK后，RDMA发送方启动重传。

- **packet drop rate** n. 丢包率（被丢弃的数据包占总发送数据包的比例）；
  1. Even a 0.4% packet drop rate can cause RDMA livelock with go-back-0. 即使0.4%的丢包率也能导致使用go-back-0的RDMA活锁。

**参考注文：**

活锁问题揭示了一个深刻的设计矛盾：RDMA传输协议假设网络完全无损，因此采用了最简单的go-back-0重传策略；但实际生产网络中丢包（哪怕极低概率）仍会发生，而go-back-0在这种情况下会导致完全性能崩溃。解决方案go-back-N看似简单，但需要供应商在NIC固件层面实现，是微软与NIC厂商合作的重要成果。

---

## 段五：Section 4.2 — PFC Deadlock（死锁问题）

**原文（关键词已标注）：**

We once believed that our network is deadlock-free because of its 【Clos network topology】 and 【up-down routing】. In such a topology, packets first climb up to one of the common ancestors of the source and the destination, then go down the path to the destination. Hence there is no 【cyclic buffer dependency】. But to our surprise, we did run into PFC deadlock when we ran a stress test in one of our test clusters.

This occurred because the unexpected interaction between PFC and 【Ethernet packet flooding】 broke the up-down routing. When a server is dead, its MAC address may still be in the ARP table but not in the MAC address table (due to different timeout values — 4 hours for ARP vs 5 minutes for MAC). When a packet destined to such a 【stale MAC address】 arrives, the switch will 【flood】 the packet to all its ports. Flooded lossless packets can traverse "downward" links, creating cyclic buffer dependencies and hence deadlock.

**Solution:** We need to stop flooding for lossless packets to prevent deadlock from happening. We chose to drop the lossless packets if their corresponding ARP entry is incomplete — i.e., the MAC address to port mapping is missing.

**The lesson:** broadcast and multicast are dangerous for a lossless network. We recommend that broadcast and multicast packets should not be put into lossless classes.

**生词解析：**

- **up-down routing** n. 上行-下行路由（在Clos网络中，流量必须先上行到公共祖先交换机再下行的路由策略，可保证无死锁）；
  1. Up-down routing guarantees deadlock-free paths in Clos-based data center networks. 上行-下行路由在Clos数据中心网络中保证无死锁路径。

- **cyclic buffer dependency** n. 循环缓冲区依赖（多个交换机缓冲区相互等待对方释放空间形成的环形依赖，是死锁的充要条件）；
  1. Cyclic buffer dependency among four switches created an unresolvable PFC deadlock. 四台交换机之间的循环缓冲区依赖形成了无法解除的PFC死锁。

- **Ethernet packet flooding** n. 以太网数据包泛洪（当交换机不知道目标MAC地址的端口时，将数据包发送到所有端口的行为）；
  1. Flooding of lossless packets can violate the up-down routing property. 无损数据包的泛洪可能破坏上行-下行路由特性。

- **stale MAC address** n. 过期MAC地址（MAC地址表中已超时失效但ARP表中仍存在的地址条目）；
  1. A stale MAC address causes the switch to flood packets to all ports. 过期MAC地址导致交换机向所有端口泛洪数据包。

- **flood** v. 泛洪（将数据包发送到除接收端口外所有端口的操作）；
  1. Flooding unknown unicast packets across a lossless network can cause deadlock. 在无损网络中泛洪未知单播数据包可能导致死锁。

- **ARP table / MAC address table** n. ARP表/MAC地址表（分别记录IP-MAC映射和MAC-端口映射的交换机表格，超时时间不同）；
  1. The disparity between ARP timeout (4h) and MAC timeout (5min) can create incomplete entries. ARP超时（4小时）和MAC超时（5分钟）的差异可产生不完整的条目。

**参考注文：**

PFC死锁的发现是本文最具洞见的工程教训之一。微软工程师原本相信Clos网络的上行-下行路由特性可以杜绝循环依赖，但实际上一种看似无害的行为（MAC地址超时引发的以太网泛洪）破坏了这一保证。解决方案是明确禁止将无损数据包放入泛洪流程——当ARP条目不完整时，直接丢弃无损数据包。这个教训推广开来：广播和组播都不应进入无损优先级队列。

---

## 段六：Section 4.3 — NIC PFC Pause Frame Storm（PFC风暴）

**原文（关键词已标注）：**

A single malfunctioning NIC of server 0 continually sends pause frames to its ToR switch, which in turn pauses all the rest ports including all the upstream ports to the Leaf switches, then to Spine switches, then to more Leaf and ToR switches, and finally to all servers. A single malfunctioning NIC may block the entire network from transmitting. We call this 【NIC PFC pause frame storm】.

The root-cause of the PFC storm problem is a bug in the NIC's receiving pipeline. The bug stopped the NIC from handling the packets it received. As a result, the NIC's receiving buffer filled, and the NIC began to send out pause frames all the time.

We implemented two 【watchdogs】 at both the NIC and the ToR switches. On the NIC side, a 【PFC storm prevention watchdog】: once the NIC micro-controller detects the receiving pipeline has been stopped for a period of time (default to 100ms) and the NIC is generating the pause frames, the micro-controller will disable the NIC from generating pause frames. On the ToR switch side, a switch watchdog monitors the server facing ports. Once a server facing egress port is queuing packets which cannot be drained and the port is receiving continuous pause frames from the NIC, the switch will disable the lossless mode for the port.

**生词解析：**

- **malfunctioning NIC** n. 故障网卡（因固件/硬件错误而行为异常的网络接口卡）；
  1. A malfunctioning NIC sending continuous pause frames can paralyze the entire network. 发送持续暂停帧的故障网卡可能导致整个网络瘫痪。

- **watchdog** n. 看门狗（监控系统运行状态并在检测到异常时自动采取纠正措施的机制）；
  1. The NIC-side watchdog disables pause frame generation when a storm is detected. 网卡侧看门狗在检测到风暴时禁用暂停帧生成。

- **micro-controller** n. 微控制器（NIC上独立于数据路径运行的小型处理器，用于固件管理）；
  1. The NIC's micro-controller monitors the receiving pipeline for storm conditions. 网卡的微控制器监控接收流水线是否出现风暴条件。

- **lossless mode** n. 无损模式（使能PFC的交换机端口配置，确保该端口不因缓冲区满而丢弃数据包）；
  1. The switch watchdog disables lossless mode for the port when a NIC PFC storm is detected. 当检测到网卡PFC风暴时，交换机看门狗禁用该端口的无损模式。

- **power-cycle** v. 强制断电重启（通过关闭再开启电源来重置设备状态的操作）；
  1. Power-cycling the malfunctioning server typically clears the NIC PFC storm. 对故障服务器强制断电重启通常能清除网卡PFC风暴。

**参考注文：**

PFC风暴展示了RDMA网络中"放大攻击面"的一个极端案例：单块服务器的网卡固件错误可以通过PFC暂停帧的逐层传播，在数秒内使整个数据中心的网络陷入瘫痪。微软在实际生产中多次遭遇此问题，最终通过NIC和交换机两侧的看门狗机制将影响范围压缩到100-200毫秒。这两个看门狗相互补充、互为保险，即使其中一个失效，另一个也能阻止风暴继续传播。

---

## 段七：Section 4.4 — Slow-Receiver Symptom（慢接收者症状）

**原文（关键词已标注）：**

We found that many servers may generate up to thousands of PFC pause frames per second. Since RDMA packets do not need the server CPU for processing, the bottleneck must be in the NIC. The NIC has limited memory resources, hence it puts most of the data structures including 【QPC (Queue Pair Context)】 and WQE in the main memory of the server. The NIC only caches a small number of entries in its own memory. The NIC has a 【Memory Translation Table (MTT)】 which translates the virtual memory to the physical memory. The MTT has only 2K entries. For 4KB page size, 2K MTT entries can only handle 8MB memory.

If the virtual address in a WQE is not mapped in the MTT, it results in a cache miss, and the NIC has to replace some old entries for the new virtual address. The NIC has to access the main memory of the server to get the entry for the new virtual address. All those operations take time and the receiving pipeline has to wait. The MTT cache miss will therefore slow down the packet processing pipeline. We call this phenomenon the 【slow-receiver symptom】.

**Mitigation:** On the NIC side, we used a large page size of 2MB instead of 4KB — with a large page size, the MTT entry miss becomes less frequent. On the switch side, we enabled 【dynamic buffer sharing】 among different switch ports.

**生词解析：**

- **QPC (Queue Pair Context)** n. 队列对上下文（存储QP连接状态的数据结构，RNIC处理每个请求时都需要访问）；
  1. QPC for each QP must be fetched from host DRAM if not cached in RNIC memory. 如果未缓存在RNIC内存中，每个QP的QPC必须从主机DRAM中获取。

- **slow-receiver symptom** n. 慢接收者症状（接收侧NIC因缓存缺失导致处理速度跟不上网络速率而发送PFC暂停帧的现象）；
  1. The slow-receiver symptom can cause PFC pause frames to propagate into the network. 慢接收者症状会导致PFC暂停帧传播到网络中。

- **large page (HugePages)** n. 大页（操作系统分配的比标准4KB更大的内存页，如2MB或1GB，减少页表条目数）；
  1. Using 2MB large pages reduces MTT entry misses in the RNIC. 使用2MB大页减少了RNIC中的MTT条目缺失。

- **dynamic buffer sharing** n. 动态缓冲共享（交换机端口从共享缓冲池动态分配内存的策略，比静态分配更高效）；
  1. Dynamic buffer sharing allows switches to absorb burst traffic without generating PFC. 动态缓冲共享允许交换机在不产生PFC的情况下吸收突发流量。

**参考注文：**

慢接收者症状揭示了RNIC资源受限带来的级联效应：NIC片上缓存太小（MTT只有2K条目），一旦注册了超过8MB的内存区域，每个超出范围的包接收都会触发PCIe往返，拖慢接收流水线，进而使NIC接收缓冲区积压并开始发送PFC暂停帧——这些暂停帧又会传播到网络中影响无辜流量。两步缓解措施（大页降低缓存缺失频率，动态缓冲共享提高交换机本地吸收能力）相互配合，有效抑制了此问题。

---

## 段八：Section 5 — RDMA in Production（生产环境）& Section 6 — Experiences（经验）

**原文（关键词已标注）：**

The 99th percentile latencies for RDMA and TCP were 90us and 700us, respectively. The 99th percentile latency for TCP had spikes as high as several milliseconds. Even the 99.9th latency of RDMA was only around 200us, and much smaller than TCP's 99th percentile latency.

**Step-by-step deployment:** We devised a step-by-step procedure to onboard RDMA: (1) lab network with tens of servers; (2) test clusters same as production; (3) production networks at ToR level only; (4) PFC at Podset level; (5) PFC up to Spine switches. The RDMA transport livelock and most bugs were detected in lab tests. The PFC deadlock and slow-receiver symptom were detected in test clusters. Only the NIC PFC pause frame storm and a few other bugs hit production networks.

**Key lessons:**

1. **Deadlock, livelock, and PFC propagation did happen.** A design that works in theory is not enough — hidden details can invalidate the design.
2. **NICs are the key to make RDMA/RoCEv2 work.** Most bugs were caused by NICs because they implement the most complicated parts of RDMA, and they are resource constrained.
3. **Be prepared for the unexpected.** RDMA management and monitoring must be an indispensable part of the project from day one.

**生词解析：**

- **99th percentile latency** n. 99百分位延迟（P99，所有请求中99%都能在该延迟以内完成，反映尾延迟）；
  1. RDMA achieves 90µs P99 latency compared to TCP's 700µs for the same service. RDMA实现了90微秒的P99延迟，相比同一服务TCP的700微秒。

- **tail latency** n. 尾延迟（高百分位（如P99、P99.9）延迟，反映最坏情况下的用户体验）；
  1. RDMA dramatically reduces tail latency by bypassing the OS kernel. RDMA通过绕过操作系统内核大幅降低尾延迟。

- **step-by-step deployment** n. 逐步部署（从小规模实验室环境到大规模生产环境逐阶段推进的部署策略）；
  1. A step-by-step deployment procedure allows bugs to be found early at smaller scale. 逐步部署流程允许在较小规模时尽早发现错误。

- **Pingmesh** n. 全网主动时延测量系统（微软开发的通过服务器互相Ping测量整网RTT的监控系统）；
  1. RDMA Pingmesh provides continuous latency measurements across all servers. RDMA Pingmesh提供覆盖所有服务器的持续时延测量。

**参考注文：**

生产数据令人印象深刻：RDMA的P99延迟（90μs）比TCP（700μs）低近8倍，P99.9也仅为200μs，远低于TCP的P99。这不仅源于内核绕过带来的延迟降低，更源于消除了偶发丢包引起的TCP重传尖刺。分阶段部署策略的有效性得到了验证：所有最危险的故障（活锁、死锁、风暴）都在产品化前被在测试集群阶段发现，仅有部分NIC固件缺陷在生产环境中才暴露。管理和监控系统与RDMA核心功能同步建设的经验尤为宝贵。

---

## 附录：核心术语速查表

| 英文术语 | 中文释义 |
|---------|---------|
| RoCEv2 | 以太网汇聚RDMA第二版 |
| commodity Ethernet | 商用以太网 |
| DSCP-based PFC | 基于DSCP的优先级流量控制 |
| PFC livelock | PFC活锁 |
| PFC deadlock | PFC死锁 |
| NIC PFC storm | 网卡PFC暂停帧风暴 |
| go-back-0 / go-back-N | 回退0/N重传策略 |
| Clos network | Clos网络拓扑 |
| up-down routing | 上行-下行路由 |
| cyclic buffer dependency | 循环缓冲区依赖 |
| Ethernet flooding | 以太网泛洪 |
| XOFF/XON threshold | 暂停/恢复阈值 |
| headroom | 余量缓冲 |
| watchdog | 看门狗 |
| slow-receiver symptom | 慢接收者症状 |
| QPC (Queue Pair Context) | 队列对上下文 |
| MTT (Memory Translation Table) | 内存转换表 |
| dynamic buffer sharing | 动态缓冲共享 |
| HugePages | 大页内存 |
| ECMP | 等价多路径路由 |
| DCQCN | 数据中心量化拥塞通知 |
| Pingmesh | 全网主动时延测量系统 |
| intra-DC communication | 数据中心内部通信 |
| tail latency (P99/P99.9) | 尾延迟 |
