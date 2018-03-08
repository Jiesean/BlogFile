---
title: Android BLE开发之初识GATT
date: 2017-02-22 12:31:03
tags: [Android,BLE,BT]
---


### 前言

BLE是在Android 4.3上被引入的，并在android 5.0上加入了ble advertise的API支持。其时，IOS上的BLE已经玩的风生水起，其中IOS的ANCS服务就是基于BLE封装的通知下发协议，而IBeacon是基于BLE广播的单向传输的应用。
Android上的BLE开发算是个后发者，但是android开放了对BLE的支持后，我们就可以又很多好玩的东西可以尝试了，比如以前IOS的蓝牙是无法与Android的蓝牙进行通信的，有了BLE这就有了可能，还有读取IOS的下发通知，自己实现和android手表的通信，读取手环的数据等等有趣的东西。

而中文社区中关于Android BLE的开发资料几乎全是android官网上[BLE开发指南](https://developer.android.google.cn/guide/topics/connectivity/bluetooth-le.html#terms)的原版翻译，但是该网站上的内容是基于Android 4.3来写的，其中的API已经过时，最新的BLE API应该是基于Android 5.0的，所以我根据官网的思路基于最新的API最简单的方式介绍Android的BLE开发。
以后还会把跟BLE相关的内容都整理下来，作为一个系列。

<!-- more -->

### 基本概念
想要进行Ble相关的开发，我们必须具备一定的基础知识，当然基础知识肯定是非常简单的。
##### 设备角色
首先要明白的是，这两种角色的区分是硬件层面上，而且是成对出现的相对概念：
** 中心设备（Central device） **：功能相对强大，用来扫描和连接周边设备的，例如手机、平板等
** 周边设备（Central device） **：功能相对简单，功耗较小，被中心设备连接以提供数据的，例如手环、智能体温计等


其实从最根本上来讲，它应该是在对建立连接的过程不同角色的一种区分。我们知道蓝牙设备要想让别人知道自己的存在，是要不间断的对外放松广播的，而另外一方则需要扫描并回复该广播包，这样才能建立连接，在这个过程中，负责广播的就是peripheral，而负责扫描的是Central。

关于两者的连接过程需要注意：
- 中心设备可以同时连接多个周边设备。
- 周边设备一旦被连接上，立刻停止广播，断开后继续广播
- 任何时候只能一个设备尝试连接，排队连接。

##### GATT
BLE技术是基于GATT进行通信的，GATT是一种属性传输协议，简单的讲可以认为是一种属性传输的应用层协议。
它的结构非常简单：
![GATT结构图](http://upload-images.jianshu.io/upload_images/1806858-265d374ad9bb425f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
你可以把他看成xml来理解：
- 每个GATT由完成不同功能的Service组成；
- 每个Service由不同的Characteristic组成；
- 每个Characteristic由一个value和一个或者多个Descriptor组成；
- Service、Characteristic相当于标签（Service相当于他的类别，Characteristic相当于它的名字），而value才真正的包含数据，Descriptor是对这个value进行的说明和描述，当然我们可以从不同角度来描述和说明，因此可以有多个Descriptor.

这样子理解可能不够准确，下面我们来举一个简单的例子进行说明：

常见的小米手环是一个BLE设备，（假设）它包含三个Service,分别是提供设备信息的Service、提供步数的Service、检测心率的Service;
而设备信息的service中包含的characteristic包括厂商信息、硬件信息、版本信息等；而心率Service则包括心率characteristic等，而心率characteristic中的value则真正的包含心率的数据，而descriptor则是对该value的描述说明，比如value的单位啊，描述啊，权限啊等。


##### GATT C/S
对GATT有了初步的了解，我们知道GATT是一种典型的C/S模式，既然是C/S那么我们就有必要对Server和client进行区分。

** GATT server ** vs. ** GATT client **。这两种角色存在的阶段则是建立连接之后，根据对话地位的不同进行区分的，很容易理解的是，保有数据的那一方我们称之为GATT server，访问数据的那一方我们称之为GATT client。

这和我们之前提到的设备角色是不同层面的概念，有必要加以区分，我们还是用一个简单的例子进行说明：

以手机和手表的例子来进行说明，手机和手机建立连接之前，我们都是用手机的蓝牙搜索功能去搜索手表的蓝牙设备，这个过程中很明显手表在进行BLE广播以便其他设备知道自己的存在，它在这个过程中就是peripheral的角色，而手机负责扫描的任务，自然扮演的就是Center了；两者建立了GATT连接后，当手机需要从手表中读取步数等传感器数据时，两者交互的数据是保存在手表中的，因此此时手表就是GATT server的角色，自然手机就作为GATT client；而当手表想要从手机读取短信电话等信息室，数据的保佑者又变成了手机，所以此时手机就是server ，而手表则是client。

##### Service/Characteristic
上面我们已经对他们有了感性的理解，接下来我们来一些实用的信息：
1. Characteristic是最小的数据逻辑单元。现在不难理解了吧。
2. value、descriptor中存储数据的解析由Server的工程师决定，并无规范，双发按照约定开发。
3. Service/Characteristic均有一个唯一的UUID标识，UUID既有16位的也有128位的，我们需要了解的是16位的UUID是经过蓝牙组织认证的，是需要购买的，当然也有一些通用的16位UUID。
例如Heart Rate服务的UUID就是0X180D,代码中表示为0X00001800-0000-1000-8000-00805f9b34fb,其他位为固定的。而128位的UUID则可以自定义。
4. GATT连接是独占的。


### 应用开发
##### 添加权限
进行蓝牙APP的开发，需要在manifest文件中加入如下的权限：
```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```
BLUETOOTH权限使得你的APP可以使用蓝牙的对话功能，例如建连和数据的传输。
BLUETOOTH_ADMIN权限允许APP启动设备的被发现以及操作蓝牙的settings。
其他更详细的查看官网上的说明[BLE开发指南](https://developer.android.google.cn/guide/topics/connectivity/bluetooth-le.html)
##### 获得蓝牙
要想进行ble的开发首先要获得设备上的蓝牙适配器并保证蓝牙是使能的，这样才能进一步的进行ble的相关操作。
1. 获得蓝牙适配器
系统启动的时候蓝牙相关的系统服务已经开启，这时候我们首先要获得系统的蓝牙服务：
```
BluetoothManager bluetoothManager =
        (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
```
通过蓝牙的系统服务得到蓝牙适配器：
```
BluetoothAdapter mBluetoothAdapter = bluetoothManager.getAdapter();
```
得到蓝牙适配器之后，我们就能进行蓝牙的相关操作，无论是经典蓝牙还是ble都可以，当然进行这些操作之前我们首先使能蓝牙。
2. 蓝牙的使能
这个当然是为了保证蓝牙是开着的，给蓝牙芯片使能。
```
if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
}
```
这段代码是调用了系统提供的开启蓝牙的对话框，点击即可使能蓝牙，其实本质也是调用BluetoothAdapter.enable()。
3. 扫描Ble设备
```
//1. Android 4.3以上，Android 5.0以下
mBluetoothAdapter.startLeScan(BluetoothAdapter.LeScanCallback LeScanCallback)
//2. Android 5.0以上，扫描的结果在mScanCallback中进行处理
mBluetoothLeScanner = mBluetoothAdapter.getBluetoothLeScanner();
mBluetoothLeScanner.startScan(ScanCallback mScanCallback);
```
注意传入的callback参数是不同的。以下我们都按照5.0的API进行。

4. 得到扫描结果
在扫描结果的callback函数中将扫描到的设备的名称和地址进行打印。
```
        @Override
        public void onScanResult(int callbackType, ScanResult result) {
            if(result != null){
                System.out.println("扫面到设备：" + result.getDevice().getName() + "  " + result.getDevice().getAddress()); 
            }
        }
```
因为扫描本身是个耗电操作，因此扫描到目标设备后应该立即体停止扫描
```
if(mTargetDeviceName.equal(result.getDevice().getName())){
        mBluetoothLeScanner.stopScan(mScanCallback);
}
```

5. 对目标设备进行连接
```
result.getDevice().connectGatt(MainActivity.this, false, mGattCallback);
```
传入的BluetoothGattCallback对象中对连接结果做处理，以及通过GATT进行通信的绝大多数的操作都在这个对象中，至于后续BluetoothGattCallback的讲解，我们将在demo以及后续的文章中进行。

### 应用demo之初识gatt
Android官网的例子中有BLE的示例，但是那个例子有些API已经过时，没有使用最新的5.0的API,而且也写得过于麻烦，不便于开发新手入门，因此我写了一个简单的demo,仅仅包含整个扫描、连接、获取service的过程，一眼便能看懂，当然这个demo的目的是增进GATT的认识。

Demo的下载地址：
[** Github Jiesean :  BleDemo **](https://github.com/Jiesean/BleDemo)

运行Demo并连接小米手环1代得到的结果如下：
```
Services num:6
扫描到Service：00001800-0000-1000-8000-00805f9b34fb
characteristic: 00002a00-0000-1000-8000-00805f9b34fb
characteristic: 00002a01-0000-1000-8000-00805f9b34fb
characteristic: 00002a02-0000-1000-8000-00805f9b34fb
characteristic: 00002a04-0000-1000-8000-00805f9b34fb
扫描到Service：00001801-0000-1000-8000-00805f9b34fb
characteristic: 00002a05-0000-1000-8000-00805f9b34fb
扫描到Service：0000fee0-0000-1000-8000-00805f9b34fb
characteristic: 0000ff01-0000-1000-8000-00805f9b34fb
characteristic: 0000ff02-0000-1000-8000-00805f9b34fb
characteristic: 0000ff03-0000-1000-8000-00805f9b34fb
characteristic: 0000ff04-0000-1000-8000-00805f9b34fb
characteristic: 0000ff05-0000-1000-8000-00805f9b34fb
characteristic: 0000ff06-0000-1000-8000-00805f9b34fb
characteristic: 0000ff07-0000-1000-8000-00805f9b34fb
characteristic: 0000ff08-0000-1000-8000-00805f9b34fb
characteristic: 0000ff09-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0a-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0b-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0c-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0d-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0e-0000-1000-8000-00805f9b34fb
characteristic: 0000ff0f-0000-1000-8000-00805f9b34fb
characteristic: 0000ff10-0000-1000-8000-00805f9b34fb
扫描到Service：0000fee1-0000-1000-8000-00805f9b34fb
characteristic: 0000fedd-0000-1000-8000-00805f9b34fb
characteristic: 0000fede-0000-1000-8000-00805f9b34fb
characteristic: 0000fedf-0000-1000-8000-00805f9b34fb
characteristic: 0000fed0-0000-1000-8000-00805f9b34fb
characteristic: 0000fed1-0000-1000-8000-00805f9b34fb
characteristic: 0000fed2-0000-1000-8000-00805f9b34fb
characteristic: 0000fed3-0000-1000-8000-00805f9b34fb
扫描到Service：0000fee7-0000-1000-8000-00805f9b34fb
characteristic: 0000fec7-0000-1000-8000-00805f9b34fb
characteristic: 0000fec8-0000-1000-8000-00805f9b34fb
characteristic: 0000fec9-0000-1000-8000-00805f9b34fb
扫描到Service：00001802-0000-1000-8000-00805f9b34fb
characteristic: 00002a06-0000-1000-8000-00805f9b34fb
```
我们可以看到总共得到了6个Service,分别是：
- 1800 [Generic Access](https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.service.generic_access.xml)
- 1801 [Generic Attribute](https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.service.generic_attribute.xml)
- fee0 安徽华米公司购买
- fee1 安徽华米公司购买
- fee7 腾讯公司购买
- 1802 Immediate Alert

下一篇文章我们将对小米手环进行读取和操作。

<p>
> ** 相关阅读： **
> [** BLE简介 **](http://www.jianshu.com/p/495f435e616d)
> [** 低功耗蓝牙BLE协议栈简介 **](http://www.jianshu.com/p/9ab3efe6147d)
