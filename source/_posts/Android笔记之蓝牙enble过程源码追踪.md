---
title: Android笔记之蓝牙enble过程源码追踪
date: 2016-12-14 21:39:44
tags: [Android,BT]
---


### 前言
在Android开发异常火热的如今，各类的Android开发的文档也异常丰富，但是很奇怪的是关于android蓝牙开发的文档确是少之又少，现在android 7都出来好久了，中文社区蓝牙开发的资料大多都停留在了android 4.3之前，仅有的新鲜的文章也是那几篇抄来抄去，也没有相关的书籍作为参考，找点资料也是心累。
所以想要把自己整理的东西慢慢写下来，除了作为自己知识的沉淀，也希望将能够分享和交流。

而追踪蓝牙使能过程目的有这么几个：
- 增进对蓝牙框架的了解
- 提供了一种Android源码的阅读方式
- 对整个Android的框架和代码结构有进一步的了解
- 了解Jni、binder、广播等机制在其中的运用

<!-- more -->

### Bt框架

![BT enable框架](http://upload-images.jianshu.io/upload_images/1806858-8f3767b60bce8e62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
enable的追踪从settings app开始，一直到最底层的driver，基本按照上图的框架，把函数的调用过程代入图中有助于理解整个蓝牙的架构。

### 源码追踪

以下为Bluetooth的enable从settings app开始往下追踪的过程，当然仅为调用过程。

1. BluetoothEnabler.java(packages\apps\settings\src\com\android\settings\bluetooth)	
```
|onSwitchChanged()
     |mLocalAdapter.setBluetoothEnabled(isChecked);
　
```
其中BluetoothEnabler实现了SwitchBar.OnSwitchChangeListener监听蓝牙开关的状态变化；当开关被点击了，onSwitchChanged被回调。
查看mLocalAdapter定义：
```
private final LocalBluetoothAdapter mLocalAdapter;
```
2. 
LocalBluetoothAdapter.java(frameworks\base\packages\settingslib\src\com\android\settingslib\bluetooth)
```
|setBluetoothEnabled()
	 |mAdapter.enable()
```
查看mAdapter的定义
```
 private final BluetoothAdapter mAdapter;
```
3. BluetoothAdapter.java (frameworks\base\core\java\android\bluetooth)
```
|enable()
	 |mManagerService.enable()
```
查看mManagerService定义
```
private final IBluetoothManager mManagerService;
```
这里是通过AIDL机制完成进程间的通信，调用的是BluetoothManagerService的enable()函数。
4. BluetoothManagerService.java(frameworks\base\services\core\java\com\android\server)
```
|enable()
	 |sendEnableMsg(false);
		|mHandler.sendMessage(mHandler.obtainMessage(MESSAGE_ENABLE,0, 0));
```
在他的内部类BluetoothHandler继承了Handler类，处理传递的消息：
```
|handleMessage(Message msg)
	 |case MESSAGE_ENABLE	
		|handleEnable(msg.arg1 == 1);
			|mBluetooth.enable()
```
查看mBluetooth的定义：
```
private IBluetooth mBluetooth;
```
5. AdapterService.java(packages\apps\bluetooth\src\com\android\bluetooth\btservice)
```
|enable(boolean quietMode)
	 |Message m = mAdapterStateMachine.obtainMessage(AdapterState.BLE_TURN_ON);
       |mAdapterStateMachine.sendMessage(m);
```
查看mAdapterStateMachine的定义：
```
private AdapterState mAdapterStateMachine;
```
6. AdapterState.java(packages\apps\bluetooth\src\com\android\bluetooth\btservice)
```
|processMessage(Message msg)
	 |case BLE_TURN_ON
		|adapterService.processStart();
```
查看定义：
```
AdapterService adapterService = mAdapterService;
```
7. AdapterService.java (packages\apps\bluetooth\src\com\android\bluetooth\btservice)
```
|processStart() 
	  | mAdapterStateMachine.sendMessage(mAdapterStateMachine.obtainMessage(AdapterState.STARTED));
```
8. AdapterState.java(packages\apps\bluetooth\src\com\android\bluetooth\btservice)
```
|processMessage(Message msg)
	 |case STARTED
		|adapterService.enableNative()
```
9. AdapterService.java(packages\apps\bluetooth\src\com\android\bluetooth\btservice)
其中定义了native函数enableNative：
```
 /*package*/ native boolean enableNative();
```
它加载了动态链接库libbluetooth_jni.so，也就意味着enableNative函数的具体实现打包在这个动态链接库中。
进到目录packages\apps\bluetooth下，有一个jni的目录，一般跟bluetooth这个原生应用有关的jni都在这里实现；下面有一个Android.mk，查看该文件，其中有打包成库的名称
```
LOCAL_MODULE := libbluetooth_jni
```
正好是我们要找的动态链接库，我们要找的函数就是在这个目录下了。
mk文件中也提供了编译的时候包含的源文件：
```
LOCAL_SRC_FILES:= \
    com_android_bluetooth_btservice_AdapterService.cpp \
    com_android_bluetooth_btservice_QAdapterService.cpp \
    com_android_bluetooth_hfp.cpp \
    com_android_bluetooth_hfpclient.cpp \
    com_android_bluetooth_a2dp.cpp \
    com_android_bluetooth_a2dp_sink.cpp \
    com_android_bluetooth_avrcp.cpp \
    com_android_bluetooth_avrcp_controller.cpp \
    com_android_bluetooth_hid.cpp \
    com_android_bluetooth_hidd.cpp \
    com_android_bluetooth_hdp.cpp \
    com_android_bluetooth_pan.cpp \
    com_android_bluetooth_gatt.cpp \
    android_hardware_wipower.cpp
```
使用命令抓取一下：
```
find .|grep "enableNative" -rn .
```
发现enable函数是在com_android_bluetooth_btservice_AdapterService.cpp中实现的。
10. com_android_bluetooth_btservice_AdapterService.cpp (packages\apps\bluetooth\jni)	
```
|enableNative(JNIEnv* env, jobject obj)
  	|sBluetoothInterface->enable()
```
查看定义
```
static const bt_interface_t *sBluetoothInterface = NULL;
```
bt_interface_t是Bluetooth.h (hardware\libhardware\include\hardware)中定义的结构体。
我们需要找到它的实现，在文件中查找sBluetoothInterface找到它赋值的地方：
```
 sBluetoothInterface = btStack->get_bluetooth_interface();
```
它是由btStack也就是蓝牙协议栈返回的，我们继续看btStack的关键代码：
```
const char *id = (strcmp(value, "1")? BT_STACK_MODULE_ID : BT_STACK_TEST_MODULE_ID);
//这句应该获得btStack实例
err = hw_get_module(id, (hw_module_t const**)&module);
err = module->methods->open(module, id, &abstraction); 
//最终转换为btStack结构体
bluetooth_module_t* btStack = (bluetooth_module_t *)abstraction;
```
找到hw_get_module函数，其中果然有加载btStack模块:
```
load(class_id, path, module);
```
11. Bluetooth.c (external\bluetooth\bluedroid\btif\src)
我们知道Bluedroid的代码在external\bluetooth\bluedroid，因此我们在其中搜索很容易找到sBluetoothInterface的实现代码。
```
|static const bt_interface_t bluetoothInterface
      |enable
        |btif_enable_bluetooth()
```
12. Btif_core.c (external\bluetooth\bluedroid\btif\src)
```
|btif_enable_bluetooth(void)
	  |bte_main_enable()
```
13. Bte_main.c (external\bluetooth\bluedroid\main)
```
|bte_main_enable()
	  |bte_hci_enable() 
         |bt_hc_if->set_power(BT_HC_CHIP_PWR_ON)
```
查看定义
```
static bt_hc_interface_t *bt_hc_if=NULL;
```
以及赋值的语句
```
bt_hc_if = (bt_hc_interface_t *) bt_hc_get_interface()
```
14. Bt_hci_bdroid.c (external\bluetooth\bluedroid\hci\src)
```
|static const bt_hc_interface_t bluetoothHCLibInterface
      |set_power
        |vendor_send_command(BT_VND_OP_POWER_CTRL, &pwr_state);
```
hci层发送了BT_VND_OP_POWER_CTRL的命令。
15. Vendor.c (external\bluetooth\bluedroid\hci\src)
```
|vendor_send_command(bt_vendor_opcode_t opcode, void *param)
	  |vendor_interface->op(opcode, param)
```
vendor_interface赋值语句：
```
VENDOR_LIBRARY_NAME = "libbt-vendor.so";
lib_handle = dlopen(VENDOR_LIBRARY_NAME, RTLD_NOW);
vendor_interface = (bt_vendor_interface_t *)dlsym(lib_handle, VENDOR_LIBRARY_SYMBOL_NAME);
```
**dlsym**根据动态链接库操作句柄与符号，返回符号对应的地址。
可以得到vendor_interface的实现代码在libbt-vendor.so这个动态链接库中。
16. Bt_vendor_qcom.c (hardware\qcom\bt\libbt-vendor\src)
全局抓取一下libbt-vendor.so，我们在hardware/qcom/bt/libbt-vendor下的Android.mk中找到了libbt-vendor.so的包含的源文件。
抓取一下bt_vendor_interface_t，发现在Bt_vendor_qcom.c找到了它的实现：
```
|const bt_vendor_interface_t BLUETOOTH_VENDOR_LIB_INTERFACE
	  |op(bt_vendor_opcode_t opcode, void *param)
	    |hw_config(nState)
```
17. Hardware.c (hardware\qcom\bt\libbt-vendor\src)
```
|hw_config(int nState)
      |property_set("bluetooth.hciattach", true)
```
进行芯片framework的config操作。

### 小结
本文为enable过程的源码追踪，总结完了吐了一口老血，但不得不说，android程序员还是厉害啊，基本上一个函数是干什么的看看函数名就知道了，以现在的水平还无法对过程进行展开分析，其中也有很多的错误，以后慢慢的补充和修改。