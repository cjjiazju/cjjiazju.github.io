---
title: "论文精读解析：LoRDMA: A New Low-Rate DoS Attack in RDMA Networks"
date: 2026-03-17
description: "论文精读解析：LoRDMA: A New Low-Rate DoS Attack in RDMA Networks"
tags: ["rdma", "paper", "security", "Collie"]
showToc: true
TocOpen: true
draft: false
---
# 论文精读解析：LoRDMA: A New Low-Rate DoS Attack in RDMA Networks

> **来源**：Network and Distributed System Security (NDSS) Symposium 2024, San Diego, CA, USA
> **作者**：Shicheng Wang, Menghao Zhang, Yuying Du, Ziteng Chen, Zhiliang Wang, Mingwei Xu, Renjie Xie, Jiahai Yang（清华大学、北航、信息工程大学、东南大学）
> **解析格式**：标题解读 → 原文（关键词标注）→ 生词解析 → 参考注文

---

## 标题解读

**LoRDMA: A New Low-Rate DoS Attack in RDMA Networks**
LoRDMA：RDMA网络中的一种新型低速率DoS攻击

- **Low-Rate DoS** n. 低速率拒绝服务攻击（以低平均流量但高峰值突发的方式触发服务性能下降的攻击）；
  1. Low-rate DoS attacks are harder to detect than traditional flood attacks. 低速率DoS攻击比传统洪泛攻击更难被检测。

- **LoRDMA** n. 本文提出的攻击名称（Low-Rate RDMA DoS的缩写）；
  1. LoRDMA exploits the interaction between PFC and DCQCN to degrade victim flows. LoRDMA利用PFC和DCQCN之间的交互来降低受害者流量的性能。

---

## 段一：Abstract（摘要）

**原文（关键词已标注）：**

Existing RDMA security studies mainly focus on the security of RDMA systems, and the security of the coupled traffic control mechanisms (represented by 【PFC】 and 【DCQCN】) in RDMA networks is largely overlooked. Through extensive experiments and analysis, we demonstrate that concurrent short-duration 【bursts】 can cause drastic performance loss on flows across multiple hops via the 【interaction between PFC and DCQCN】. We propose the 【LoRDMA attack】, a low-rate DoS attack against RDMA traffic control mechanisms. By monitoring 【RTT】 as the feedback signal, LoRDMA can adaptively 1) coordinate the bots to different target switch ports to cover more victim flows efficiently; 2) schedule the burst parameters to cause significant performance loss efficiently. The results show that compared to existing attacks, the LoRDMA attack achieves higher victim flow coverage and performance loss with much lower attack traffic and detectability. The communication performance of typical distributed machine learning training applications (NCCL Tests) can be degraded from 18.23% to 56.12% under the LoRDMA attack.

**生词解析：**

- **DCQCN (Datacenter Quantized Congestion Notification)** n. 数据中心量化拥塞通知（NVIDIA RNIC默认的端到端拥塞控制算法，基于ECN和AIMD机制）；
  1. DCQCN uses ECN marks to detect congestion and adjusts flow rates using AIMD. DCQCN使用ECN标记检测拥塞，并使用AIMD调整流速。

- **burst** n. 突发流量（短时间内发送大量数据的流量模式，峰值速率远高于平均速率）；
  1. Short-duration line-rate bursts can overwhelm switch port queues in microseconds. 短时间的线速突发流量可在微秒内使交换机端口队列溢出。

- **interaction between PFC and DCQCN** n. PFC与DCQCN的交互（PFC的逐跳传播可误导DCQCN的拥塞检测，导致非共链路的流量被错误减速）；
  1. The PFC-DCQCN interaction can spread congestion to flows with no direct link sharing. PFC-DCQCN交互可将拥塞扩散至没有直接链路共享的流量。

- **RTT (Round-Trip Time)** n. 往返时延（数据包从发送到接收确认的总时间，反映网络拥塞状态）；
  1. RTT is linearly correlated with queue length and serves as a congestion indicator. RTT与队列长度线性相关，可作为拥塞指标。

