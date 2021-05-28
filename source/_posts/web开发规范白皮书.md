---
title: web开发规范白皮书-vue部分
tags: 开发规范
categories: 项目总结
date: 2021/05/28
---
## 前言：
为了进一步提升团队同学的编码质量和研发效率，我们会分几个方向来归纳总结我们已有的开发经验和借鉴其他团队的经验，来帮助我们团队的新人同学能够快速成长起来，尽量减少重复踩坑，并且以此作为团队日常代码revivew的标准，后期会根据实际情况新增或调整。


### 1. 始终在 v-for 中使用 :key
在需要操纵数据时，将key属性与v-for指令一起使用可以让程序保持恒定且可预测。
这是很有必要的，这样Vue就可以跟踪组件状态，并对不同的元素有一个常量引用。在使用动画或Vue转换时，key 非常有用。
如果没有key ，Vue只会尝试使DOM尽可能高效。 这可能意味着v-for中的元素可能会出现乱序，或者它们的行为难以预测。 如果我们对每个元素都有唯一的键引用，那么我们可以更好地预测Vue应用程序将如何精确地处理DOM操作。

```
<!-- 不好的做法-->
<div v-for='product in products'>  </div>

<!-- 好的做法 -->
<div v-for='product in products' :key='product.id'>

```

### 2. data 应始终返回一个函数
声明组件data时，data选项应始终返回一个函数。 如果返回的是一个对象，那么该data将在组件的所有实例之间共享(引用类型)

```
// 不好的做法
data: {
  name: 'My Window',
  articles: []
}
```
但是，大多数情况下，我们的目标是构建可重用的组件，因此我们希望每个组件返回一个惟一的对象。我们通过在函数中返回数据对象来实现这一点。

```
// 好的做法
data () {
  return {
    name: 'My Window',
    articles: []
  }
}
```

### 3. 不要在同个元素上同时使用v-if和v-for指令
为了过滤数组中的元素，我们很容易将v-if与v-for在同个元素同时使用。

```
// 不好的做法
<div v-for='product in products' v-if='product.price < 500'>
```
问题是在 Vue 优先使用v-for指令，而不是v-if指令。它循环遍历每个元素，然后检查v-if条件。

```
// 伪代码
this.products.map(function (product) {
  if (product.price < 500) {
    return product
  }
})
```

这意味着，即使我们只想渲染列表中的几个元素，也必须遍历整个数组。
一个更聪明的解决方案是遍历一个计算属性，可以把上面的例子重构成下面这样的：

```
<div v-for='product in cheapProducts'>
 
computed: {
  cheapProducts: () => {
    return this.products.filter(function (product) {
      return product.price < 100
    })
  }
}
```

这么做有几个好处：

+ 渲染效率更高，因为我们不会遍历所有元素
+ 仅当依赖项更改时，才会重使用过滤后的列表
+ 这写法有助于将组件逻辑从模板中分离出来，使组件更具可读性


### 4. 用正确的定义验证我们的 props

在设计大型项目时，很容易忘记用于props的确切格式、类型和其他约定。如果你在一个更大的开发团队中，你的同事不会读心术，所以你要清楚地告诉他们如何使用你的组件
因此，我们只需编写props验证即可，不必费力地跟踪组件来确定props的格式

```
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

### 5. 组件命名带上统一的前缀，对于不同类型的前缀可以带上不同的前缀

例如统一前缀：
```
lcDialog.vue
lcPopup.vue

// 基础组件
lcBaseButton.vue
lcBaseCell.vue

