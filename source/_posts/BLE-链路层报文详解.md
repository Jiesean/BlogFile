---
title: BLE 链路层报文详解
date: 2017-07-06 17:12:20
tags: [BLE,BT]
---


### 报文结构

报文是构成链路层的基石。报文就是携带着标签的数据，有一个设备发送，其他设备接收。

![报文结构](http://upload-images.jianshu.io/upload_images/1806858-134803fb30d4aebb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 比特序列
1. 数据按照字节传输时，总是从最低位开始传输的，例如0x01是1000000
2. 多个字节组成的数据总是从低字节开始传输，例如0x0103发送序列是11001000


<!-- more -->

### 前导

前导序列是一个01010101或者10101010的8bit交替序列。如果接入地址的第一位是0，前导序列则是01010101，否则是10101010，这样的设计是为了保证报文的前9位都是交替的。

这个前导序列的作用主要是为了做 数据接收的训练，比如频率同步，符号时间评估，AGC训练。通过这段数据的训练使得接收器调整到合适的状态对接下来的数据进行接收。

### Access Address 

接入地址是一个32位的地址，包含两种类型:
- 广播接入地址（广播数据，扫描或者发起连接）
- 数据接入地址（两个设备建立连接之后）

考虑到无线通信存在的噪音干扰和其他链路的干扰，就涉及了接入地址用来排除噪音和其他干扰数据包。
例如广播介入地址是一个固定值10001110100010011011111011010110b (0x8E89BED6) ，当接受到广播后验证接入地址正确后才认为他是个广播报文而不是噪音。
而对于数据报文的介入地址则是随机地址，不同的连接有不同的值，这是为了进行专线交流用的。

![广播报文](http://upload-images.jianshu.io/upload_images/1806858-2341d1bd92c63567.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从BLE空口包中的Link Layer 可以看到每个报文都有一个Access Address，Adv_pkt有一个固定Access Address，而intiator发con_req时会包含一个Con_Access_Addr，连接之后的data pkt都是用的这个新的Access Addr了；  每次重新断开建立连接，Access Address会不一样，都会重新随机生成。

如下图，广播包使用固定的0x8E89BED6
![ADV_PKT](http://upload-images.jianshu.io/upload_images/1806858-779b9530b1fe7a83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如下图，连接请求包使用协商通知新的地址，此处为0x5b71c59b
![CONNECT_IND](http://upload-images.jianshu.io/upload_images/1806858-9b7381b38f45ead0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如下图，连接过程中使用的地址则为connect_ind中协商
![LE DATA](http://upload-images.jianshu.io/upload_images/1806858-fc8a5fc42ee0c527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实关于Access Address是还有一些要求的，如不能有连续的6个1 bit位或6个连续的0 bit位，因此这样算下来，满足作为Access Address的4byte的组合中有231个是可用的

### 报头
报头虽然只有8bit，但是包含了不少的信息，而且他的内容因是广播报文还是数据报文差异很大。我们将依次来分析他们的结构和内容。
##### 广播报文的Header

![ADV PKT Header](http://upload-images.jianshu.io/upload_images/1806858-c0e29b25e0d00fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

广播报文类型包括7种：
- ADV_IND —— 通用广播指示
- ADV_DIRECT_IND —— 定向连接指示
- ADV_NONCONN_IND —— 不可连接指示
- ADV_SCAN_IND —— 可扫描指示
- SCAN_REQ —— 主动扫描请求
- SCAN_RSP —— 主动扫描响应
- CONNECT_REQ —— 连接请求

其中根据其作用域，还可以将他们分为三类，ADV_* 属于广播数据单元，
SCAN_* 属于扫描数据单元，CONNECT_* 属于发起数据单元。

发送地址类型和接收地址类型都是BLE地址类型，只有两种，公有地址和随机地址，因而用一个bit就可以进行区分。公有地址是需要向IEEE申请和购买的类似于公网ip，但是随机地址则不需要。

##### 数据报文的Header

![DATA PKT Header](http://upload-images.jianshu.io/upload_images/1806858-931f06dba1943804.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

逻辑链路标识符，主要是用来判断数据报文的类型：
- 链路控制报文（11）—— 链路层用来进行管理连接的控制报文，这种类型的报文直接交给链路层来进行处理。
- 上层报文的开始（10）—— 表明该报文是属于host，需要交给host才能进行解读和处理，因为host最多能够发送27个字节的数据，但是无法一次性放入单个链路层数据报文，因此需要将它截成几段进行传输，该标志位用于标识host报文的初识包。
- 上层报文的延续（01）—— 标识host数据包的后续包。

以一个L2CAP数据包为例，可以分为以下结构：
![L2CAP数据包的链路层分包组成](http://upload-images.jianshu.io/upload_images/1806858-363fc8e0896014e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样带来两种后果：第一是链路层并不知道一个上层数据包究竟有多少个链路层报文组成，连续的数量不确定；第二是总是可以发送长度为零的延续包，这些长度为零的空包可以用来进行消息的确认。

** 序列号SN **建立连接后的第一个数据包序号为零；每次发送新的数据包时，其序列号与上一个数据包的序列号不同；
也就是说，如果序列号与上一个数据包的序列号相同则为重传报文，如果不同则是新的报文。

** 期待序列号NESN **期待序列号，就是告诉对方自己期待的序列号。这其实起到一个数据包确认的作用，如果确认数据包的期待序列号和发送的序列号相同，则说明对方期待我们重传刚刚的数据包，如果是不同的，则是希望我们发送新的数据报文。

** 更多数据 **更多数据位是用来告之对方，我方是否还有下一个数据包要发送，如果该位设置为1，则表明仍有数据要传送，对方则选择保持连接；设为0，则表明没有更多数据要进行发送，对方则会断开连接，节省能量。

一图以概之

![SN/NESN/MD说明实例](http://upload-images.jianshu.io/upload_images/1806858-e1b89fbc9dfde035.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. SN和NESN除了可以用来确认数据，保证数据的准确送达。
2. NESN除了确认数据，还可以用来进行流控制，当我方缓存区紧张时，可以不发送NESN给对方，使对方重发来等待我方释放足够的缓存区在进行确认处理。
3. 当报文CRC校验失败时，认为报文传输失败，要求重传。
4. 在同一连接里，出现两次数据包接受出错，则停止该连接，进行重新连接发送数据，这说明该信道很大的可能被干扰导致质量较差，需要重新连接更换信道。

### 长度
长度这个数据段很简单，表示数据段的实际数据的长度。

对于广播报文，该段是由6个bit组成，剩余两个bit留作未来使用；
对于数据报文，该段是由5个bit成，剩余三个bit留作未来使用；

他们长度需要bit不相同的原因是，广播数据报文多了6个字节的设备地址需要携带，因为需要6bit来标识长度；

### 数据（净荷）
最大传输的数据是31个字节，但是如果数据被加密，需要留出4个自己进行数据完整性校验
### 循环冗余校验码
24bit的循环校验码，可以校验所有基数位以及所有2，4位错误，显然其并不能校验所有位错误，但是出于低功耗考虑，这是一个妥协的产物。