---
title: angular从入门到入土系列02
tags: angular
categories: angular
date: 2021/03-23
---

### 组件

#### 1. 组件是什么？
    angular最基本的构造块

#### 2. 如何创建组件？

a. 命令的方式创建
```bash
ng generate(g) component(c) componentName(组件路径+名称，路径不加的话默认根路径) --module moduleName(指定这个组件归属于哪一个模块)
```

b. 手动创建
在对应的目录新建对应的ts,html,css文件即可

>>> 两种组件模板声明

```typescript
// 引用外部文件方式
@Component({
  selector: 'app-component-overview',
  templateUrl: './component-overview.component.html',
  styleUrls: ['./component-overview.component.css']
})
```

```typescript
// 直接写在组件内部
@Component({
  selector: 'app-component-overview',
  template: '<h1>Hello World!</h1>',
  styles: ['h1 { font-weight: normal; }']
})
```

### 3. 组件有哪些生命周期?
1. 具体见文档

2. 有哪些注意点？

+ @angular/core 有导出对应额生命周期，需要自己去“实现”，比如 OnInit 的钩子方法叫做 ngOnInit
```javascript
@Directive()
export class PeekABooDirective implements OnInit {
  constructor(private logger: LoggerService) { }

  // implement OnInit's `ngOnInit` method
  ngOnInit() { this.logIt(`OnInit`); }

  logIt(msg: string) {
    this.logger.log(`#${nextId++} ${msg}`);
  }
}
```

+ 你不需要实现所有的生命周期，只需要实现自己需要用的就可以了
+ 在构造函数外部执行复杂的初始化。组件的构造应该便宜又简单。比如，你不应该在组件构造函数中获取数据。当在测试中创建组件时或者决定显示它之前，你不应该担心新组件会尝试联系远程服务器。
+ 在 Angular 设置好输入属性之后设置组件。构造函数应该只把初始局部变量设置为简单的值。请记住，只有在构造完成之后才会设置指令的数据绑定输入属性。如果要根据这些属性对指令进行初始化，请在运行 ngOnInit() 时设置它们。

+ 在实例销毁时进行清理：把清理逻辑放到ngOnDestro中， 这个逻辑必然就会在组件执行销毁指令之前执行。这里是释放资源的地方，这些资源不会被自动垃圾回收。如果不这样做，会存在内存泄漏的风险
  + 取消可订阅对象和DOM事件
  + 清楚定时器
  + 反注册该指令在全局或应用服务中注册过的所有回调

> ngOnDestroy() 方法也可以用来通知应用程序的其它部分，该组件即将消失

### 4. 视图封装是什么？有几种模式？

在angular中，组件的样式被封装进了自己的视图中，而不会影响到应用程序的其他部分。
通过在组件元数据上设置视图封装模式，可以分别控制每个组件的封装模式，共分为以下几种：
+ `ShadowDom 模式`(https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM);组件的视图被附加到这个 Shadow DOM 中，组件的样式也被包含在这个 Shadow DOM 中。(译注：不进不出，没有样式能进来，组件样式出不去。)
+ `Emulated 模式`（默认值）通过预处理（并改名）CSS 代码来模拟 Shadow DOM 的行为，以达到把 CSS 样式局限在组件视图中的目的。 更多信息，见附录 1。(译注：只进不出，全局样式能进来，组件样式出不去)
+ `None`意味着 Angular 不使用视图封装。 Angular 会把 CSS 添加到全局样式中。而不会应用上前面讨论过的那些作用域规则、隔离和保护等。 从本质上来说，这跟把组件的样式直接放进 HTML 是一样的

如何设置？
通过组件元数据中的encapsulation属性来设置组件的视图封装模式
```typescript
// warning: few browsers support shadow DOM encapsulation at this time
encapsulation: ViewEncapsulation.ShadowDom
```

具体说明见文档


### 5. 组件之间是如何进行交互的？

+ 通过输入性绑定把数据从父组件传到子组件
  `@Input` 装饰器
  ```
  import { Component, Input } from '@angular/core';

  import { Hero } from './hero';

  @Component({
    selector: 'app-hero-child',
    template: `
      <h3>{{hero.name}} says:</h3>
      <p>I, {{hero.name}}, am at your service, {{masterName}}.</p>
    `
  })
  export class HeroChildComponent {
    @Input() hero: Hero;  // 
    @Input('master') masterName: string; // 第二个 @Input 为子组件的属性名 masterName 指定一个别名 master, 不推荐这么干
  }

  // 父组件调用
  <app-hero-child *ngFor="let hero of heroes" [hero]="hero" [master]="master"> </app-hero-child>
  ```

+ 通过setter截听输入属性值的变化

```
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-name-child',
  template: '<h3>"{{name}}"</h3>'
})
export class NameChildComponent {
  @Input()
  get name(): string { return this._name; }
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';
}
```

+ 通过ngOnChange()来截听输入属性值的变化
利用OnChanges生命周期钩子来检测输入值的变化并作出回应，当需要监视多个、交互式输入属性的时候，这个方法比setter更合适


+ 父组件监听子组件的事件, `@Output` `EventEmitter` `emit`

+ 父组件和子组件通过本地变量互动
父组件不能通过使用数据绑定的方式来使用子组件的属性和方法，但可以在父组件的模板里，新建一个本地变量来代表子组件，然后利用这变量来读取子组件的方法和属性

```
// 宿主组件模板
  <button (click)="timer.start()">Start</button>
  <button (click)="timer.stop()">Stop</button>
  <div class="seconds">{{timer.seconds}}</div>
  <app-countdown-timer #timer></app-countdown-timer>
