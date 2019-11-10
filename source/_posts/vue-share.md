---
title: vue开发心得分享
tags: vue
categories: vue
date: 2019/10/09
---

## 开发心得分享

#### 思想

######  做到不信任

1. 对产品不信任（流程、细节）
   - 在看原型、流程阶段把自己的疑问或想法都提出来，实际参与编码的人能站在代码层去考虑到一些产品想不到的问题，提前发现问题也是帮自己填坑

   2. 对后端不信任（返回值、内容校验）
       - 对于后端返回来的值一定要时刻记得判断非空、null判断，一般页面报错就是这么来的，后端说绝对不会是不可信的，百密一疏常有
       - 一般来说提示信息由后端定义，或者前后端约定好，后端只返回错误代码，前端定义代码对应报错文案，避免出现调试类型的提示，这里也需要对报错文案进行空值判断

#### 编码封装

1. 前端字典，可以很好的保持统一性，B端项目效果显著。碰上某些场景部分字典不可用的状况下，可以封装一个方法，将不可用的值的key传入，返回可用的值的对象

   ```js
   // 设备类型
   dict_device_type: [
     {
       key: 2,
       value: '标准ICE-003'
     },
     {
       key: 4,
       value: '迷你ICE-004'
     },
   ]
   // 使用起来直接复制粘贴，改个名字就行
   <select v-for="item of dict_device_type">
     <options label="item.value" value="item.key"></options>
   </select>
   // 分页页码配置
   dict_page_size: [10,20,30]
   // 要修改分页配置的时候，可以一个地方修改，全局都改，保持统一
   searchData = {
     pageNum:1,
     pageSize:dict_page_size
   }
   ```
   

   
2. 工具类方法的提取

   ```js
   import Vue from 'vue'
   const that = Vue.prototype
   // 公用方法管理
   const funs = {
   	/**
        * 清空对象的所有属性的值
        */
       emptyObj(obj) {
           if (typeof obj === 'object') {
               for (let key of Object.keys(obj)) {
                   obj[key] = '';
               }
               return obj;
           }
       },
   
       /**
        * 本地存储（持久化）
        */
       setS(k, v, t) {
           if (typeof v === "object") {
               v = JSON.stringify(v);
           }
           if (t == 'session') {
               window.sessionStorage.setItem(k, v);
           } else {
               window.localStorage.setItem(k, v);
           }
       }
   }
   that._fun = fun;
   // 使用的时候直接引入
   import 'util.js'
   this._fun.setS('userInfo',{name:'张三',age:18})
   ```

   

3. ui库的引用，新建一个专门的文件

   ```js
   import Vue from 'vue'
   
   import { Button, Divider, Form, message } from 'ant-design-vue'
   Vue.use(Button);
   Vue.use(Divider);
   Vue.use(Form);
   Vue.use(message);
   
   // 全局弹窗配置
   message.config({
     top: `120px`,
     duration: 2,
     maxCount: 3,
   });
   Vue.prototype.$message = message;
   
   // 入口引入：import './config/antDesign'
   ```

4. 表单校验的提取

   ```js
   // 定义项目中表单的规则
   addOrderRules: {
     name: [
       { required: true, message: '请输入活动名称', trigger: 'blur' },
       { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
     ],
     region: [
       { required: true, message: '请选择活动区域', trigger: 'change' }
     ],
     date1: [
       { type: 'date', required: true, message: '请选择日期', trigger: 'change' }
     ],
     date2: [
       { type: 'date', required: true, message: '请选择时间', trigger: 'change' }
     ],
     type: [
       { type: 'array', required: true, message: '请至少选择一个活动性质', trigger: 'change' }
     ],
     resource: [
       { required: true, message: '请选择活动资源', trigger: 'change' }
     ],
     desc: [
       { required: true, message: '请填写活动形式', trigger: 'blur' }
     ]
   }
   // 使用
   <el-form :model="ruleForm" :rules="addOrderRules" ref="ruleForm"></el-form>
   ```

   

5. 高频率使用的方法挂载到实例上

  ```js
  // 工具类的挂载：Vue.prototype.funs = _funs;
  ```

6. 错误统一处理

   - 在拦截器里面对200的业务错误和200以外的网络错误做统一报错处理

   ```js
   // 响应拦截
   service.interceptors.response.use(function (response) {
       // Do something with response data
       if (response && response.data.return_code == '02') {
           that.$message.error(response.data.return_msg || '请求失败')
       }
       return response;
   }, function (error) {
       // Do something with response error
       const errObj = {
           400:'错误请求',
           401:'未授权，请重新登录',
           403:'拒绝访问',
           404:'请求错误，未找到该资源',
           405:'请求方法未允许',
           408:'请求超时',
           500:'服务器出错',
           501:'网络未实现',
           502:'网络错误',
           503:'服务不可用',
           504:'网络超时',
           505:'http版本不支持该请求'
           
       }
       if (error && error.response) {
         that.$message.error(errObj[error.response.status] || `连接错误					${error.response.status}`);
       } else {
           that.$message.error('连接到服务器失败')
       }
       return Promise.reject(error);
   });
   ```



#### 细节

1. for循环性能

   ```js
   https://github.com/jawil/blog/issues/2
   循环类型	耗费时间(ms)
   for	约11.998
   for cache	约10.866
   for 倒序	约11.230
   forEach	约400.245
   for in	约2930.118
   for of	约320.921
   正常推荐原始的for循环
   for in 一般是用在对象属性名的遍历上的，由于每次迭代操作会同时搜索实例本身的属性以及原型链上的属性，所以效率低下
   ```

2. 原始for循环

   ```js
   // 保存数组的length长度
   let len = arr.length
   不要在for里面直接写
   i < arr.length,这样每次循环都会去获取数组的length
   ```

3. 一次性函数

   ```js
   var sca = function () {
       console.log('msg')
       sca = function () {
           console.log('new-msg')
       }
   }
   sca() // msg
   sca() // new-msg
   sca() // new-msg
   ```

   