- **NCCL Tests** n. NCCL测试（NVIDIA集合通信库NCCL的标准测试套件，用于评估GPU间集合通信性能）；
  1. NCCL Tests measure AllReduce, AllGather, and other collective communication primitives. NCCL测试测量AllReduce、AllGather等集合通信原语的性能。

**参考注文：**

本文填补了RDMA安全研究的一个空白：此前工作聚焦于RDMA系统安全（如内存访问控制），而忽视了底层流量控制机制（PFC+DCQCN）的安全性。LoRDMA的核心发现是：精心构造的短时突发流量不仅影响与之直接竞争链路的流量（直接受害者），还能通过PFC传播错误地使完全不共享链路的流量（间接受害者）遭受DCQCN速率削减。这种"以小博大"的攻击以低于2%的网络主机作为僵尸机，即可对近100%的流量造成显著性能损失。

---

## 段二：Section 2 — Background（背景）

**原文（关键词已标注）：**

**PFC机制：**

In RoCEv2, the hop-by-hop flow control mechanism, PFC, is deployed to guarantee the loss-free property. When the ingress queue length exceeds a certain threshold (X-OFF), the switch sends a "PAUSE" frame to its upstream entity to stop packet transmission. And the transmission restarts with a "RESUME" frame when the queue drains below another threshold (X-ON).

Despite zero packet loss, PFC results in many performance problems due to congestion spreading, such as 【head-of-line blocking (HLB)】, 【unfairness (victim flows)】, PFC storm, and even PFC deadlock.

**DCQCN拥塞控制：**

DCQCN detects congestion based on the egress queue length. The switch (Congestion Point) monitors egress queues and marks packets with 【ECN (Explicit Congestion Notification)】 according to the 【RED (Random Early Detection)】 algorithm. The receiver (Notification Point) then responds to ECN-marked packets with 【congestion notification packets (CNPs)】. After receiving CNPs, the sender runs an 【AIMD rate adjustment mechanism】, based on which the sending rate can be cut rapidly and recovered in a more moderate manner. Note that DCQCN has no slow start phase like TCP. When a flow starts, it sends at full line rate.

**生词解析：**

- **head-of-line blocking (HLB)** n. 队头阻塞（PFC暂停整个优先级队列，导致低优先级或无关流量也被阻塞的现象）；
  1. HLB caused by PFC affects flows that are not involved in the original congestion. PFC导致的队头阻塞会影响与原始拥塞无关的流量。

- **ECN (Explicit Congestion Notification)** n. 显式拥塞通知（交换机在数据包中标记拥塞信号的机制，不丢包）；
  1. ECN allows switches to signal congestion before queues overflow. ECN允许交换机在队列溢出前发出拥塞信号。

- **RED (Random Early Detection)** n. 随机早期检测（根据队列长度以概率方式标记或丢弃数据包的主动队列管理算法）；
  1. RED marks packets with ECN when queue length exceeds a threshold. 当队列长度超过阈值时，RED用ECN标记数据包。

- **CNP (Congestion Notification Packet)** n. 拥塞通知包（接收方发回给发送方，通知其降低发送速率的控制包）；
  1. Receiving a CNP triggers the sender to reduce its RDMA transmission rate. 收到CNP会触发发送方降低其RDMA传输速率。

- **AIMD (Additive Increase/Multiplicative Decrease)** n. 加性增加/乘性减少（速率恢复时缓慢线性增加、拥塞时快速乘法降低的速率调整策略）；
  1. AIMD's slow recovery means one burst can suppress victim flow rate for tens of milliseconds. AIMD的缓慢恢复意味着一次突发可以将受害者流量速率抑制数十毫秒。

- **Congestion Point / Notification Point / Reaction Point** n. 拥塞点/通知点/反应点（DCQCN中的三个角色：检测拥塞的交换机、发送CNP的接收方、调整速率的发送方）；
  1. The switch as Congestion Point marks packets; the receiver as Notification Point sends CNPs. 交换机作为拥塞点标记数据包；接收方作为通知点发送CNP。

