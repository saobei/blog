---
title: Typescript 最佳实践
tags: 
    - typescript
    - 实战
categories: typescript
date: 2019/10/10
---

### 不能忽略的options

聊一下一些默认关闭着的、却又非常影响类型系统的实际效果的选项。下面这些选项，如果你有类型洁癖，都可以开启。若你想得到类型推导系统带来的好处，建议至少开启noImplicitAny和strictNullChecks。
若你只想获得编辑器的提示，又不想陷入类型的泥淖中，可以保留默认值。

+ noImplicitAny  当存在不明确的any时报错。
+ noImplicitThis 当this 无法正确推导时报错
+ alwaysStrict  在解析时使用严格模式，并且为每个文件增加 "use strict"
+ strictBindCallApply  对使用bind 、call 和 apply 调用的函数进行严格的检查
+ strictNullChecks 严格检查可空类型 （对于可以根据上下文逻辑，确定非空的地方，可使用感叹号!来去除类void的类型）。
+ strictFunctionTypes 严格检查函数的参数 （主要针对参数继承的情况，用的比较少）
+ strictPropertyInitialization  严格检查非空的类的属性是否都在constructor中进行初始化
+ strict 开启noImplicitAny、noImplicitThis、alwaysStrict、strictBindCallApply、strictNullChecks、strictFunctionTypes和strictPropertyInitialization。
+ noImplicitReturns 当不存在明确的return 语句时报错
+ noUnusedLocals  当存在没有使用的局部变量时报错。
+ noUnusedParameters  当存在没有使用的函数参数时报错（如果需要占位，可使用下划线_）。
+ suppressImplicitAnyIndexErrors 对于隐式使用索引类型的地方进行检查
+ suppressExcessPropertyErrors  是否检查过量的属性（酌情使用，建议开启）


### 副作用 side effect

前端开发始终都在和副作用作斗争。拿到底什么是副作用呢？举一个简单的例子就可以解释清楚了，假如我们写了一段代码，我们本地测试ok，就提交集成测试了，大多数比较正常的请款是测试失败。一般来说，失败的原因集中在，某种特殊的情况下，后端接口的返回与预期不一致。在这个例子中，API调用属于副作用的一种。通常意义上的副作用是，调用函数后，除了函数的返回值之外，还会对主调用函数产生附加影响。

假如我们把整个页面应用看作一个函数，那么他的副作用一般来源于一下几个方面：

+ 网络通信（http 调用， ws等）
+ 文件或数据IO（localStorage、IndexedDB、sessionStorage、File等）
+ 用户交互（按钮点击、表单输入、URL改变等）

这些副作用时时刻刻在影响我们应用的状态，可以说是Bug的源泉。然而，我们却很少善待这一部分的工作。比如，我们通常喜欢快刀斩乱麻似的定义API的结构、然后进入开发环节；我们不假思索的将数据存储在localStorage确很少关心什么时候删除。

遗憾的是，TS不会帮我们解决这些Bug，但它给我们提供了一个解决这些问题的思路，那就是在有Side Effect的地方定义更为明确的类型，比如：

```typescript

// 一般做法
const getDataFromStorage = (key: string) => {
  const str = window.localStorage.getItem(key);
  return str ? JSON.parse(str) : null;
};
const someData = getDataFromStorage("DATA_KEY");

// 推荐做法
interface DataType {
    name: string;
    age: number;
}
function getDataFromStorage<T>(key: string): T | null {
    const str = window.localStorage.getItem(key);
    return str ? JSON.parse(str) : null;
}
const someData = getDataFromStorage<DataType>("DATA_KEY");

```

这里只列举了localStorage的情况，对于其他副作用，也是同样的。我们可以基于该例子推广到一般处理副作用的函数上，对于这里函数，最好能满足这些特点：

+ 明确返回值的可能性，比如对于 getDataFromStorage 的返回值可能是 null 、也可能是 T类型
+ 提供可定制的泛型参数，比如对于getDataFromStorage函数而言，根据key的参数不同，返回值的类型可能不同，这里我们可以用泛型提供这种定制的可能性。
+ 提供明确的泛型类型，比如对于getDataFromStorage函数而言，这里明确了类型是DataType而不是由TS默认推导出的{}类型（在未来的TS版本中，它会被改为unknow类型）

