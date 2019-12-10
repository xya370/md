## jQuery(五)： Deferred
#### 有啥用
  通常来说，js请求数据，无论是异步还是同步，都不会立即获取到结果，通常而言，我们一般是是使用回调函数再执行，
  而 deferred就是解决jQuery的回调函数方案，总的来说，deferred对象就是为了将某个回调函数延迟到某个时机再执行。
1. ajax链式写法：
  ```
    //一般写法：
    $.ajax({
      url: '',
      success: function(){},
      error: function(){},
    })
    
    //deferred
    $.ajax(url)
      .done(function(){}) //相当于success
      .fail(function(){})
  ```

2. 指定同一操作的多个函数，允许添加多个函数
  写法也很简单，直接添加在后面就可以了。
  ```
    $.ajax(url)
      .done(function(){})
      .fail(function(){})
      .done(function(){})
  ```
  
3. 为多个函数添加指定回调，
  可以为多个不同的函数添加同一个回调事件
  ```
    $.when($.ajax(url),$.ajax(url2))
      .done()
      .fail()
      
  ```
  
  为两个函数执行操作，如果<font color="#D86658">都成功了</font>就执行done中的回调，如果<font color="#D86658">有一个失败或全部都失败</font>，就执行fail中的回调
4. 普通操作的回调
  deferred允许任何操作都可以使用deferred对象的方法，指定回调函数
  ```
    var wait = function(de){
      var test = function(){
        console.log('开始');
        de.resolve();
      }
      setTimeout(test, 3000);
      return de;
    }
    
    $.when(wait($.Deferred()))
    .done(function(){
      console.log('已完成')
    })
    .fail(function(){
      console.log('失败')
    })
  ```
  
  注意： $.when()的参数只能是deferred对象。

#### 咋处理
1. 关于resolve && rejected
  在上面的时候，会注意到一个resolve，并且会觉得这种链式写法很眼熟，且对promise有一个简单了解的话，大概就知道了。
  ```
    promise: 同样也是用于处理异步函数，将异步操作队列化处理
    
    简单的promise
    new promise (function(resolve,rejected){
        resolve('成功')
    })
    .then(function(){
      
    })
    
    promise.then 接受两个参数： 
    一、 resolve 代表成功时调用的函数
    二、 rejected 代表失败时调用的回调
    promise的三个状态值： pending(初始状态值), fulfilled(操作成功),rejected(操作失败)
  ```
  $.deferred 同样也是有三个不同的状态：未完成，已完成，已失败，当状态处于已完成(resolve)下回自动调用done()中的回调函数，而resolve()就是人为将状态值修改为已完成，同理可证rejected();
  总的来说，核心就是：根据不同的状态值调用回调。
2. API
    * $.when()
    * $.progress()
    * $.promise() 返回promise对象，用于观察某类型的所有行动绑定到集合，排队与否，或是否完成。
    * $.Deferred()
    * $.done()
    * $.fail()

#### 学习下
  来看下jQ的源码是怎么处理的：
  ```
    Deferred: function(func) {
      var tuples = [
        // action, add listener, callbacks,
        // ... .then handlers, argument index, [final state]
        ["notify", "progress", jQuery.Callbacks("memory"),
          jQuery.Callbacks("memory"), 2
        ],
        ["resolve", "done", jQuery.Callbacks("once memory"),
          jQuery.Callbacks("once memory"), 0, "resolved"
        ],
        ["reject", "fail", jQuery.Callbacks("once memory"),
          jQuery.Callbacks("once memory"), 1, "rejected"
        ]
      ],
      state = 'pending',

      // 延迟对象
      deferred = {},
      promise = {
        state: function() {
          return state
        },
        then: function(){},
        promise: function(obj) {
          return obj != null ? jQuery.extend(obj, promise): promise;
        }
      }
      ...
    }
  ```
  
  从代码来看，定一个了数组tuples,以及初始状态值。tuples存储了三个状态下的所需参数，来看下存储了写什么内容：    
  [状态, 对应的处理函数, 利用callbacks创建的回调队列， then方法的回调队列， index, 最终的状态值]，    
  我们可以看到最终的状态值只有reject 和resolve才有。
  ok,已经知道deferred的本质是根据不同的状态调用不同的方法，并且使用callbacks添加函数，那么把tuples遍历一下，生成队列；    
  源码：
  ```
    tuples.forEach(function(tuple){
      var list = tuple[2], // 获取到jQuery.callbacks返回，创建一个队列
        stateString = tuple[5], //获取到最终状态描述
        
        //promise[ progress | done | fail ] = list.add
        promise[tuple[1]] = list.add;
        
       // 如果最终状态值存在，即处于 reject|| resolve 状态下；
      if (stateString) {
        list.add(
          function() {
            state = stateString;
          }
          ....
        )
      }
      // 延迟对象状态 deferred.resolve()
      //deferred[ 'resolve' | reject | notify] = function(){}

      deferred[tuple[0]] = function() {
        deferred[tuple[0]+"Width"](this === deferred ? promise : this, arguments);
        return this;
      }
      
      //jQuery.callbacks.fireWith
      //执行队列，调用处理函数，绑定执行时的上下文
      deferred[tuple[0] + "With"] = list.fireWith;
    })
    promise.promise(deferred);
    return deferred;
  ```
  已经遍历生成了3个队列，并将三个状态方法挂载在了延迟对象上。    
  从代码中可以看出，在调用deferred[ reject | resolve]时，其实是调用了deferred[ rejectWith | resolveWith]方法，本质上是对callbacks.fireWith的调用，以用来执行添加的回调函数，同时设置函数的上下文。
  并且可以看的到，deferred[proress | done | fail] 其实是copy了callbacks.add方法，将回调函数添加在了执行队列中。
  
  