**参考注文：**

PFC和DCQCN原本是互补机制：PFC防止丢包（逐跳），DCQCN减轻PFC引起的副作用（端到端）。但两者的交互产生了安全漏洞：PFC的传播速度（微秒级逐跳）远快于DCQCN的响应速度（端到端，几十微秒），这意味着攻击者只需造成短暂的局部拥塞，PFC就会在DCQCN来得及反应之前将影响扩散到更广范围。

---

## 段三：Section 3 — Motivating Measurement（动机测量）

**原文（关键词已标注）：**

**Two types of victim flows:**

(1) **Direct victim flows (F2, F3)**: F2 and F3 are directly "cut off" by the bursts because of the egress queue congestion in SW5.P4. The packets of F2 and F3 get 【ECN marked】, making DCQCN decrease their rate consequently. It takes a longer time for them to recover to the original rate because of the 【AIMD property】.

(2) **Indirect victim flows (F1, F4)**: F1 and F4, surprisingly, also experience performance degradation despite having 【no link sharing】 with the bursts. When the bursts overwhelm SW5.P4, a large number of packets in F2 and F3 accumulate quickly in the ingress ports. Their queue length rapidly exceeds the PFC X-OFF threshold, generates PFC PAUSE frames to the upstream switch ports (SW3.P2 and SW4.P2), and stops their packet transmission. PFC dominates the control system 【transiently】 and spreads the congestion upstream. Consequently, the upstream egress queues quickly build up and results in an extra rate cut on F1 and F4 by DCQCN. In other words, the congestion spread by PFC 【misleads】 the congestion detection of DCQCN.

**Performance loss properties:**

- **Direct victim flows**: Rate cut happens in microseconds; AIMD recovery takes ~60ms. A 1ms burst can suppress flow rate to 0 for 60ms.
- **Indirect victim flows**: P Li converges to an upper bound when τ is long — longer duration adds no extra damage. P Li upper bound increases with δ but with diminishing returns above 500Gbps.

**生词解析：**

- **direct victim flow** n. 直接受害流（流量路径经过被突发堵塞的交换机端口的流量）；
  1. Direct victim flows are cut off by ECN marks triggered by queue congestion. 直接受害流因队列拥塞触发的ECN标记而被削减速率。

- **indirect victim flow** n. 间接受害流（流量路径不经过被堵塞端口但通过PFC传播受到影响的流量）；
  1. Indirect victim flows are degraded without any direct queue contention with attack traffic. 间接受害流在与攻击流量没有直接队列竞争的情况下遭受性能降级。

- **performance loss factor (ΔR)** n. 性能损失因子（原始流速R₀与受攻击后最低流速Rlo之差，衡量攻击的最大影响程度）；
  1. The performance loss factor quantifies how much a flow's rate drops under attack. 性能损失因子量化了流量在攻击下速率下降了多少。

- **performance loss (PL)** n. 性能损失（流量在攻击期间损失的总带宽，即速率-时间图中阴影面积）；
  1. Performance loss is defined as the integral of rate deficit over the burst period. 性能损失被定义为突发周期内速率亏损的积分。

- **AIMD recovery** n. AIMD恢复（DCQCN速率从0加性增加回到原始值所需的时间，通常约60ms）；
  1. AIMD's slow additive recovery allows a 1ms burst to suppress flows for 60ms. AIMD缓慢的加性恢复使1毫秒的突发可以抑制流量达60毫秒。

- **mislead** v. 误导（PFC扩散的拥塞使DCQCN误认为非攻击链路上存在真实拥塞）；
  1. PFC spread misleads DCQCN into cutting rates of flows on unaffected links. PFC扩散误导DCQCN削减未受影响链路上的流量速率。

