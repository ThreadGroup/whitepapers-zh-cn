2015 年 7 月 13 日

Thread 技术白皮书仅供参考。

Thread Group 的成员可以获取完整的技术规范。要加入 Thread Group 并获取权限，请访问以下链接：[http://threadgroup.org/Join.aspx](http://threadgroup.org/Join.aspx)。

如果你已经是 Thread Group 的成员，则可以在 Thread Group 的门户网站中获取完整的技术规范：[http://portal.threadgroup.org](http://portal.threadgroup.org)。

如果对该白皮书由任何疑问或建议，请发送邮件至 [help@threadgroup.org](help@threadgroup.org)。

This document and the information contained herein is provided on an “AS IS” basis and THE THREAD GROUP DISCLAIMS ALL WARRANTIES EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO (A) ANY WARRANTY THAT THE USE OF THE INFORMATION HEREIN WILL NOT INFRINGE ANY RIGHTS OF THIRD PARTIES (INCLUDING WITHOUT LIMITATION ANY INTELLECTUAL PROPERTY RIGHTS INCLUDING PATENT, COPYRIGHT OR TRADEMARK RIGHTS) OR (B) ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE OR NONINFRINGEMENT.

IN NO EVENT WILL THE THREAD GROUP BE LIABLE FOR ANY LOSS OF PROFITS, LOSS OF BUSINESS, LOSS OF USE OF DATA, INTERRUPTION OF BUSINESS, OR FOR ANY OTHER DIRECT, INDIRECT, SPECIAL OR EXEMPLARY, INCIDENTAL, PUNITIVE OR CONSEQUENTIAL DAMAGES OF ANY KIND, IN CONTRACT OR IN TORT, IN CONNECTION WITH THIS DOCUMENT OR THE INFORMATION CONTAINED HEREIN, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH LOSS OR DAMAGE. 

Copyright © 2015 Thread Group, Inc. All rights reserved.

---

# Thread 协议栈基础 <!-- omit in toc -->

2015 年 7 月

修订历史

| 版本 | 日期 | 注解 |
| :-- | :-- | :-- |
| 1.0 | 2014 年 11 月 29 日 | 首次发布 |
| 2.0 | 2015 年 7 月 13 日 | 公开发布 |

# 目录 <!-- omit in toc -->

- [引言](#引言)
  - [特征概述](#特征概述)
  - [IEEE 802.15.4](#ieee-802154)
  - [无单点故障](#无单点故障)
- [设备类型](#设备类型)
  - [边界路由器](#边界路由器)
  - [路由器](#路由器)
  - [适任路由器的终端设备/潜在路由器](#适任路由器的终端设备潜在路由器)
  - [嗜睡终端设备](#嗜睡终端设备)
- [IP 协议栈基础](#ip-协议栈基础)
  - [寻址](#寻址)
  - [6LoWPAN](#6lowpan)
  - [ICMP](#icmp)
  - [UDP](#udp)
- [网络拓扑](#网络拓扑)
  - [网络地址和设备](#网络地址和设备)
  - [Mesh 网络](#mesh-网络)
- [路由与互联](#路由与互联)
  - [MLE 消息](#mle-消息)
  - [路由发现和修复](#路由发现和修复)
  - [路由](#路由)
  - [重试和确认](#重试和确认)
- [加入 Thread 网络](#加入-thread-网络)
  - [发现](#发现)
  - [Commissioning](#commissioning)
  - [附着](#附着)
  - [MLE 消息](#mle-消息-1)
  - [DHCPv6](#dhcpv6)
- [管理](#管理)
  - [ICMP](#icmp-1)
  - [设备管理](#设备管理)
- [持久数据](#持久数据)

# 引言

## 特征概述

Thread 是一个低成本、低功耗、安全可靠的无线设备通信标准。该协议栈是专为基于 IP 网络的互联家居应用设计的，并且可以支持各种应用层协议。

以下是 Thread 协议栈和网络的特征：

* 易于部署和维护：组建及维护网络的协议简单，允许系统自配置和路由自修复。
* 安全可靠：未经授权的设备无法加入到 Thread 网络，并且网络中的所有通信都是加密保护的。
* 规模可扩展：支持将小型家域网（数个设备）扩展成大型家域网（数百个设备）。网络层会根据预期的使用情况进行优化。
* 范围适当：结合 mesh 组网技术，提供足以覆盖常规家庭的家域网。在物理层采用扩频技术，提供良好的抗干扰能力。
* 无单点故障：即使个别设备出现故障或丢失，也不会影响网络的安全性和可靠性。
* 低功耗：如果使用 AA 电池供电，并配以合适的睡眠周期，设备通常可以工作数年。

图 1 展示了 Thread 协议栈的概览。

![Figure 1. Overview of Thread Stack](./figure/figure1.png)

## IEEE 802.15.4

Thread 基于 [IEEE 802.15.4](https://standards.ieee.org/standard/802_15_4-2006.html) PHY（Physical，物理）和 MAC（Media Access Control，媒体访问控制）层，并在 2.4GHz 频段下以 250kbps 速率工作。Thread 使用 IEEE 802.15.4-2006 版本的规范。

802.15.4 的 MAC 层用于基本的消息处理和拥塞控制。该 MAC 层包括一个用于设备监听空闲信道的 CSMA（Carrier Sense Multiple Access，载波侦听多路访问）机制，以及一个用于处理相邻设备间可靠通信的重试和消息确认的链路层。消息使用 MAC 层的加密和完整性保护机制，所使用的密钥是在网络链路建立和配置中确定的。这些机制确保了网络层能够提供可靠的端到端通信。

## 无单点故障

在 Thread 网络系统中，设备不会表现出单点故障。虽然系统中有许多设备执行特殊的功能，但是 Thread 协议栈的设计使它们可以在不影响 Thread 网络内通信的情况下进行替换。例如，一个嗜睡（sleepy）子系需要一个父系来进行通信，而该父系发生了单点故障。由于其父系不可用，嗜睡设备将选择另一个父系，而这个转变的过程对用户是透明的。

虽然系统设计成无单点故障，但在某些拓扑下，将会有个别设备没有后备能力。例如，在只有单个网关的系统中，如果网关断电，则没有备用的网关可供切换。

路由器或边界路由器可以在 Thread 网络中担任领导者（Leader）角色。该角色需要为网络的某些功能作出决策。例如，领导者需要分配路由器地址并允许新的路由器请求。领导者是选举出来的，如果领导者发生故障，则由另一个路由器或边界路由器担任领导者角色。这种自操作确保了无单点故障。

# 设备类型

## 边界路由器

边界路由器（Border Router）是一种特殊的路由器，它提供从 802.15.4 网络到其他物理层上的邻近网络（例如，Wi-Fi 和以太网）的连接。边界路由器为 802.15.4 网络内的设备提供服务，包括用于离网（off-network）操作的路由服务。Thread 网络中可能有一个或多个边界路由器。

## 路由器

路由器（Router）为网络设备提供路由服务。路由器还为试图加入网络的设备提供加入和安全服务。路由器的接收器总是打开的。路由器可以降级其功能并成为潜在路由器（Router-eligible End Devices）。

## 适任路由器的终端设备/潜在路由器

REED（Router-eligible End Devices，适任路由器的终端设备/潜在路由器）具有成为路由器的能力，但由于网络拓扑或条件，这些设备暂时不充当路由器。这些设备通常不会为 Thread 网络中的其它设备转发消息或提供加入和安全服务。在必要的时候，Thread 网络会将 REED 升级为路由器，该升级过程无需用户交互。

## 嗜睡终端设备

嗜睡终端设备（Sleepy End devices）是主机设备（host device）。它们仅通过它们的父路由器进行通信，并且无法为其他设备转发消息。

# IP 协议栈基础

## 寻址

Thread 中的设备支持 [\[RFC 4291\]](https://www.ietf.org/rfc/rfc4291) 中指定的 IPv6 寻址架构。设备将具有一个或多个 ULA（Unique Local Address，唯一本地地址）或 GUA（Global Unicast Address，全局单播地址）。

启动网络的设备选择一个 /64 前缀，这个前缀会在整个 Thread 网络中使用。前缀是本地分配的 Global ID，通常称为 ULA 前缀 [\[RFC 4193\]](https://www.ietf.org/rfc/rfc4193)，还可以称为 mesh local ULA 前缀。Thread 网络还可以具有一个或多个边界路由器，每个边界路由器都可以具有/没有一个用于生成额外 GUA 的前缀。Thread 网络中的设备使用其扩展的 MAC 地址来派生出其接口标识符（如 [\[RFC 4944\]](https://www.ietf.org/rfc/rfc4944) 第 6 部分中所定义的），并从中使用众所周知的本地前缀 FE80::0/64 来配置一个链路本地 IPv6 地址（如 [\[RFC 4862\]](https://www.ietf.org/rfc/rfc4862) 和 [\[RFC 4944\]](https://www.ietf.org/rfc/rfc4944) 所述）。

这些设备还支持合适的多播地址。这包括链路本地全节点多播，链路本地全路由多播和领域本地（realm-local）多播。

如 [\[IEEE802154\]](https://standards.ieee.org/standard/802_15_4-2006.html) 中所述，每个加入 Thread 网络的设备都会分配到一个 16-bit 短地址。对于路由器，使用地址字段中的高位分配此地址且低位设置为 0，以表示路由器地址。然后，父系使用其高位和合适的低位为其子系分配一个 16-bit 短地址。这样，Thread 网络中的其他设备就可以简单地通过使用其地址字段的高位来推断其子系的路由位置。

图 2 展示了 Thread 的短地址。

![Figure 2. Thread Short Address](./figure/figure2.png)

## 6LoWPAN

所有设备均使用 [\[RFC 4944\]](https://www.ietf.org/rfc/rfc4944) 和 [\[RFC 6282\]](https://www.ietf.org/rfc/rfc6282) 中定义的 6LoWPAN。

在 Thread 网络内使用报头压缩（header compression），并且在设备传输消息中尽可能地压缩 IPv6 报头，以最小化传输包的大小。

Thread 支持 mesh 报头，以便更有效地压缩 mesh 内的消息和支持链路层转发（如 [路由与互联](#路由与互联) 部分所述）。Mesh 报头还允许消息的端到端分片（fragmentation），而非 [\[RFC 4944\]](https://www.ietf.org/rfc/rfc4944) 中指定的逐跳分片。Thread 协议栈使用 route-over 配置。

Thread 设备不支持 [\[RFC 6775\]](https://www.ietf.org/rfc/rfc6775) 中指定的邻居发现（neighbor discovery），因为使用了 DHCPv6 来为路由器分配地址。终端设备和 REED 由其父路由器分配短地址。然后，使用该短地址来配置 mesh local ULA，该 ULA 用于网络内部（intra-network）通信。

有关 6LoWPAN 使用和配置的详情，请参见 “**Thread Usage of 6LoWPAN**” 白皮书。Thread 规范的第 3 章详细介绍了特定 6LoWPAN 配置的使用。

## ICMP

Thread 设备支持 ICMPv6（Internet Control Message Protocol version 6，互联网控制消息协议第六版）协议 [\[RFC 4443\]](https://www.ietf.org/rfc/rfc4443) 和 ICMPv6 错误消息，以及 echo 请求和 echo 应答消息。

## UDP

Thread 协议栈支持 [\[RFC 768\]](https://www.ietf.org/rfc/rfc768) 中定义的 UDP（User Datagram Protocol，用户数据报协议），以用于设备间的消息传送。

# 网络拓扑

## 网络地址和设备

Thread 协议栈支持 Thread 网络中所有路由器之间的 full mesh 互联。

拓扑的实际情况根据 Thread 网络中的路由器数量而决定。如果只有一个路由器或边界路由器，则形成具有单个路由器的基本星型拓扑。如果有多个路由器，则会自动形成 mesh 拓扑。图 3 展示了 Thread 网络的基本拓扑和设备类型。

![Figure 3. Basic Thread Network Topology and Devices](./figure/figure3.jpg)

## Mesh 网络

Mesh 网络允许无线电转发其他无线电的消息，从而使无线电系统更加可靠。例如，如果一个节点不能直接向另一节点发送消息，则 mesh 网络通过一个或多个中间节点转发消息。如 [路由与互联](#路由与互联) 部分所述，Thread 网络的本质是所有路由器节点都会维护彼此间的路由与互联，因此，mesh 网络将不断地被维护和连接。一般情况下，Thread 网络中的活跃路由器限制为 32 个。但是，其将使用 64 个路由器地址来容许回收路由器地址。

在 mesh 网络中，嗜睡终端设备或 REED 不会为其它设备路由。这些设备将消息发送到其父系（路由器）。父系路由器会为其子系处理路由操作。

# 路由与互联

Thread 网络通常最多有 32 个活跃路由器，活跃路由器为消息使用下一跳路由（基于设备路由表）。设备路由表由协议栈维护，以确保 Thread 网络中的所有路由器都具有与其他路由器的互联和最新路径。设备路由表使用 RIP（Routing Information Protocol，路由信息协议）算法（该算法来自 [\[RFC 1058\]](https://www.ietf.org/rfc/rfc1058) 和 [\[RFC 2080\]](https://www.ietf.org/rfc/rfc2080)，但不使用其特定的消息格式）。在 Thread 网络中，所有路由器使用 MLE（Mesh Link Establishment，Mesh 链路建立）来压缩格式以与其他路由器交换其路由成本。

> Note：从 IP 的角度来看，Thread 网络支持路由器和主机。主机要么是嗜睡终端设备，要么是 REED。

## MLE 消息

MLE 消息（参见 [\[draft-kelsey-intarea-mesh-link-establishment-06\]](https://datatracker.ietf.org/doc/draft-kelsey-intarea-mesh-link-establishment/)，在 Thread 规范的第 4 章消息链路建立中对 Thread 进行了扩展）用于建立和配置安全的无线电链路、检测相邻设备、并维护 Thread 网络中设备间的路由成本。MLE 消息通过单跳链路本地单播和路由器间多播来传输。

在拓扑和物理环境发生变化时，使用 MLE 消息来标识、配置和保护到相邻设备的链路。MLE 还用于分发跨 Thread 网络共享的配置值，如信道和 PAN（Personal Area Network，个域网）ID。这些消息可以使用 MPL（Multicast Protocol for Low power and Lossy Networks，用于低功耗和有损网络的多播协议）指定的简单泛洪（simple flooding）进行转发。（有关详情，请参阅 [\[draft-ietf-roll-trickle-mcast-09\]](https://tools.ietf.org/html/draft-ietf-roll-trickle-mcast-09)）。

在两个设备间建立路由成本时，MLE 消息还确保了非对称链路成本的考虑。非对称链路成本在 802.15.4 网络中很常见。为确保双向消息传递的可靠性，考虑双向链路的成本非常重要。

## 路由发现和修复

低功耗 802.15.4 网络通常使用请求式（On-demand）路由发现。然而，请求式路由发现的网络开销和带宽十分高昂，因为路由发现请求会泛洪网络。

在 Thread 网络中，所有路由器定期将包含链路成本信息的单跳 MLE 广告包交换到所有邻居路由器，并且将路径成本交换到 Thread 网络中的其他路由器。通过这些定期的本地更新，Thread 网络中的所有路由器都将具有到其他路由器的最新路径成本信息，因此无需请求式路由发现。如果路由不可用，路由器可以选择下一个最合适的路由来到达目的地。这种自愈路由机制使得路由器可以快速地检测到其他路由器的离开，并计算出最佳路径以维护与网络中其他设备的互联。

根据来自相邻设备的传入消息的链路成本来得出每个方向上的链路质量。此传入链路成本映射到一个从 0 到 3 的链路质量上。值 0 表示为未知成本。链路成本是对接收消息的 RSSI（Received Signal Strength Indicator，接收信号强度指标）的度量。

表 1 概述了链路质量和链路成本。

![Table 1. Link Quality and Link Cost](./table/table1.png)

图 4 展示了 Thread 网络上各种链接成本的示例。

![Figure 4. Examples of Various Link Costs on a Thread Network](./figure/figure4.jpg)

Thread 网络中任何其他节点的路径成本是到达该节点的链路成本的最小总和。即使在网络的无线电链路质量或拓扑发生变化时，路由器也会监控这些成本，并使用定期的 MLE 广告消息来通过 Thread 网络传播新成本。路由成本基于两个设备间的双向链路质量。

为了通过一个简单的示例进行说明，可想象一个具有共享安全材料的 pre-commissioned 网络，其中所有设备同时启动。每个路由器将定期向单跳邻居发送一个广告，该广告最初仅用成本作为填充。在内部，每个路由器将存储未在广告中发送的下一跳信息。

因为已知的唯一路由器是直接邻居，所以前几个广告的路径成本等于链路成本，如图 5 所示。

![Figure 5. Forming a Thread Network after Power Cycle](./figure/figure5.jpg)

但随后，路由器将开始听到来自邻居的广告，这些广告包含两跳或更多的其他路由器的成本，它们的表使用了多跳路径成本作为填充，然后传播到更远，最终直到网络中所有路由器之间都存在连通信息，如图 6 和图 7 所示。

![Figure 6. Thread Network Formation: Route Advertisements](./figure/figure6.jpg)

![Figure 7. Thread Network Formation: Multi-hop](./figure/figure7.jpg)

当路由器从邻居收到一个新的 MLE 广告时，它将已具有该设备的邻居表条目或者将添加一个条目。MLE 广告包含来自邻居的传入成本，以在路由器的邻居表中更新。MLE 广告还包含其他路由器的更新路由信息，并且该信息将在设备路由表中更新。

通过查看子系地址的高位来确定其父路由器地址，从而完成到子设备的路由。一旦设备知道父路由器，它就具有该设备的路径成本信息和下一跳路由信息。

活跃路由器的数量受限于单个 802.15.4 包中可包含的路由和成本信息的数量。目前该限制为 32 个路由器，但提供了 64 个活跃路由器地址，以允许路由器地址老化。

## 路由

设备使用 IP 来路由转发包。设备路由表使用路由器的 mesh local ULA 地址的压缩形式和合适的下一跳进行填充。

距离矢量路由（Distance vector routing）用于获取到 Thread 网络上的路由器地址的路由。在 Thread 网络上进行路由时，该 16-bit 地址的高 6 位定义了目标路由器的路由器地址。如果目标地址的低位为 0，则最终目的地为路由器。否则，目标路由器将负责根据 16-bit 目标地址的低位来转发到最终目标。

对于 Thread 网络外的路由，边界路由器将通知服务特定前缀的领导者，并且该信息在 MLE 包内作为 Thread 网络数据被分发。该 Thread 网络数据包括：前缀数据（即前缀本身）、6LoWPAN 上下文、边界路由器和该前缀的 DHCPv6 服务器。如果设备要使用该前缀配置 IPv6 地址，则它将使用 SLAAC（Stateless Address Autoconfiguration，无状态地址自动配置）或联系相应的 DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）服务器。Thread 网络数据还包括一个路由服务器列表，其是默认边界路由器的路由器地址。

领导者需要决定 REED 的升级或路由器的降级。领导者还负责分配和管理路由器地址。然而，此领导者中包含的所有信息都存在于其他路由器中，如果该领导者发生故障，则另一个路由器将自动选举并在没有用户干预的情况下接管成为领导者。

## 重试和确认

虽然在 Thread 协议栈中使用 UDP，但仍然需要可靠的消息交付。可靠的消息交付使用一系列轻量级机制完成，如下所示：

* MAC 级重试：每个设备使用来自下一跳的 MAC 确认，并且如果未收到 MAC ACK 消息，则将在 MAC 层重试消息。
* 应用级重试：应用级可以进行确定（如果消息的可靠性是一个关键参数），并且可以在必要时实现应用级的重试机制。

# 加入 Thread 网络

加入设备在参与 Thread 网络之前必须经历各个阶段：

* 发现（Discovery）
* Commissioning
* 附着（Attaching）

在 Thread 网络中，所有的加入都是用户发起的。一旦加入，设备将完全参与 Thread 网络，并可与 Thread 网络内外的其他设备和服务交换应用层信息。

## 发现

加入设备必须发现 Thread 网络并与路由器建立联系以进行 commissioning。加入设备扫描所有信道，在每个信道上发出一个信标请求，并等待信标响应。信标（beacon）包含一个有效载荷，该有效载荷包括网络 SSID（Service Set Identifier，服务集标识符）和一个许可加入信标（如果 Thread 网络接受该新成员）。一旦设备发现了 Thread 网络，它将使用 MLE 消息确立一个相邻的路由器，然后通过该路由器可以执行 commissioning。

如果设备已经获得 commissioning 信息，则无需发现，因为它已经有足够的信息来直接附着到 Thread 网络。

## Commissioning

Thread 提供两种 commissioning 方法：

* 使用带外（out-of-band）方法将 commissioning 信息直接配置到设备上。commissioning 信息允许加入设备在其被引入网络后立即加入到合适的 Thread 网络。
* 在加入设备和智能手机、平板电脑或网站上的 commissioning 应用之间建立 commissioning 会话。Commissioning 会话安全地将 commissioning 信息传递给加入设备，以允许其在完成 commissioning 会话后附着到合适的 Thread 网络。

> Note：一般情况下，Thread 网络不使用仅通过信标有效载荷中的许可加入标志进行加入的 802.15.4 方法。此方法最常用于没有用户界面或带外通道的设备的按钮类型加入。在存在多个可用网络的情况下，此方法可能存在设备转向问题，并且可能存在安全问题。

## 附着

带有 commissioning 信息的加入设备与父路由器联系，然后通过交换 MLE 链路配置消息来通过父路由器附着到 Thread 网络。设备将作为终端设备或 REED 附着到 Thread 网络，并且父路由器会为其分配一个 16-bit 短地址，如图 8 所示。

![Figure 8. Attaching to a Thread Network with a Known Key](./figure/figure8.jpg)

一旦 REED 已附着，它可能会发出一个地址请求以成为路由器，然后领导者会为其分配一个路由器地址。

## MLE 消息

一旦设备附着到 Thread 网络，它就需要各种信息来维护其在网络中的参与。MLE 提供在整个网络中分发网络数据的服务，并在邻居间交换链路成本和安全帧计数器。

MLE 消息分发或交换以下信息：

* 相邻设备的 16-bit 短地址和 64-bit EUI 64 长地址
* 设备能力信息，包括其是否为嗜睡终端设备以及嗜睡主机设备的睡眠周期
* 邻居链路成本（如果是路由器）
* 设备间的安全材料和帧计数器
* 到 Thread 网络中所有其他路由器的路由成本
* 更新网络数据，例如 MAC 中使用的信道，PAN ID 和信标有效载荷参数

> Note：除了加入设备获得所需安全材料前的发现期间，MLE 消息都是加密的。

## DHCPv6

DHCPv6 [\[RFC 3315\]](https://www.ietf.org/rfc/rfc3315) 是一种基于 UDP 的 client-server 协议，用于管理网络中设备的配置。DHCPv6 使用 UDP 从 DHCP 服务器中请求数据。

边界路由器上的 DHCPv6 服务用于配置：

* 网络地址（Network addresses）
* 设备要求的多播地址（Multicast addresses required by devices）
* 主机名服务（Hostname services）

# 管理

## ICMP

所有设备都支持 ICMPv6 错误消息，以及 echo 请求和 echo 应答消息。

## 设备管理

设备上的应用层可以访问一组设备管理和诊断信息，这些信息可以在本地使用，或收集并发送到其他管理设备。

Thread 在 802.15.4 MAC 层上使用的信息包括：

* EUI 64 地址
* 16-bit 短地址
* 能力信息（Capability information）
* PAN ID
* 发送和接收的包（Packets sent and received）
* 发送或接收时丢弃的包（Packets dropped on transmit or receive）
* 安全错误（Security errors）
* MAC 重试次数（Number of MAC retries）

Thread 在网络层上使用的信息包括：

* IPv6 地址丢失（IPv6 address lost）
* 邻居表（Neighbor table）
* 子系表（Child table）
* 路由表（Routing table）

# 持久数据

在现场操作的设备可能由于各种原因而被意外或故意重置。已被重置的设备需要重启网络操作（无需用户干预）。为此，需要将一组信息存储在非易失性存储器中。这包括：

* 网络信息（如 PAN ID）
* 安全材料（使用的每个密钥）
* 来自网络的寻址信息以形成设备 IPv6 地址
