---
title: Android BLE开发之操作IOS ANCS
date: 2017-03-27 18:27:52
tags: [Android,BLE,ANCS]
---

### 前言
之前写过两篇有关于ANCS的文章，最近一段时间老是有人问关于得到ANCS服务的问题，因为IOS ANCS不同于其他的Peripheral一样对周边所有的蓝牙设备广播自己，而是仅有连接上配对并连接上IOS设备的可见，我想这对于Android、IOS、嵌入式等的开发都是一样的。

现在将以前写的Android ble操作ANCS的demo修改了一下，并集中对于ANCS的相关问题进行说明。

## 发现ANCS
熟悉IOS的都知道，IOS设备上的蓝牙是有很大限制的，只能连接手表、耳机等周边设备，甚至同样是IOS平台的设备都不能进行互联。但是BLE的出现给了我们使用蓝牙技术进行通信的可能。

这里我们用到的是IOS系统提供的ANCS服务获取IOS分发的通知，包括消息、来电、计划等，但是这个服务对于我们是不可见的，他并不主动进行广播，我们使用BLE scan 并不能扫描到ANCS这个服务。

<!-- more -->

** 那么是不是意味着我们就无法找到这个ANCS服务了呢？ **
答案是否定的，经过调查我们发现ANCS是基于GATT做的封装，也就是他是一个BLE的gatt server，只是对通信过程加入了自定义的协议，他跟其他的Ble service是同等的，比如常见的Heart Rate。因此我们考虑通过其他的service连接上这个GATT server,然后在获取ANCS服务的思路。


