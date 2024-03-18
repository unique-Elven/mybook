john test.txt --format=crypt
 john爆破出现Using default input encoding: UTF-8 No password hashes loaded (see FAQ)
 出现这个提示不是说明破解失败了，而是这个hash以前被你破解过，可以使用以下命令查看破解出的密码是什么。

john --show password.txt

msf永恒之蓝命令执行后：
hashdump得到的值拿到最后32位字母数字组合可以在下图,注意类型是NTLM

![[Pasted image 20240318114658.png]]