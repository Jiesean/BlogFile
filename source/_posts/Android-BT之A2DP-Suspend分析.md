---
title: Android BT之A2DP Suspend分析
date: 2017-05-07 19:22:11
tags: [Android,BT]
---

#### 适用场景
Android手机连接蓝牙耳机播放音频，另外一台手机给DUT打电话，电话进来后，音频暂停，播放电话语音。
对方挂断后，蓝牙耳机继续播放音频。
#### 常见issue
针对该场景下可能出现一些异常的bug需要进行分析，或者有些厂商会对该问题进行feature的开发。
1. 电话打进来，音乐继续播放没有暂停。
2. 电话挂断后，音乐没有继续播放。

<!-- more -->

#### 概要分析
(1) telephony模块接收到in_comming call的事件后，调用蓝牙的API接口phoneStateChanged，告知蓝牙电话进来了需要suspend A2DP.
(2) 蓝牙调用processCallState去suspend A2DP。
(3) 协议栈和蓝牙耳机同步A2DP的状态变化。
(4) A2DP Suspend之后讲状态变化返回Java层。
(5) Java层发送广播，告知其他模块，A2DP suspend状态,完成整个suspend的过程，音频暂停。

![A2DP_Suspend](http://upload-images.jianshu.io/upload_images/1806858-a667a50f3a64acc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(6) tele收到广播后，路由HandsFree数据给蓝牙，通过蓝牙传输给蓝牙耳机。
(7) 电话挂断后，telephony模块接收到电话挂断事件，HandsFree建立的soc链路断开。
(8) tele调用蓝牙接口修改suspend状态为false, A2DP数据继续传输。

#### 代码及协议分析
一下通过对于蓝牙模块的log几个重要的点进行分析，来说明整个过程。
```
//当电话incoming，tele调用蓝牙interface,使A2DP suspend 
05-05 15:06:33.262 D/HeadsetStateMachine( 7307): A2dp Play State Changed: Current State: 11Prev State: 10A2pSuspend: true
//snoop 里看到A2DP suspend
2,165	1 5 Single Packet	Master	SUSPEND	5	12	 00:00:14.132396	2017/5/5 15:06:33.204058
```

以上我们看到蓝牙在电话打进来时，A2DP正常的suspend,但是当电话挂断时，tele模块应该调用蓝牙的接口修改A2DP的状态

```
//电话断开，tele模块调用bt接口，hci snoop中收到断开指令
Success	Success	2,195 Event Disconnection Complete Success 0x0006 2017/5/5 15:07:41.350643	7 00:00:00.117054 4	
//handsfree层断开
2,197 Master 5 ..+CIEV: 1,0..	Call Status indicator's status report 27 00:01:05.315589	2017/5/5 15:07:41.351906	
//btif处理断开
05-05 15:07:41.351 D/bt_btif ( 7307): btif_hf_upstreams_evt: event=BTA_AG_AUDIO_CLOSE_EVT
//java层收到断开，发送广播通知
05-05 15:07:41.374 D/HeadsetStateMachine( 7307): Audio state 00:1E:7C:01:61:84: 12->10
```
以上说明蓝牙模块已正常断开。
