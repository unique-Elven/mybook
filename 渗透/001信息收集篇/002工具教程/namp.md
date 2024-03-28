[nmap详细使用教程\_nmap使用教程-CSDN博客](https://blog.csdn.net/smli_ng/article/details/105964486)
nmap -sn 目标ip //只看主机状态
>[!TIP]
对多个不连续的主机进行扫描
命令语法：nmap    192.168.1.1 192.168.1.2 baidu.com 192.168.1.3
不显示是因为没开机，没该主机对于连续范围内的主机进行扫描
对于连续范围内的主机进行扫描
语法：nmap 192.1681.1-255
对整个子网进行扫描
语法：nmap  192.168.1.1/24
# 主机发现
## 1.1 ARP协议
就是无论有没指定别的协议发现，能用ARP协议发现的，Nmap都会默认优先使用ARP协议发现

## 1.2 ICMP协议

nmap -PE [www.baidu.com](http://www.baidu.com) -sn

request请求 reply回答

收到回复说明在线，没回复不一定说明不在线

nmap -PM [www.baidu.com](http://www.baidu.com) -sn

nmap -PP [www.baidu.com](http://www.baidu.com) -sn

## 1.3 TCP协议

nmap -PS [www.baidu.com](http://www.baidu.com) -sn
发送SYN，返回ACK 判断主机在线，否则无应答

nmap -PA [www.baidu.com](http://www.baidu.com) -sn
直接发送ACK，返回RST 判断主机在线，否则无应答

nmap -PU [www.baidu.com](http://www.baidu.com) -sn
不准、少用、UDP协议

# 端口发现

端口的分类：

将端口的使用情况的不同，简单讲端口分为以下几类：

- 公认端口
- 注册端口
- 动态/私有端口

>[!TIP]
Nmap中对端口状态的定义：
Open
closed
filtered 没到达目的地，不确定是否开启
unfiltered 未被过滤但是无法访问，不确定开启
open|filtered 无法确定开放还是被过滤

扫描80端口为例
nmap -sS -p 80 \http://www.baidu.com
SYN扫描，半开扫描

nmap -sT -p 80 \http://www.baidu.com
connect扫描，全开扫描，建立连接，用的少，被写入日志被查出来

指定范围
	扫描全部的端口
	nmap -sT -p “\*” \http://www.baidu.com
	
	扫描使用频率最高的五个端口
	nmap -sT --top-ports 5 http://www.baidu.com

**使用FIN扫描**
有的时候TCP SYN不是最佳的扫描默认，目标主机可能有IDS/IPS系统的存在，防火墙可能过滤掉SYN数据包。而发送一个
FIN标志的数据包不需要完成TCP的握手。
nmap -sF  192.168.3.74


**idle扫描（需要指定另外一台主机IP地址，并且目标主机的IPID是递增的）**
idlescan是一种理想的扫描方式，它使用另一台网络上的主机替你发送数据包，从而隐藏自己。
nmap -sI  192.168.3.227 192.168.3.74
# 扫描目标操作系统

nmap -O 192.168.43.226

nmap-os-db 把不同设备系统对数据包的响应时特征记录下来，然后执行时验证即可识别系统

指纹技术

猜测，不一定准

nmap -O -osscan-guess 192.168.43.226

# 扫描目标服务

原来检测的端口对应的都是默认的服务，也不一定准！！！
如果要精确地识别服务，也有办法。不仅显示服务还会显示出提供服务的软件。
nmap -sV 192.168.43.226

保存扫描结果，最常用的是XML格式
nmap -sV 192.168.43.226 -oX Report.xml