- **diminishing return** n. 边际效益递减（增加投入带来的额外收益越来越少的现象）；
  1. Adding more bots beyond 5 shows diminishing returns on indirect victim flow degradation. 超过5个僵尸机后，增加更多僵尸机在间接受害流降级上显示出边际效益递减。

**参考注文：**

本节是LoRDMA的核心理论发现。通过精心设计的ns-3实验，作者揭示了一个此前未被注意的漏洞：短时突发→PFC传播→DCQCN误判的三步链条，使攻击影响能够越过拓扑边界，影响完全不共享链路的流量。关键数值：2个僵尸机的200Gbps突发即可将直接受害流速率降至0；间接受害流的损失在突发持续约1ms后收敛，且受益于更多僵尸机但边际效益递减。这些特性为后续构建高效低成本攻击奠定了基础。

---

## 段四：Section 3.4 — Vulnerabilities in RDMA Traffic Control（流量控制漏洞总结）

**原文（关键词已标注）：**

**Vulnerability 1: Wide congestion spread at low attack cost.**

By carefully congesting certain switch port queues using bursts from multiple bots, the attackers can spread congestion broadly, and consequently mislead DCQCN to cut off victim flows in an 【indirect way】. The attackers can directly congest fewer links by causing short-duration high link load using bursts, and exploit PFC to indirectly shut down more links while keeping their load low. They can therefore cover more target flows with fewer 【high-load congested links】 and lower 【link correlation between bursts and victim flows】 — both of which are important footprints in security defense.

**Vulnerability 2: Short-duration bursts cause long-term flow degradation at low detectability.**

A burst lasting for <1ms can significantly decrease the victim flow rate to nearly 0. However, due to the 【AIMD property】 of DCQCN rate adjustment, it takes ~60ms to recover to the original rate. The average burst rate for each bot is only 【1.6Gbps】 (1ms burst every 60ms period), while the average victim flow rate is 50Gbps, which is only 50% of the original rate.

Short-duration (~1ms) bursts with low average rate (~1Gbps) are difficult to capture for the prevalent 【second-level network monitoring tools】 currently. And the bursts are also difficult to be noticed at end-host, because these unexpected traffic is immediately dropped by RNICs without noticing upper-level applications.

**生词解析：**

- **indirect congestion spread** n. 间接拥塞扩散（通过PFC传播使无直接竞争关系的链路也受到拥塞影响的现象）；
  1. Indirect congestion spread allows attacking fewer links while affecting more flows. 间接拥塞扩散允许攻击更少链路同时影响更多流量。

- **link correlation** n. 链路相关性（攻击流量与受害流量之间共同经过的链路数量，相关性低则攻击更隐蔽）；
  1. Low link correlation between attack and victim traffic makes detection harder. 攻击与受害流量之间的低链路相关性使检测更难。

- **second-level monitoring tool** n. 秒级监控工具（以秒为粒度采集和分析网络指标的监控系统，无法捕获毫秒级突发）；
  1. Second-level monitoring tools miss millisecond-duration bursts used in LoRDMA. 秒级监控工具无法捕获LoRDMA中使用的毫秒级突发。

- **detectability** n. 可检测性（攻击行为被防御系统发现的难易程度，攻击者希望最小化此值）；
  1. LoRDMA's low average burst rate significantly reduces its detectability. LoRDMA的低平均突发速率显著降低了其可检测性。

- **average burst rate** n. 平均突发速率（在整个攻击周期（含静默期）内，攻击流量的平均带宽）；
  1. An average burst rate of 1.6Gbps can suppress a 100Gbps flow to 50% with LoRDMA. 1.6Gbps的平均突发速率可以通过LoRDMA将100Gbps的流量抑制到50%。

**参考注文：**

两个核心漏洞共同构成了LoRDMA的攻击价值主张：第一，通过利用间接受害效应，攻击者只需拥塞少数端口就能影响大量流量，降低了被检测到的"直接证据"；第二，AIMD的慢恢复特性使极短的突发（<1ms）产生几十倍时间跨度的性能影响，使得攻击的平均流量极低（~1.6Gbps vs受害流量100Gbps），现有秒级监控系统完全无法捕获。

