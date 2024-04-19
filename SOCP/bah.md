
|   |   |   |
|---|---|---|
|bah|easy|qdpmcms利用、mysql利用、host碰撞、shellinaboxd使用、pspy分析隐藏进程提权|


```c
┌──(kali㉿kali)-[~]
└─$ sudo netdiscover -i eth0 -r 192.168.44.141/24
```

```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.141 
```
80 3306
```css
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://192.168.44.141/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,png -e
```