![how_to_find_ancs.png](http://upload-images.jianshu.io/upload_images/1806858-4b0ec347f14faf2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 发现设备
想要连接蓝牙设备我们首先要知道他的设备地址，但是IOS设备的蓝牙是不主动广播的，但是我们知道IOS是支持BLE广播的。

这里我们就可以借由这个功能让我们得到IOS设备的目标地址，当然你可以自己实现一个简单的APP去startLeAdvertisment,因为我并不会IOS开发所以这里借助了一个第三方APP(LightBlue)来虚拟一个peripherial,例如Heart Rate.

然后在我们的Android APP中进行扫描，就能扫描到这个名称为Heart Rate的周边设备。
```
@Override
public void onScanResult(int callbackType, ScanResult result) {
    Log.d(TAG, "onScanResult Device Address :" + result.getDevice());

    BluetoothDevice device = result.getDevice();

    if (device.getName() != null && device.getName().equals("Heart Rate")) {
         mTargetDevice = device;
    }
}
```

##### 连接设备
使用我们上一步得到的BluetoothDevice对象或者设备地址链接到Gatt操作。
安卓代码示例如下：
```
device.connectGatt(getApplicationContext(), false, mGattCallback);
```
并在连接回调中，获得连接的状态：
```
private class LocalBluetoothGattCallback extends BluetoothGattCallback {

   @Override
   public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
       if (newState == BluetoothProfile.STATE_CONNECTED) {
           Log.d(TAG, "connected");

           mConnectedGatt = gatt;
       }
       if (newState == BluetoothProfile.STATE_DISCONNECTED) {
           Log.d(TAG, "disconnected");
           mConnectedGatt = null;
       }
   }

}
```

##### 发现ANCS
当连接上GATT后，我们就可以调用discover函数去发现所有的服务。
```
gatt.discoverServices();
```
然后在发现服务的回调中就可以根据UUID获得ANCS服务。
```
@Override
public void onServicesDiscovered(BluetoothGatt gatt, int status) {
    if (status == BluetoothGatt.GATT_SUCCESS) {
        BluetoothGattService ancsService = gatt.getService(UUID.fromString(Constants.service_ancs));
        if (ancsService == null) {
            Log.d(TAG, "ANCS cannot find");
        } else {
            Log.d(TAG, "ANCS find");

        	mANCSService = ancsService;
        	mDataSourceChar = ancsService.getCharacteristic(UUID.fromString(Constants.characteristics_data_source));
        	mPointControlChar = ancsService.getCharacteristic(UUID.fromString(Constants.characteristics_control_point));
        	mNotificationSourceChar = ancsService.getCharacteristic(UUID.fromString(Constants.characteristics_notification_source));


        }
    }
}
```
获得ANCS后，我们也可以通过UUID获得ANCS的三个characteristic.

##### 绑定设备
当我们进行ANCS操作的时候，就会弹出配对的请求的对话框要求我们来完成配对，如果我们不进行配对的话，就无法对ANCS进行操作，同时GATT连接也会经常自动断开连接。

因此我们一般是在扫描到设备后就与设备进行配对，完成配对后再与设备进行连接。

先判断是否已经在配对列表中，是，则进行连接，不是，则进行配对：
```
 //已经绑定，该设备在绑定的设备名单里面
 if (mBluetoothAdapter.getBondedDevices().contains(device)) {

     device.connectGatt(getApplicationContext(), false, mGattCallback);
     mBluetoothLeScanner.stopScan(mScanCallback);
 } else {//未绑定的设备
     device.createBond();
 }
```
通过接受系统的BondStateChanged广播接受绑定成功的消息：
```
if (BluetoothDevice.ACTION_BOND_STATE_CHANGED.equals(action)) {
    if (intent.getIntExtra(BluetoothDevice.EXTRA_BOND_STATE, -1) == BluetoothDevice.BOND_BONDED) {
        showMessage("Bluetooth bond success！");
    }
}
```

## 操作ANCS
操作ANCS就是操作ANCS服务下的三个Characteristic,其操作也无非是BLE characteristic的三种操作：
- 读取（read）
- 写入（write）
- 通知（setNotification）

详细可见最后一章参考文章《Android BLE开发之玩转小米手环》。

因为需要通过对Notification Source、Data Source、Control Point进行读写通知操作完成所有的功能，因此对操作的流程、通知数据包的格式、命令的格式进行了规定，相当于应用层的协议，具体的可以参考ANCS分析的两篇文章。

- Notification Source（setNotification）:获取通知基本信息
- Data Source（setNotification）：获取通知的详细信息
- Control Point （write）: 写入通知控制命令

接下来我们主要从代码上来说明如何操作。

##### 获取通知 
1. Data Source通知开启
```
 private void setNotificationEnabled(BluetoothGattCharacteristic characteristic) {
    mConnectedGatt.setCharacteristicNotification(characteristic, true);
    BluetoothGattDescriptor descriptor = characteristic.getDescriptor(UUID.fromString(Constants.descriptor_config));
    if (descriptor != null) {
        descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
        mConnectedGatt.writeDescriptor(descriptor);
    }
}
```
2. Data Source开启后，Notification Source通知开启。
3. Notification Source的回调中获取通知基本信息。
```
System.out.println(
    "EventId:" + String.format("%d", nsData[0]) + "\n" +
    "EventFlags:" + String.format("%02x", nsData[1]) + "\n" +
    "Category id:" + String.format("%d", nsData[2]) + "\n" +
    "Category Count:" + String.format("%d", nsData[3]) + "\n" +
    "NotificationUId:" + String.format("%02X", nsData[4]) + String.format("%02X", nsData[5])+ String.format("%02X", nsData[6]) + String.format("%02X", nsData[7]) + "\n"
);
```
4. 往Control Point中写入获取更多通知信息的命令。
```
 private void getMoreAboutNotification(byte[] nsData) {
        byte[] getNotificationAttribute = {
        (byte) 0x00,
        //UID
        nsData[4], nsData[5], nsData[6], nsData[7],
        //app id
        (byte) 0x00,
        //title
        (byte) 0x01, (byte) 0xff, (byte) 0xff,
        //message
        (byte) 0x03, (byte) 0xff, (byte) 0xff
    };

    if (mConnectedGatt != null) {
        mPointControlChar.setValue(getNotificationAttribute);
        mConnectedGatt.writeCharacteristic(mPointControlChar);
    }
}
```

5. Data Source的回调中获取更多信息。
具体解析参见相关阅读中ANCS分析的两篇文章。


##### 执行相应动作
1. 解析eventFlags中的通知动作。
```
    public int getAction() {

        action = 0;
        //positive标志位为１
        if ((eventFlags & 0x08) > 0) {
            action = action + 1;
        }
        //negative标志位为１
        if ((eventFlags & 0x10) > 0) {
            action = action + 2;
        }
        return action;
    }
```
2. 写入动作命令到Control Point中。
```
byte[] action = {
        (byte) 0x02,
        //UID
        nid[0], nid[1], nid[2], nid[3],
        //positive action id(二选一)
        (byte) 0x00,
        //negative action id(二选一)
        (byte)0x01,
};
```

## Demo
Github地址：[ANCSReader](https://github.com/Jiesean/ANCSReader)
- 接收系统通知基本的信息，包括标题、类型（消息、来电等）、状态（产生、修改、删除）
- 接收通知的详细信息，包括内容、应用、时间等各种信息。
- 对通知采取相应的操作



![](http://upload-images.jianshu.io/upload_images/1806858-f82a0fad85cc0339.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 相关阅读
** Android BLE开发相关知识 **
[Android BLE开发之初识GATT](http://www.jianshu.com/p/29a730795294)
[Android BLE开发之玩转小米手环](http://www.jianshu.com/p/a274e17fc66a)

** ANCS相关知识 **
[苹果通知中心服务ANCS协议分析](http://www.jianshu.com/p/2ddf76ab85b0)
[苹果通知中心服务ANCS协议分析二](http://www.jianshu.com/p/b82db7b6312f)