```

### 6. 不要在“created”和“watch”中调用方法
一些同学经常犯的一个错误是他们不必要地在created和watch中调用方法。 其背后的想法是，我们希望在组件初始化后`立即运行`watch。

```
created: () {
  this.handleChange()
},
methods: {
  handleChange() {
    // stuff happens
  }
},
watch () {
  property() {
    this.handleChange()
  }
}
```

但是，Vue为此提供了内置的解决方案，这是我们经常忘记的Vue watch属性
我们要做的就是稍微重组watch并声明两个属性：

1. handler (newVal, oldVal)-这是我们的watch方法本身
2. immediate: true 代表初始化监听之后立马运行检测方法

```
// 好的做法
methods: {
handleChange() {

// stuff happens
}
},
watch () {
property {

immediate: true
handler() {
  this.handleChange()
}
}
}
```

### 7. template模板表达式应该只有基本的 JS 表达式
在模板中添加尽可能多的内联功能是很自然的。但是这使得我们的模板不那么具有声明性，而且更加复杂，也让模板会变得非常混乱，同时也会降低页面渲染性能

```
//不好的做法
{{
  fullName.split(' ').map(function (word) {
    return word[0].toUpperCase() + word.slice(1)
  }).join(' ')
}}
```

更好的方式是借用computed 来计算这个值，同时也可以利用computed的缓存特性，增强页面渲染的性能

```
// 好的做法
{{ normalizedFullName }}


// The complex expression has been moved to a computed property
computed: {
  normalizedFullName: function () {
    return this.fullName.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ')
  }
}
```

### 8. 父子组件通信首选props 和 emit
要保持一个单向的数据流向，父子组件通信子组件接受父组件传入的props，然后通过emit事件反馈给父组件，不能直接修改pros数据，dev模式下子组件直接修改props vue会有warning

```
// parent
<child-compnnent :direction="up" @confirm="confirm">
...
methods: {
    confirm(e) {
        console.log("收到子组件传入的数据", e)
    }
}
...

// child
...
props: {
    direction: {
        type: string,
        default: "down"
    },
},
methods: {
    hander() {
        // 不要这么干
        this.props.direction = "right"
    },
    childConfirm() {
        this.$emit("confirm", this.xxx)
    }
}
...
```


### 9. [开启路由懒加载](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)
vue 路由懒加载其实依赖于 webpack 的 code-spliting 以及 vue 的异步组件，关于 vue 的异步组件可以看动态组件 & 异步组件，而异步组依赖动态 import

```
const Foo = () => import('./Foo.vue')

const router = new VueRouter({
  routes: [{ path: '/foo', component: Foo }]
})

```
##### 把组件按组分块
有时候我们想把某个路由下的所有组件都打包在同个异步块 (chunk) 中。只需要使用 命名 chunk (opens new window)，一个特殊的注释语法来提供 chunk name (需要 Webpack > 2.4)。

```
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```
Webpack 会将任何一个异步模块与相同的块名称组合到相同的异步块中。

### 10. 模块化路由配置
在中大型项目中，会有很多的页面或模块，常出现路由嵌套的情况。此时，建议以路由层级进行模块拆分。文件结构如下：

```
├── router
│   ├── index.js
│   ├── home.js
│   ├── login.js
```

根据业务模块来租住代码模块划分

```
import homeRoutes from './home'
import loginRoutes from './login'

const routes = [
  {
    path: '/',
    redirect: '/login'
  },
  { 
    name: 'Home'
    path: '/home'
    component: Home,
    children: [...homeRoutes]
  },
  {
    name: 'Login',
    path: 'login',
    component: Login,
    children: [...loginRoutes]
  }
]

export default new VueRouter({
  routes
})
```

### 11. [模块化组织Vuex状态](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart),同时使用mapState、mapGetters、mapMutations和mapAction精简代码

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。
为了解决以上问题，建议使用模块化组织Vuex，将store分割成模块。

```
import Vue from 'vue'
import Vuex from 'vuex'
import cart from './modules/cart'
import products from './modules/products'
import createLogger from '../../../src/plugins/logger'

Vue.use(Vuex)

const debug = process.env.NODE_ENV !== 'production'

export default new Vuex.Store({
  modules: {
    cart,
    products
  },
  strict: debug,
  plugins: debug ? [createLogger()] : []
})
```

### 12. 规范组件选项的顺序

```
// 外部引入配置一律按照对应的类别分类放在一起
// components
// 一些组件import

// service
// 一些服务

// d.ts
// 一些ts类型声明

// config
// 一些配置文件

