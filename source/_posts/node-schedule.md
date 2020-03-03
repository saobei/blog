---
title: node-schedule定时任务
tags: 
	- Nest.js
	- 入门实战-定时任务
categories: nestjs
date: 2019/12/11
---

### 内容概要
- 在实际开发项目中，会遇到很多定时任务的工作。比如：定时发送推送通知，定时更新某些数据等等
这是我们第一时间想到的就是可以写个定时器，来完成相应的需求，而在在node.js中可以使用node-schedule来完成定时任务
nestjs是一个node框架当然也可以使用node-schedule来完成定时任务。下面就用示例来说明一下node-schedule的用法.

安装依赖
    npm install node-schedule

### 开启一个定时任务
```js
const schedule = require('node-schedule');

function scheduleTask(){
    schedule.scheduleJob('30 * * * * *', ()=>{
        console.log('scheduleTask:' + new Date());
    }); 
}

scheduleTask();
// schedule.scheduleJob的回调函数中写入要执行的任务代码，一个定时器就完成了！传入的'30 * * * * *'带来的结果是每分钟的30秒时都会执行
```

### 定时任务中的通配符解释
    * * * * * *
    ┬ ┬ ┬ ┬ ┬ ┬
    │ │ │ │ │  |
    │ │ │ │ │ └ day of week (0 - 7) (0 or 7 is Sun)
    │ │ │ │ └───── month (1 - 12)
    │ │ │ └────────── day of month (1 - 31)
    │ │ └─────────────── hour (0 - 23)
    │ └──────────────────── minute (0 - 59)
    └───────────────────────── second (0 - 59, OPTIONAL)

6个占位符从左到右分别代表：秒、分、时、日、月、周几

每分钟的第30秒触发： '30 * * * * *'

每小时的1分30秒触发 ：'30 1 * * * *'

每天的凌晨1点1分30秒触发 ：'30 1 1 * * *'

每月的1日1点1分30秒触发 ：'30 1 1 1 * *'

每年的1月1日1点1分30秒触发 ：'30 1 1 1 1 *'

每周1的1点1分30秒触发 ：'30 1 1 * * 1'

### 范围触发

```js
const schedule = require('node-schedule');

function scheduleTask(){
    schedule.scheduleJob('1-10 * * * * *', function(){
        console.log('scheduleTask:' + new Date());
    }); 
}

scheduleTask();
// 每分钟的1-10秒都会触发  其它占用符使用方法一样
```

### 递归规则定时器

```js
const schedule = require('node-schedule');

function scheduleTask(){

    let rule = new schedule.RecurrenceRule();
    // rule.dayOfWeek = 2;
    // rule.year = 2020; 
    // rule.month = 3;
    // rule.dayOfMonth = 1;
    // rule.hour = 1;
    // rule.minute = 30;
    rule.second = 0;
    
    schedule.scheduleJob(rule, function(){
       console.log('scheduleTask:' + new Date());
    });
   
}

scheduleTask();
// 每分钟第60(0)秒时就会触发
```

### 对象文本语法定时器
```js
const schedule = require('node-schedule');

function scheduleTask(){
    //dayOfWeek
    //month
    //dayOfMonth
    //hour
    //minute
    //second
    schedule.scheduleJob({hour: 17, minute: 0, dayOfWeek: 5}, function(){
        console.log('scheduleTask:' + new Date());
    });
   
}

scheduleTask();
// 每周五的下午17：00分触发
```
### 取消定时器
调用定时器对象的cancl方法即可
```js
const schedule = require('node-schedule');

function scheduleCancel(){

    let counter = 1;
    const j = schedule.scheduleJob('* * * * * *', function(){
        
        console.log('定时器触发次数：' + counter);
        counter++;
        
    });

    setTimeout(function() {
        console.log('定时器取消')
        j.cancel();   
    }, 5000);
}

scheduleCancel();
// 5000ms后取消则触发五次后将会取消
```

### nestjs中使用定时器
这里有这样一个需求，每周五的下午6点定时更新数据库中某些字段；
这里我们就可以实现一个接口用来更新数据库表字段，而关键就在于每周五的下午6点去更新，这时我们就可以启一个定时任务，定时调用接口达到定时更新的目的。
例如：
有接口 http://localhost:3000/photo/updateStatus 定时将数据库中数据删除(不建议删库，一般设置字段表示数据状态，更新该字段为指定值表示删除状态)

在main.ts引入并使用
```ts
import { Logger } from '@nestjs/common';
import { NestFactory, FastifyAdapter } from '@nestjs/core';
import { AppModule } from './app.module';
import * as cors from 'cors';
import { join } from 'path';
import fly from 'flyio';

// tslint:disable-next-line:no-var-requires
const schedule = require('node-schedule');

function scheduleTask() { // 定时任务
    const timing = schedule.scheduleJob('0 0 0 * * *', () => {
        Logger.log('定时任务');
        fly.get('http://localhost:3000/photo/updateStatus').then(res => {
            Logger.log(JSON.stringify(res.data));
        }).catch(err => {
            Logger.log(JSON.stringify(err));
        }); // 定时任务更改数据
    });
}
scheduleTask(); // 开启定时任务

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.useStaticAssets(join(__dirname, '..', 'public'));
    app.use(
        cors({
            credentials: true,
        }),
    );
    await app.listen(3000);
}
bootstrap();
```

### nest-schedule
[nest-schedule](http://npm.taobao.org/package/nest-schedule)是一个基于node-schedule实现的用于nest.js的任务库
...还未尝试使用

