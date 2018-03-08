---
title: Android BLE开发之玩转小米手环
date: 2017-03-01 19:19:35
tags: [Android,BLE]
---


### 前言

上一篇文章中我们已经认识了gatt的基本机构以及如何获得gatt中的Service以及Characteristic，接下来我们将学习对于Characteristic的基本操作，并使用这些基本操作，来操纵小米手环，实现一些有趣的功能。

我们可以大体想想一下小米手环所实现的功能：
- 计步获取
- 震动
- 电量信息获取
- Led颜色控制
- User信息
- 睡眠监测
等等。

<!-- more -->

这里我们选取一些有意思的功能进行实现，比如震动，led控制，计步获取等。

### porfile和协议的获取
因为之前我们已经可以对轻松的获得小米手环GATT的结构，所有的service、characteristic、descriptor等的UUID，但是我们对于每一个UUID所实现的功能，以及每个功能所使用的解析方式（也就是协议）是无从得知的。

这里我们能够想到的解决这些问题的方式有两种：
- 网络资料中获取。显然网上肯定有很多破解小米手环的文章，但是我们发现所有的文章里面也只有对获取步数等基本的信息，而且很多尝试后都是错误的。
- 自己动手反编译小米运动APP。首先这个APP肯定经过了混淆编译的，反编译后的源码阅读是很困难的，但是也不是不可读的，只要愿意花费时间，还是可以完全破解他的。但是毕竟时间有限，我们结合我们想要的，查找想要的信息。

最终我们获得以下的characteristic的UUID:
``` 
    //alertchar
    public static final UUID IMMIDATE_ALERT_CHAR_UUID = UUID.fromString("00002a06-0000-1000-8000-00805f9b34fb");
    //计步char:读取该UUID下的value数组 第0 个数据就是 步数
    public static final UUID STEP_CHAR_UUID = UUID.fromString("0000ff06-0000-1000-8000-00805f9b34fb");
    //电量信息
    public static final UUID BATTERY_CHAR_UUID = UUID.fromString("0000ff0c-0000-1000-8000-00805f9b34fb");
    //用户信息char
    public static final UUID USER_INFO_CHAR_UUID = UUID.fromString("0000ff04-0000-1000-8000-00805f9b34fb");
    //控制点char
    public static final UUID CONTROL_POINT_CHAR_UUID = UUID.fromString("0000ff05-0000-1000-8000-00805f9b34fb");

```
而我们想要实现的功能都将是对这些characteristic进行操作完成的。

### Characteristic的基本操作

任何BLE功能的实现都要对characteristic进行操作，
主要包括有三种：
1. 读取特征值
2. 写入特征值
3. 特征值的变化通知

以下我们将用代码说明如何进行读写特征值：

** 读取特征值 **

当从GATT中获取到该特征后，利用该特征的对象读取他的值：
```
mGatt.readCharacteristic(characteristic);
```
然后从读操作的回调函数中获取特征值：
```
@Override
public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
   Log.d(TAG, "onCharacteristicRead UUID : " + characteristic.getUuid());
   byte[] data = characteristic.getValue();
}
```

** 写特征值 **

写特征值的操作和读是一样的：
gatt写操作：
```
byte[] value;
characteristic.setValue(value);
mGatt.writeCharacteristic(characteristic);
```
回调中获取写的结果
```
@Override
public void onCharacteristicWrite(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
    Log.d(TAG, "onCharacteristicWrite UUID: " + characteristic.getUuid() + "state : " + status);
}
```
** 通知变化 **

这个名字好像并不能很好的说明这个功能是干什么的，但是看英文名字就一目了然了，setCharacteristicNotification,很显然就是设置监听特征值，监听到它发生变化后，就会触发回调函数：
```
mGatt.setCharacteristicNotification(characteristic, enable)
BluetoothGattDescriptor clientConfig = characteristic.getDescriptor(Profile.notificationDesUUID);
clientConfig.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
//clientConfig.setValue(BluetoothGattDescriptor.DISABLE_NOTIFICATION_VALUE
mGatt.writeDescriptor(clientConfig);
```
在Descritor的写入回调：
```
 @Override
 public void onDescriptorWrite(BluetoothGatt gatt, BluetoothGattDescriptor descriptor, int status) {
    Log.d(TAG, "onDescriptorWrite");
 gatt.readCharacteristic(batteryChar);

}
```
### 功能实现
我们知道了那个Characteristic实现什么功能，并知道了如何操作，在得到他们之间的协议（即怎么解析value值）皆可以实现以上的所有功能，下面我结合我们知道的简单的说一下他们的实现。
具体实现细节查看Demo的源码中xParser.java：
##### 计步
计步功能的可以由两种实现：
1. 每隔一段时间读取一下步数。
2. 事实显示步数，即每走一步更新一次。

很显然1是通过读取特征值，2是通过变化通知实现的。
读取特征值，解析后表示步数。
```
byte stepNum = characteristic.getValue;
```
##### 震动
震动很明显是通过写入特征值实现的，相当于往手环写入命令。
写入Immidate Alert特征值能够实现震动。
经测试，写入的数值为 0x01,0x2,0x03,0x04均能实现震动，震感好像是一样的，震动的次数好像不同，还未详细进行统计。
##### 电量信息
电量信息，包括了电量百分比显示、充电状态、上一次充电时间等均可以读取。
当然电量也是既可以读取，也可以通知了。
整个电量的信息通过解析电量特征值获得：
```
byte[] batteryInfo = characteristic.getValue;
```
##### Led控制
尚未实现，后续补充。
### Demo: MiBandReader
Github地址：[** MiBandReader **](https://github.com/Jiesean/MiBandReader.git)

- 震动(通过Immidate Alert实现，0x02震动10次)
- 获取实时步数
- 电量信息(百分比电量/充电状态/上次充电时间/充电次数)
- 修复连接断开问题，主动或者通过代码配对
 

### 调戏别人的手环
首先我们要知道的手环手边之类的周边设备有这样的一个特性：** 一旦建立了GATT连接就不再被扫描到 **，所以理论上我们是没有机会对别人的手环手表下手的，但是我们不能忽略的一个问题是，大家对于耗电的恐惧。
你扫描一下你周围的Ble设备就会发现有不少的Band，watch之类的ble设备，都没有跟手机进行连接，因为他们可能觉得看着蓝牙那个标志就觉得耗电吧，也不管你低功耗高功耗，直接一关了之。

所以，机会是有的，但是要考虑风险和后果。调戏之前先确定好调戏的对象具有充分的可调戏性。危险动作，后果自负，别问我怎么知道的！

>相关阅读
[Android BLE开发之初识GATT](http://www.jianshu.com/p/29a730795294)