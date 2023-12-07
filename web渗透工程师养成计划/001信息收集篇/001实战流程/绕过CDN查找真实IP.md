😱[11种绕过CDN查找真实IP方法 - STARTURN - 博客园](https://www.cnblogs.com/qiudabai/p/9763739.html)
## 判断CDN

nslookup

超级ping
	http://ping.chinaz.com/
	http://ping.aizhan.com/
	https://www.17ce.com/

## 绕过cdn找真实ip

域名解析历史记录
	通过以下这些网站可以访问dns的解析，有可能存在未有绑cdn之前的记录。
	https://dnsdb.io/zh-cn/ ###DNS查询
	https://x.threatbook.cn/ ###微步在线
	http://viewdns.info/ ###DNS、IP等查询
	https://tools.ipip.net/cdn.php ###CDN查询IP
	[https://sitereport.netcraft.com/?url=域名](https://sitereport.netcraft.com/?url=域名)
	https://securitytrails.com 🌏
	https://dnsdb.io/zh-cn/
	




部分cdn只针对国内的ip访问，如果国外ip访问域名 即可获取真实IP

全世界DNS地址：
	[http://www.ab173.com/dns/dns_world.php](http://www.ab173.com/dns/dns_world.php)
	https://dnsdumpster.com/
	https://dnshistory.org/
	http://whoisrequest.com/history/
	https://completedns.com/dns-history/
	http://dnstrails.com/
	[https://who.is/domain-history/](https://who.is/domain-history/)
	http://research.domaintools.com/research/hosting-history/ http://site.ip138.com/
	http://viewdns.info/iphistory/
	https://dnsdb.io/zh-cn/
	https://www.virustotal.com/
	https://x.threatbook.cn/
	http://viewdns.info/
	[http://www.17ce.com/](http://www.17ce.com/)
	http://toolbar.netcraft.com/site_report?url= https://securitytrails.com/
	https://tools.ipip.net/cdn.php

ico图标通过空间搜索找真实ip    下载图标 放到fofa识别

通过censys找真实ip
Censys工具就能实现对整个互联网的扫描，Censys是一款用以搜索联网设备信息的新型搜索引擎，能够扫描整个互联网，Censys会将互联网所有的ip进行扫面和连接，以及证书探测。
若目标站点有https证书，并且默认虚拟主机配了https证书，我们就可以找所有目标站点是该https证书的站点。

通过协议查询

找文件
	phpinfo.php     \_SERVER["SERVER_ADDR"]
	测试环境  服务器环境脚本

子域名解析ip----通过寻找子域名再通过子域名查询解析的服务器的IP可以是真实的服务器
	域名批量解析
	[http://tools.bugscaner.com/domain2ip.html](http://tools.bugscaner.com/domain2ip.html)


邮箱发送-----如果网站有邮箱发送功能可以试再密码找回邮箱头文件可能是服务器的真实ip

验证IP 
	直接访问IP
	端口扫描 获取端口服务与目标是否一致
	绑定hosts进行访问
	
空间搜索
	根据body的特征在fofa sohdan搜索
	图标搜索
	[360网络空间测绘 — 因为看见，所以安全](https://quake.360.cn)

### ## 1.1. 利用SSL证书寻找真实IP

证书颁发机构(CA)必须将他们发布的每个SSL/TLS证书发布到公共日志中，SSL/TLS证书通常包含域名、子域名和电子邮件地址。因此SSL/TLS证书成为了攻击者的切入点。

获取网站SSL证书的HASH再结合Censys

利用Censys搜索网站的SSL证书及HASH，在https://crt.sh上查找目标网站SSL证书的HASH

再用Censys搜索该HASH即可得到真实IP地址

## 1.2. F5 LTM解码法

当服务器使用F5 LTM做负载均衡时，通过对set-cookie关键字的解码真实ip也可被获取，

例如：Set-Cookie: BIGipServerpool_8.29_8030=487098378.24095.0000，先把第一小

节的十进制数即487098378取出来，然后将其转为十六进制数1d08880a，接着从后至前，

以此取四位数出来，也就是0a.88.08.1d，最后依次把他们转为十进制数10.136.8.29，也就

是最后的真实ip。

rverpool-cas01=3255675072.20480.0000; path=/

3255675072 转十六进制 c20da8c0 从右向左取 c0a80dc2 转10进制 192 168 13 194

## 1.1. banner

获取目标站点的banner，在全网搜索引擎搜索，也可以使用AQUATONE，在Shodan上搜索相同指纹站点。

可以通过互联网络信息中心的IP数据，筛选目标地区IP，遍历Web服务的banner用来对比CDN站的banner，可以确定源IP。

欧洲：

http://ftp.ripe.net/pub/stats/ripencc/delegated-ripencc-latest

北美：

https://ftp.arin.net/pub/stats/arin/delegated-arin-extended-latest

亚洲：

ftp://ftp.apnic.net/public/apnic/stats/apnic/delegated-apnic-latest

非洲：

ftp://ftp.afrinic.net/pub/stats/afrinic/delegated-afrinic-latest

拉美：

ftp://ftp.lacnic.net/pub/stats/lacnic/delegated-lacnic-extended-latest

获取CN的IP

http://www.ipdeny.com/ipblocks/data/countries/cn.zone 

例如：

找到目标服务器 IP 段后，可以直接进行暴力匹配 ，使用zmap、masscan 扫描 HTTP banner，然后匹配到目标域名的相同 banner

zmap -p 80 -w bbs.txt -o 80.txt

使用zmap的banner-grab对扫描出来80端口开放的主机进行banner抓取。

cat /root/bbs.txt |./banner-grab-tcp -p 80 -c 100 -d http-req -f ascii > http-banners.out

根据网站返回包特征，进行特征过滤

location: plugin.php?id=info:index

https://fofa.so/

title="T00LS | 低调求发展 - 潜心习安全 - T00ls.Net"

https://www.zoomeye.org/

title:"T00LS | 低调求发展 -潜心习安全 -T00ls.Net"

[https://quake.360.cn/](https://quake.360.cn/)

response:"T00LS | 低调求发展 - 潜心习安全 - T00ls.Net"

1、ZMap号称是最快的互联网扫描工具，能够在45分钟扫遍全网。https://github.com/zmap/zmap

2、Masscan号称是最快的互联网端口扫描器，最快可以在六分钟内扫遍互联网。

https://github.com/robertdavidgraham/masscan
## 1.1. 长期关注

在长期渗透的时候，设置程序每天访问网站，可能有新的发现。每天零点 或者业务需求增大 它会换ip 换服务器的。
## 1.1. 流量攻击

发包机可以一下子发送很大的流量。

这个方法是很笨，但是在特定的目标下渗透，建议采用。

cdn除了能隐藏ip，可能还考虑到分配流量，

不设防的cdn 量大就会挂，高防cdn 要大流量访问。

经受不住大流量冲击的时候可能会显示真实ip。

站长->业务不正常->cdn不使用->更换服务器。
## 1.1. 被动获取

被动获取就是让服务器或网站主动连接我们的服务器，从而获取服务器的真实IP

如果网站有编辑器可以填写远程url图片，即可获取真实IP

如果存在ssrf漏洞 或者xss 让服务器主动连接我们的服务器 均可获取真实IP。
## 1.1. 扫全网获取真实IP

https://github.com/superfish9/hackcdn

https://github.com/boy-hack/w8fuckcdn
















目前很多网站使用了cdn服务，用了此服务 可以隐藏服务器的真实IP，加速网站静态文件的访问，而且你请求网站服务时，cdn服务会根据你所在的地区，选择合适的线路给予你访问，由此达网站加速的效果，cdn不仅可以加速网站访问，还可以提供waf服务，如防止cc攻击，SQL注入拦截等多种功能，再说使用cdn的成本不太高，很多云服务器也免费提供此服务。在进行黑盒测试的时候，往往成了拦路石，所以掌握cdn找真实ip成了不得不掌握的一项技术。
![[Pasted image 20231205203529.png]]