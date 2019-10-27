---
title: HTML 书写规范
tags: html
categories: 项目总结
---

### 语法

+ 缩进使用soft tab (4个空格)
+ 嵌套节点应该缩进
+ 在属性上，使用双引号，不要使用单引号
+ 属性名全都小写，用中划线做分隔符
+ 不要在自动闭合标签结尾处使用斜线（[HTML5规范](https://html.spec.whatwg.org/multipage/syntax.html#syntax-start-tag)指出他们是可选的）

```
<!DOCTYPE html>
<html>
    <head>
        <title>Page title</title>
    </head>
    <body>
        <img src="images/company_logo.png" alt="Company">

        <h1 class="hello-world">Hello, world!</h1>
    </body>
</html>
```

### HTML5 doctype

在页面开头使用这个简单地doctype来启用标准模式，使其在每个浏览器中尽可能一致的展现；

虽然doctype不区分大小写，但是按照惯例，doctype大写 ([关于属性时大写还是小写](https://stackoverflow.com/questions/15594877/is-there-any-benefits-to-use-uppercase-or-lowercase-letters-with-html5-tagname))

### lang属性

根据HTML5规范：

> 应在html标签上加上lang属性。这会给语音工具和翻译工具帮助，告诉它们应当怎么去发音和翻译。

更多关于 lang 属性的说明 [在这里](https://html.spec.whatwg.org/multipage/semantics.html#the-html-element)

在sitepoint上可以查到 [语言列表](https://www.sitepoint.com/iso-2-letter-language-codes/)

但sitepoint只是给出了语言的大类，例如中文只给出了zh，但是没有区分香港，台湾，大陆。而微软给出了一份更加详细的[语言列表](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/)，其中细分了zh-cn, zh-hk, zh-tw。

```
<!DOCTYPE html>
<html lang="en-us">
    ...
</html>
```

### 字符编码

通过声明一个明确的字符编码，让浏览器轻松、快速的确定适合网页内容的渲染方式，通常指定为'UTF-8'。

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
    </head>
    ...
</html>
```

### IE兼容模式

用 `meta` 标签可以指定页面应该用什么版本的IE来渲染
如果你想要了解更多，请点击[这里](https://stackoverflow.com/questions/6771258/what-does-meta-http-equiv-x-ua-compatible-content-ie-edge-do)；

不同doctype在不同浏览器下会触发不同的渲染模式（[这篇文章总结的很到位](https://hsivonen.fi/doctype/)）。

```
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    </head>
    ...
</html>
```

### viewport
我们在做响应式布局的时候，肯定要考虑到适配移动端的屏幕，大多数同学也一定复制粘贴过下面这段代码：

```
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">

```
添加了这段代码以后，我们在移动端看到的显示效果就非常好，整个页面不会缩成一团。但是很多时候我们只是拿来用了，没有去理解这段代码究竟干了什么，为什么会影响移动端页面的布局效果，它又是怎么起作用的。
这篇[手记](https://www.imooc.com/article/34720)讲明白了，meta 标签viewport 的原理和作用

### 引入CSS, JS
根据HTML5规范, 通常在引入CSS和JS时不需要指明 type，因为 text/css和 text/javascript 分别是他们的默认值。

```
<!-- External CSS -->
<link rel="stylesheet" href="code_guide.css">

<!-- In-document CSS -->
<style>
    ...
</style>

<!-- External JS -->
<script src="code_guide.js"></script>

<!-- In-document JS -->
<script>
    ...
</script>
```

### 属性顺序

属性应该按照特定的顺序出现以保证易读性；

+ class
+ id
+ name
+ data-*
+ src , for, type, href, value, max-length, max, min, pattern 
+ placeholder, title, alt
+ aria-*, role
+ required, readonly, disabled

class是为高可复用组件设计的，所以应处在第一位；

id更加具体且应该尽量少使用，所以将它放在第二位。

```
<a class="..." id="..." data-modal="toggle" href="#">Example link</a>

<input class="form-control" type="text">

<img src="..." alt="...">
```

### JS生成标签

在JS文件中生成标签让内容变得更难查找，更难编辑，性能更差。应该尽量避免这种情况的出现。
