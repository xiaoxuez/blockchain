bft



本文提出了Zyzzyva1，一种使用推测来降低成本并简化BFT状态机复制设计的新协议[18,34]。与传统的状态机复制协议[9,32,40]一样，主要提出了客户端对其他副本的请求的顺序。在Zyzzyva中，与传统协议不同，复制品推测性地执行请求，而不运行昂贵的协议协议来确定订单。因此，正确的副本状态可能会有所不同，并且副本可能会向客户端发送不同的响应。尽管如此，客户端的应用程序观察到复制状态机的传统和强大的抽象，它以可线性化[13]的顺序执行请求，因为答复带有足够的历史信息给客户端以确定答复和历史记录是否稳定并最终保证最终承诺。如果推测回复和历史记录稳定，则客户端使用回复。否则，客户端会等待系统收敛于稳定的回复和历史记录。



Zyzzyva面临的挑战是确保对正确客户的响应变得稳定。 最终，副本负责确保来自正确客户端的所有请求最终完成，但等待回复并且历史稳定的客户端可以通过提供将导致请求在当前视图内快速稳定的信息来加速进程 或触发视图更改。 请注意，由于客户不需要提交请求，而只是为了保持稳定，因此客户可以按照一到两个阶段而不是惯常的三个请求行事[9,32,40]。



鉴于BFT复制协议已经在客户端上实施的职责[3,9,10,21,32,40]，将输出提交完全移交给客户端并不是一个很大的步骤，但这一小步骤带来了巨大的收益。首先，Zyzzyva利用推测执行 - 复制品在订单完全建立之前执行请求。其次，Zyzzyva利用快速协议协议[11,20,24]在少至三个消息延迟的情况下建立请求排序。第三，一旦客户知道请求的顺序，协议子协议就停止处理请求，从而避免在副本上建立这些知识所需的工作。

这些选择导致设计Zyzzyva时遇到两个关键挑战。首先，我们必须仔细地指定请求在客户端完成的条件，并定义协议，检查点和视图变更子协议，以保留请求在单个正确的状态机上执行的抽象。直接地，当正确的客户端可以安全地处理对该请求的回复。为了帮助客户确定何时适合回复，Zyzzyva会将历史信息附加到客户收到的回复中，以便客户可以判断回复是否基于相同的请求顺序。 



#### 协议概述

3f+1个节点replicas执行，由一系列views组成，在一个view内，一个单点的节点replica被指定为主要负责子协议共识的leader。

客户端发送一个请求给到leader, leader转发给其余replicas, 收到请求的replicas执行请求并将结果返回给客户端，客户端完成1个请求有两种方式，首先，如果客户端收到3f+1相互一致的响应结果（包括history和application-level的回复），然后客户端便认为请求完成并执行之。其次，如果客户端收到再2f+1 ~ 3f之间个数的一致的结果，客户端将手机2f+1的结果然后分发commit certificate给replicas，一旦2f+1的replicas确认收到commit certificate，客户端将会认为请求完成并且执行响应的回复。

如果有足够的replicas认为leader故障，然后view将会进行更换，一个新的leader将会产生。

在本节的其余部分中，我们描述基本协议并概述其正确性的证明[16]。 在§4中，我们描述了一些优化，这些优化全部在我们的原型中实现，通过用消息认证码（MAC）代替公钥签名来降低加密成本，通过批量请求提高吞吐量，通过缓存out-of 通过优化只读请求来提高读取性能，通过使大多数副本发送散列而不是完整回复来减少带宽，通过仅为首选法定数包括MAC来减少开销，并且通过包括额外的附加功能来提高存在故障节点的性能 证人副本。



#### 节点状态和检查点协议

为了明确地讨论我们的讨论，我们首先讨论由每个副本维护的状态，如图2所总结的。每个副本i维护它已执行的请求的有序历史记录，以及最大提交证书副本，提交证书（定义如下）由我看到，涵盖了我存储历史的最大前缀。这个提交证书覆盖的序列号最高的请求的历史记录是承诺的历史记录，以下的历史记录是推测历史记录。如果n是提交历史记录中的任何请求的最高序列号，我们说一个提交证书的序列号为n。

每个CP INTERVAL请求都会创建一个检查点。副本维护一个稳定的检查点和一个相应的稳定应用程序状态快照，并且它可以存储多达一个暂定检查点和相应的暂定应用程序状态快照。尝试性检查点和应用程序状态提交的过程与早期BFT协议所使用的过程类似[9,10,17,32,40]，因此我们将详细讨论推迟到我们的扩展技术报告[16]。但是，简要总结一下：当正确的副本生成临时检查点时，它会向所有副本发送签名检查点消息。该消息包含检查点中包含的任何请求的最高序列号以及对应的试验性检查点和应用程序快照的摘要。当收集由不同副本签名的f + 1匹配检查点消息时，正确的Zyzzyva副本会将检查点和相应的应用程序快照视为稳定。为了限制历史的大小，副本（1）在提交的检查点之前截断历史记录，（2）在处理自上次提交的检查点以来的2×CP INTERVAL请求之后阻止处理新的请求。最后，每个副本维护一个响应缓存，其中包含每个客户端的最新订购请求的副本以及相应的响应。

