---
title: angular相关知识分享
tags: angular
categories: angular
date: 2020/03-09
---

### angular相关知识分享 

---

#### resolve的使用

##### 1、定义

```js
官方定义：一个接口，某些类可以实现它以扮演一个数据提供者
解释：类似于路由前置钩子，可以在路由加载到新页面的时候，将数据提前获取到，从而解决加载新页面白屏的问题，当然也可以使用loading或者骨架屏之类的解决方案
```

##### 2、创建resolve

> index-resolve.service.ts

```js
import { Resolve } from '@angular/router';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
@Injectable(); // 表明将这个服务标记为可注入的
// 导出一个实现resolve的类
export class IndexResolveService impolements Resolve<any> {
  constructor(
  	private indexService: IndexService
  ){}
  // 必须声明resolve方法，返回一个Observable
  resolve(): Observable<any>{
    return this.indexService.getData();
  }
}
```

##### 3、使用

> Index-routing.module.ts

```js
import { IndexResolveService } from 'index-resolve.service.ts';
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
const routes: Routes = [
  {
    path:'xxx',
    component: 'xxx',
    resolve: { indexData: IndexResolveService } // 在路由中定义resolve属性
  }
]
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
  providers: [IndexResolveService] // 注入服务
})
```

> Index.component.ts

```js
import { ActivatedRoute } from '@angular/router';
import { OnInit } from '@angular/core';
export class IndexComponent implements OnInit {
  constructor(
  	private route: ActivatedRoute // ActivatedRoute里面包含了路由的相关信息
  ){}
  ngOnInit(): void{
    this.route.data.subscribe(res=>{
			const data = res.indexData;
    })
  }
}
```

##### 4、操作符

- map
  1. 定义

  ```js
  定义：将给定的project函数应用于源Observable发出的每个值，并将结果值作为Observable发出
  解释：跟array的map类似，可以对里面的值进行操作，然后以Observable发射出去
  ```
  
  2. 使用
  
  ```js
  import { map } from 'rxjs/internal/operators';
  this.indexService.getData().pipe(
  	map((res)=>{
      return xxx;
    })
  )
  ```

- forkJoin，zip

  1. 定义

  ```js
  forkJoin定义：当所有可观测值完成时，从每个可观测值发出最后一个（最新）发出的值
  解释：类似promise.all，当所有的Observable流结束后，发出每个流的最后一个发出的值
  
  zip定义：当每个传入的可观测值第一次完毕时，合并流然后发出去，当每个传入的可观测值第二次、第三次...完毕时，又分别合并发出去
  解释：跟forkJoin一样都是合并流发出，但是zip会根据流本身发出的次数多次发出，而forkJoin仅发射最后一次
  ```

  2. 使用

  ```js
  import { forkJoin } from 'rxjs';
  resolve(): Observable<any>{
    return forkJoin([
    	this.indexService.getData(),
      this.indexService.getData1()
    ])
  }
  ```

- first，last，take

  1. 定义

  ```js
  fitst定义：只发出由源Observable所发出的值中第一个（或第一个满足条件的值）
  解释：在所有的流中取每个流第一次发出的值后结束
  
  last定义：只发出由源Observable所发出的值中最后一个
  解释：在所有的流中取每个流最后一次发出的值后结束
  
  take定义：只发出由源Observable所发出的值中最初发出的n个值
  解释：在所有的流中取每个流最初发出的n个值后结束
  ```

  2. 使用

  ```js
  import { first } from 'rxjs/internal/operators';
  import { forkJoin } from 'rxjs';
  resolve(): Observable<any>{
    return forkJoin([
    	this.indexService.getData(),
      this.indexService.getData1()
    ]).pipe(first())
  }
  ```

  

#### pipe的使用

##### 1、创建pipe

> index.pipe.ts

```js
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'index', // 管道名称
  pure: true // 是否为纯管道，纯管道表示transform()方法只有在输入参数变化时才会调用，默认 true
})
export class IndexPipe implements PipeTransform {
  transform(value, name) {
    // value: 需要转换的值
    // name: 传入的参数
    ...
    return xxx;
  }
}
// 定义多个管道
@Pipe({})
export class ...
```

##### 2、使用

> index.module.ts

