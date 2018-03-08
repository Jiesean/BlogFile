---
title: 低功耗蓝牙BLE协议栈简介
date: 2016-10-24 12:50:52
tags: [BLE,BT]
---


BLE,blooth low power，即蓝牙低功耗技术。
该技术具有低成本、短距离、可互操作的特性，工作在免许可的2.4GHz ISM射频频段。
###协议栈

![BLE协议栈](http://upload-images.jianshu.io/upload_images/1806858-7bf1550dbb716f69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<!-- more -->

蓝牙系统核心包括射频收发器，基带和协议栈。核心系统协议包括射频(RF)协议、链路控制(LC)协议、链路管理(LM)协议、逻辑链路的控制和适配(L2CAP)协议。 蓝牙核心系统最底三层是射频，链路控制，链路管理协议，通常会把这三者归为一个子系统——蓝牙控制器。把往上的其他层一起称为为蓝牙主机。在蓝牙控制器和蓝牙主机之间实现通信通常需要有主机-控制器接口，Host to Controller Interface(HCI)。蓝牙系统的具体应用apps，就是建立在蓝牙主机之上。而host部分由蓝牙软件厂商开发和维护，control部分由蓝牙的硬件厂商提供，两部分通过hci(主机控制器接口)进行通信和数据交互。

##### Direct Test Mode 
厂商提供的测试模块，可以通过HCI或者串口直接控制蓝牙的物理层来让它收发数据包
#####Physical Layer（PHY）物理层
负责数据和语音的发送和接收，特点是短距离、低功耗。蓝牙天线一般体积小、重量轻，属于微带天线。
1Mbps自适应跳频GFSK（高斯频移键控），运行在免费的工业频段2.4GHz。
#####Link Layer（LL）链路层
LL层为RF控制器，控制设备处于准备（standby）、广播、监听/扫描（scan）、初始化、连接，这五种状态中一种。
五种状态切换描述为：未连接时，设备广播信息，另外一个设备一直监听或按需扫描，两个设备连接初始化，设备连接上了。
发起聊天的设备为主设备，接受聊天的设备为从设备，同一次聊天只能有一个意见领袖，即主设备和从设备不能切换。

#####Host-Controller Interface（HCI）主机控制器接口

HCI层为接口层，向上为主机提供软件应用程序接口（API），对外为外部硬件控制接口，可以通过串口、SPI、USB来实现设备控制。
#####L2CAP逻辑链路控制和适配协议
L2CAP层提供数据封装服务，允许逻辑上的点对点通讯。
基于包的协议，将包传输到HCI，对于无主机系统，就将包传给链路管理器LM。支持多路复用，包的分割和重组，以及向上层协议提交服务质量信息。
#####Security Manager（SM）安全管理
SM层提供配对和密匙分发，实现安全连接和数据交换。
#####Attribute Protocal（ATT）属性协议
ATT层负责数据检索，允许设备向另外一个设备展示一块特定的数据称之为属性，在ATT环境中，展示属性的设备称之为服务器，与它配对的设备称之为客户端。链路层的主机从机和这里的服务器、客服端是两种概念，主设备既可以是服务器，也可以是客户端。从设备毅然。
#####Generic Attribute Profile（GATT）通用属性协议
GATT层定义了使用 ATT 的服务框架和配置文件（profiles）的结构。BLE 中所有的数据通信都需要经过GATT。
它定义两个 BLE 设备通过叫做 **Service** 和 **Characteristic** 的东西进行通信。GATT 就是使用了 ATT（Attribute Protocol）协议，ATT 协议把 Service, Characteristic遗迹对应的数据保存在一个查找表中，次查找表使用 16 bit ID 作为每一项的索引。


![gatt结构](http://upload-images.jianshu.io/upload_images/1806858-c13b1f25f4da4525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####Generic Access Profile（GAP）通用访问协议
GAP直接与应用程序或配置文件（profiles）通信的接口，处理设备发现和连接相关服务。另外还处理安全特性的初始化。对上级，提供应用程序接口，对下级，管理各级职能部门，尤其是指示LL层控制室五种状态切换，指导保卫处做好机要工作。
GAP给设备定义了若干角色，其中主要的两个是：外围设备（Peripheral）和中心设备（Central）。
**外围设备**：这一般就是非常小或者简单的低功耗设备，用来提供数据，并连接到一个更加相对强大的中心设备。例如小米手环。
**中心设备**：中心设备相对比较强大，用来连接其他外围设备。例如手机等.


>图片来自BLUETOOTH SPECIFICATION Version 4.2和网络。
内容主要参考BLUETOOTH SPECIFICATION Version 4.2 ，
部分来自有网络http://blog.csdn.net/ooakk/article/details/7302425

