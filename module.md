## 模块加载器
### 模块化
#### 模块化系统
  最初使用js的时候是直接使用script标签引入，这样无可避免的问题就是，全局变量暴露，代码之间逻辑混乱，可维护性差等一系列问题，
  为了解决问题，提出了多种方案，其中一种就是模块化编程，虽然现在大家基本都在使用模块化的方式，但是，还是需要具体了解一下的；
  
  模块化编程： 简单来说就是将一个整体系统，分割为多个方法模块组合而成。
  具有以下的几个优点：
  * 解决了全局变量的污染问题和命名冲突问题
  * 提高了代码的可维护性，
  * 提高了代码的复用性
  * 多模块组合的方式，也更加利于多人员的开发方式，提升了开发效率
  * 解决了文件的依赖问题，开发人员不需要再关注文件的引用顺序。

#### 规范
##### 同步 CommonJS
  CommonJS是模块化编程的先行者，CommonJS加载模块是同步加载的，主要应用在NodeJs服务器编程上
  CommonJs要求编程者遵循一样的开发规范：
  1. 模块标识(module)应遵循一样的书写规范.
  2. 定义全局函数require, 通过模块标识来引入其他模块.
  3. 如果require引入失败，require需要报一个异常。
  4. 模块标识通过exports暴露对外的api, exports只能是一个对象，暴露的api作为exports属性存在.<br>
  虽然CommonJs很棒，但是由于是同步加载，导致在浏览器端使用注定是有问题，但是参考CommonJs，于是就有了下面的异步加载主角了

##### AMD规范 (requirejs)
  AMD： 异步模块加载定义，采用异步加载的方式，在回调中执行逻辑
  特点： 
  1. 依赖前置
  2. 延迟加载
  AMD规范一般是通过require.js来进行，可以通过require.js来看AMD的具体规范；
  1. 使用全局函数define来定义模块。
 
    ```
      // 独立模块
      define({
        module: function(){}
      })
      // 非独立模块
      define([module2, module1], function(m2, m1){})
    ```
    
  2. 使用require来加载模块
  
    ```
      /**
       * [params] modules type Array 需要加载的模块
       */
      require([modules], callback)
    ```
    虽然AMD已经实现了浏览器端模块加载的需求，但是仍然还有一些不足之处，
    虽然AMD中模块是异步加载的，但是依然需要等到模块全部提前加载完毕，才会调用方法，这样在初次调用时，就会加载较慢，提高开发成本，

##### CMD (seajs)
  CMD: 通用模块定义，和AMD加载方式不同，CMD是按需就近加载，而不是提前全部加载。
  同AMD一样，使用第三方seaJs来实现，seaJs 推崇一个模块就是一个文件。
  规范：
  1. 使用全局函数defined定义模块
  
  
    ```
      /**
       * [params] require 模块导入
       * [params] exports 模块导出       
       * [params] module type Object
       */
      define(function(require, exports, module){
        //js
      })
    ```
  2. 模块加载，使用seajs.use方法进行加载

##### es6模块化
  在es6之前,js模块化是需要引入第三方文件实现的，直至es6的模块化。

  特点：
  1. es6中每一个模块都是一个文件，
  2. es6是在编译时对模块进行加载的，而之前的CommonJS,AMD,CMD都是在运行时对模块进行加载。
  3. es6的模块化是自动使用严格模式的，无论是否添加严格模式的声明，都会使用严格模式

  使用：
  + export (导出模块)
  + import (引入模块)
    
     ```
        export myAge = 12; // age.js
        import {myAge} from "./age.js"
     ```
#### 书写约定 - 自定义加载器
##### 定义
  1. 所有的js文件都使用模块的方式书写，一个文件只包含一个模块
  2. 全局函数define定义模块
  3. factory 调用时，始终传入require, exports, module,这三个参数在所有模块代码中都可用
  模块：
  1. 模块代码中，第一个参数为require
  2. require不能重写
  3. require 参数必须是字符串，不能是表达式


### 加载器结构
  1. 模块部分： [每个模块都是主构造函数的实例对象]
    + 数据初始化，
      * url 
      * deps : 依赖存储
    + 模块存储： cache缓存对象，模块存储在cache对象中.
      * 存储方式： {"模块路径"： new Module()}
      * 通过资源定位方法获取模块绝对路径
  2. 资源部分
    + 资源定位， 通过resolve()方法将模块名解析为路径。（判断文件是否有后缀/模块短名称配置 + 当前文件相对路径）
    + 动态加载， 动态创建script,加载同时，模块加载器解析当前模块所依赖的模块以数组形式存储。
    + 依赖管理： 解析依赖，-> 获取绝对路径 -> 动态加载，递归加载模块。
      
### module构造器设计
  1. status[1-6] //获取，缓存，加载，准备执行，正在执行，执行完毕
  2. 动态加载：
    * 提取模块名称 - 路径检测【名称、短名称，后缀检测】 - 生成路径 - 动态加载