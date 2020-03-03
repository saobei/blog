---
title: js的一些常用简写操作
tags: 
	- js
categories: 项目总结
date: 2019/12/10
---

### 内容概要
有助于大家更加熟练的运用 JavaScript 语言来进行开发工作 -- 关于简写 大家一起慢慢补充0_0哦

### 声明变量
```js
//在函数开始之前，对变量进行赋值是一种很好的习惯。在申明多个变量时,我们可能会：
let x;
let y;
let z = 3;

// 简写为
let x, y, z=3;
```
### 三目运算符
```js
// 通常我们经常会写if...else..语句 例如
var a = 0;
var b;
if (a > 10) {
    b = 'greater than 10';
} else {
    b = 'less than 10';
}
// 可简写为
var b = a > 10 ? 'greater than 10' : 'less than 10';
```
### if 语句
```js
// 在使用 if 进行基本判断时，可以省略赋值运算符。
if (a === true) / if (a === undefined || a === null || a === '' || a===0) 
// 可简写为
if( a ) / if(!a)
......
```
### 十进制数
```js
// 可以使用科学计数法来代替较大的数据，如可以将 100000 简写为 1e5。
let a = 100000;
// 可简写为
let a = 1e5;
```
### 模板字符串

```js
// 过去我们习惯了使用“+”将多个变量转换为字符串
const url = 'http://' + host + ':' + port + '/' + path;
// ES6 提供了相应的方法，我们可以使用反引号和 $ { } 将变量合成一个字符串。
const url = `http://${host}:${port}/${path}`;
```

### 未完待续，大家快来补充啊...





