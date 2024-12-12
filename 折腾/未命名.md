termux文件导入导出
手机安装termux后就可以像Linux一样操作啦, 但是遇到两个问题:

怎样让termux访问手机的内部存储
怎样将termux的根目录文件传递到手机呢
answer1
想要让termux访问手机存储, 只需要输入这行命令:

termux-setup-storage # 申请获取系统文件权限
就可以啦!

answer2
termux的根目录是在/data/data/com.termux/files/home/, 默认情况下(手机没root)手机的文件管理器是访问不了的, 会提示权限不够或者是根本看不到, 那我们怎样拿到termux里面的文件呢?

其实手机的内部存储入口就在当前文件夹下sotrage/shared里面, 你可以cd进入到这个文件目录, 再ls一下会发现他就是你手机的内部存储
还有一种方法就是直接输入命令 cd /storage/emulated/0/, 其切换到的目录就是手机内部存储的目录
最后只需要cp或者mv某文件到该目录就可以实现将termux目录中的文件传递到手机存储啦!
