---
title: CSS — BEM 命名规范
tags: css
categories: 项目总结
---

### 什么是BEM 命名规范
+ [Bem](https://en.bem.info/) 是块（block）、元素（element）、修饰符（modifier）的简写，由 Yandex 团队提出的一种前端 CSS 命名方法论。

>  -中划线：仅作为连接符使用，表示某个块或者某个子元素的多单词之间的连接记号

>  __ 双下划线：双下划线用来连接块和块的子元素

> _ 单下划线：单下划线用来描述一个块和一个块或块的子元素的一种状态

type-block__element_modifier

### 块（block）

一个块是设计或布局的一部分，它具有具体且唯一的意义，要么是语义上要么是视觉上
在大多数情况下，任何独立的页面元素（或复杂或简单）都可以被视作一个块。它的HTML容器会有一个唯一的CSS类名，也就是这个块的名字。
针对块的CSS类名会加一些前缀（ ui-），这些前缀在CSS中有类似 命名空间 的作用

一个块的正式（实际上是半正式的）定义有下面三个基本原则：

1. css中只能使用类名（不能是ID）
2. 每一个块名应该有一个命名空间（前缀）
3. 每一条CSS规则必须属于一个块。

例如：一个自定义列表 .list 是一个块，通常自定义列表是算在 module 类别的，在这种情况下，一个 list 列表的block写法应该为:

```
.list
```

### 元素（element）

块中的子元素是块的子元素，并且子元素的子元素在 bem 里也被认为是块的直接子元素。一个块中元素的类名必须用父级块的名称作为前缀。

如上面的例子，li.item 是列表的一个子元素，

```
 // 普通写法
 .list{}
 .list .item{}

 // BEM 写法
 .list{}
 .list__item{}
```

### 修饰符（modifier）

一个“修饰符”可以理解为一个块的特定状态，标识着它持有一个特定的属性。
用一个例子来解释最好不过了。一个表示按钮的块默认有三个大小：小，中，大。为了避免创建三个不同的块，最好是在块上加修饰符。这个修饰符应该有个名字（比如：size ）和值（ small，normal 或者 big ）。
如上面的例子中，表示一个选中的列表，和一个激活的列表项

```
// 普通写法
.list{}
.list.select{}
.list .item{}
.list .item.active{}

// BEM 写法
.list{}
.list_select{}
.list__item{}
.list__item_active{}
```


### LESS 书写规范

使用 .less 后缀的文件来存储变量、混合代码以及最终合并压缩。 

|  子less | 注解 |
|:-----|-----:|
|`lib-base.less`    |  预定义的变量，例如颜色、字号、字体      |
|`lib-mixins.less`  |  用于混合的代码，例如渐变、半透明的混合    |
|`lib-reset.less`   |  初始化                                |
|`lib-ui.less`      |  颗粒化ui功能                           |
|`xxx.less`         |  模块样式                                |

.less 文件的引用顺序会对最终编译的样式的作用域和优先级产生影响，请尽量按照由底层到自定义的顺序来引用。

### 关于风格

##### 1. 原则上不会出现2层以上选择器嵌套
使用BEM原则，用命名来解耦，所有类名都为一层，增加效率和复用性

##### 2. 两层选择器嵌套出现在.mod-xxx__item_current子元素的情况，如下：
使用推荐的嵌套写法

```
// 常规写法：
.xxx{}
.xxx__item{}
.xxx__item_current{}
// 嵌套写法
.xxx__item_current .mod-xxx__link{}

// 推荐
.xxx{}
.xxx__item{}
.xxx__item_hightlight{}
.xxx__product-name{}
.xxx__link{}
.xxx__ming-zi-ke-yi-hen-chang{}

// 嵌套写法
.xxx__item_current{
    .xxx__link{}
}


```

对应HTML 结构如下

```
<ul class="xxx">
    <li class="xxx__item">第一项
        <div class="xxx__product-name">我是名称</div>
        <span class="xxx__ming-zi-ke-yi-hen-chang">看类名</span>
        <a href="#" class="xxx__link">我是link</a>
    <li>
    <li class="xxx__item xxx__item_current">第二项 且 当前选择项
        <div class="xxx__product-name">我是名称</div>
        <a href="#" class="xxx__item-link">我是link</a>
    <li>
    <li class="xxx__item xxx__item_hightlight">第三项 且 特殊高亮
         <div class="xxx__product-name">我是名称</div>
        <a href="#" class="xxx__item-link">我是link</a>
    <li>
</ul>
```


### BEM 解决问题
组件之间的完全解耦，不会造成命名空间的污染，如：.mod-xxx ul li 的写法带来的潜在的嵌套风险。

### 性能
BEM 命名会使得 Class 类名变长，但经过 gzip 压缩后这个带宽开销可以忽略不计

```
xxx__item_hightlight
```