---

## 段五：Section 4 — The LoRDMA Attack（攻击设计）

**原文（关键词已标注）：**

**A. Threat Model（威胁模型）**

We assume that the network infrastructure can be trusted, but there may be potential malicious hosts which seek to degrade the service performance of the entire network. Attackers can craft 【line-rate burst】 traffic in multiple ways: (1) continuously generating line-rate 【mice flows】 (flow size ≤ BDP) shorter than 1 RTT to avoid DCQCN rate cutting; (2) using 【IBV_QPT_RAW_PACKET】, a raw Ethernet programming feature enabling high-performance kernel-bypass traffic generation.

**C. Key Observation & Attack Overview（核心观察）**

RTT, which has been proven to present a linear mapping with the queue length, can well reflect the convergence of the burst impact. By monitoring RTT, the attackers can qualitatively infer whether the attack impact converges under the current attack pattern and adaptively adjust the burst parameters.

Traditional probing tools (ping, traceroute) fail to get the accurate RTT of RDMA communication. We find a side channel: when using 【RDMA CM (Connection Manager)】, a host replies with a 【ConnectReject】 packet upon receiving a ConnectRequest from an unknown host. Both packets are in RoCEv2 protocol and can reflect the RTT of the RDMA communication.

**D. Coordination（协调过程）**

The goal of coordination: 1) select the ports which can cover more flows directly and indirectly, and 2) carefully set the number of bots on each target port. A heuristic function H(F,α)(p) = α|Fi(p)| + (1−α)|Fd(p)| describes the weighted sum of covered direct and indirect victim flow numbers. The attackers stop adding bots when range⟨RTTi⟩ ≈ range⟨RTTd⟩, indicating ΔRi has converged.

**E. Schedule（调度过程）**

The schedule procedure aims to obtain an efficient burst duration τ with high performance loss. The attackers gradually reduce the burst duration τ using a 【bisection method】 while monitoring the RTT₁ sequence. The ⟨RTT₁⟩ during burst shows a "peak pattern" followed by return to baseline. The attackers trim the lower RTT₁ sub-sequence without reducing the attack impact.

**生词解析：**

- **line-rate burst** n. 线速突发（以网络接口最大速率发送的短时高强度流量）；
  1. A line-rate burst at 100Gbps can fill a switch buffer in microseconds. 100Gbps的线速突发可在微秒内填满交换机缓冲区。

- **mice flow** n. 鼠流（数据量很小（≤带宽时延积BDP）、持续时间极短的流量）；
  1. Mice flows smaller than 1 RTT are not throttled by DCQCN's congestion control. 小于1个RTT的鼠流不受DCQCN拥塞控制的限制。

- **IBV_QPT_RAW_PACKET** n. 原始数据包QP类型（RDMA verbs API中允许直接构造以太网帧的高级功能）；
  1. IBV_QPT_RAW_PACKET enables kernel-bypass line-rate traffic generation. IBV_QPT_RAW_PACKET支持内核绕过的线速流量生成。

- **RDMA CM (Connection Manager)** n. RDMA连接管理器（管理RDMA连接建立过程的软件组件）；
  1. RDMA CM exchanges ConnectRequest/ConnectReject packets over RoCEv2. RDMA CM通过RoCEv2交换连接请求/拒绝数据包。

- **ConnectReject** n. 连接拒绝包（RDMA CM收到未知主机的连接请求时回复的RoCEv2控制包）；
  1. ConnectReject packets carry RTT information and can be used as probes. ConnectReject数据包携带RTT信息，可用作探针。

- **bisection method** n. 二分法（通过不断折半缩小搜索范围找到最优参数的算法）；
  1. The bisection method efficiently finds the minimum burst duration with sufficient attack impact. 二分法高效找到具有足够攻击影响的最小突发持续时间。

