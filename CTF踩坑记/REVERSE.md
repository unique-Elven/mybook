# Srand,rand,time()伪随机数
例题：Simple Encryptor[Hack The Box :: Login](https://app.hackthebox.com/challenges/Simple%2520Encryptor)
题解：
[Simple Encryptor reverse engineering challenge Writeup - Blog by Hiroki Fujino](https://hiroki6.dev/posts/simple-encryptor)

[HackTheBox — Simple Encryptor Write Up | by southbre | Medium](https://medium.com/@southbre/hackthebox-simple-encryptor-308949f7023c)

```css
`__ROL4__`是uint32循环左移，`__ROL2__`是uint16循环左移，`__ROL1__`是uint8循环左移

`__ROR4__`是uint32循环右移，`__ROR2__`是uint16循环右移，`__ROR1__`是uint8循环右移

这些名称都是ida的定义，可以在`ida根目录\plugins\defs.h`中详细查看：
```