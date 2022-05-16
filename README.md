<font color="silver">翻译自：</font><br>
[https://docs.libp2p.io/concepts/](https://docs.libp2p.io/concepts/)<br>
[https://docs.libp2p.io/reference/glossary/](https://docs.libp2p.io/reference/glossary/)<br>

# 概念
libp2p 涵盖了很多领域，并且可能涉及不熟悉的概念和术语。本节将介绍 libp2p 中涉及的基本概念。
 
* [传输](#传输)
* [NAT穿透](#NAT穿透)
* [安全通信](#安全通信)
* [中继连接](#中继连接)
* [协议](#协议)
* [对等端点标识](#对等端点标识)
* [内容路由](#内容路由)
* [对等端点路由](#对等端点路由)
* [寻址](#寻址)
* [安全性考虑](#安全性考虑)
* [发布/订阅](#发布订阅)
* [流多路复用](#流多路复用)

>本节不完整，许多文章也是残缺的。为帮助填补空白，请参阅每篇文章中链接的问题，以添加您的意见，并帮助我们确定未完成工作的优先级。

>*”Peer” 视情况翻译为 “对等端点” 或 “节点”，“peer-to-peer” 翻译为 “p2p” -译注*

## 传输<a id='传输'></a>
当您的计算机连接到 Internet 上的计算机时，很有可能您正在使用 TCP/IP协议 发送比特位和字节，这是 Internet 协议（处理数据包的寻址传输与传输控制）非常成功的组合，它确保通过网络发送的数据以正确的顺序被完全的接收。

由于 TCP/IP协议 的普遍且得到很好的支持，它通常是网络应用程序的默认选择。在某些情况下，TCP协议 会增加太多开销，因此应用程序可能会使用 UDP协议，这是一种更简单的协议，但可靠性得不到保证。

虽然 TCP 和 UDP（以及 IP）是当今最常用的协议，但它们绝不是唯一的选择。可以替代的方案在较低级别的如可以发送原始以太网数据包或蓝牙帧的，和较高级别的基于 UDP 的 QUIC协议。

在 libp2p 中，我们把这些与比特位传输相关的称之为基础协议，而 libp2p 的核心要求之一是与传输无关。这意味着使用哪种传输协议由开发人员决定，实际上一个应用程序可以同时支持许多不同的传输。

### 监听和拨号
传输是根据两个核心操作定义的，监听和拨号。

监听意味着您可以使用传输实现（基于传输协议）所提供的方式，接受来自其他端点的传入连接。例如，Unix 平台上的 TCP 传输可以使用 bind 和 listen 系统调用，让操作系统将给定 TCP 端口上的流量路由到应用程序。

拨号是向监听端建立一个传出连接。就像监听一样，具体方式由实现决定，但 libp2p 实现中的每个传输都将共享相同的编程接口。

### 地址
在您可以对一个端点拨号并打开连接之前，您需要知道如何与它们取得联系。因为每种传输都可能需要自己的地址方案，libp2p 使用一种称为 `multiaddress` 或 `multiaddr`（*以下统称“多地址” -译注*） 的方案来约定并编码许多不同的地址。

[寻址](#寻址) 更详细地介绍了工作原理，但对多地址如何工作的简单概述有助于理解拨号和监听接口。

下面是 TCP/IP 传输的多地址示例：

```shell
/ip4/7.7.7.7/tcp/6543
```

这等效于更熟悉的 `7.7.7.7:6543` 结构，但它的优点是对协议有了更明确的描述。有了多地址，您一眼就能看出 `7.7.7.7` 地址属于 IPv4 协议，`6543` 属于 TCP。

有关更复杂的示例，请参阅 [对址](#寻址)。

拨号和监听都可以处理多地址。监听时提供您想监听的地址，拨号时提供您要拨号的地址。

对一个远程对等端点拨号时，多地址应包括您尝试联系的 `PeerId`（*对等端点地址 -译注*）。这让 libp2p 建立了一个可以防止冒充的安全信道。

包含 `PeerId` 的多地址示例：

```shell
/ip4/1.2.3.4/tcp/4321/p2p/QmcEPrat8ShnCph8WjkREzt5CPXF2RwhYxYBALDcLC1iV6
```

`/p2p/QmcEPrat8ShnCph8WjkREzt5CPXF2RwhYxYBALDcLC1iV6` 中使用其公钥的哈希【hash】来唯一的标识一个远程端点。有关更多信息，请参阅 [PeerId](#PeerId)。

>启用对等端点路由后，您可以仅使用对等端点的 `PeerId` 拨号，而无需事先知道它们的传输地址。

### 支持多种传输
libp2p 应用程序通常需要同时支持多种传输。例如，您可能希望您的服务可以通过 TCP 从长期运行的守护进程中保持连接，同时还接受来自 Web 浏览器中运行的 Websocket 连接。

负责管理传输的 libp2p 组件称为 “switch”（*该术语不翻译，下同 -译注*），它还协调 [协议协商](https://docs.libp2p.io/concepts/protocols/#protocol-negotiation)、[流多路复用](https://docs.libp2p.io/concepts/stream-multiplexing)、[建立安全通信](https://docs.libp2p.io/concepts/secure-comms/) 和其他形式的“连接升级”。

该 switch 为拨号和监听提供了一个单一的“入口点”，并使您的应用程序代码不必担心特定的传输和“连接堆栈”的其他部分。

>术语“swarm”（*该术语不翻译，下同 -译注*）以前用于指代现在称之为“switch”的东西，代码库中的某些地方仍然使用“swarm”术语。

## NAT穿透<a id='NAT穿透'></a>
互联网由无数网络组成，通过基础[传输协议](https://docs.libp2p.io/concepts/transport/)连接在一起，形成共享的地址空间。

当信息流在网络边界之间移动时，通常会发生一个称为网络地址转换的过程。网络地址转换 (NAT) 将地址从一个地址空间映射到另一个地址空间。

NAT 允许许多机器共享一个公网地址，这对于 IPv4 协议的持续运行至关重要，否则它无法通过其 32 位地址空间满足现代网络人口的需求。

例如，当我连接到我的家庭 wifi 时，我的计算机获得的 IPv4 地址为 10.0.1.15。这是为专用网络内部（*以下简称内网 -译注*）使用而保留的一系列 IP 地址的一部分。当我对一个公用网络（*以下简称公网 -译注*） IP 地址建立一个传出连接时，路由器会用它自己的公网 IP 地址替换我的内网 IP。当数据从另一端返回时，路由器将转换回内部地址。

虽然 NAT 通常对传出连接是透明的，但监听传入连接需要一些配置。路由器监听单个公网 IP 地址，但内部网络上的任意数量的机器都可以处理该请求。要处理请求，您的路由器必须配置为将某些流量发送到特定机器，通常是通过将一个或多个 TCP 或 UDP 端口从公网 IP 映射到内网。

虽然通常可以手动配置路由器，但并不是每个想要运行 p2p 应用程序或其他网络服务的人都有能力这样做。

我们希望 libp2p 应用程序可以在任何地方运行，而不仅仅是在数据中心或具有稳定公网 IP 地址的机器上。为了实现这一点，以下是目前 libp2p 中可用的 NAT 穿透的主要方法。

### 自动路由器配置
许多路由器支持端口转发的自动配置协议，最常见的是 [UPnP](https://en.wikipedia.org/wiki/Universal_Plug_and_Play) 或 [nat-pmp](https://en.wikipedia.org/wiki/NAT_Port_Mapping_Protocol)。

如果您的路由器支持其中一种协议，libp2p 将尝试自动配置端口映射，以允许它监听传入流量。如果网络和 libp2p 的实现都支持，这通常是最简单的选项。

>对自动 NAT 配置的支持因 libp2p 的实现而异。查看 [current implementation status](https://libp2p.io/implementations/#nat-traversal) 以获取详细信息。

### 打孔【Hole-punching】 (STUN)
当一台内网机器“拨出”并连接到公网地址时，路由器将把一个公共端口映射到内网 IP 地址以用于连接。在某些情况下，路由器还会接受该端口上的传入连接并将它们路由到相同的内网 IP。

当使用支持 IP 的传输时，libp2p 将尝试利用此行为，使用一个名为 `SO_REUSEPORT` 的套接字选项，使用相同的端口进行拨号和监听。

如果我们的对等端点处于有利的网络环境中，他们将能够建立一个传出连接，并“免费”获得一个可公开访问的监听端口，但他们可能永远不会知道这一点。不幸的是，拨号程序无法自行发现分配给连接的端口。

但是，外部的对等端点可以告诉我们他们观察到我们的地址。然后，我们可以获取该地址并将其通告给我们 [对等端点路由网络](https://docs.libp2p.io/concepts/peer-routing/) 中的其他对等端点，让他们知道在哪里可以找到我们。

基础的 [STUN](https://en.wikipedia.org/wiki/STUN)（NAT 会话穿透实用程序）协议是对等端点相互通知他们观察到的地址的基本前提，它[描述](https://tools.ietf.org/html/rfc3489)了一个客户端/服务器协议（为那些发现的公开可达的 IP 地址和端口的组合）

libp2p 的核心协议之一是 [identify 协议【识别协议】](https://github.com/libp2p/specs/pull/97)，它允许一个对等端点向另一个对等端点询问一些识别信息。对等端点之间发送[公钥](https://docs.libp2p.io/concepts/peer-id/)和其他一些有用信息来进行识别，也包括对等端点所观察到的地址集合。

这种外部发现机制的作用与 STUN 相同，但不需要一组“STUN 服务器”。

identify 协议允许一些对等端点跨 NAT 进行通信，否则这些 NAT 将无法穿透。

### 自动NAT<a id='自动NAT'></a>
虽然上述[identify 协议](https://github.com/libp2p/specs/pull/97)允许对等端点相互通知他们观察到的网络地址，但并非所有网络都允许在拨号的同一端口上进行传入连接。

再一次，其他对等端点可以帮助我们观察我们的情况，他们通过尝试他们所观察到的地址来对我们拨号。如果这成功了，我们可以依据其他对等端点能够对我们拨号的情况，就可以开始传播我们的监听地址。

一种称为 自动NAT 的 libp2p 协议允许对等端点（提供了 自动NAT 服务）请求回拨。

>自动NAT 目前通过 [go-libp2p-autonat](https://github.com/libp2p/go-libp2p-autonat) 在 go-libp2p 中实现*（rust 也已实现 -译注）*。

### 中继连接【Circuit Relay】(TURN)
在某些情况下，对等端点无法穿透他们的 NAT 以使其可以公开访问。

libp2p 提供了一个[中继协议](https://docs.libp2p.io/concepts/circuit-relay/)，允许对等端点通过一个的中间对等端点进行间接通信。

这与其他系统中的 [TURN 协议](https://tools.ietf.org/html/rfc5766) 具有类似的功能。

## 安全通信<a id='安全通信'></a>
这篇文章即将发布！

请[参阅此问题](https://github.com/libp2p/docs/issues/19)以跟踪进度并提出建议。

## 中继连接<a id='中继连接'></a>
中继连接是一种[传输协议](#传输)，两个对等端点通过第三方中继对等端点进行路由通信。

在许多情况下，对等端点将无法[穿透 NAT 或/和防火墙](#NAT穿透)，以使其公开访问。或者它们可能没有使用相同的允许直接通信的基础[传输协议](#传输)。

为了在 NAT 等连接障碍面前实现对等架构，libp2p 定义了一个称为 [p2p-circuit](https://github.com/libp2p/specs/tree/master/relay) 的协议。当一个对等端点无法监听公网地址时，它可以拨出到中继对等端点，并打开一个长连接。其他对等端点可以使用 `p2p-circuit` 地址通过中继对等端点拨号，以将流量转发到其目的地。

 中继协议受 [TURN](https://tools.ietf.org/html/rfc5766) 的启发，TURN 是 NAT 穿透技术的 [交互式连接创建集合](https://tools.ietf.org/html/rfc8445) 的一部分。

>中继连接是端到端加密的，这意味着充当中继的对等端点无法读取或篡改流经该连接的任何流量。

中继协议的一个重要方面是它不是“透明的”。换句话说，发送者和接收者都知道信息流正在被中继。这很有用，因为接收者以看到用于打开连接的中继地址，并且可以潜在地使用它来构建返回发送者的路径。它也不是匿名的——所有参与者都使用他们的 `PeerId` 来识别，包括中继节点。

### 协议版本
今天有两个版本的中继连接协议，[v1](https://github.com/libp2p/specs/blob/master/relay/circuit-v1.md) 和 [v2](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md)。我们建议使用后者而不是前者。有关两者的详细比较，请参阅[中继连接 v2 版规范](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md#introduction)。如果没有明确说明，本文档将描述中继连接 v2 版协议。

### 中继地址
一个中继连接使用[多地址](#multiaddr)识别，该多地址包括了正在通信的 `PeerId`，也包括了中继端点自身 `PeerId`。

假设我有一个 `PeerId` 为 QmAlice 的对等端点。我想把我的地址告诉我的朋友 QmBob，但我的 NAT 不允许任何人直接对我拨号。

我可以构造的最基本的 `p2p-circuit` 地址如下所示：

`/p2p-circuit/p2p/QmAlice`

上面的地址很有趣，因为它既不包括我们要联系的对等端点的地址，也不包括可以传输流量的中继对等端点的地址。如果没有这些信息，对等端点对我拨号的唯一机会就是发现一个中继对等端点并希望他们可以与我联接。

更好的地址是 `/p2p/QmRelay/p2p-circuit/p2p/QmAlice`。这包括特定中继对等端点 `QmRelay` 的身份标识。如果对等端点已经知道如何打开到 `QmRelay` 的连接，他们将能够联系到我们。

还有更好的是在地址中包含中继对等端点的传输地址。假设我已经建立了一个到特定中继对等端点 `QmRelay` 的连接。他们通过 identify 协议告诉我，他们正在侦听 IPv4 地址为 `7.7.7.7`， 端口为 `55555` 上的 TCP 连接。我可以构造一个地址，描述通过该传输上的特定中继到我的路径：

`/ip4/7.7.7.7/tcp/55555/p2p/QmRelay/p2p-circuit/p2p/QmAlice`

在 `/p2p-circuit/` 之前的所有内容都是中继对等端点的地址，其中包括传输地址和它们的 `PeerId`（QmRelay）。 `/p2p-circuit/` 之后是我在连接另一端的 `PeerId`（QmAlice）。

通过将完整的中继路径提供给我的朋友 `QmBob`，他们能够快速建立中继连接，从而无需“四处询问”是否有通往 `QmAlice` 的中继。

>在[传播地址](https://docs.libp2p.io/concepts/peer-routing/)时，最好提供中继地址，其中包括中继对等端点的传输地址。如果中继有许多传输地址，则可以通过每个地址传播 p2p 连接。

### 流程
下面的序列图描述了一个中继连接流程：
![](https://raw.githubusercontent.com/libp2p/specs/master/relay/circuit-v2.svg)

1. 节点 A 位于 NAT 和/或防火墙之后，例如通过 [自动NAT](#自动NAT) 服务检测到。
2. 因此，节点 A 向中继 R 请求预约。节点 A 请求中继 R 代表它监听传入的连接。
3. 节点 B 想要建立到节点 A 的连接。鉴于节点 A 没有传播任何直接地址，而只是传播一个中继地址，节点 B 连接到中继 R，请求中继 R 中继到 A 的连接。
4. 中继 R 将连接请求转发给节点 A，并最终中继 A 和 B 发送的所有数据。


## 协议<a id='协议'></a>
当您编写网络应用程序时，处处都有协议，而 libp2p 中的协议尤其丰富。

本文关注的协议类型是使用 libp2p 本身构建的，使用核心 libp2p 抽象，如[传输](#传输)、[对等端点标识](#对等端点标识)、[寻址](#寻址)等。

在本文中，我们将这种使用 libp2p 构建的协议称为 libp2p 协议，但您也可能会看到它们被称为“有线协议（wire protocols）”或“应用程序协议”。

这些协议定义了您的应用程序并提供了其核心功能。

本文将介绍一些 [libp2p 协议的关键定义](#什么是libp2p协议)，概述协议协商过程，并概述 libp2p 中包含的一些提供关键功能的核心协议。

### 什么是 libp2p 协议？<a id='什么是libp2p协议'></a>

#### 协议 ID
libp2p 协议具有唯一的字符串标识符，在首次打开连接时用于[协议协商](#协议协商)过程。

按照惯例，协议 id 具有类似路径的结构，版本号作为最后组成部分：

`/my-app/amazing-protocol/1.0.1`

对协议的格式或语义的重大更改应该会产生一个新的版本号。有关在拨号和监听过程中版本选择如何工作的更多信息，请参阅[协议协商](#协议协商)部分。

>虽然从技术上讲，libp2p 可以接受任何字符串作为有效的协议 id，但使用带有版本号的路径结构既对开发人员友好，也可以更轻松地[按版本进行匹配](#)。

#### 处理函数
为了接受连接，libp2p 应用程序将使用它们的协议 ID 与 [switch](#Switch)（又名“swarm”）或更高级别的接口（如 [go 的 Host 接口](https://github.com/libp2p/go-libp2p-core/blob/master/host/host.go)）注册协议的处理函数。

当一个被注册的协议 ID 标记的信息流传入时，将调用处理函数。如果您使用[匹配函数](https://docs.libp2p.io/concepts/protocols/#using-a-match-function)注册处理函数，则可以选择是否接受一个对协议 ID 不精确的字符串匹配，例如，匹配协议的[语义版本](https://docs.libp2p.io/concepts/protocols/#match-using-semver)。

#### 二进制流
libp2p 协议传输的“媒介”是具有以下属性的双向二进制流：

* 双向、可靠的二进制数据传输
	* 每一方都可以随时从流中读取和写入
	* 数据的读取顺序与写入的顺序相同
	* 可以是“半开放”的，即关闭写打开读，也可以关闭读打开写
* 支持背压（Supports Backpressure）
	* 读操作不会被快速的写操作淹没

在幕后，libp2p 还将确保流的[安全](#安全通信)和高效的[多路复用](#流多路复用)。这对协议处理程序是透明的，协议处理程序通过流来读取和写入未加密的二进制数据。

二进制数据的格式以及发送内容的机制、发送时间和发送人都由协议决定。下面概述了 libp2p 内部协议中使用的一些[常见模式](https://docs.libp2p.io/concepts/protocols/#common-patterns)，以供启发。

### 协议协商<a id='协议协商'></a>
当拨出以启动新流时，libp2p 将发送您要使用的协议的协议 ID。另一端的监听对等端点将根据注册的协议处理函数检查传入的协议 ID。

如果监听对等端点不支持请求的协议，它将结束流，拨号的对等端点可以使用不同的协议再次尝试，又或者是最初请求的协议的不同版本。

如果支持该协议，则监听对等端点回显协议 ID，并使用约定的协议通过流来发送将来的数据。

一个给定的流或连接使用何种协议达成一致的过程称为协议协商。

### 匹配协议 ID 和版本
注册协议处理函数时，可以使用两种方法。

第一种方法接受两个参数：协议 ID 和处理函数。如果其（请求的）传入流的协议 ID 完全匹配，则将使用新流作为参数调用处理函数。

#### 使用匹配函数
第二种协议注册需要三个参数：协议 ID、协议匹配函数和处理函数。

当其（请求的）传入流的协议 ID 没有任何精确匹配时，协议 ID 将传递至所有已注册的匹配函数。如果有任何返回 `true`，则将调用关联的处理函数。

这使您可以灵活地进行自己的“模糊匹配”，并定义对应用程序有意义的协议匹配规则。

#### 使用 semver 匹配
如果您想同时支持多个版本，您可能需要使用语义版本控制（又名 [semver](https://semver.org/)）。

在 go-libp2p 中，一个名为 `MultistreamSemverMatcher` 的辅助函数可以用作协议匹配函数，以查看注册的协议版本是否可以满足传入的请求。

js-libp2p 作为 [js multistream select](https://github.com/multiformats/js-multistream-select/) 的一部分，提供了类似的匹配功能。

### 特定协议的拨号
当远程对等端点拨号以打开新流时，（发起）对等端点会发送他们想要使用协议的协议 ID。（接收）远程对等端点将使用上述匹配逻辑来接受或拒绝协议。如果协议被拒绝，（发起）对等端点可以重试。

拨号时，您可以选择提供协议 ID 列表而不是单个 ID。当您提供多个协议 ID 时，它们将被依次尝试，如果远程对等端点至少支持一个协议，则将使用第一个成功的匹配项。如果您支持多个协议版本，这可能很有用，因为如果（接收）远程对等端点尚未采用最新版本，您可以使用最新版本也可以回退到旧版本。

### 核心 libp2p 协议
除了开发 libp2p 应用程序时编写的协议外，libp2p 本身还定义了几个用于核心功能的基础协议。

#### 常见模式
下面描述的协议都使用[协议缓冲区](https://developers.google.com/protocol-buffers/)（又名 protobuf）来定义消息模式。

使用非常简单的约定通过连接线路交换消息，该约定在二进制消息有效负载前加上一个整数，表示有效负载的长度（以字节为单位）。长度被编码为 [protobuf varint](https://developers.google.com/protocol-buffers/docs/encoding#varints)（可变长整数）。

#### Ping
| 协议 ID | spec | 实现 |
| :----: | :----: | :----: |
| `/ipfs/ping/1.0.0` | N/A | [go](https://github.com/libp2p/go-libp2p/tree/master/p2p/protocol/ping)&nbsp;&nbsp;&nbsp;&nbsp;[js](https://github.com/libp2p/js-libp2p-ping)&nbsp;&nbsp;&nbsp;&nbsp;[rust](https://github.com/libp2p/rust-libp2p/blob/master/protocols/ping/src/lib.rs) |

ping 协议是一种简单的状态检查，对等端点可以使用它来快速查看其他对等端点是否在线。

在初始协议协商后，拨出对等端点发送 32 字节的随机二进制数据。监听对等端点响应数据，拨出对等端点将验证响应并测量请求和响应之间的延迟。

#### Identify【识别协议】<a id='Identify'></a>
| 协议 ID | spec | 实现 |
| :----: | :----: | :----: |
| `/ipfs/id/1.0.0` | [identify spec](https://github.com/libp2p/specs/pull/97/files) | [go](https://github.com/libp2p/go-libp2p/tree/master/p2p/protocol/identify)&nbsp;&nbsp;&nbsp;&nbsp;[js](https://github.com/libp2p/js-libp2p-identify)&nbsp;&nbsp;&nbsp;&nbsp;[rust](https://github.com/libp2p/rust-libp2p/tree/master/protocols/identify/src) |

`identify` 协议允许对等端点交换彼此的身份标识信息，特别是它们的公钥和已知网络地址。

identify 协议通过使用上表中的 identify 协议 ID 与对等端点建立新流来工作。

当远程对等端点打开新的流时，他们将填写一个 identify  [protobuf](https://github.com/libp2p/go-libp2p/blob/master/p2p/protocol/identify/pb/identify.proto) 消息，其中包含有关他们自己的信息，例如他们的公钥和 `PeerId`。

重要的是，identify 消息包含一个 `observedAddr` 字段，该字段包含对等端点观察到请求进入的[多地址](#multiaddr)。这有助于对等端点确定他们的 NAT 状态，因为它允许他们查看其他对等端点观察到的公共地址，并将其与自己的观察到的网络情况进行比较。

##### identify/push【识别/推送】
| 协议 ID | spec & 实现 |
| :----: | :----: |
| `/ipfs/id/push/1.0.0` | 与上面的 [identify](#Identify) 相同 |

与 `identify` 略有不同，`identify/push` 协议发送相同的识别消息，但它是主动发送的，而不是响应请求。

如果一个对等端点开始监听一个新地址，建立一个新的[中继连接](#中继连接)，或者使用 identify 协议从其他对等端点获取到它的公共地址，这将非常有用。在创建或获取新的地址后，对等端点可以将新地址推送给它当前知道的所有对等端点。这样可以使每个人的路由表保持最新状态，并使其他对等端点更有可能发现新地址。

#### secio
| 协议 ID | spec | 实现 |
| :----: | :----: | :----: |
| `/secio/1.0.0` | [secio spec](https://github.com/libp2p/specs/tree/master/secio) | [go](https://github.com/libp2p/go-libp2p-secio)&nbsp;&nbsp;&nbsp;&nbsp;[js](https://github.com/libp2p/js-libp2p-secio)&nbsp;&nbsp;&nbsp;&nbsp;[rust](https://docs.rs/libp2p-secio/0.26.0/libp2p_secio/)(*译者添加*) |

`secio`（安全输入/输出的缩写）是一种类似于 TLS 1.2 的加密通信协议。

>Secio 现在已弃用，我们建议不要使用它。它已被 [TLS 1.3](https://github.com/libp2p/specs/tree/master/tls) 和 [Noise](https://github.com/libp2p/specs/tree/master/noise) 取代，成为首选的安全传输。有关详细信息，请参阅此[博客文章](https://blog.ipfs.io/2020-08-07-deprecating-secio/)。

#### kad-dht
| 协议 ID | spec | 实现 |
| :----: | :----: | :----: |
| `/ipfs/kad/1.0.0` | [kad-dht spec](https://github.com/libp2p/specs/pull/108) | [go](https://github.com/libp2p/go-libp2p-kad-dht)&nbsp;&nbsp;&nbsp;&nbsp;[js](https://github.com/libp2p/js-libp2p-kad-dht)&nbsp;&nbsp;&nbsp;&nbsp;[rust](https://github.com/libp2p/rust-libp2p/tree/master/protocols/kad) |

`kad-dht` 是基于 [Kademlia](https://en.wikipedia.org/wiki/Kademlia) 路由算法的[分布式哈希表](https://en.wikipedia.org/wiki/Distributed_hash_table)(DHT)，并进行了一些修改。

libp2p 使用 DHT 作为其对等路由和内容路由功能的基础。

#### 中继连接
| 协议 ID | spec | 实现 |
| :----: | :----: | :----: |
| `/libp2p/circuit/relay/0.1.0` | [circuit relay spec](https://github.com/libp2p/specs/tree/master/relay) |  [go](https://github.com/libp2p/go-libp2p-circuit)&nbsp;&nbsp;&nbsp;&nbsp;[js](https://github.com/libp2p/js-libp2p-circuit)&nbsp;&nbsp;&nbsp;&nbsp;[rust](https://docs.rs/libp2p/0.44.0/libp2p/relay/v2/relay/struct.CircuitId.html)(*译者添加*) |

如[中继连接小节](#中继连接)中所述，libp2p 提供了一种协议，用于在两个对等端点无法直接相互连接时通过中继对等端点传输流量。有关使用中继的更多信息，请参阅此小节，包括有关中继地址的说明以及如何在难以处理的 NAT 后面启用自动中继连接。

## 对等端点标识<a id='对等端点标识'></a>
对等端点标识（通常写为 `PeerId`）是对整个 p2p 网络中特定对等端点的唯一引用。

`PeerId` 不仅是每个对等端点身份的唯一标识符，而且可以通过公钥进行验证。

### 什么是 `PeerId`
每个 libp2p 对等端点拥有一个私钥，它对所有其他对等端点保密。每个私钥都有一个对应的公钥，该公钥与其他对等端点共享。

公钥和私钥（或“密钥对”）一起 允许对等端点建立[安全通信](#安全通信)通道。

从概念上讲，`PeerId` 是对等端点公钥的[加密哈希](https://en.wikipedia.org/wiki/Cryptographic_hash_function)。当对等端点建立安全通道时，哈希值可用于验证 用于保护通道的公钥是否与 用于识别对等端点的公钥相同。

[PeerId 规范](https://github.com/libp2p/specs/pull/100) 详细介绍了用于 libp2p 公钥的字节格式，以及如何对密钥进行哈希以生成有效的 `PeerId`。

PeerIds 使用[多重哈希](#Multihash)（multihashes）格式编码，该格式向哈希本身添加了一个小的头，用于标识生成它的哈希算法。

### Peer Ids 如何表示为字符串？
PeerIds 是[多重哈希](#Multihash)，被定义为紧凑的二进制格式。

很常见的情况是，使用与[比特币相同的字母表](https://en.bitcoinwiki.org/wiki/Base58#Alphabet_Base58)，将多重哈希编码到 [base 58](https://en.wikipedia.org/wiki/Base58) 中。

下面的 `PeerId` 是一个使用 base58 编码的多重哈希：

`QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`

虽然可以用许多文本格式（例如十六进制、base64 等）表示多哈希，但 PeerIds 始终使用 base58 编码，在编码为字符串时没有[多基前缀](https://github.com/multiformats/multibase)（multibase prefix）。

### 多地址中的 PeerIds
`PeerId` 可以做为一个 `/p2p` 地址中的参数，编码到一个多地址中。

如果我的对等端点 ID 是 `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`，我的 libp2p 多地址将是：

```shell
/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
```

与其他多地址一样，一个 `/p2p` 地址可以封装到另一个多地址中，以组成一个新的多地址。例如，我可以将上面的 地址与[传输](#传输)地址 `/ip4/7.7.7.7/tcp/4242` 结合起来，生成这个非常有用的地址：

```shell
/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
```

这提供了足够的信息来通过 TCP/IP 传输 对特定的对等端点拨号。如果其他对等端点已经接管了该 IP 地址或端口，这将立即显而易见，因为他们将无法控制用于生成嵌入地址中的 `PeerId` 的密钥对。

有关 libp2p 中地址的更多信息，请参阅[寻址](#寻址)

>libp2p 地址的多地址协议最初写成 `/ipfs`，后来改名为 `/p2p`。两者是等价的，并且在多地址中具有相同的二进制表示。以字符串格式呈现哪一个取决于所使用的多地址库的版本。

### 对等端点信息
另一种与对等端点标识相关的常见 libp2p 数据结构是 `PeerInfo` 结构。

`PeerInfo` 将 `PeerId` 与对等端点正在监听的一组[多地址](#multiaddr)组合在一起。

libp2p 应用程序通常会保留一个“对等端点仓库”或“对等端点帐薄”，为他们知道的所有对等端点维护一个 `PeerInfo` 对象的集合。

当拨出到其他对等端点时，对等端点仓库就像一种“电话簿”；如果对等端点仓库中有对等地址，我们可能不需要使用[对等路由](https://docs.libp2p.io/concepts/peer-routing/)发现它们的地址。

>您可以帮助改进这篇文章！请参考此问题提出建议，并让我们知道如何提供帮助。

## 内容路由<a id='内容路由'></a>
这篇文章即将发布！

请[参阅此问题](https://github.com/libp2p/docs/issues/23)以跟踪进度并提出建议。

## 对等端点路由<a id='对等端点路由'></a>
这篇文章即将发布！

请[参阅此问题](https://github.com/libp2p/docs/issues/13)以跟踪进度并提出建议。

## 寻址<a id='寻址'></a>
灵活的网络需要灵活的寻址系统。由于 libp2p 旨在跨多种网络工作，因此我们需要一种方法以一致的方式处理许多不同的寻址方案。

多地址是一种将多层寻址信息编码为一个“经得起未来考验”的路径结构。它[定义](https://github.com/multiformats/multiaddr)的通用传输和隧道协议（overlay protocols）不仅具有优化的机器编码，而且也具有人类可读性，并允许将多个寻址层组合在一起使用。

例如：`/ip4/127.0.0.1/udp/1234` 对两个协议及其基本寻址信息进行编码。 `/ip4/127.0.0.1` 告诉我们我们想要 IPv4 协议的 `127.0.0.1` 环回地址，`/udp/1234` 告诉我们想要将 UDP 数据包发送到端口 `1234`。

随着我们进一步的组合，事情变得更加有趣。例如，多地址 `/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N` 使用 libp2p 的[注册协议 ID](https://github.com/multiformats/multiaddr/blob/master/protocols.csv) `/p2p/` 和我的 IPFS 节点公钥的[多重哈希](#Multihash)来唯一标识我的本地 IPFS 节点。

有关对等端点标识与公钥加密的关系的更多信息，请参阅[对等端点标识](#对等端点标识)。

假设我的 `PeerId` 为上述的 `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`，我的公网 IP 是 `7.7.7.7`。我启动我的 libp2p 应用程序并监听 TCP 端口 `4242` 上的连接。

现在我可以开始[向我的所有朋友分发多地址](#对等端点路由)了，格式为 `/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`。将我的“位置多地址”（我的 IP 和端口）与我的“身份多地址”（我的 libp2p `PeerId`）结合起来，生成一个包含两个关键信息的新多地址。

现在，我的朋友们不仅知道在哪里可以找到我，而且他们提供该地址的任何人都可以验证另一端的机器是否真的是我，或者至少，他们控制着我的 `PeerId` 的私钥。他们还知道（通过 `/p2p/` 协议 ID）我很可能支持常见的 libp2p 交互，例如打开连接和协商我们可以使用哪些应用程序协议进行通信。这还不错！

这可以扩展到多层寻址和抽象。例如，用于中继连接的地址将传输地址与多个对等端点标识组合在一起，形成一个描述“中继连接”的地址：

```shell
/ip4/7.7.7.7/tcp/4242/p2p/QmRelay/p2p-circuit/p2p/QmRelayedPeer
```

### 更多信息
有关更多详细信息，请参阅[多地址规范](https://github.com/multiformats/multiaddr)，其中包含许多实现的链接。

## 安全性考虑<a id='安全性考虑'></a>
libp2p 使得在两个对等端点之间建立[加密的、经过身份验证的通信通道](#安全通信)变得简单，但是在构建健壮的 p2p 系统时还需要考虑其他重要的安全问题。

此处描述的许多问题都没有“完美的解决方案”，而现存的解决方案和缓解策略可能会有其他领域的权衡和妥协。作为一个通用框架，libp2p 试图为应用程序开发人员提供解决这些问题的工具，而不是采取 所有使用 libp2p 构建的系统 都无法接受的任意的安全方法。

另一个需要考虑的方面是，特定类型的攻击在理论上是可行的这一事实，但并不就自动意味着它是实际的、明智的、值得的或有效的。要评估其理论攻击的实际可利用性，请考虑攻击者为使攻击达到一定的成功而必须花费的资源数量、类别和成本。

### 身份标识和信任
每个 libp2p 对等端点都由其 [PeerId](#对等端点标识) 唯一标识其身份，该 ID 来自一个私钥。对等端点 ID 及其相应的密钥允许我们对远程对等端点进行身份验证，这样我们就可以确保我们与正确的对等端点交谈，而不是冒名顶替者。

然而，就安全性而言，身份验证通常只是“验证”故事的一半。许多系统还需要授权，或确定“谁可以做什么”的能力。

libp2p 不提供“开箱即用”的授权框架，因为 p2p 系统的需求差异很大。例如，一些网络可能根本不需要授权，只需要简单地接受来自任何对等端点的请求，而其他网络可能需要根据角色级别，对请求的资源或服务进行明确的授权。

在 libp2p 上设计一个授权系统，你可以依赖 `PeerId` 的身份验证，并在 `PeerId` 和权限之间建立关联，`PeerId` 与传统授权框架中的“用户名”具有相同的功能，以及对等端点的私钥作为“密码”。然后，您的[协议处理](#协议)程序可以拒绝来自不受信任的对等端点的请求。

当然，也可以在 libp2p 上构建不基于 PeerId 的其它类型的授权系统。例如，您可能希望单个 libp2p 对等端点可供许多人使用，每个人都有一个传统的用户名和密码。这可以通过定义一个授权协议来完成，该协议接受用户名和密码，并在凭据有效时使用签名令牌进行响应。暴露敏感资源的协议可以在访问前请求一个令牌。

设计为完全去中心化的系统通常是“默认开放的”，允许任何对等端点参与核心功能。然而，此类系统可能会受益于维护某种“声誉”系统，以识别有缺陷或恶意的参与者，并阻止或忽略它们。例如，每个对等端点可以根据协议的规则，基于行为的有用性和“正确性”，为其他对等端点分配分值，并在决定是否处理给定请求时，将分值考虑在内。

一个以对等端点协作以相互评估的 完全去中心化的声誉管理系统，并不在 libp2p 的范围内。然而，许多 libp2p 的核心开发人员和社区成员都对该领域的研究和开发感到兴奋，并且也欢迎您在 [libp2p 论坛](https://discuss.libp2p.io/) 上发表意见。

### 具有“滥用潜力”的协作系统
libp2p 的一些最有用的内置协议是协作协议，利用网络中的其他对等端点来执行有益于所有人的任务。例如，存储在 Kad-DHT 上的数据 在与数据相关联的键值最靠近的 一组对等端点之间复制，无论这些对等端点是否对数据有任何特别的兴趣。

协作系统天生就容易受到不良行为者的滥用，尽管我们正在研究限制此类攻击影响的方法，但它们在今天的 libp2p 中是可能的。

### Kad-DHT
Kad-DHT 协议是一个[分布式哈希表](#DHT)，为所有参与者提供一个共享的键/值存储系统。除了键/值查找之外，DHT 是 libp2p 的[对等路由](#对等端点路由)和[内容路由](#内容路由)接口的默认实现，因此在发现 网络上的其他对等端点和服务方面 发挥着重要作用。

#### 女巫攻击【Sybil Attacks】
DHT 和 p2p 系统通常容易受到称为[女巫攻击](https://en.wikipedia.org/wiki/Sybil_attack)的一类攻击，在这种攻击中，攻击者启动大量具有不同身份的 DHT 对等端点（通常称为“女巫”）以淹没网络并获得优势地位。

DHT 查询在完成之前，可能需要通过多个对等端点进行路由，每个对等端点都有机会通过返回错误数据或根本不返回数据来修改查询响应。通过控制大量女巫节点（与网络规模成正比），不良行为者会增加查找路径中的查询概率。为了针对特定的键值，他们可以根据 DHT 的距离度量生成与目标键值“接近”的 ID，从而进一步提高他们在查找路径中的机会。

应用程序可以通过对存储在 DHT 中的值进行签名或使用内容寻址来防止数据被修改，其中存储值的加密哈希做为键【key】，如 [IPFS](https://ipfs.io/) 中。这些策略允许您检测数据是否已被篡改，但是它们不能从一开始就阻止篡改的发生，也不能阻止恶意节点简单地假装数据不存在并完全删除它。

与女巫攻击非常相似，日蚀攻击【Eclipse Attack】也使用大量受控节点，但目标略有不同。 日蚀攻击不是修改传输中的数据，而是针对特定的对等端点，目的是扭曲他们对网络的“视图”，通常是为了防止它们到达任何合法的对等端点（从而“掩盖”真实网络）。这种攻击执行起来相当耗费资源，需要大量恶意节点才能完全有效。

日蚀和女巫攻击很难防御，因为它可以生成无限数量的有效对等端点 ID。许多针对女巫攻击的实际缓解措施都依赖于以某种方式使 ID 生成“昂贵”，例如，通过要求具有现实世界相关成本的工作量证明，或者通过从中央可信机构“铸造”和签名。这些缓解措施超出了 libp2p 的范围，但可以在应用程序层采用，以使女巫攻击更加困难和/或成本过高。

受 [S/Kademlia 论文](https://telematics.tm.kit.edu/publications/Files/267/SKademlia_2007.pdf) 的启发，我们目前正计划实施一种并行查询多个不相交查找路径（不共享任何公共中间对等端点的路径）的策略。这将大大增加找到“诚实”节点的机会，即使某些节点返回不诚实的路由信息​​。

### 发布/订阅
libp2p 的[发布/订阅](#Pubsub)协议允许对等端点向给定“主题”内的其他对等端点传播消息。

默认情况下，`gossipsub` 实现 将使用作者的私钥对所有消息进行签名，并在进一步接受或传播消息之前需要有效的签名。这可以防止消息在传输过程中被更改，并允许接收者对发送者进行身份验证。

但是，作为一种协作协议，对等端点可能会干扰消息路由算法，从而破坏通过网络的消息流。

我们正在积极研究如何减轻恶意节点对 `gossipsub` 路由算法的影响，特别关注防止女巫攻击。我们希望这将创建一个更健壮、更抗攻击的“发布订阅”协议，但它不太可能阻止由攻击者发起的 所有类型的可能攻击。

## 发布/订阅
发布/订阅是一个系统，对等端点聚焦在他们感兴趣的主题【Topic】周围。对等端点关注了某个主题，称之为订阅了该主题：

![](https://docs.libp2p.io/concepts/publish-subscribe/subscribed_peers.png)

对等端点可以向主题发送消息。每条消息都会传递给订阅该主题的所有对等端点：

![](https://docs.libp2p.io/concepts/publish-subscribe/message_delivered_to_all.png)

发布/订阅的用法示例：

* 聊天室。每个房间都是一个发布/订阅主题，客户端发布聊天消息，房间中的所有其他客户端都会收到这些消息。
* 文件共享。每个发布/订阅主题代表一个可以下载的文件。上传者和下载者在发布/订阅主题中传播他们拥有哪些文件，并协调之后在发布/订阅系统之外进行的下载。

### 设计目标
在 p2p 发布/订阅系统中，所有对等端点都参与在整个网络中传递消息。对等发布/订阅系统有几种不同的设计，它们提供了不同的权衡。理想的属性包括：

* 可靠：所有消息都会传递给订阅该主题的所有对等端点。
* 快速：消息传递迅速。
* 高效：网络不会被过多的消息副本淹没。
* 高可用：对等端点加入和离开网络时不会中断网络。没有中心故障点。
* 大规模：主题可以拥有大量的订阅者并处理大量的消息。
* 简单：该系统易于理解和实施。每个对等端点只需要记住少量的状态。

libp2p 目前使用一种叫做 `gossipsub` 的设计。它的名字来源于这样一个事实：对等端点彼此八卦他们看到了哪些消息，并使用这些信息来维护一个消息传递网络。

### 发现
在对等端点订阅主题之前，它必须找到其他对等端点并与它们建立网络连接。 发布/订阅系统无法自行发现对等端点。相反，它依赖应用程序来代替自己寻找新的对等端点，这个过程称为周边对等端点发现。

发现对等端点的潜在方法包括：

* 分布式哈希表
* 本地网络广播
* 与现有对等端点交换对等端点列表
* 集中式跟踪器或集合点
* 引导节点列表

例如，在一个 BitTorrent 应用程序中，下载文件的过程中已经使用了上述大多数方法。通过重用 BitTorrent 应用程序正常运行时发现的对等端点，该应用程序也可以建立一个强壮的发布/订阅网络。

询问发现的对等端点是否支持发布/订阅协议，如果支持，则将其添加到发布/订阅网络。

### 对等端点类型
在 `gossipsub` 中，对等端点通过全消息【Full-message】对等连接（*以下简称全消息连接/网络 -译注*）或（仅）元数据【Metadata-only】对等连接（*以下简称元数据连接/网络 -译注*）进行相互连接。整个网络结构由这两个网络组成：

![](https://docs.libp2p.io/concepts/publish-subscribe/types_of_peering.png)

#### 全消息
全消息连接用于在整个网络中传输消息的完整内容。这个网络是稀疏连接的，每个对等端点只连接到几个其他对等端点。 （在 `gossipsub` 规范中，这个稀疏连接的网络称为网格，其中的对等端点称为网格成员。）

限制全消息连接的数量很有用，因为它可以控制网络流量；每个对等端点仅将消息转发给其他几个对等端点，而不是所有对等端点。每个对等端点都有一个要连接的对等端点的目标数量。在此示例中，理想情况下，每个对等端点都希望连接到 3 个其他对等端点，但可以满足 2-4 个连接：

![](https://docs.libp2p.io/concepts/publish-subscribe/full_message_network.png)

>在本指南中，以紫色突出显示的数字可以由开发人员配置。

连接度【Peering Degree】（也称为网络度或 D）控制网络的速度、可靠性、可用性和效率之间的权衡。更高的连接度有助于消息更快地传递，更有可能到达所有订阅者，并且有更少的机会因对等端点的离开而中断网络。但是，高连接度也会导致 在整个网络中发送每条消息的 额外冗余副本，从而增加参与网络所需的带宽。

在 libp2p 的默认实现中，理想的网络连接度为 6，可接受的范围为 4-12。

#### 元数据
除了稀疏连接的全消息网络外，还有一个密集连接的元数据网络。该网络由非全消息对等端点之间的所有网络连接组成。

元数据网络共享有关哪些消息是可用的，并执行有助于维护全消息网络的功能。

![](https://docs.libp2p.io/concepts/publish-subscribe/metadata_only_network.png)

### 嫁接【Grafting】和修剪【Pruning】
对等连接是双向的，这意味着对于任何两个连接的对等端点，两个对等端点都认为他们是全消息连接，或者是元数据连接。

任一对等端点都可以通过通知另一方来更改连接类型。嫁接是将元数据连接转换为全消息的过程。修剪是相反的过程；将全消息连接转换为元数据：

![](https://docs.libp2p.io/concepts/publish-subscribe/graft_prune.png)

当一个对等端点的全消息连接的节点太少时，它将随机嫁接一些元数据连接节点以成为全消息连接节点：

![](https://docs.libp2p.io/concepts/publish-subscribe/maintain_graft.png)

相反，当一个对等端点有太多的全消息连接时，它会随机将其中一些修剪为元数据：

![](https://docs.libp2p.io/concepts/publish-subscribe/maintain_prune.png)

在 libp2p 的实现中，每个对等端点每秒执行一系列检查。这些检查称为心跳【Heartbeat】。在此期间进行嫁接和修剪。

### 订阅和取消订阅
对等端点跟踪其直接连接的对等端点订阅了哪些主题【Topic】。使用此信息，每个对等端点都能够构建一张他们周围的主题订阅图，并且知道哪些对等端点订阅了这些主题：

![](https://docs.libp2p.io/concepts/publish-subscribe/subscriptions_local_view.png)

通过发送订阅【Subscribe】和取消订阅消息来跟踪订阅。当两个对等端点之间建立新连接时，它们首先会相互发送他们订阅的主题列表：

![](https://docs.libp2p.io/concepts/publish-subscribe/subscription_list_first_connect.png)

然后随着时间的推移，每当一个对等端点订阅或取消订阅某个主题时，它都会向每个对端等点发送一个订阅或取消订阅消息。无论接收对等端点是否订阅了相关主题，这些消息都会发送到所有连接的对等端点：

![](https://docs.libp2p.io/concepts/publish-subscribe/subscription_list_change.png)

订阅和取消订阅消息与嫁接和修剪消息密切相关。当一个对等端点订阅一个主题时，它会选择一些对等端点成为该主题的全消息节点，并在订阅消息的同时向它们发送嫁接消息：

![](https://docs.libp2p.io/concepts/publish-subscribe/subscribe_graft.png)

### 发送消息
当一个对等端点想要发布一条消息时，它会向其连接的所有全消息网络中的节点发送一个副本：

![](https://docs.libp2p.io/concepts/publish-subscribe/full_message_send.png)

类似地，当一个对等端点从另一个对等端点接收到一条新消息时，它会存储该消息并将副本转发给 它连接到的所有其他全消息网络中的节点：

![](https://docs.libp2p.io/concepts/publish-subscribe/full_message_forward.png)

在 [gossipsub 规范](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/README.md#controlling-the-flood)中，对等端点也称为路由器，因为它们在通过网络路由消息时具有此功能。

同行记住最近看到的消息列表。这让对等端点仅在他们第一次看到的消息采取行动，并忽略已经看过的消息不让其重新传输。

对等端点也可以选择验证收到的每条消息的内容。是否有效和无效取决于应用程序。例如，聊天应用程序可能会强制所有消息必须短于 100 个字符。如果应用程序告诉 libp2p 一条消息无效，那么该消息将被丢弃并且不会通过网络进一步复制。

### Gossip
对等端点 "gossip" 他们最近看到的消息。每个对等端点每秒钟随机选择 6 个元数据连接，并向它们发送最近看到的消息列表。

![](https://docs.libp2p.io/concepts/publish-subscribe/gossip_deliver.png)

“gossip” 让对等端点有机会注意到 他们有可能丢失全消息网络上的的消息。如果对等端点注意到它反复丢失消息，那么它可以与拥有消息的对等端点 建立新的全消息连接。

以下是如何通过元数据连接 请求特定消息的示例：

![](https://docs.libp2p.io/concepts/publish-subscribe/request_gossiped_message.png)

在 [gossipsub 规范](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/README.md#control-messages)中，宣布最近看到的消息的 “gossip” 称为 “*IHAVE*” 消息，而对特定消息的请求称为 “*IWANT*” 消息。

### 扇出【Fan-out】
对等端点可以将消息发布到未订阅的主题。关于如何做到这一点，有一些特殊的规则可以帮助确保可靠地传递这些消息。

当一个对等端点第一次想要向一个它没有订阅的主题发布消息时，它会随机选择 3 个订阅该主题的对等端点（如下所示的 3 个）并将它们记为该主题的扇出节点：

![](https://docs.libp2p.io/concepts/publish-subscribe/fanout_initial_pick.png)

与其他类型的对等连接不同，扇出是单向的；它们总是从主题外的对等端点 指向订阅该主题的对等端点。订阅该主题的对等端点不会被告知他们已被选中，并且仍将连接视为元数据连接。

每次发送者想要发送消息时，它都会将消息发送给它的扇出节点，然后它们在主题中分发消息：

![](https://docs.libp2p.io/concepts/publish-subscribe/fanout_message_send.png)

如果发送者去发送一条消息，但注意到自上次以来他们的一些扇出节点消失了，他们将随机选择其它的扇出节点，并将它们加到最多 6 个。

当一个对等端点订阅一个主题时，如果它已经有一些扇出节点，它会更希望它们成为全消息连接节点：

![](https://docs.libp2p.io/concepts/publish-subscribe/fanout_grafting_preference.png)

在 2 分钟内没有向某个主题发送任何消息后，该主题的所有扇出节点都被遗忘：

![](https://docs.libp2p.io/concepts/publish-subscribe/fanout_forget.png)

### 网络数据包
对等端点实际通过网络相互发送的数据包 是本指南中看到的所有不同消息类型的组合（应用程序消息、have/want、订阅/取消订阅、嫁接/修剪）。这种结构允许在单个网络数据包中批量发送多个不同的请求。

以下是整个网络数据包结构的图形表示：

![](https://docs.libp2p.io/concepts/publish-subscribe/network_packet_structure.png)

有关用于编码网络数据包的确切[协议缓冲区](https://developers.google.com/protocol-buffers)纲要，请参阅[规范](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/README.md#protobuf)。

### 状态
以下是每个对等端点参与发布/订阅网络时必须记住的状态摘要：

* 订阅：订阅的主题列表。
* 扇出主题：这些是最近发送但未订阅的主题。对于每个主题，都会记住最后向该主题发送消息的时间。
* 当前连接到的对等端点列表：对于连接到的每个对等端点，状态包括他们订阅的所有主题，以及每个主题的对等连接是全连接、元数据还是扇出。
* 最近查看的消息：这是最近查看的消息的缓存。它用于检测和忽略重传的消息。对于每条消息，状态包括发送者和序列号，这足以唯一标识任何消息。对于最近的消息，会保留完整的消息内容，以便可以将其发送给请求该消息的任何对等端点。

![](https://docs.libp2p.io/concepts/publish-subscribe/state.png)

### 更多信息
有关影响 `gossipsub` 设计的 其他发布/订阅设计的 更多详细信息和讨论，请参阅 [gossipsub 规范](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/README.md)。

实现细节参见 [go-libp2p-pubsub](https://github.com/libp2p/go-libp2p-pubsub) 源码中的 [gossipsub.go](https://github.com/libp2p/go-libp2p-pubsub/blob/master/gossipsub.go) 文件，是 gossipsub 在 libp2p 中的规范实现。

## 流多路复用<a id='流多路复用'></a>
流多路复用允许多个独立的逻辑流共享一个公共的底层传输介质。

libp2p 应用程序通常在对等端点之间打开许多独立的通信流，并且可能与给定的远程对等端点同时打开多个并发流。流多路复用允许我们 在与对等端点交互的整个生命周期内， 分摊建立新[传输](#传输)连接的开销。我们还只需要处理一次 [NAT穿透](#NAT穿透)，就可以打开我们需要的任意数量的流，因为它们都将共享相同的底层传输连接。

多路复用绝非 libp2p 独有的。大多数通信网络都涉及某种多路复用，因为传输介质通常很稀缺，需要许多参与者共享。例如，TCP/IP 堆栈通过底层网络连接，复用许多 TCP 流，并且使用唯一的端口号来区分流。 libp2p 的流多路复用器位于传输堆栈的“上方”，并允许多个流在一个 TCP 端口或其他原始传输连接上流动。

libp2p 为流多路复用器提供了一个通用[接口](#接口)，并提供了多种[实现](#实现)。应用程序可以支持多个多路复用器，如果远程对等端点不支持首选选择，这将允许您回退到广泛支持的多路复用器。

### 它在 libp2p 堆栈的位置
libp2p 的多路复用发生在“应用层”，这意味着它不是由操作系统的网络堆栈提供的。但是，编写 libp2p 应用程序的开发人员很少需要直接与流多路复用器交互，除非在初始配置时去控制启用哪些模块。

#### Switch / Swarm
libp2p 在一个称为 switch（或“swarm”，具体取决于实现）的组件中，维护有关已知对等端点和现有连接的一些状态。该 switch 提供了一个拨号和监听接口，该接口可以获取用于给定连接的流多路复用器的细节。

配置 libp2p 时，应用程序会启用流多路复用模块，switch 在拨号和监听连接时会使用该模块。如果远程对等端点支持任何相同的流多路复用实现，则 switch 将在建立连接时选择并使用它。如果您拨号时，switch 已经与对等端点建立了开放连接，则新流将自动在现有连接上多路复用。

在连接建立过程的早期，就使用哪个流多路复用器达成一致。对等端点使用[协议协商](#协议协商)来约定一个共同支持的多路复用器，它将“原始”传输连接升级为能够打开新流的多路复用连接。

### 接口和实现
#### 接口<a id='接口'></a>
[流多路复用接口](https://github.com/libp2p/interface-stream-muxer)定义了如何将流多路复用模块应用于连接，以及多路复用连接支持哪些操作。

#### 实现<a id='实现'></a>
libp2p 中有多个流多路复用模块。请注意，并非所有 libp2p 的语言实现 都支持所有的流多路复用器。

##### mplex
mplex 是为 libp2p 开发的协议。该规范定义了一个简单的多路复用协议，该协议在 libp2p 的语言实现中得到广泛支持：

* Go: [go-mplex](https://github.com/libp2p/go-mplex)
* Javascript: [js-mplex](https://github.com/libp2p/js-libp2p-mplex)
* Rust: [rust-libp2p mplex module](https://github.com/libp2p/rust-libp2p/tree/master/muxers/mplex)

##### yamux
[yamux](https://github.com/hashicorp/yamux) 是由 [Hashicorp](https://www.hashicorp.com/) 设计的多路复用协议。

yamux 提供比 mplex 更复杂的流控制，并且可以通过单个连接扩展到数千个多路复用流。

go 和 rust 目前支持 yamux：

* Go: [go-smux-yamux](https://github.com/whyrusleeping/go-smux-yamux)
* Rust: [rust-libp2p yamux module](https://github.com/libp2p/rust-libp2p/tree/master/muxers/yamux).

##### quic
[QUIC](https://en.wikipedia.org/wiki/QUIC) 是包含一个“原生”流多路复用器的[传输](#传输)协议。libp2p 将自动使用原生多路复用器，用于使用 quic 传输的流。

[go-libp2p-quic-transport](https://github.com/libp2p/go-libp2p-quic-transport) 目前支持 quic。(*[rust-quick-transport](https://docs.rs/libp2p/0.44.0/libp2p/core/multiaddr/enum.Protocol.html#variant.Quic) -译注*)

##### spdy
SPDY 是 Google 开发的协议，是 HTTP/2 的前身。 SPDY 实现了一个流多路复用器，一些 libp2p 的实现支持它：

* Go: [go-smux-spdystream](https://github.com/whyrusleeping/go-smux-spdystream)
* Javascript: [js-libp2p-spdy](https://github.com/libp2p/js-libp2p-spdy)

##### muxado
[muxado](https://github.com/inconshreveable/muxado) 是一个 go 的流多路复用库，由 go-libp2p 通过 [go-smux-muxado](https://github.com/whyrusleeping/go-smux-muxado) 支持。

&emsp;
&emsp;
# 附：术语
## 引导节点【Boot Node】
p2p 网络中的新节点通常通过一组称为“引导节点”的节点与 p2p 网络建立初始连接。有关这些引导节点的信息（例如地址）是嵌入在应用程序二进制文件中或作为配置选项提供。

引导节点充当入口点，向新节点提供网络中其他节点的列表。连接到引导节点后，新节点可以连接到网络中的其他节点，从而不再依赖引导节点。

## 中继连接【Circuit Relay】
在愿意并且能够充当中介的第三方对等端点的帮助下，使无法直接通信的对等端点之间能够建立通信的方法。

在许多真实的 p2p 网络中，由于各种原因，不是所有对等端点之间的能够直接通信的。例如，一个或多个对等端点可能位于防火墙后面或存在 NAT 穿透问题。又或者，对等端点可能没有共同的基础传输协议。

在这种情况下，只要每个节点都能够与愿意的中继节点建立连接，就可以在节点之间“架起桥梁”。如果我只会说 TCP 而你只会说 websockets，我们仍然可以在双语朋友的帮助下溜达。

中继连接根据中继规则在 libp2p 中实现，中继规则定义了中继连接协议机制和寻址方案。

## 客户端/服务器【Client / Server】<a id='cs'></a>
一种由中央“服务器”程序定义的网络架构，这些程序为一组（通常更大的）“客户端”程序提供服务和资源。通常客户端不直接相互通信，而是通过服务器路由所有通信，服务器本质上是网络中最具特权的成员。

## 分布式哈希表【DHT】<a id='DHT'></a>
一个分布式哈希表，其内容分布在参与节点的网络中。与进程内哈希表非常相似，值与键相关联并且可以通过键检索。大多数 DHT 以确定性的方式 将一部分可寻址键值空间 分配给节点，这可以高效的路由到给定路径的节点。

由于 DHT 是许多 p2p 系统的基础，libp2p 提供了一个基于 [Kademlia](https://en.wikipedia.org/wiki/Kademlia)-DHT 的 [Go](https://github.com/libp2p/go-libp2p-kad-dht) 和 [javascript](https://github.com/libp2p/js-libp2p-kad-dht) 实现。(*[rust-kad-dht](https://docs.rs/libp2p/0.44.0/libp2p/kad/index.html) -译注*)

libp2p 使用 DHT 作为其对等路由实现之一的基础，使用 libp2p 构建的系统通常使用 DHT 来提供有关内容的元数据、广播服务可用性等。

## 连接【Connection】
libp2p 连接是允许对等端点读取和写入数据的通信通道。

对等端点之间的连接是通过传输建立的，可以将其视为“连接工厂”。例如，TCP 传输允许您创建使用 TCP/IP 作为其底层基础的连接。

## 拨号【Dial】<a id='Dial'></a>
打开与另一个对等端点的 libp2p 连接的过程称为“拨号”，接受连接称为“监听”。拨号和监听的实现 一起构成了传输。

## 监听【Listen】
接受传入的 libp2p 连接的过程称为“监听”，它允许其他对等端点“拨号”并打开与您的对等端点的网络连接。

## 多播DNS【mDNS】<a id='mDNS'></a>
多播 DNS 是一种用于在本地网络上发现服务的协议。 libp2p 的对等路由实现之一是利用 mDNS 快速有效地发现本地对等端点。

## 多地址【multiaddr】<a id='multiaddr'></a>
多地址是一种将多层寻址信息编码为单个“经得起未来考验”的路径结构。

例如：`/ip4/127.0.0.1/udp/1234` 对两个协议及其基本寻址信息进行编码。 `/ip4/127.0.0.1` 告诉我们我们想要 IPv4 协议的 `127.0.0.1` 环回地址，`/udp/1234` 告诉我们想要将 UDP 数据包发送到端口 `1234`。

可以组合多个地址来描述地址的多个“层”。

有关更多详细信息，请参阅[寻址](#寻址)或[多地址](https://github.com/multiformats/multiaddr)规范，其中包含许多实现的链接。

## 多重哈希【Multihash】<a id='Multihash'></a>
[多重哈希](https://github.com/multiformats/multihash)是一种约定，用于以紧凑的、确定性的编码方式表示许多不同[加密哈希函数](https://en.wikipedia.org/wiki/Cryptographic_hash_function)的输出，以适应未来的变化。

哈希是许多系统（例如 git）的核心，但许多系统只存储哈希输出本身，因为哈希函数的选择是系统的一个隐式设计参数。这带来了一个不幸的后果，那就是很难改变你对系统使用哪种哈希函数的看法！

多重哈希编码用于生成 输出的哈希函数的类型，以及输出的字节长度。这会在原始哈希输出中添加一个两字节的标头，作为这两个字节的回报，该标头允许当前和未来的系统通过利用公共库轻松识别和验证许多哈希函数。随着新函数的添加，您可以更轻松地扩展应用程序或协议以支持它们，因为旧的和新的哈希输出将很容易相互区分。

在 libp2p 中，多重哈希最突出的用途是在 [PeerId](#PeerId) 中，它包含对等端点公钥的哈希。然而，使用 libp2p 构建的系统，尤其是 [IPFS](https://ipfs.io/)，将多重哈希用于其他目的。在 IPFS 的情况下，由于 IPFS 使用 libp2p 并共享相同的 PeerId 约定，因此多重哈希同时用于标识内容和其他对等端点。

在 IPFS 中，多重哈希是 [CID 或内容标识符](https://docs.ipfs.io/concepts/content-addressing/)的关键组成部分，而 CID 的“v0”版本是一段内容的“原始”多重哈希。 “现代” CID 将某些内容的多重哈希与一些紧凑的上下文元数据相结合，允许 IPFS 等内容寻址系统在哈希寻址数据之间创建更有意义的链接。有关 p2p 系统中哈希链接数据结构的更多信息，请参阅 [IPLD](https://ipld.io/)。

多重哈希通常表示为 [base58 编码](https://en.wikipedia.org/wiki/Base58)的字符串，例如 `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`。前两个字符 `Qm` 是 SHA-256 哈希算法的多重哈希标头，长度为 256 位，对于所有使用 SHA-256 的 base58 编码的多重哈希都是一样的。

## 多路复用【Multiplexing】<a id='Multiplexing'></a>
多路复用是指在单个逻辑“介质”上组合多个通信流的过程。例如，我们可以在单个 TCP 网络连接上 维护多个独立的数据流，当然，这是在单个物理连接（以太网、wifi 等）上多路复用的。

多路复用允许对等端点在单个连接上提供多种[协议](#Protocol)，从而减少网络开销并使 [NAT穿透](#NAT穿透) 更加高效。

libp2p 支持流多路复用的多种实现。 [mplex 规范](https://github.com/libp2p/specs/tree/master/mplex)定义了一个简单的协议，并以多种语言实现。其他支持的多路复用协议包括 [yamux](https://github.com/hashicorp/yamux) 和 [spdy](https://www.chromium.org/spdy/spdy-whitepaper)。

有关跨 libp2p 语言实现的多路复用状态，请参阅 [Stream Muxer 实现](https://libp2p.io/implementations/#stream-muxers)。

## multistream<a id='multistream'></a>
[multistream](https://github.com/multiformats/multistream) 是一种轻量级约定，用于“标记”二进制数据流，带有标识流内容的短标头。

libp2p 使用 multistream 来标识用于对等端点之间通信的[协议](#协议)，并使用相关项目 [multistream-select](https://github.com/multiformats/multistream-select) 进行[协议协商](#协议协商)。

## 网络地址转换【NAT】<a id='NAT'></a>
网络地址转换通常是地址从一个地址空间映射到另一个地址空间，这经常发生在专用网络与全球互联网的边界。它在 IPv4 网络（仍然是绝大多数）中尤其重要，因为 IPv4 的地址空间非常有限。使用 NAT，本地专用网络可以在内部网络中拥有大量地址，同时仅使用一个公网 IP。

NAT 在实践中的一个不幸影响是，从内网到公网的传出连接 比从公网连接到内网的传入连接 要容易得多。这是因为在内网上监听连接的机器需要明确地告诉负责 NAT 的路由器，它应该将给定端口（OS 网络层的多路复用）的流量转发到监听机器。

这在客户端/服务器模型中不是什么问题，因为到服务器的传出连接为路由器提供了足够的信息，可以将响应 路由回它需要去的客户端。

在 p2p 模型中，接受来自其他节点的连接通常与启动它们一样重要，这意味着我们经常需要我们的节点可以从全球互联网公开访问。 NAT穿透 有许多可行的方法，其中一些在 libp2p 中实现。

## NAT穿透【NAT Traversal】
NAT穿透 是指跨越 [NAT](#NAT) 边界与其他机器建立连接的过程。当跨越 IP 网络之间的边界时（例如，从本地网络到全球互联网），会发生[网络地址转换](#NAT)过程，该过程将地址从一个空间映射到另一个空间。

例如，我的家庭网络有一个内网 IP 地址范围 (10.0.1.x)，这是为专用网络保留的地址范围中的一部分。如果在我的计算机上启动一个程序来监听其内网地址上的连接，那么来自公网的用户将无法联系到我，即使他们知道我的公网 IP 地址。这是因为我还没有让我的路由器知道我的程序。当从 Internet 连接到我的公网 IP 地址时，路由器需要确定将请求 路由到哪个内网 IP 以及哪个端口。

有很多方法可以通知路由器您要公开的服务。对于用户路由器，可能有一个管理界面可以为任意范围的 TCP 或 UDP 端口设置映射。在许多情况下，路由器将允许使用 libp2p 支持的名为 [upnp](https://en.wikipedia.org/wiki/Universal_Plug_and_Play) 的协议自动注册端口。如果启用，libp2p 将尝试向路由器注册您的服务以进行自动 NAT穿透。

在某些情况下，自动 NAT穿透 是不可能的，通常是因为涉及到多层 NAT。在这种情况下，我们仍然希望能够进行通信，并且我们特别希望能够达到 并允许其他对等端点拨入 和使用我们的服务。这是中继连接的动机之一，它是一种涉及“中继”节点的协议，该节点可公开访问并可以代表其他人路由流量。一旦建立了中继连接，在特别难以处理的 NAT 后面的对等端点可以通告中继连接的多地址，中继节点将代表我们接受传入连接并通过中继连接向我们发送流量。

## 节点【Node】<a id='Node'></a>
“节点”这个词在一般编程环境中是相当多的，在 p2p 网络圈中尤其如此。

一种常见的用法是，“节点”指的是 在宇宙中某个时间和地点运行的 对等软件系统的单个实例。例如，`我在 AWS 中运行一个 orbit-db 节点（我认为它在版本 3.2.0 上）`。在这种用法中，“节点”是指参与网络的整个软件程序（unix中的守护进程）。在本文档中，我们经常为此使用“[对等端点【peer】](#Peer)”，这两个术语在各种 p2p 软件讨论中经常互换使用。

另一个完全不同的含义是 [node.js](https://nodejs.org/)，一个 javascript 运行时环境，它是 [javscript libp2p 实现](https://docs.libp2p.io/reference/)支持的运行时之一。一般来说，当“node”指的是 node.js 时，从上下文中应该很清楚。

我们社区的许多成员都对许多情况下的图感到兴奋，因此在讨论各种主题时经常使用“节点和边”的图术语。图相关讨论的一些常见上下文：

* 在讨论对等网络的[拓扑](#Topology)或结构时，“节点”通常用于连接对等端点图的上下文中。该图的有效构造和遍历是有效[对等路由](#对等路由)的关键。
* 在讨论数据结构时，“节点”通常用于指代结构的关键元素。例如，一个链表由许多“节点”组成，其中包含一个值和一个将其连接到下一个节点的链接（或者，在图术语中，指链接节点形成的“边”）。由于许多有用且有趣的数据结构可以被描述为图，因此在讨论它们的属性时，图论的许多术语都适用。特别是，IPFS 自然非常适合存储和操作形成[有向无环图或 DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) 的数据结构。

对于我们社区中的许多人来说，一个特别有趣的数据结构是 IPLD，即星际键连资料。与 libp2p 类似，IPLD 源于 IPFS 的实际需求，但在 IPFS 之外的许多环境中广泛有用和有趣。 IPLD 讨论通常涉及此处讨论的所有类型的“节点”。

## 覆盖网络【Overlay】
“覆盖网络”或简称“覆盖”是指对等网络的逻辑结构，它“覆盖”在用于较低级别网络通信的底层传输机制之上。

p2p 系统通常由一个或多个覆盖网络组成，这些覆盖网络决定了如何识别和定位对等端点、消息如何在整个系统中传播以及其他关键属性。

libp2p 中使用的覆盖网络的示例是基于 [Kademlia](https://en.wikipedia.org/wiki/Kademlia) 的 [DHT](#DHT) 实现，以及由各种[发布/订阅](#Pubsub)实现的参与者形成的网络。

## 对等端点【Peer】<a id='Peer'></a>
p2p 网络中的单个参与者。虽然给定的对等端点可能支持许多[协议](#协议)，但它有一个 `PeerId`，用于向其他对等端点标识自己。通常与[节点](#Node)同义使用。

## PeerId<a id='PeerId'></a>
一个[对等端点](#Peer)的唯一、可验证的身份标识符，另一个对等端点不可能伪造或冒充。在 libp2p 中，对等端点的身份由它们的 `PeerId` 标识，它是全球唯一的，并且允许其他对等端点获取某个对等端点的[加密公钥](https://en.wikipedia.org/wiki/Public-key_cryptography)。

`PeerId` 最常见的形式是对等端点公钥的[多重哈希](#Multihash)，可用于从 DHT 中获取整个公钥以进行加密或签名验证。也有实验支持将小公钥直接嵌入或“内联”到 `PeerId` 中，但是，这是一个[持续讨论](https://github.com/libp2p/specs/issues/138)的领域，在最终确定之前 在生产系统中应谨慎对待。

加密对等端点身份的一个重要属性是它们与[传输](#传输)解耦，允许对等端点验证其他对等端点的身份，而不管它们可能使用什么底层网络进行通信。这也使它们比基于位置的标识符（例如，IP 地址）具有更长的“保质期”，因为身份在地址更改时保持稳定。

## Peer store
一种数据结构，用于存储已知对等端点的 [PeerId](#PeerId)，以及可用于与它们通信的已知[多地址](#multiaddr)。

## 对等路由【Peer routing】<a id='对等路由'></a>
对等路由是在给定对等端点 ID 的情况下，为网络中的对等端点发现网络“路由”或地址的过程。

它还可能包括本地对等端点的“周边”发现，例如通过[多播 DNS](#mDNS)。

libp2p 中的主要对等端点路由机制 是使用[分布式哈希表](#DHT)来定位对等端点，利用 Kademlia 路由算法来有效定位对等端点。

## 点对点【Peer-to-peer (p2p)】
点对点 (p2p) 网络是参与者（称为[对等端点](#Peer)或[节点](#Node)）在或多或少“平等的基础上”直接相互通信的网络。这并不一定意味着所有对等端点都是相同的；有些可能在整个网络中扮演不同的角色。然而，对等网络的一个定义特征是它们不需要一组特权“服务器”，这些“服务器”的行为与其“客户端”完全不同，就像在主要的[客户端/服务器模型](#cs)中那样。

## 发布/订阅【Pubsub】<a id='Pubsub'></a>
一种通信模式，其中参与者“订阅”其他参与者“发布”的更新，通常是关于一个命名的“主题”。

libp2p 定义了一个[发布/订阅规范](https://github.com/libp2p/specs/blob/master/pubsub/README.md)，其中包含支持语言的多个实现的链接。 发布/订阅是一个持续研究和开发的领域，针对不同的用例和环境优化了多种实现。

## 协议【Protocol】<a id='Protocol'></a>
通常，用于网络通信的一组规则和数据结构。

libp2p 由许多协议组成，并利用了操作系统或运行时环境提供的许多其他协议。

大多数核心 libp2p 功能是根据协议定义的，并且 libp2p 协议是使用 [multistream](#multistream) 标头标识的。

## 协议协商【Protocol Negotiation】
就给定通信流使用何种协议达成一致的过程。

在 libp2p 中，使用称为 [multistream](#multistream) 的约定来标识协议，该约定在包含唯一名称（包括版本标识符）的流的开头添加一个小标头。

当两个对等端点第一次连接时，他们交换握手以就使用的协议达成一致。

libp2p 握手的实现称为 [multistream-select](https://github.com/multiformats/multistream-select) 选择。

详见[协议协商小节](#协议协商)。

## 流【Stream】
TODO：区分各种类型的“流”。可以参考

* 原始 TCP 连接
* multistream 连接的一个组件
* node.js streams/pull-streams

## Swarm<a id='Swarm'></a>
可以指相互连接的对等端点的集合。

在 libp2p 代码库中，“swarm”可以是指允许节点与节点交互的模块，尽管该组件后来被重命名为“[switch](#Switch)”。

## Switch<a id='Switch'></a>
一个 libp2p 组件，负责将多个传输组合到一个接口中，允许应用程序代码向对等端点[拨号](#Dial)，而无需指定要使用的[传输](#传输)。

除了管理传输之外，switch 还协调“连接升级”过程，该过程将传输层的“原始”连接提升为支持[协议协商](#协议协商)、[流多路复用](#Multiplexing)和[安全通信](#安全通信)的连接。

由于历史原因，有时称为“[swarm](#Swarm)”。

## 拓扑【Topology】<a id='Topology'></a>
在 p2p 环境中，通常是指 p2p 网络通信时形成的的形状或结构。

## 传输【Transport】
在 libp2p 中，`传输`是指让我们将比特位从一台机器移动到另一台机器的技术。这可能是操作系统提供的 TCP 网络、浏览器中的 websocket 连接或任何其他能够实现[传输接口](https://github.com/libp2p/interface-transport)的东西。

请注意，在某些环境中，例如在浏览器中运行的 javascript，并非所有传输都可用。在这种情况下，可以在可以支持许多常见传输的对等端点的帮助下建立[中继连接](#中继连接)。这样的中继可以充当某种“传输适配器”，允许无法直接相互通信的对等端点进行交互。例如，浏览器中只能建立 websocket 连接的对等端点可以通过能够建立 TCP 连接的对等端点进行中继，这将能够与更广泛的对等端点进行通信。

