如果能够做到这几点，我无法保证项目中的Bug数量会降低一个数量级，但至少会让不清楚API历史的新人踩更少的坑，从而提高团队的效率。


### 泛型优于联合类型（union type）

联合类型可以说是在ts中最容易被滥用的类型之一。
先看一个例子：

```typescript

interface Bird {
  fly(): void;
  layEggs(): boolean;
}
interface Fish {
  swim(): void;
  layEggs(): boolean;
}
// 获得小宠物，这里认为不能够下蛋的宠物是小宠物。现实中的逻辑有点牵强，只是举个例子。
function getSmallPet(...animals: Array<Fish | Bird>): Fish | Bird {
  for (const animal of animals) {
    if (!animal.layEggs())
      return animal;
  }
  return animals[0];
}

let pet = getSmallPet();
pet.layEggs(); // okay 因为layEggs是Fish | Bird 共有的方法
pet.swim(); // errors 因为swim是Fish的方法，而这里可能不存在

```

这个例子存在三个问题：

+ 类型定义使getSmallPet变得局限。从代码逻辑看，它的作用是返回一个不下蛋的动物，返回的类型指向的是Fish或Bird。但我如果只想在一群鸟中挑出一个不下蛋的鸟呢？通过调用这个方法，我只能得到一个或者是Fish、或者是Bird的神奇生物。
+ 代码重复、难以扩展。比如，我想再增加一个乌龟。我必须找到所有类似Fish | Bird的地方，然后把它修改为Fish | Bird | Turtle。
+ 类型签名无法提供逻辑相关性。我们再审视一下类型签名，完全无法看出这里为什么是Fish | Bird而不是其它动物，它们两个到底和逻辑有什么关系才能够被放在这里。


我们用泛型来解决这个问题：

```typescript

// 将共有的layEggs抽象到Eggable接口
interface Eggable {
  layEggs(): boolean;
}
interface Bird extends Eggable {
  fly(): void;
}
interface Fish extends Eggable {
  swim(): void;
}

// 使用泛型，并用泛型约束声明接收的动物必须是继承了Eggable接口的类型，也展示了这个函数的作用范围是Eggable
function getSmallPet<T extends Eggable>(...animals: Array<T>): T {
  for (const animal of animals) {
    if (!animal.layEggs())
      return animal;
  }
  return animals[0];
}
// 可以用于继承了Eggable的任意类型，比如这里想找到鱼里面不能下蛋的鱼也是可以的
let pet = getSmallPet<Fish>();
pet.layEggs(); // okay
pet.swim(); // okay

```


### 巧用typeof优于自定义类型

在副作用一节，强调了为副作用定义良好类型的重要性。这一节恰好相反，是为了强调在没有副作用的代码中，使用typeof偷懒的重要性。在前端页面，大多数数据来源于服务器，但有少部分数据是写死在前端的。对于这些写死的数据，除本系列第二篇文章提到一种处理手段外，我们有时候还需要这样的类型参与到计算中，这时，可以使用typeof来获取我们想要的类型。

举个例子：
比如，我们定义一个URL列表，并且要为类型是通知的标题加上[通知]两个子，代码如下：

```typescript

enum URLTypes {
    NORMAL,
    NOTIFICATION
}
const urlconfigs = [
    {
        type: URLTypes.NORMAL,
        title: "普通页面",
        component: 'div'
    },
    {
        type: URLTypes.NOTIFICATION,
        title: "通知页面",
        component: 'p'
    }
]

// 提取urlconfigs 的类型 ，以便给getTitle函数使用
type UrlConfigItem = typeof urlconfigs[0];

function getTitlte(item: UrlConfigItem, title: string) {
    if(item.type === URLTypes.NOTIFICATION) {
        return `[通知]-${title}`;
    }
    return title;
}

```

这样，我们可以借助于TS，少些一些类型定义。另一个通常能够用到这个技巧的地方是类的函数，有时，我们希望将一个函数的返回值传给另一个函数，而这时我们又不想对函数接收的参数进行重复的类型定义，这种情况下，我们可以使用ReturnType获得函数的返回类型，然后用于另一个函数的参数。

### 未完待续..

