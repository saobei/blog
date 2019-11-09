---
title: Typescript 极速入门
tags: 
    - typescript
categories: typescript
---

### 1、基本类型和拓展类型

Typescript 与 JavaScript 共享基本类型，拓展了一些类型

+ 元组 Tuple
+ 枚举 Enum
+ any和void
+ 从不类型 never

#### 共有基本类型

``` typescript

let a: number = 0;
let b: string = "扫呗无线前端";
let c: boolean = true;
let d: undefined = undefined;
let e: null = null;

```

#### object 类型，统指非number, string, null, undefined, boolean等基本类型的复杂类型

```typescript

declare function bar(arg: object | null): number;

bar({});        //  ok
bar(() => {});  //  ok
bar(null);      //  ok
bar(12121);     //  error

// 或者这种写法
interface demofunc {
    (arg: object): number
}
let func: demofunc = (arg): number => {
    return 1
}

func({}); // ok
func(()=> {}) // ok
func(null); // error
func(12121) // error
func("str") // error

// 数组
let arr: Array<string> = ['12121212']; // 数组泛型写法
let arr1: number[] = [1,2,3];    // 简写
```

####  元组 Tuple

元组类型允许你用固定数量的元素表示数组，这些元素的类型是已知的，但不必相同

```typescript

// 声明一个元组类型
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error

// 当我们去访问x 的值的时候, 如果访问到不存在的属性的时候，ts会立马给出错误提示
console.log(x[0].substring(1)); // OK
console.log(x[1].substring(1)); // Error, 'number' does not have 'substring'

// 当越界访问x得时候
x[3] = "world"; // Error, Property '3' does not exist on type '[string, number]'.

console.log(x[5].toString()); // Error, Property '5' does not exist on type '[string, number]'.
```

####  enum 枚举
enum类型是对JavaScript标准数据类型的一个补充。 像C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字。

```typescript
// 默认情况从0开始为元素编号，也可手动为1开始
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;

let colorName: string = Color[2];
console.log(colorName);  // 输出'Green'因为上面代码里它的值是2
```

枚举的应用场景适合定义一些固定的状态，比如应用得状态

```typescript

enum status {
    isFetching,
    pendding,
    success,
    error
}

```
####  any和void、从不类型never
+ any 用于不确定值得类型情况下，给变量定义any类型，但是不能多用，不然用ts就失去意义了
+ void表示空值返回，比如
```typescript
declare function test(): void{
    console.log("啥都没有返回")
}
```
+ never表示永远也到不了的情况，也叫从不类型
```typescript

function error(message: string): never {
    throw new Error(message);
}

function fail() {
    return error("Something failed");
}

// 函数返回绝不能具有无法到达的终点
function infiniteLoop(): never {
    while (true) {
    }
}

```

####  类型断言
简略的定义是：可以用来手动指定一个值的类型。
有两种写法，尖括号和as:

```typescript
let bar: any = "str";

let len: number = (<string>bar).length;
let len1: number = (bar as string).length;
```

### 2. 泛型 Generics

先来看一个demo

```typescript
function demo(arg: number): number {
    return arg
}
```
demo这个函数直接返回了传入的值，这个时候确定的是number类型，所以也直接返回了number类型
软件工程的一个主要部分就是构建组件，构建的组件不仅需要具有明确的定义和统一的接口，同时也需要组件可复用。支持现有的数据类型和将来添加的数据类型的组件为大型软件系统的开发过程提供很好的灵活性。

在C#和Java中，可以使用"泛型"来创建可复用的组件，并且组件可支持多种数据类型。这样便可以让用户根据自己的数据类型来使用组件。

####   泛型方法

在ts里，声明泛型方法
```typescript
function identity<T>(arg: T): T {
    return arg;
}
```
调用泛型方法

```typescript
identity<string>("112121");

// 或者
identity("12121");  // 编译器会根据传入参数来自动识别对应的类型。

```

####   泛型与any
typescript的特殊类型any在使用的时候可以代替任意的类型，看似和泛型很像，但实际上有很大差别

```typescript
function anyDemo(arg: any):any {
    console.log(arg.length);
    return arg;
}

function genericsDemo<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);
    return arg;
}
```
+ anyDemo, 因为arg的类型是any,所以可以打印length属性，但是如果arg的实际输入类型不是带有length属性的变量，会抛出异常
+ genericsDemo， 定义了arg类型是数组的泛型类型，所以一定有length属性，所以不会抛出异常


### 3. 自定义类型类型

#### interface 与 type的区别

##### 相同点

######  都可以描述一个对象或函数

```typescript
// interface
interface Blog{
    title: string;
}
interface funcs {
    (name: string): void; 
}
let bar:funcs = function (name) {
    console.log(name)
}
bar("12121")

// type
type Blog = {
    title: string;
}
type funcs = (name: string) => void

let bar: funcs = function (name) {
    console.log(name)
}
bar("112121")

```

######  拓展（extends）与 交叉类型（Intersection Types）
interface是可以extends,但是type是不允许extends和implement的。但是type却可以通过交叉类型实现interface 的extends行为，并且两者并不是相互独立的，也就是说 interface 可以 extends type, type 也可以 与 interface 类型 交叉 。
虽然效果差不多，但是两者语法不同。

```typescript
// interface extends interface
interface Name { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}


// type 与 type交叉
type Name = { 
  name: string; 
}
type User = Name & { age: number  };


// interface extends type
type Name = { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}


// type 与 interface 交叉
interface Name { 
  name: string; 
}
type User = Name & { 
  age: number; 
}

```

##### 不同点


######  type 可以而 interface 不行

+ type 可以声明基本类型别名，联合类型，元组等类型

```typescript
// 基本类型别名
type Name = string;

// 联合类型
type status = string | number

// 元组
type arr = [string, number]
```

+ type 语句中还可以使用 typeof 获取实例的 类型进行赋值

```typescript

// 当你想获取一个变量的类型时，使用 typeof
let div = document.createElement('div');
type B = typeof div

```

+ 其他骚操作

```typescript

type StringOrNumber = string | number;  
type Text = string | { text: string };  
type NameLookup = Dictionary<string, Person>;  
type Callback<T> = (data: T) => void;  
type Pair<T> = [T, T];  
type Coordinates = Pair<number>;  
type Tree<T> = T | { left: Tree<T>, right: Tree<T> };

```

######   interface 可以而 type 不行

```typescript
// interface 能够声明合并
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
*/

```



