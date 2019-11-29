---
title: es6内容解读
tags: js
categories: 项目总结
date: 2019/11/29
---

### 赋值解构

- 赋值解构使得声明变量更简洁。特点是左边多个变量，右边是一个对象或数组。


```js
// 解构出对象的属性
let aa = {b: '11', c:'22'};
let {b,c} = aa;
console.log(b,c);	// 11 22
```

```js
// 解构出对象的属性并重新取名
let a = {b: '11', c:'22'};
let {b: bb,c: cc} = aa;
console.log(bb,cc);	// 11 22
```

```js
// 解构数组
let [a, b] = [1, 2, 3];
console.log(a,b);	// 1 2
```

```js
// 解构时给变量默认值
[a=5, b=7] = [1];
console.log(a,b);	// 1 7
```

```js
// 解构 rest的使用。...能解构剩余属性值
[a, b, ...rest] = [10, 20, 30, 40, 50];
console.log(a,b,rest);	// 10 20 [30, 40, 50]
({a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40});
console.log(a,b,rest);	// 10 20 {c: 30, d: 40}
```

### Generator生成器

- 可以理解为一个状态机，封装了多个内部状态，每次调用就遍历并执行一个状态。这样就可以使得多个函数共同配合完成某个任务。


```js
function* helloWorldGenerator() {
  yield 'hello';	
  yield 'world';
  return 'ending';	// 以return作为结束执行的标志
}
var hw = helloWorldGenerator();
hw.next();	// 执行后的数据为一个对象：{ value: 'hello', done: false }
hw.next();	// { value: 'world', done: false }
hw.next();	// { value: 'ending', done: true }
hw.next();	// { value: undefined, done: true }

// 也可以直接return 不必一直调用next() 
hw.return('aaa');	// { value: 'aaa', done: true }

// yield* fn() 这种用法表示generator的嵌套执行
```

- Generator可以作为一个暂缓执行函数


```js
function* f() {
  console.log('执行了！')
}

var generator = f();	// 如果这里的f()是普通函数，那么这里就执行了f()里面的内容

setTimeout(function () {
  generator.next()
}, 2000);
```

- Generator有3个方法 next()  throw()  return()。它们将yield替换成值、throw语句、return语句。next里面传值时会被当做上一个yield的值

- Generator轻松遍历所以数组成员

```js
function* iterTree(tree) {
  if (Array.isArray(tree)) {	// 如果是Array
    for(let i=0; i < tree.length; i++) {
      yield* iterTree(tree[i]);	// 将Array的每一项当参数继续传入函数执行
    }
  } else {	// 否则直接返回
    yield tree;
  }
}
const tree = [ 'a', [[1, 2, 3],'b', 'c']];
for(let x of iterTree(tree)) {	// for of 能循环执行yield
  console.log(x);	// a 1 2 3 b c
}
```

- Generator实现协程。A B两个方法交替执行完成同一个任务

```js
// A函数
function* A(){
  console.log("1");
  yield "B";//交由B处理
  console.log("11");
  yield "B";//交由B处理
}
// B函数
function* B(){
   console.log("2");
   yield "A";//交由A处理
   console.log("22");
   yield "A";//交由A处理
   console.log("222");//上菜
}
var fnA = A();
var fnB = B();
//流程控制
function run(fn){
   var v = fn.next();
   if(v.value =="A"){
      run(fnA);	// 当函数返回值是A 再次进入run 达到循环调用的目的
   }else if(v.value =="B"){
      run(fnB);
   }
}
run(fnB);//开始执行 2 1 22 11 222
```

- 生成器本身带有Symbol.iterator。Symbol：唯一标识id，iterator：生成器

### 监听对象属性变化

- **Object.defineProperty**通过监听对象属性的读取和修改触发函数执行业务代码

```js
 let obj = {
    _hello:'hello world' //表示私有变量
};
Object.defineProperty(obj,'hello',{
    get() {
        console.log('get');
        return this._hello;	// 这里会返回一个值修改obj的hello属性
    },
    set:function (value) {
        console.log('set');
        this._hello = value; // 使得obj的私有属性_hello与hello属性保持一致
    }
});
console.log(obj.hello);
obj.hello = 'goodbye';
console.log(obj.hello);
```

