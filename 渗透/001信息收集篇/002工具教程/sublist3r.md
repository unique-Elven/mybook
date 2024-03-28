[GitHub - aboul3la/Sublist3r: Fast subdomains enumeration tool for penetration testers](https://github.com/aboul3la/Sublist3r)

**中文翻译**

-h ：帮助

-d ：指定主域名枚举子域名

-b ：调用subbrute暴力枚举子域名

-p ：指定tpc端口扫描子域名

-v ：显示实时详细信息结果

-t ：指定线程

-e ：指定搜索引擎

-o ：将结果保存到文本

-n ：输出不带颜色

**默认参数扫描子域名**

python sublist3r.py -d baidu.com

**使用暴力枚举子域名**

python sublist3r -b -d baidu.com