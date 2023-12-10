常用参数说明
hydra <<<-l LOGIN|-L FILE> <-p PASS|-P FILE>> | <-C FILE>> <-e ns>
<-o FILE> <-t TASKS> <-M FILE <-T TASKS>> <-w TIME> <-f> <-s PORT> <-S> <-vV>
server service <OPT>
-R
继续从上一次进度接着破解
-S
大写，采用 SSL 链接
-s <PORT>
小写，可通过这个参数指定非默认端口
-l <LOGIN>
指定破解的用户，对特定用户破解
-L <FILE>
指定用户名字典
-p <PASS>
小写，指定密码破解，少用，一般是采用密码字典
-P <FILE>
大写，指定密码字典
-e <ns>
可选选项，n：空密码试探，s：使用指定用户和密码试探
-C <FILE>
使用冒号分割格式，例如“登录名:密码”来代替-L/-P 参数
-M <FILE>
指定目标列表文件一行一条
-o <FILE>
指定结果输出文件
-f
在使用-M 参数以后，找到第一对登录名或者密码的时候中止破解
-t <TASKS>
同时运行的线程数，默认为 16
-w <TIME>
设置最大超时的时间，单位秒，默认是 30s
-v / -V
显示详细过程
server
目标 ip
service
指定服务名，支持的服务和协议：telnet ftp pop3[-ntlm] imap[-ntlm] smb smbnt
http[s]-{head|get} http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3
mssql mysql oracle-listener postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn
icq sapr3 ssh2 smtp-auth[-ntlm] pcanywhere teamspeak sip vmauthd firebird ncp afp
等等
OPT
可选项