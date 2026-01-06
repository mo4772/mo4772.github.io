---
layout:     post
title:      Webrtc FEC分析(二) FEC的封装.md
subtitle:   Webrtc FEC
date:       2025-09-01
author:     mo4772
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Webrtc
    - FEC
---

在前面的文章提到，**fec包和fec的生成规则都需要告知接收端**，所以这些信息怎么携带，组织，需要有一套规则。在webrtc中是按照rfc5109中定义的规则来封装。

rfc5109定义了两个方面的规则：

1. 定义了一种如何生成fec，叫做ULP(webrt中使用的是简化版规则)。
2. 定义了fec包的封装规则(webrtc使用该规则)。


## ULP(Uneven Level Protection)
不均等保护的基本思想是，并不是所有的数据都被保护，被包含的强度也不一样。

强度越强，也就是冗余数据越多，那么带宽占用越大；冗余数据少，抗丢包弱。冗余多少数据与带宽大小间需要权衡。

但是媒体数据中，各类型数据的重要度不一样，关键帧肯定比P帧更重要，所以可以对重要的数据加大冗余，对不重要的数据减少冗余或不加冗余数据。**这是ULP的基本理念，webrtc中也是运用了该理念。**



更进一步，每个包的各部分的重要度都不一样，并不是一定要用整个包的数据进行冗余。可以按需来确定要参与冗余的数据包的长度，同一个包的不同部分可以携带在多个fec中，在一个fec包可以携带多个冗余数据，这样就被称做不同的level，每个level都是独立的，而**webrtc没用使用该规则**，是直接使用整包来计算冗余。

### 示例说明
摘自rfc5109 section-5的关于ULP的说明示例



![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/rfc5109_ulp.png)

上面的例子是用两个fec包保护4个包，Packet A，B，C，D都是媒体包，产生两个FEC packet 1和2。



FEC 1是由A，B部分数据(**当然这个数据长度可以按需变化)**做异或生成，它包含一个冗余数据，叫做L0。

FEC 2包括包含了两个冗余数据L0和L1，L0是由包C，D包生成。L1是由A，B，C，D包生成。



下面是摘自rfc5109对该示例的说明：



> ULP FEC packet #1 has only one level, which protects packets A and B. Instead of applying parity operation to the entire packets of A and B, it only protects a length of data of both packets.  The length,which can be chosen and changed dynamically during a session, is called the protection length.
>



<font style="color:rgba(0, 0, 0, 0.85);">ULP FEC Packet #1 仅包含一个保护级别，用于保护数据包 A 和 B。与对 A 和 B 的完整数据包执行奇偶校验运算不同，它仅对这两个数据包的一段数据长度进行保护。这段可在会话过程中动态选择和调整的数据长度，被称为保护长度。</font>

<font style="color:rgba(0, 0, 0, 0.85);"></font>

> ULP FEC packet #2 has two protection levels.  The level 0 protection is the same as for ULP FEC packet #1 except that it is operating on packets C and D.  The level 1 protection is using parity operation applied on data from packets A, B, C, and D.  Note that level 1 protection operates on a different set of packets from level 0 and has a different protection length from level 0, so are any other levels.  
>



<font style="color:rgba(0, 0, 0, 0.85);">ULP FEC Packet #2 包含两个保护级别。其中，0 级保护与 ULP FEC Packet #1 的保护方式相同，不同之处在于它作用于数据包 C 和 D。1 级保护则通过对数据包 A、B、C 和 D 的数据执行奇偶校验运算来实现。需要注意的是，1 级保护所作用的数据包集合与 0 级不同，且保护长度也与 0 级不同，其他所有级别亦是如此。</font>



## FEC数据的封装
它定义如下几点规则：

1. fec包以rtp包的进行封装，有rtp头。
2. 媒体rtp包和fec包独立封装，它们各自有自己的序列号，pt值，时间戳值。
3. fec数据封装在rtp包的payload中。



如下图所示为fec包的结构图：

![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_packet.png)



### FEC的封包格式
fec数据在Rtp的payload中，如下所示：

![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_packet_1.png)



FEC Header + FEC Level Header + FEC Payload 组成。

#### FEC Header
FEC Header是固定的10个字节，如下:

![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_packet_2.png)



E：保留的扩展标志位，必须设置为0

L：长掩码标志位。当L设置为0时，ulp header的mask长度为16bits，当L设置为1时，ulp header的mask长度为48bits(就是参与冗余的包更多)。

P、X、CC、PT的值由此fec包所保护的RTP包的对应值通过XOR运算得到。

**<font style="color:#DF2A3F;">SN base: 设置为此fec包所保护的RTP包中的最小序列号。</font>**

TS recovery: 由此fec包所保护的RTP包的timestamps通过XOR运算得到。

Length recovery:由此fec包所保护的RTP包的长度通过XOR运算得到。



**<font style="color:#DF2A3F;">其中最重要的就是SN base，fec包通过它与媒体包关联。</font>**

#### FEC Level Header
![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_packet_3.png)

**<font style="color:#DF2A3F;">Protection Length：fec payload的长度。</font>**

**<font style="color:#DF2A3F;">mask：保护掩码，它就是生成配方，用来说明是哪几个媒体包生成了该冗余数据，长度为2个字节或者6个字节（由fec header 的L标志位决定）。</font>**

**<font style="color:#DF2A3F;"></font>**

**<font style="color:#DF2A3F;">mask cont 指示mask的长度，只是fec heaher 的L标志位被设置，才需携带。</font>**

****

**<font style="color:#DF2A3F;">例如：SN base等于100，mask值等于9b80，对应的二进制为 1001 1011 1000 0000，那么就可以知道第0、3、4、6、7、8个RTP包被此级别所保护，被保护的RTP包的序列号分别是100、103、104、106、107、108。</font>**

**<font style="color:#DF2A3F;"></font>**

## **Webrtc中的实现**
**webrtc采用了RFC 5109中定义的fec封包方式，但是并没有使用多level，fec包中始终只有 Level 0，并且是使用整个媒体包做冗余(生成方式与RFC 2733中定义的相同)，见这个例子**[**rfc5109生成一个fec level的示例**](https://datatracker.ietf.org/doc/html/rfc5109#section-10.1)**。**

****

****

## 引用
[rfc5109生成一个fec level的示例](https://datatracker.ietf.org/doc/html/rfc5109#section-10.1)

[rfc5109生成两个fec level的示例](https://datatracker.ietf.org/doc/html/rfc5109#section-10.2)