```


+ 父组件调用@ViewChild
方便父组件的类（component.ts控制器）调用子组件的数据和方法，上面所说的本地变量的方式无法满足。这种是当父组件的类需要访问时，把子组件作为viewChild注入到父组件里

+ 父组件和子组件通过服务的方式进行通信
父组件和它的子组件共享同一个服务，利用该服务在组件家族内部实现双向通讯。


### 6. 聊一下组件的样式

+ 范围化的样式(上述提到的视图封装模式)
> 在 @Component 的元数据中指定的样式只会对该组件的模板生效。

+ 特殊的选择器
:host 使用 :host 伪类选择器，用来选择组件宿主元素中的元素（相对于组件模板内部的元素）
:host 选择是是把宿主元素作为目标的唯一方式。注意：宿主不是组件自身模板的一部分，而是父组件模板的一部分。
要把宿主样式作为条件，就要像函数一样把其它选择器放在 :host 后面的括号中。
当再次将宿主元素作为目标的时候，只有当其带有同样的名称的时候，在会生效。

:host-context 
有时候，基于某些来自组件视图外部的条件应用样式是很有用的。 例如，在文档的 元素上可能有一个用于表示样式主题 (theme) 的 CSS 类，你应当基于它来决定组件的样式。这时可以使用 :host-context() 伪类选择器。它也以类似 :host() 形式使用。它在当前组件宿主元素的祖先节点中查找 CSS 类， 直到文档的根节点为止。在与其它选择器组合使用时，它非常有用


#### 7. 如何动态加载组件？
见代码示例

#### 8. 模板相关
细节见文档

+ 将ngForm与模板变量一起使用
引用指令的exportAs名字来引用不同的值
```
<form #itemForm="ngForm" (ngSubmit)="onSubmit(itemForm)">
  <label for="name">Name <input class="form-control" name="name" ngModel required />
  </label>
  <button type="submit">Submit</button>
</form>

<div [hidden]="!itemForm.form.valid">
  <p>{{ submitMessage }}</p>
