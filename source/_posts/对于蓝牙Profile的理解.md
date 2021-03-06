---
title: 对于蓝牙Profile的理解
date: 2017-06-22 23:27:18
tags: [BT]
---


#### 什么是Profile？
众所周知，蓝牙中有很多的profile，我们接触和学习蓝牙相关的开发不可避免的需要弄懂什么是Profile ,但它对于新手而言似乎没那么容易弄懂，即使是有经验者也很难形象的描述profile的含义，这里我尝试写下自己的理解，以便记录和总结，日后有新的理解不断更新。

Profile中文译名有很多，比如配置文件，剖面，应用协议，轮廓等，每一种翻译代表了一种对于profile的不同理解，以我个人的理解来说，可能中文中并没有那么合适的词与之对应，但我觉得** 剖面 **这个说法可能更贴切一点。

因为profile其实是蓝牙对应于每一个具体的应用场景以及每一种应用的不同的协议栈，也就是说它其实是实现某种功能对应的自下而上的协议的组合。类似于对于横向协议的纵向组合。

<!-- more -->

这里我们不得不简单的介绍一下蓝牙的协议栈组成结构。
![参考 计算机网络第五版](http://upload-images.jianshu.io/upload_images/1806858-80fcbfee9e8266d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上是蓝牙协议栈的概要结构示意图，我们大致的看一眼，会觉得他是不符合OSI和TCP/IP的网络模型的。
- 无线电层也就相当于OSI模型中的物理层，他主要负责无线传输和调制解调。
- 链路控制或者叫做基带层，负责控制时间槽以及数据帧的组装。
- 链路管理负责设备之间逻辑信道的建立，例如电源管理、配对、加密和服务质量。
- HCI接口层是连接上下的管道，一般来说，接口以上的部分由蓝牙设备实现，以下的部分由蓝牙芯片来实现。
- L2CAP层可以携带变长的帧，以及提高可靠性。发送数据包的剖面通常需要使用该协议。
- 在上面是具体的类似于应用层的协议，完成特定的功能。
- **profile剖面对应的则是垂直的条形快，他们各自定义了实现特定功能所包含的协议切片，一个特定的剖面，例如GATT就只包含需要的协议，而不包含那些不需要的。**

#### 蓝牙有什么Profiles ?
蓝牙中有很多的Profile, 我没有找到确切的资料总共有多少种profile,但我们常见的莫过于那几种，而且porile之间也并非平行的关系，他们是相互依赖组合构成的，存在明显的层级关系的。

![参考 BT Spec 4.2](http://upload-images.jianshu.io/upload_images/1806858-b642475951354ea2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是一个层级划分，所有的profile都是直接或间接依赖于GAP的，都是GAP的superset,然后是用于构成多数Application profile的generic profile，这里有四种：

- ** 通用接入剖面（GAP，General Access Profile）**：定义两个蓝牙单元如何发现对方并建立连接，保证两个蓝牙单元，无论其生产厂商及进行的应用，可以通过蓝牙交换信息从而发现个单元支持何种应用。所有蓝牙单元都必须支持GAP以保证基本的互操作性和共存性。
- ** 服务发现应用剖面（SDAP，Service Discovery Application Profile）**：定义如何发现蓝牙单元支持的业务，该剖面可以用来搜索已知的特定业务，也可以用来进行普通业务浏览搜索。
-  **串行端口剖面（SPP，Serial Port Profile）**：定义如何在两个设备之间建立虚拟串行端口，并用蓝牙将其连接。采用串行端口剖面可在蓝牙单元上仿真基于RS-232控制信令的串行线缆，该剖面可保证高达128kbit/s的数据速率。
- ** 普通对象交换剖面（GOEP，General Object Exchange Profile）**：定义处理对象交换的应用需采用的协议和程序，基于GOEP的应用模型（如文件传输、同步等）假定链路和信道已经建立如GAP所述，GOEP描述从一个蓝牙设备Push数据到另一个蓝牙设备的程序，还规定如何在两个单元之间Pull数据。

其他剖面成为应用剖面，主要面向各个应用。


#### 为什么设计那么多Pofiles ？

自从开始接触蓝牙，我就有一个疑问，就是为什么蓝牙有那么多的profile,以至于他把自己的协议栈弄得如此的复杂 ？ 而不像其他的网络协议一样只负责为通信实体提供信道，将其他的交给应用去做呢？

至今我仍然没有找到很好的资料去解释这一问题，但我们可以大概的从此类问题通用的角度去考虑， 我们有几个不错的角度：

#####1. 组织架构
这里借用《计算机网络 第五版》中的一段话：
>真的有必要分清楚所有应用的细节，并且为每一种应用提供不同的协议栈吗？也许没有这个必要。但是，由于存在多个不同的工作组，他们分别负责设计标准的不同部分，因此，每个工作组都只关注特定的问题，从而形成了自己的profile.ni可以把这个看成是Conway法则在起作用。或许蓝牙标准根本不用25个协议栈，两个就可以了，一个用于文件传输，另外一个用于流式实时通信。

这里他的观点是因为蓝牙兴趣小组是各自为战的，因此缺少必要的协同而导致的蓝牙协议栈的分裂，最终形成了几十个协议栈并存的局面。

也就是最初各个协议的标准可能是由各个公司自己研发，最终经过蓝牙标准组织认定的。

#####2. 历史发展

由于组织架构的原因，各个公司组织将自己设计的通信标准纳入到了蓝牙标准中去，形成了特有的profile式的协议栈结构，后来随着技术发展，新的事物新的技术不断出现，当需要为蓝牙标准添加新的场景的时候，就只能遵循现有的蓝牙技术框架，不断地为其添加profile。

#####3. 顶层设计

虽然没有任何材料的佐证，但是我觉得蓝牙协议栈的问题可能不仅仅是组织架构问题和松散兴趣联盟话语权的妥协，我始终觉得一个得以流行全世界的一种技术，一定经过了一定指向性和预见性的顶层设计的，一定是经过利弊权衡后的结果，而绝非简单的Conway法则的必然呈现。

我能够想到的就是对比于其他的网络协议核心的特点就是** 协议栈定制性 **, 而相对于其他的而言就是通用性和扩展性的上的缺陷，我们来从概念上思考一下，我们可以猜测到一下的优点：

(1) 避免了通用性带来的资源浪费和设计冗余，定制化可以针对特定的应用优化通信流程，帧结构等提高传输效率，稳定性和节省成本。
(2) 分散设计带来的设计成本的减少，拼接式的协议栈构最大程度的接纳每一种场景设计而避免了协议并入的冲突，减少了各个企业成员之间的协同成本，提高了设计效率。
(3) 特定的终端不必要仅仅需要实现特定的profile即可实现目的，适用于功能单一而且低功耗终端。
(4) 减小了企业的设计成本和难度，利于蓝牙技术的推广。
(5) 推动了场景标准化，打通设备和应用阻隔。


当然以上的很多都是我自己的猜测，需要更多的资料去论证，先记录下来，以后不断修正。

#### 对于各profile的应用和未来的思考
2011年之前我们还拿着诺基亚，用着每月30M的2G网络，不得不使用手机蓝牙和朋友们交换照片，mp3，电子书，可是当智能机时代，4G网络，家庭WIFI的到来，很少人再用蓝牙去传输一个小小的文件了，甚至我们都使用其他的任何局域自组网技术，直接走Internet来传输了。随着时代合计数的进步，很多的蓝牙profile必然会被抛弃，而留下的将会是特定化用途的不可取代的profile。

其实很多的蓝牙技术我们生活中也很少能够见到，以有限的未来来看，我觉的能够保存不错的活跃度的profile有两种：
1. 跟无线耳机音箱有关的，handsfree,A2DP,AVRCP等。这些profile用于处理电话，音频等相关的场景。
2. GATT based profile。该profile是蓝牙低功耗标准，随着以后智能化穿戴设备和各种随身传感器的兴起，BLE将会是蓝牙的一个突破口。

以我自己的观点来看，在近几年，我们主要会以蓝牙作为个人自组网的连接方式，而WIFI会作为室内或者家庭的组网方式。

当然未来的事情谁也说不准，我们在过去的几年里见识过，预见不了未来并不意味着没必要去想象未来，只有做好准备，他来的时候，你才会淡定的说，你和我想的差不多。

#### 参考
>**Bluetooth spec 4.2**
**《计算机网络 第五版》**
> **《蓝牙协议及其实现》**