export default {
  name: '',
  parent: null,
  extends: null,
  minxins: [],
  components: {},
  inheritAttrs: false,
  model: {},
  props: {},
  data () {
    return {}
  },
  computed: {},
  watch: {},
  // 生命周期钩子，按调用顺序编写
  beforeCreate () {},
  ...,
  destroyed () {},
  methods: {},
  directives: {},
  filters: {},
  // 使用render函数时，置于末尾
  render () {}
}
```

### 13. 始终为组件样式设置作用域
全局样式容易污染其他组件样式。在vue组件中一旦使用了全局的style，那么你必将陷入无限的梦魇，因为你根本不知道什么时候组件的样式就被全局样式污染了。因此，建议始终为组件样式设置作用域。


### 14. 化繁为简的计算属性
将复杂计算属性分割为尽可能多的更简单的属性。简单、专注的计算属性减少了信息使用时的假设性限制，所以需求变更时也用不着那么多重构了。如：

```
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}
```

简化后：
```
computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```

### 15. 使用Slots可以使你的自定义组件更强大和便于理解
在大型项目进行过程中，构建的业务组件可能会出现频繁变动的情况，如果单纯靠props 来控制数据输入，会随着项目的复杂度提升而变得越来越难以维护，合理利用slot可以把更多自定义的操作放给调用方，可以尽量解耦，便于拓展维护,例如：

```
<template>
 <div class="c-base-popup">
  <div v-if="$slot.header" class="c-base-popup__header">
   <slot name="header">
  </div>
  <div v-if="$slot.subheader" class="c-base-popup__subheader">
   <slot name="subheader">
  </div>
  <div class="c-base-popup__body">
   <h1>{{ title }}</h1>
   <p v-if="description">{{ description }}</p>
  </div>
  <div v-if="$slot.actions" class="c-base-popup__actions">
   <slot name="actions">
  </div>
  <div v-if="$slot.footer" class="c-base-popup__footer">
   <slot name="footer">
  </div>
 </div>
</template>
 
<script>
export default {
 props: {
  description: {
   type: String,
   default: null
  },
  title: {
   type: String,
   required: true
  }
 }
}
</script>
```


### 16.使用$config访问环境变量（这个思路不错，推荐）
项目定义了一些全局变量

```
config
├── development.json
└── production.json
```
我通常使用this.$config获取，尤其是当我在模板中时。 一如既往扩展Vue对象非常容易：

```
// NPM
import Vue from "vue";
 
// PROJECT: COMMONS
import development from "@/config/development.json";
import production from "@/config/production.json";
 
if (process.env.NODE_ENV === "production") {
 Vue.prototype.$config = Object.freeze(production);
} else {
 Vue.prototype.$config = Object.freeze(development);
}
```

### 17.显示一个大的数据时应该使用Vue虚拟滚动条
在页面中显示多行或需要循环大量数据时，你已经注意到该页面渲染速度很快变慢。要解决此问题，您可以使用[vue-virtual-scoller](https://github.com/Akryum/vue-virtual-scroller)。

```
npm install vue-virtual-scroller

<template>
 <RecycleScroller
  class="scroller"
  :items="list"
  :item-size="32"
  key-field="id"
  v-slot="{ item }"
 >
  <div class="user">
   {{ item.name }}
  </div>
 </RecycleScroller>
</template>
```

### 18.追踪第三方程序包的大小
多人合作一个项目时，如果没人关注安装的依赖包数量很快变的难以置信。为了避免程序变慢（尤其是在移动网络环境），我这VSC中使用import cost package这样就可以编辑器中看到导入的包有多大，并且找出大的原因。

例如在最近的项目中，导入了整个lodash库（压缩后24kB）。 有啥子问题？ 仅仅使用cloneDeep方法。 通过import cost package找到了问题
```
npm remove lodash
npm install lodash.clonedeep
```

在使用的地方导入：
```
import cloneDeep from "lodash.clonedeep";
```
为了进一步优化，我们还可以使用Webpack Bundle Analyzer包通过树状图来可视化Webpack输出文件的大小
