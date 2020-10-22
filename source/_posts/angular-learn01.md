---
title: angular从入门到入土系列01
tags: angular
categories: angular
date: 2020/10-22
---

### angular从入门到入土系列01 -- Angular各个版本差异和应用架构设计

#### 学习路线规划

+ Angular各个版本差异和应用架构设计
+ 过一遍[官方文档](https://angular.cn/guide/architecture-modules)，理清专业术语和掌握基础知识 
+ [Angular In Depth](https://zhuanlan.zhihu.com/DepthInAngular)文章过一遍
+ [30天精通Rxjs系列](https://ithelp.ithome.com.tw/users/20103367/ironman/1199)
+ [读完教程和文档](https://www.typescriptlang.org/docs/handbook/intro.html)，掌握Ts基础
+ 学习组件库 ng-zorro-antd， Angular Material
+ 上手项目实践


#### 第一步 了解背景

对于刚接触Angular的新手来说，第一步无非是打基础，也是基础的一步

AngularJs是google发布的第一个MVC框架，带来了许多新特性，为前端开发提供了很多新的思路，线上直到现在还有许多大型产品在用AngularjJs.

AngularJs实际上是Angular2+，官方推崇类似Chrome的版本迭代理念，让开发者淡化对版本号的变化感知，希望这个框架可以持续发展。Angular是个重架构，提供了模块、组件、装饰器、服务等多种功能组件，编译方式也从JIT(just in time)逐渐过度到AOT(Ahead of time)，不断的优化性能及功能。

而TS的加入也让angular如虎添翼，ts吸取了许多面向对象的编程语言优势，比如java,这样让前端的规模化协作开发不再是难题，工程质量也大幅提升。但是ng也有自身的问题，学习曲线陡峭，对开发人员的整体数值要求还是过高；还有ngzone的性能也是个问题，对外不是太友好，默认截获了所有的浏览器事件
和方法，调优难度和第三方产品对接难度都比较大。


> Angular版本差异？如何选择合适的版本？

+ 2009 -- AngularJs 

    AngularJS 诞生于2009年，bai由Misko Hevery 等人创建，后为Google所收购。是一款优du秀的前端zhiJS框架，已经被用于Google的多款产品当中。AngularJS有着诸多特性，最为核心的是：MVC、模块化、自动化双向数据绑定、语义化标签、依赖注入等等

+ 2016 -- Angular2 
    
    整个重写了1.x, 变得更加简洁，最核心的概念只剩下一个，那就是componnent,其他所有一切都是围绕着Component展开的。从这点看Angular2无疑是收到了React的强力影响，毕竟React的核心概念也只有一个，也是component，所以在使用ng2的时候，大家只需要会写Component就行了，其他的那些什么服务啊，路由啊，管道啊，都是小工具而已。接下来罗列一下Angular2与之前的1.x相比带来的核心改变:
    
    +  Angular删掉了$scope的概念。
        
        在ng1.x里面，$scope是一个相当强大又相当可怕的东西，一言不合就让开发者自己$apply
    +  删掉了ng-controller指令
        
        Controller终于和Component合体了
    +  大幅度演进了脏值检测机制

        双向绑定之所以能够工作，都是因为底层有脏值检测这么个神奇的东西，而实际上ng.1x里面的脏值检测机制的运行效率是非常差的，这就是为什么大家一直在吐槽绑定的对象不能太多、太深的原因。那么，在ng2中，大幅演进了这一机制，不仅引入了单向绑定，还引入了各种绑定策略，例如：只检测一次、利用jit动态生成脏值检测代码等等。毫无疑问，有了这些工具之后，数据绑定效率不再是问题。

    +  嵌套路由问题

        大家都知道，在ng1.x里面有个非常讨厌的问题，官方的路由机制是不能嵌套的，这就导致大家在开发的过程中不得不依赖第三方的ui-router.ng2中没有这个问题了，因为ng2的路由是基于Component的，天然就支持嵌套。
    +  依赖注入机制改造
    +  框架整体上基于Typescript开发， 这是最大的一个变更

+ 2017.03 -- Angular4

    支持向后兼容，优化视图引擎，剥离动画包，以及一些其他功能的改进
    
+ 2017.11 -- Angular5   只是遵循语义化版本号规范的正常迭代

    + 构建优化
    + 编译器改进, 改进了Angular编译器来支持增量编译,重新构建变得更快，特别是对生产环境的构建和AOT编译，增强的装饰器可以通过更精细化的去除空格来减小产生的包.   
    + StaticInjector取代ReflectiveInjector依赖注入器,为了更多的减少polyfills，5.0中使用了StaticInjector注入器来替换原有的ReflectiveInjector注入器，这种注入器不再里来与ReflectPolyfill，可以大幅减少应用程序体积
    + Zone执行速度的提升,5.0中默认提供的zones已经优化过，速度大幅提升，并且在应用程序中绕过zonee区域更加关于应用程序的性能。
    + exportAs多命名支持, 5.0中提供了组件/指令的多命名支持，在对用户不修改代码情况下进行组件的迁移操作等非常有用，将一个组件导出多个名字，可以让组件已一个新名字来使用而达到不破坏现有代码的目的


+ 2018.05 -- Angular6
    Angular v6 新版本重点关注工具链以及工具链在 Angular 中的运行速度问题.Angular v6 是统一整体框架、Material 和 CLI 三大 Angular 组件的第一个版本，此次更新更多地关注于工具链上，以使其具有更好的可移植性。

+ 2018.10 -- Angular7
    
    Angular7  这是跨整个平台的主要版本，包括核心框架，Angular Material和具有同步主要版本的CLI工具。

+ 2019 - Angular8

    作为一个期待已久的重大版本更新，Angular 8 为框架、Angular Material 和命令行界面工具 Angular CLI 带来了大量的改进和新功能。

    +  默认启用差异化加载（Differential loading）
    +  新的渲染引擎 Ivy, 作为新的渲染引擎，Ivy 旨在彻底缩减代码尺寸并增强系统灵活性。与目前的 Angular View Engine 相比，Ivy 具有以下优势：
        
        +  通过 Angular 编译器生成的代码更具可读性，更易调试
        +  更快的构建速度
        +  有效减少负载大小，浏览器用于下载和解析应用程序的时间将更短
        +  更好的模板类型检查，以便在项目构建初期就可捕获更多 Bug
        +  优秀的向后兼容性
    + 使用动态导入进行路由配置,在 Angular 8 中，我们可以使用路由以延迟加载部分应用程序，这是通过在路由配置中使用 loadChildren 键来实现的。
    + 对 Web Worker 的支持, 在 Angular 8 之前，使用 Web Worker 存在这样的问题：在 worker 中运行的代码不能与应用程序的其余部分位于同一 JavaScript 脚本文件中，它必须是分开的。因此，对于曾经希望借助 Angular CLI 等工具，自动将 JavaScript 文件拆分、绑定到更少文件夹下的效果往往不佳,
        Angular 8 的新特性之一就是改进了使用 Angular CLI 捆绑 WebWorker 的支持，这项改进意味着我们将走向多并发、自动化的 Web Worker 之路。

+ 2020 -- Angular9

    这是涵盖整个平台的主要版本，包括框架，Angular Material和CLI。此版本默认情况下将应用程序切换到Ivy编译器和运行时，并引入了改进的组件测试方法
    
    + 默认使用 Ivy 编译器， Ivy 在 Angular8 时即可使用，但需要自行在 tsconfig.json 中增加配置以开启
    + 更可靠的 ng update
        
        +  始终使用最新的 CLI。从 CLI 的 8.3.19 版本开始，我们现在在升级过程中使用了 TARGET 版本的 CLI。这意味着在未来，更新将始终由最新的 CLI 自动处理
        + 进度更新更清晰。ng update 现在做了更多的工作来告诉你幕后的情况。对于每次迁移，你都会看到迁移中的更多信息。
        + 更容易对更新进行调试。 默认情况下，ng update 运行所有迁移，并在磁盘上留下最终结果更改供你检查。版本 9 的更新还引入了新的 --create-commits 标志。当你运行 ng update --create-commits 时，该工具会在每个迁移动作后提交代码库的状态，这样你就可以逐步理解或调试我们对代码所做的更改。
    
    + providedIn 的新选项

        当你在 Angular 中创建一个 @Injectable 服务时，你必须选择它应该添加到注入器的什么位置。除了以前的 root 和 module 这两种选项之外，还有两个新选项：

        +  platform: 指定 providedIn: 'platform' 可以在一个特殊的单例平台注入器中使用该服务，该注入器由该页面上的所有应用共享。
        + any: 在每个注入该令牌的模块（包括惰性加载模块）中提供一个唯一的实例

    + TypeScript 3.7 支持
        
        Angular 已经更新，可以用 TypeScript 3.6 和 3.7 了，也包括 TypeScript 3.7 中非常受欢迎的可选串联（optional chaining）特性。为了与生态系统保持同步，还更新了其他生态系统依赖的版本，比如 Zone.JS 和 RxJS

    + 移除 tslib 依赖项 ,Angular 现在不再依赖 tslib。在早期版本的 Angular 中它是必需的，并且是依赖项的一部分。如果你不用 Angular CLI，则可能需要安装这个包。


+ 2020 --  Angular10

    这是跨越整个平台（包括框架、Angular Material 和 CLI）的一次主要版本更新

    [Angular 10正在接近终点线：带来哪些颠覆性变化](https://baijiahao.baidu.com/s?id=1673431402914533055&wfr=spider&for=pc)


#### 第二步 全局看Angular应用架构设计

##### 1. 组件设计-显示组件与功能组件