```js
import { IndexPipe } from 'index.pipe.ts';
@NgModule({
  declarations: [IndexPipe], // 声明管道
  ...
  exports: [IndexPipe] // 导出管道
})
```

> index.component.html

```html
<div>
  {{value | index : 'xxx'}}
</div>
```



#### ngrx的使用

##### 1、安装

``` js
npm i @ngrx/store -S
yarn add @ngrx/store
ng add @ngrx/store // 如果使用angular-cli 6+ ，可以直接在项目中添加
```

##### 2、创建动作action

> animal.action.ts

```js
import { createAction, props, Action } from '@ngrx/store';
import { Animal } from 'animal.model.ts'; // {name: string; age: number}
// 方式A 8.x版本
export const CreateAnimal = createAction(
	'[CreateAnimal Page] CreateAnimal', // 通常[]里面声明使用动作的页面，后面声明触发动作的操作
  props<{animal: Animal}>() 					// 定义动作接收的参数
)
export const DeleteAnimal = createAction(
	'[DeleteAnimal Page] DeleteAnimal',
  props<{animalLen: number}>()
)

// 方式B 7.x版本
export const ActionTypes = {
  CREATE_ANIMAL: '[CreateAnimal Page] CreateAnimal',
  DELETE_ANIMAL: '[DeleteAnimal Page] DeleteAnimal'
}
export class CreateAnimal implememts Action {
  readonly type = ActionTypes.CREATE_ANIMAL; // type用来描述发生了什么动作
  public animal: Animal;										 // 定义动作接收的参数
  constructor() {}
}
export class DeleteAnimal implememts Action {
  readonly type = ActionTypes.DELETE_ANIMAL;
  public animalLen: number;
  constructor() {}
}
export type actions = CreateAnimal | DeleteAnimal; // 以别名的形式导出
```

##### 3、创建减速器reducer

> animal.reducer.ts

```js
import { createReducer } from '@ngrx/store';
import { CreateAnimal, DeleteAnimal } from 'animal.action.ts';
import { Animal } from 'animal.model.ts'; // {name: string; age: number}
import * as AnimalActions from 'animal.action.ts';

const initData: Animal = {
  name: 'pig',
  age: 12
}
// 方式A
const _animalReducer = createReducer(
  	[initData],
    on(CreateAnimal, (state: Animal[], { animal }) => ([...state, animal])),
    on(DeleteAnimal, (state: Animal[], { animalLen }) => {
      state.splice(animalLen, 1)
      return state;
    }
  )
export function animalReducer(state, action){
  return _animalReducer(state, action);
}

// 方式B
export function animalReducer(state: Animal[] = [initData], action: AnimalActions.actions){
  switch(action.type){
    case AnimalActions.ActionTypes.CREATE_ANIMAL:
      return [...state, action.animal];
    case AnimalActions.ActionTypes.DELETE_ANIMAL:
      state.splice(action.animalLen, 1);
      return state;
    default:
      return state;
  }
}
```

##### 4、创建state

> animal.state.ts

```js
import { Animal } from 'animal.model.ts'; // {name: string; age: number}
export interface AnimalState {
  readonly animal: Animal[];
}
```

##### 5、使用

> app.module.ts

```js
import { NgModule } from '@angular/core';
import { StoreModule } from '@ngrx/store';
import { animalReducer } from 'animal.reducer.ts';

// StoreModule.forRoot({})用户全局注册
@NgModule({
  imports: [
    StoreModule.forRoot({ animal: animalReducer })
  ],
})
export class AppModule {}
```

> create.component.ts

```js
import { Obserbable } from 'rxjs';
import { OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Animal } from 'animal.model.ts'; // {name: string; age: number}
import { AnimalState } from 'animal.state.ts';
import * as AnimalActions from 'animal.action.ts';
export class CreateAnimalComponent implements OnInit {
  animal: Observable<Animal[]>;
  constructor(
  	private store: Store<AnimalState>
  ){
    this.animal = store.select('animal');
  }
	createAnimal(){
    // 以函数形式定义带的动作用法 方式A
    this.store.dispatch(CreateAnimal({
      name: 'cat',
      age: 12
    }))
    // 以class形式定义的动作用法 方式B
    this.store.dispatch(new AnimalActions.CreateAnimal({
      name: 'cat',
      age: 12
    }))
  }
}
```

