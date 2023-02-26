### 本章内容概述和教学目标
- IS-IS (Intermediate System to Intermediate System， 中间系统到中间系统）是一种链路状态路由协议，在服务提供商网络中被广泛应用。IS-IS 与 OSPF 在许多方面非常相似，例如运行 IS-IS 的直连设备之间会通过 Hello 报文发现彼此，然后建立邻居关系，并交互链路状态信息，这些链路状态信息表现为 LSP (Link-State Packet， 链路状态报文）。每一台运行 IS-IS 的设备都会产生 ISP，设备产生的 LSP 会被泛洪到网络中适当的范围，所有的设备都将自己产生的、以及网络中泛洪的LSP 存储在自己的 LSDB 中。IS-IS 设备基于自己的LSDB 采用 SPF (Shortest Path First， 最短路径优先）算法进行计算，最终得到 IS-IS 路由信息。另外，与OSPF 一样，IS-IS 也支持层次化的网络架构，支持 VLSM,支持手工路由汇总等功能。
- IS-IS 早期被 ISO (International Organization for Standardization，国际标准化组织）标准化时，是为OSI (Open Syster Interconnection， 开放式系统互联）协议栈服务的，它是为 CLNP (ConnectionLess Network Protocol，无连接网络协议）设计的动态路由协议。需注意的是 OSI 与TCP/IP 是两个不同的协议栈。到目前为止，本书所讨论的内容都是围绕 TCP/P 协议栈展开的，关于该协议栈相信大家已经非常熟悉了。我们可以简单地将 OSI 协议栈中的 CINP 理解为 TCP/IP 协议栈中的卫协议，两者实现的功能非常类似。最初的 IS-IS 是无法工作在TCP/IP 环境中的，随着 TCP/IP 风靡全球，IETF (InternetEngineering Task Force， Internet 工程任务组）对 IS-IS 进行了扩展，使得它能够同时支持 IP 路由，这种 IS-IS 被称为集成 IS-IS(Integrated IS-IS)。由于在当今的通信网络中，TCP/IP 己经成了绝对的主流协议栈，因此如今我们所讨论的 IS-IS 几乎都指的是集成 IS-IS。在本书的后续章节中，除非特别说明，否则IS-IS 指的就是集成 IS-IS。



- 本章学习目标:
<br>
<br>



### 4.1.1 常用术语
- 在正式学习IS-IS 之前，有几个术语需要大家提前熟悉。这些术语中，有许多是与OSI 协议栈相关的，这些术语被沿用下来。
  - ISO (International Organization for Standardization， 国际标准化组织): 这是一个全球性质的非政府组织，成立于 1946 年，从其名称可以看出该组织的使命，即在国际上促进各领域的标准化实现。ISO 的一个广为人们所知的成果便是ISO9000 质量体系，另外，大家非常熟悉的 OSI 参考模型也是 ISO 的杰作。
  - IS (Inter mediate System，中间系统): 指的是 OSI 中的路由器。 
  - IS-IS (Intermediate System to Intermediate System， 中间系统到中间系统): 用于在 IS 之间实现动态路由信息交互的协议。
  - CINP (Comnection-Less Network Protocol，无连接网络协议）: 这是 OSI 的无连接网络协议，它与 TCP/I P中的卫协议的功能类似。
  - LSP (Link-State Packet，链路状态报文)：这是 IS-IS 用于描述链路状态信息的关键数据，类似 OSPF 的 LSA。IS 将网络中的 LSP 搜集后装载到自己的 LSDB(Link State Database，链路状态数据库)中，然后基于这些 LSP 进行路由计算。- LSP 分为两种：
    - Level-1 LSP  
    - Level-2 LSP


