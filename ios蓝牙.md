# iOS的蓝牙研究

目前在iOS中开发的蓝牙框架有三种：

* **GameKit.framework：iOS7之前的蓝牙通讯框架，从iOS7开始过期，这个一般是用于游戏开发；**
* **MutipeerConnectivity.framework :iOS7开始引入的新的蓝牙通讯开发框架，用于取代GameKit,多用于文件传输；**
* **CoreBlueTooth.framework:可用于第三方蓝牙设备交互，必须基于蓝牙4.0以上；**

前两个框架使用起来较为简单，但是仅支持iOS设备，传输内容仅限于沙盒或者照片库中用户选择的文件，并且第一个框架只能在同一个应用之间进行传输，因此大部分情况下，iOS的蓝牙开发是使用CoreBlueTooth进行开发。

# CoreBlueTooth

## 简介

* 可用于第三方的蓝牙设备交互，设备必须支持蓝牙4.0
* iPhone的设备必须是4S或者更新，iPad设备必须是iPad mini或者更新
* iOS的系统必须是iOS6或者更新 
* 应用场景：
  * 运动手环 
  * 智能家居
  * 拉卡拉蓝牙刷卡器


## 核心概念

* CBCentralManager：中心设备（用来连接到外部设备的管家）左侧
* CBPeripheralManager：外部设备（第三方的蓝牙4.0设备）右侧
  ![](http://upload-images.jianshu.io/upload_images/1093584-c59f14e18a39a343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 服务和特征，特征的属性\(service and characteristic\)：蓝牙设备添加若干服务，服务內添加若干特征，特征就是具体键值对，提供对数据的读取和写。蓝牙中心和外设数据的交换基于服务的特征。


![](http://upload-images.jianshu.io/upload_images/1093584-9b2b139594bd1c35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开发步骤

## Central端操作

一个中心端操作包含以下：

1. 启动一个Central端管理器对象；

``` obj-c
self.mgr = [[CBCentralManager alloc] initWithDelegate:self queue:nil];
```

1. 搜索并连接正在广告的Peripheral设备；
2. 扫描Peripheral中的服务和特征；
3. 与Peripheral做数据交互；
4. 订阅Characteristic的通知；
5. 断开连接