- 代理**Proxy**类似于defineProperty的加强版

```js
let obj = {
     hello:'hello world'
 };
let proxy = new Proxy(obj, {
    get(target, property) {
        console.log('get');
        return target[property];
    },
    set(target,property,value) { // target当前对象 property属性 value修改的属性值
        console.log('set');
        target[property] = value;
    }
});
console.log(proxy.hello);
proxy.hello = 'goodbye';
console.log(proxy.hello);
```

### async await的用法

async和await配合使用常用来解决回调地狱的问题，可以使代码更简洁

```js

function takeLongTime() {	// 返回一个promise对象
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);//这里用定时器模拟异步操作
    });
}
async function test() {		// async申明的函数本质上是一个promise对象
    const v = await takeLongTime();
    console.log(v);	// 只有等待takeLongTime()执行完才打印信息，在此函数体内就把异步操作变同步了
}
test();
```

异步操作结束->通知到resolve()函数->通知到await可以执行下面代码了。用generate生成器也可以做到这一点。

### call、apply、bind的区别

每个函数都包含这3个方法，它们能修改this的指向

```js
var a = {
    user:"Ice",
    fn:function(e,ee){
        console.log(this.user); //Ice
		console.log(e+ee);
    }
}
var b = a.fn; // 申明变量b继承变量a的一个方法
b(1, 2);  // underfined NaN b继承了a的方法但是它的this指向的是全局变量
b.call(a,1,2); // call使b的this指向了a
b.apply(a,[1,2]); // 同上，区别在于传参不同
b.bind(a);	// 这里 bind返回了一个新的函数，这里并不会立即执行
```

### for...in和for...of

for循环相比于用api循环的优点在于可以主动用break结束循环，用return结束函数。

```js
// for in遍历对象
var obj = {a:1, b:2, c:3};
for (var prop in obj) {
  console.log(prop);	// a b c
}
// for in遍历数组
var obj = ['a', 'b', 'c'];
for (var prop in obj) {
  console.log(prop);	// ‘0’ ‘1’ ‘2’ 由此可见for in会自动添加下标
}
//for of遍历数组、字符串、Set、Map等
var a = [{a:1}, {b:2}, {c:3}];
for(let val of a) {
	console.log(val);	// {a:1} {b:2} {c:3} 遍历出值
}
```

for...of其实隐式调用了Iterator方法（不断调用iterator.next()）

执行效率：常规for循环 > forEach > for...of  >  map >  for...in

### 面向对象Class

1. ##### 构造函数

   - 构造函数其实就是一个普通的函数。

   - 构造函数的写法：函数体内用this声明一些共有属性，用.prototype来申明共有方法，用new 构造函数()创建一个对象继承构造函数的属性和方法。通过原型的继承，它节约了内存。

   - js创建对象时都有一个_proto_的内置属性，它指向了构造函数的prototype。构造函数.prototype.constructor又指向构造函数本身

     ```js
     var person = function(name){
        this.name = name
      };
     person.prototype.getName = function(){
          console.log('名字是:'+this.name); 
     }
     var aa = new person('cat');
     var bb = new person('dog')
     aa.getName(); 	//名字是:cat
     bb.getName();	//名字是:dog
     console.log(aa.__proto__ === person.prototype);  //true
     console.log(person.prototype.constructor === person);  //true
     ```

2. ##### Class

   - 用class改造上面的构造函数。实例化一个对象的时候constructor里面的内容会执行一遍。

   ```js
   class person {
       constructor(name) {
           console.log('constructor执行了');
           this.name = name;
       }
       getName() {
           console.log('名字是:'+this.name);
       }
   }
   var a = new person('cat');
   var b = new person('dog');
   a.getName();  //名字是:cat
   b.getName();  //名字是:dog
   ```

   - class可以通过extends来继承，constructor内使用super()调用父类的构造方法

     ```js
     class personA extends person {
     	constructor(name, food) {
     		super(name);
     		this.food = food;
     	}
     	myfood() {
             console.log('吃东西：'+this.food)
         }
     }
     var a = new personA('cat', 'apple');
     a.myfood();
     a.getName();
     ```
