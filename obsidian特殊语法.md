保存csdn网页到本地打开时自动跳转到首页解决方法:::::::::应该是找到display:none这个div进行删除<div style="display:none;">在Obsidian最新更新的14.0版本中，终于上线了“Callout”功能。

使用这样的符号就可以启用callout模块: `> [!INFO]`.

```text
> [!INFO]
> 这里是callout模块
> 支持**markdown** 和 [[Internal link|wikilinks]].
```

呈现出来是这样的

![](https://pic3.zhimg.com/80/v2-86ef1d3141e0b3bc915e2aad7a621162_720w.webp)

## 样式

默认有12种风格。每一种有不同的颜色和图标。

只要把上面例子里的 `INFO` 替换为下面任意一个就行。

- note
- abstract, summary, tldr
- info, todo
- tip, hint, important
- success, check, done
- question, help, faq
- warning, caution, attention
- failure, fail, missing
- danger, error
- bug
- example
- quote, cite

## 标题和内容

可以自定义标题，然后直接不要主体部分，比如

```text
> [!TIP] Callouts can have custom titles, which also supports **markdown**!
```

![](https://pic3.zhimg.com/80/v2-ff3e8ccd97a28fe54982b7c303de90f6_720w.webp)

## 折叠

可以使用 `+` 默认展开或者 `-` 默认折叠正文部分。

```text
> [!FAQ]- Are callouts foldable?
> Yes! In a foldable callout, the contents are hidden until it is expanded.
```

显示为:

![](https://pic1.zhimg.com/80/v2-9a1c6964e3badb6b52fae96e630515cc_720w.webp)

## 如果需要自定义

Callout的类型和图标是用CSS来描述，颜色是`r, g, b` 三色组，图标有相应的 icon ID (比如`lucide-info`)。也可以自定义SVG图标。

```css
.callout[data-callout="my-callout-type"] {
    --callout-color: 0, 0, 0;
    --callout-icon: icon-id;
    --callout-icon: '<svg>...custom svg...</svg>';
}
```



在Obsidian里面写文章时，时常需要给不同文字添加高亮背景/颜色；但是Markdown却没有这个功能，于是查询发现Obsidian编辑器同时支持md+css+html。


## **字体颜色**

![](https://pic1.zhimg.com/80/v2-90bc938829305628797598fe43e79de8_720w.webp)

```text
<font color="red" size="1">这里是红色</font>
```

其中`red`代表红色，也可以根据下表更换为其他颜色；`1`代表字体大小。

![](https://pic2.zhimg.com/80/v2-9b170fc8c712021a59613fbc4d4c590d_720w.webp)