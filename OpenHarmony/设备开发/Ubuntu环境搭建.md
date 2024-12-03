采用的是ubuntu18.04，搭建之前打快照以便于恢复

根据官网的教程搭建环境
[搭建开发环境-HUAWEI DevEco Device Tool使用指南-OpenHarmony设备开发-HarmonyOS设备开发](https://device.harmonyos.com/cn/docs/documentation/guide/ide-install-windows-ubuntu-0000001194073744#section186614181716)


中间遇到的问题是vscode插件没有显示
![[Pasted image 20240518113127.png]]
在这篇文章遇到了一样的道友[如何在Ubuntu开发环境下安装DevEco Device Tool？-鸿蒙开发者社区-51CTO.COM](https://ost.51cto.com/posts/11501)

这是官方社区提供的解决方案
[devicetool-linux-tool-3.0.0.300.sh安装 找不到 .vscode-华为开发者论坛 | 华为开发者联盟](https://developer.huawei.com/consumer/cn/forum/topic/0202772043716470323?fid=0103702273237520029)

当时遇到这种情况，请重新按照下面的步骤安装救能解决这个bug
```C
首先安装VScode：具体方法自己搜吧

然后具体具体操作：

1. su

2. cd /root 

3. mkdir .vscode

4. 关闭 VSCODE

5. sudo ./devicetool-linux-tool-3.0.0.300.sh -- --install-plugins
```