</div>
```
如果没有ngForm这个属性值，itemForm引用的是HTMLFormElement也就是form元素。而Component和Directive之间的差异在于Angular在没有指定属性值的情况下，Angular会引用Component,而Directive不会改变这种隐式引用（即他的宿主元素）
而是用了NgForm之后,itemForm就是对ngForm指令的引用，可以用它来跟踪表单组件中每个控件的值和有效性
与原生的form不同，ngForm指令有一个form属性。如果itemForm.form.valid无效，那么ngForm的form属性就会让你禁用提交按钮。

+ 在嵌套模板中访问
内部模板可以访问外模板定义的模板变量。

```
<input #ref1 type="text" [(ngModel)]="firstExample" />
<span *ngIf="true">Value: {{ ref1.value }}</span>
```

这种情况下，有一个包含这个 <span> 的隐式 <ng-template>，而该变量的定义在该隐式模板之外。访问父模板中的模板变量是可行的，因为子模板会从父模板继承上下文
等同于下面和这个情况

```
<input #ref1 type="text" [(ngModel)]="firstExample" />

<!-- New template -->
<ng-template [ngIf]="true">
  <!-- Since the context is inherited, the value is available to the new template -->
  <span>Value: {{ ref1.value }}</span>
</ng-template>
```

但是从外部的父模板访问本模板中的变量是行不通的

```
<input *ngIf="true" #ref2 type="text" [(ngModel)]="secondExample" />
<span>Value: {{ ref2?.value }}</span> <!-- doesn't work -->
```

等同于

```
<ng-template [ngIf]="true">
  <!-- The reference is defined within a template -->
  <input #ref2 type="text" [(ngModel)]="secondExample" />
</ng-template>
<!-- ref2 accessed from outside that template doesn't work -->
<span>Value: {{ ref2?.value }}</span>
```

### 9. 指令相关

+ 属性指令
说明见文档

常见的内置属性指令
  + NgClass -- 添加和删除一组css类 ,要添加或删除单个类请使用 [class] 类绑定而不是 NgClass
  + NgStyle -- 添加和删除一组Html样式 ,有些情况需要考虑使用 [style] 样式绑定，而不是NgStyle
  + NgModel -- 将数据双向绑定添加到html元素



+ 结构指令

常见的内置结构指令
  + NgIf -- 从模板创建或销毁视图
  + NgFor -- 为列表中的每一个条目重复渲染一个节点
  + NgSwitch -- 一组在备用视图之间切换的指令


+ 为啥要在前面加一个 * ？
星号是一个用来简化更复杂语法的“语法糖”。 从内部实现来说，Angular 把 *ngIf 属性 翻译成一个 <ng-template> 元素 并用它来包裹宿主元素，

```
<div *ngIf="hero" class="name">{{hero.name}}</div>
```

```
<ng-template [ngIf]="hero">
  <div class="name">{{hero.name}}</div>
</ng-template>
```
ng-template 永远不会渲染出来。只有最终产出的结果会生成出来

+  显示与隐藏NgIf

隐藏元素时，该元素及其所有后代仍保留在 DOM 中。这些元素的所有组件都保留在内存中，Angular 会继续做变更检查。它可能会占用大量计算资源，并且会不必要地降低性能。

NgIf 工作方式有所不同。如果 NgIf 为 false，则 Angular 将从 DOM 中删除该元素及其后代。这销毁了它们的组件，释放了资源，从而带来更好的用户体验。

如果要隐藏大型组件树，请考虑使用 NgIf 作为显示/隐藏的更有效替代方法。

+ 防范空指针错误
ngIf 另一个优点是你可以使用它来防范空指针错误。显示/隐藏就是最合适的极简用例，当你需要防范时，请改用 ngIf 代替。如果其中嵌套的表达式尝试访问 null 的属性，Angular 将引发错误。

```
<div *ngIf="currentCustomer">Hello, {{currentCustomer.name}}</div>
```

+ 带 tracckBy的*ngFor
你可以使用 trackBy 来让它更加高效。向该组件添加一个方法，该方法返回 NgFor 应该跟踪的值。这个例子中，该值是英雄的 id。如果 id 已经被渲染，Angular 就会跟踪它，而不会重新向服务器查询相同的 id

```

// component
trackByItems(index: number, item: Item): number { return item.id; }

// html
<div *ngFor="let item of items; trackBy: trackByItems">
  ({{item.id}}) {{item.name}}
</div>
```