- **bot** n. 僵尸机（被攻击者控制、用于发起攻击的主机）；
  1. LoRDMA requires only ~2% of network hosts as bots to affect nearly all flows. LoRDMA只需约2%的网络主机作为僵尸机即可影响几乎所有流量。

**参考注文：**

LoRDMA攻击分两步：协调（决定僵尸机攻击哪些端口、投入多少）和调度（决定最优突发持续时间）。两步都依赖RTT作为反馈信号——这是本文的关键工程创新：通过构造RDMA CM连接请求并接收拒绝包，攻击者可以测量网络中任意主机的往返时延，而无需控制目标主机，从而自适应地调整攻击参数。协调的停止条件是间接受害流的RTT范围趋近于直接受害流的RTT范围（说明ΔRi已收敛）；调度用二分法在不影响攻击效果的前提下最小化突发持续时间以降低检测风险。

---

## 段六：Section 5 — Evaluation（评估）

**原文（关键词已标注）：**

**Coordination Performance:**

Compared with the Crossfire attack, the LoRDMA attack covers more victim flows (~10% higher on average) and causes higher performance loss factor ΔR. LoRDMA has fewer congested ports directly overwhelmed by the bursts (~50% on average), and lower queue contention (~20% on average) between the burst traffic and target flows — i.e., lower attack footprint.

**Impact on Real Applications (ns-3 Fat-tree, k=8):**

Very few bots (~2% in the network) can cause performance loss on nearly 100% flows and a mean performance degradation ratio of 8.11% to 52.7%. For cloud storage, the maximum co-flow completion time (CFCT) impact ratio can reach 251.6%.

**Real Cloud RDMA Cluster (Kuaishou Technology):**

All NCCL communication primitives suffer significant performance degradation, from 18.23% (AlltoAll) to 56.12% (AllGather). ToR1 switch generates a large number of PFC PAUSE frames upstream, while ToR3 switch receives numerous CNP packets destined downstream, validating the PFC-misleading-DCQCN insight.

**生词解析：**

- **Crossfire attack** n. Crossfire攻击（一种经典链路洪泛攻击，通过控制僵尸机选择性阻塞目标链路）；
  1. LoRDMA achieves higher coverage than Crossfire with lower direct queue contention. LoRDMA以更低的直接队列争用实现了比Crossfire更高的覆盖率。

- **attack footprint** n. 攻击足迹（攻击流量在网络中留下的可观测痕迹，足迹越小越难被检测）；
  1. LoRDMA's indirect attack strategy reduces its footprint compared to direct link flooding. LoRDMA的间接攻击策略与直接链路洪泛相比减少了其攻击足迹。

- **co-flow completion time (CFCT)** n. 共流完成时间（同一应用下多个相关流量全部完成传输的总时间）；
  1. Short flows suffer higher CFCT impact ratio as performance loss counts more against small transfers. 短流遭受更高的CFCT影响比，因为性能损失占小传输量的比例更高。

- **NCCL (NVIDIA Collective Communications Library)** n. NVIDIA集合通信库（深度学习分布式训练中实现GPU间AllReduce等集合操作的高性能库）；
  1. NCCL Tests measure GPU communication bandwidth for AllReduce and AllGather. NCCL测试测量AllReduce和AllGather的GPU通信带宽。

- **AllReduce / AllGather / ReduceScatter / AlltoAll** n. 集合通信原语（分布式训练中用于同步参数和梯度的基本操作类型）；
  1. AllReduce aggregates gradients across all workers in distributed deep learning. AllReduce在分布式深度学习中跨所有工作节点聚合梯度。

**参考注文：**

评估结果在三个层面验证了LoRDMA的有效性：ns-3模拟证明了理论分析的正确性；大规模Fat-tree仿真证明了2%僵尸机即可对几乎所有流量造成影响；在真实的快手云RDMA集群上，实际验证了18%-56%的机器学习通信性能下降，并通过网络诊断信息（PFC帧和CNP包的分布）直接验证了PFC-DCQCN交互漏洞的存在。

---

## 段七：Section 6 — Defense Schemes（防御方案）

