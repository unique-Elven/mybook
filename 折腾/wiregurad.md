

原理讲解
[WireGuard - lsgxeva - 博客园](https://www.cnblogs.com/lsgxeva/p/14105764.html)

[kali安装、配置wireguard\_wg-quick returend64-CSDN博客](https://blog.csdn.net/weixin_44471703/article/details/126963908)


[如何设置 Linux WireGuard VPN 客户端——VPN Unlimited](https://www.vpnunlimited.com/zh/help/manuals/wireguard/linux?srsltid=AfmBOor4ZKcYGN6IoKzmhjuoNBYtSW1zsN4sbHrGhtz5AjIKKFrkYhtm)


这篇文章深入学习配置文件，让我突然有思路了，怎么添加一个客户端连接
[Wireguard笔记(一) 节点安装配置和参数说明 - Milton - 博客园](https://www.cnblogs.com/milton/p/14178344.html)


这个介绍，超级贴心，看了很多文章后发现这个又有收获：1. AllowedIPs = 0.0.0.0/0 表示客户端所有流量都通过VPN服务器中转，即全局代理
[体验Wireguard的简单之美 - 阅心笔记](https://opswill.com/articles/wireguard-howtos.html)

又学到了--遇到运营商UDP限速（QOS）
解决方法：使用[Phantun](https://github.com/dndx/phantun)将UDP伪装成TCP连接
[Wireguard安装 | 一心净土](https://www.tanwen.net/blog/wireguard)

部署web管理界面
[WireGuard 配置教程：使用 wg-gen-web 来管理 WireGuard 的配置 · 云原生实验室](https://icloudnative.io/posts/configure-wireguard-using-wg-gen-web/)

 如何搭建多节点间路由的WireGuard
 [如何搭建多节点间路由的WireGuard | 积少成多, 聚沙成塔](https://kiritow.com/chained-wireguard-setup/)