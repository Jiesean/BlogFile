---
title: BLE简介
date: 2016-12-18 22:37:00
tags: [BLE,BT]
---


>[Introduction to Bluetooth Low Energy](https://learn.adafruit.com/introduction-to-bluetooth-low-energy/introduction)

## 简介
Bluetooth Low Energy (BLE)，也经常被称为**Bluetooth Smart**，它是传统蓝牙的子集，在Bluetooth 4.0 core specification中被引入。虽然BLE和传统蓝牙有很多重叠的地方，但是它是有自己独特的历史血统的，BLE被Bluetooth SIG即蓝牙标准纳入之前，一直是诺基亚发展的一个叫做Wibree的室内项目。

对工程师们来说虽然有很多种的无线协议可供选择，但是BLE是实现和现代移动平台通信的最简单的方式，尤其是对于苹果设备而言，BLE可以说是唯一的可以避免为了成功的提交苹果商店的硬件设计选择。

以下的简介将使你有一个快速对BLE的总体的了解，尤其是数据如何组织，以及设备如何通过广播来告之其存在，让你可以连接他们并进行数据的传输。

<!-- more -->

## BLE平台支持
以下列出的设备和平台均支持蓝牙 4.0和BLE:
- iOS5+ (iOS7+ preferred)
- Android 4.3+ (numerous bug fixes in 4.4+)
- Apple OS X 10.6+
- Windows 8 (**XP, Vista and 7 only support Bluetooth 2.1**)
- GNU/Linux Vanilla BlueZ 4.93+

# GAP 

GAP是**Generic Access Profile**的缩写，它控制着蓝牙的连接和广播过程。GAP协议使得设备可以被其他设备识别，并决定两个设备如何交互。
##### 设备角色

GAP为设备定义了各种各样的角色，但两个最关键的概念是中心设备（**Central** devices）和周边设备（*Peripheral** devices）。

周边设备是指那些体积较小，功耗较小，资源有限的设备，它可以连接更加强大的中心设备，例如各种手环，手表，心跳检测器等；中心设备通常指的是手机或者平板等拥有更强大的计算和存储的设备。

下图可以详细的说明了整个广播的过程，以及广播负载和扫描回复负载是如何工作的。
周边设备会设置一个特定的广播的时间间隔（Advertising Interval），每个时间段的开始，它会发送其广播数据包，时间间隔越长节省电量，但是不能及时被扫描到。这是很明显的道理。

周边设备的广播包括两种，一种是Advertising data,一种是scan response data，广播数据就是上面所说的，是必选，每一广播间隔开始都会不间断发送，通过它来告诉中心设备自己的存在；扫描回复则是可选的，它包含了设备的基本信息，比如设备的名字和地址等，只有当中心设备对他感兴趣，发送扫描回复请求时，周边设备才会发送扫描回复数据包作为反馈。

![广播示意图](http://upload-images.jianshu.io/upload_images/1806858-145908cdba8bf6bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 广播网络拓扑
尽管大多数情况下，周边设备广播自己都是为了建立连接后使用GATT services、characteristics完成更多的双向的数据交换，但也有些情况是不需要连接的，只需要周边设备将数据广播出去即可。

这种情况主要存在于需要周边设备**同时**给多个中心设备发送数据，而一旦建立连接，数据的收发就只对在建立连接的两台设备间可见。

周边设备可以发送一段包含少量自定义数据的**31字节**的广播或者扫描回复包给在监听范围的所有设备，这就是典型的BLE广播的过程。

通俗点的讲就是BLE的广播有两种作用，一种是告诉中心设备自己的存在然后等待连接；另一种就是单纯的对外广播信息，苹果的iBeacon就是后者的定性应用，它在广播包Manufacturer Specific Data中插入了一段自定义的数据，用来完成特定的功能。

![microcontrollers_BroadcastTopology](http://upload-images.jianshu.io/upload_images/1806858-ec9d45893d12e809.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

周边设备和中心设备一旦建立连接，周边设备的广播过程就停止了，也不能再发送广播包了，接下来就要通过GATT services和characteristic进行通信。

### GATT

GATT全称**Generic Attribute Profile**，中文名叫通用属性协议，它定义了**services**和**characteristic**两种东西来完成低功耗蓝牙设备之间的数据传输。它是建立在通用数据协议**Attribute Protocol (ATT)**,之上的，ATT把services和characteristic以及相关的数据保存在一张简单的查找表中，该表使用16-bit的id作为索引。

一旦两个设备建立了连接，GATT就开始发挥作用，同时意味着GAP协议管理的广播过程结束了。但是必须要知道的是，建立GATT连接必要经过GAP协议。

最重要的事情，GATT连接是**独占的**，也就意味着一个BLE周边设备同时只能与一个中心设备连接。一旦周边设备与中心设备连接成功，直至连接断开，它不再对外广播自己的存在，其他的设备就无法发现该周边设备的存在了。

周边设备和中心设备要完成双方的通信只能通过建立GATT连接的方式。

##### 网络连接拓扑

下图展示了BLE设备如何工作的。一个周边设备只能同时连接一台中心设备，但是中心设备可以连接多台周边设备。

如果两个周边设备需要进行数据的交换的话，就必须经由中心设备的中转。
一旦周边设备和中心设备建立了连接，通信就是双向的了，对比前面的GAP广播的网络拓扑，通信是单向的，只能由周边设备往中心设备广播数据。

![GATT网络拓扑](http://upload-images.jianshu.io/upload_images/1806858-115e2ea4e0bf65d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### GATT事务

GATT是基于典型的C／S模式，其中周边设备通常扮演**GATT Server**，它保有services、characteristic以及查找表，也就是数据的存储是在**GATT Server**中；而例如手机平板等中心设备通常是**GATT Client**，他们向**GATT Server**发送请求。一切都是主设备GATT Client发起，然后接受来自从设备GATT Server的相应。

当连接建立之后，周边设备会给中心设备建议一个连接间隔（Connection Interval），然后中心设备每个时间间隔都会重连查看是否有新数据可以获取。但是这个连接间隔只是一个建议，因为中心设备可能忙于与其它周边设备通信或者系统资源不可得而并不能完全遵循。

下图展示了周边设备和中心设备的数据交换的流程，可以看出每一次事务都是由中心设备发起的，而周边设备只负责应答。

![通信事务流程](http://upload-images.jianshu.io/upload_images/1806858-a12534b707378d56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Services and Characteristics**

BLE GATT通信是基于嵌套的**Profiles**, **Services** and**Characteristics**结构之上的，下图是其框架：

![GATT数据存储结构](http://upload-images.jianshu.io/upload_images/1806858-cfc71e2b28e99a8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Profiles**

Profile并不是真实存在的一种结构，而是多个完成某一特定功能services的集合，或者说是对这个特定结合的功能的描述，或者名称。以[Heart Rate Profile](https://developer.bluetooth.org/TechnologyOverview/Pages/HRP.aspx)为例，它包括Heart Rate Service 和 Device Information Service，他们都是为了完成测量心率这个功能而存在的service。

更详细的可以查看[Profiles Overview](http://developer.bluetooth.org/TechnologyOverview/Pages/Profiles.aspx#GATT)。
- **Services**

每个service拥有一个唯一标识UUID,可以是官方认证的16bit的id，也可以是128位的自定义id。service是GATT数据的逻辑分类，它包含一个或者多个characteristic。

官方通过了一些标准 Service，完整列表在[这里](https://developer.bluetooth.org/gatt/services/Pages/ServicesHome.aspx)。以 [Heart Rate Service](https://developer.bluetooth.org/gatt/services/Pages/ServiceViewer.aspx?u=org.bluetooth.service.heart_rate.xml)为例，可以看到它的官方通过 16 bit UUID 是 0x180D
，包含 3 个 Characteristic：*Heart Rate Measurement*, *Body Sensor Location* 和 *Heart Rate Control Point*，实现该service第一个*Heart Rate Measurement*是必选的，其他两个是可选的。

- **Characteristics**

Characteristic也拥有一个16-bit 或者128-bit的UUID，它是GATT通信中的最小的逻辑数据单元，它封装了一个单一的数据点，当然这个数据点可能包含一组相关的数据，比如加速度传感器的x/y/z三个坐标轴的数据。

实际上，和 BLE 外设打交道，主要是通过 Characteristic。你可以从 Characteristic 读取数据，也可以往 Characteristic 写数据。这样就实现了双向的通信。你可以使用Characteristic实现一个类似串口（UART）的 Sevice，这个 Service 中包含两个 Characteristic，一个被配置只读的通道（RX），另一个配置为只写的通道（TX）。