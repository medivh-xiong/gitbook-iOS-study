## iOS的蓝牙研究


目前在iOS中开发的蓝牙框架有三种：

* **GameKit.framework：iOS7之前的蓝牙通讯框架，从iOS7开始过期，这个一般是用于游戏开发；**
* **MutipeerConnectivity.framework :iOS7开始引入的新的蓝牙通讯开发框架，用于取代GameKit,多用于文件传输；**
* **CoreBlueTooth.framework:可用于第三方蓝牙设备交互，必须基于蓝牙4.0以上；**

前两个框架使用起来较为简单，但是仅支持iOS设备，传输内容仅限于沙盒或者照片库中用户选择的文件，并且第一个框架只能在同一个应用之间进行传输，因此大部分情况下，iOS的蓝牙开发是使用CoreBlueTooth进行开发。

## CoreBlueTooth
### 简介
* 可用于第三方的蓝牙设备交互，设备必须支持蓝牙4.0
*