**原文（关键词已标注）：**

**PFC-driven network anomaly detection.** Many DoS detection systems consider the number of congested queues and the major queue contention contributor as key metrics to locate the culprit attacker. However, such philosophy may be less effective due to congestion spreading caused by PFC, because the attack traffic may have a lower congested queue number and contention contribution. The key to defend the LoRDMA attack is to take the PFC pause frames spreading into the 【causality dependency construction】. By analyzing the causality of the spreading of PFC pause frames, the root cause of the congestion can be found.

**Fine-grained burst monitor.** The millisecond-level burst duration requires that the monitoring granularity should be at the corresponding level. Second-level monitoring granularity cannot pinpoint the 1ms burst. Besides, the 【line-rate start property of DCQCN】 makes the attack burst less outstanding in RDMA networks, rendering rate-based detection less effective.

**Network fabric level isolation.** The feasibility of the LoRDMA attack lies in the sharing of the network infrastructure between hosts. 【Network fabric isolation】 can theoretically eliminate the infrastructure sharing and provide bounded bandwidth guarantees for each tenant. However, currently fabric isolation mechanisms in RDMA networks are still lacking.

**生词解析：**

- **causality dependency** n. 因果依赖（PFC暂停帧的传播链路，帮助溯源拥塞根本原因的分析框架）；
  1. Causality dependency analysis traces PFC spreading to identify the attack source. 因果依赖分析追踪PFC扩散以识别攻击来源。

- **line-rate start** n. 线速启动（DCQCN默认新流以线速发送，不像TCP有慢启动阶段）；
  1. Line-rate start of RDMA flows makes distinguishing attack bursts from normal traffic harder. RDMA流量的线速启动使区分攻击突发和正常流量更加困难。

- **network fabric isolation** n. 网络结构隔离（在网络传输层面为不同租户提供带宽保证和流量隔离的机制）；
  1. Network fabric isolation would prevent attacker traffic from contending with victim queues. 网络结构隔离将防止攻击者流量与受害者队列竞争。

**参考注文：**

现有防御机制面临三重挑战：检测方面，LoRDMA的间接攻击特性使得传统基于直接队列拥塞的检测方法失效；监控方面，毫秒级突发超出了现有秒级工具的分辨率；从根本上，RDMA网络缺乏链路层隔离机制。作者提出的基于PFC因果传播分析的新型检测思路是最有前景的方向，但需要大量工程工作，被留作未来研究。

---

## 附录：核心术语速查表

| 英文术语 | 中文释义 |
|---------|---------|
| LoRDMA | 低速率RDMA DoS攻击 |
| Low-Rate DoS | 低速率拒绝服务攻击 |
| DCQCN | 数据中心量化拥塞通知 |
| PFC-DCQCN interaction | PFC与DCQCN交互 |
| direct victim flow | 直接受害流 |
| indirect victim flow | 间接受害流 |
| performance loss factor (ΔR) | 性能损失因子 |
| performance loss (PL) | 性能损失 |
| burst peak rate (δ) | 突发峰值速率 |
| burst duration (τ) | 突发持续时间 |
| burst period (T) | 突发周期 |
| AIMD | 加性增加/乘性减少 |
| ECN | 显式拥塞通知 |
| CNP | 拥塞通知包 |
| line-rate burst | 线速突发 |
| mice flow | 鼠流 |
| bot | 僵尸机 |
| RTT probing | RTT探测 |
| bisection method | 二分法 |
| RDMA CM | RDMA连接管理器 |
| ConnectReject | 连接拒绝包 |
| attack footprint | 攻击足迹 |
| coordination | 协调（选择目标端口） |
| schedule | 调度（优化突发参数） |
| NCCL | NVIDIA集合通信库 |
| AllReduce | 全规约集合通信操作 |
| Crossfire attack | Crossfire链路洪泛攻击 |
| causality dependency | 因果依赖分析 |
| co-flow completion time | 共流完成时间 |
| network fabric isolation | 网络结构隔离 |
