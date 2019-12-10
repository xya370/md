##### underscore_1: map();
  map()是underscore.js中一个处理数组和对象的方法。
  params: 1. array || obj 
          2. callback 
          3. content 上下文指向
###### 使用： 
  ```
    var obj = {a:1, b: 2, c: 3}
    var arr = [1.3,5,6,76]
    _.map(arr, function(value, index, obj){
      return value*3
    });
  ```
  
  这个代码如果我自己写的话当然很简单的想法了，就是判断传入参数类型，进行遍历，然后判断是否有回调，有的话，就执行，并将结果返回出去，没有的话，就将原值返回出去。
  下面看下源码是怎样的吧。
###### 调用过程：
   1. map():
   ```
    _.map = function (obj, iteratee, context) {
      iteratee = cb(iteratee, context);
      var keys = !isArray(obj) && Object.keys(obj);
      var length = (keys || obj).length;
      var result = Array(length);
      for(var index = 0; index < length; index++) {
        var current = keys ? keys[index] : index;
        result[index] = iteartee(obj[current], index, obj);
      }
    }
   
    map方法在一开始调用了一个cb方法，对iteratee进行重写，下面看下cb
   ```
   
   2. cb()
   ```
    var defaultCb = function(value) {
      return value;
    }
    /**
     * argcount 在生成迭代的时候，对迭代器有几个参数的要求，默认3
     * */
    var cb = function(value, context, argcount) {
      if (value == null ) return defaultCb;
      if (isFunc(value)) return optimizeCb(value, context, argCount);
      ...
    }
    
    这里其实没什么好说的，如果没有回调，就给他一个默认方法。重点应该是在optimizeCb方法上。
   ```
   
   3. optimizeCb()
   ```
    var optimizeCb = function(func, context, argCount) {
      if (context == void 0 ) return func;
      switch (argCount == null ? 3 : argCount) {
        case 1: return function(value) {
          return func.call(context, value)
        }
        ...
        case 3: return function(value, index, object) {
          return func.call(context, value, index, object);
        }
      }
    }
    
    在optimizeCb中，会注意到underscore使用了void 0 来代替undefined进行判断。
    void是js的一个函数，但是，似乎很少用到，由于void的返回值是undefined，于是，被代替undefined以用来判断，毕竟，相比undefined，可以改写这一点，void更可靠一点
    可以看到，在optimizeCb中使用了call函数，对func重新绑定了上下文，并返回。
   ```
  
  其实在map方法中，主要的是iteratee设计(迭代设计)，迭代可以基本大家都知道，但是，却不清楚具体。
  迭代： 简单的说：就是通过某一种方法对数组，对象以及类数组中的每个元素进行处理。
  一个迭代器，至少有两个基础部分：
  1. 被迭代集合
  2. 当前迭代过程
  而map中的迭代过程则是一个函数，就是传入的处理函数。至于cb函数，则是根据传参的不同情况创建迭代过程，为每次的迭代服务。
  optimizeCb函数是对迭代函数的一个优化
  
#### 洗牌算法：
    简单思路： 将集合视为牌堆，不停的从牌堆中抽牌构成新的牌堆，直至新牌堆的排数到达预设数量。
              在一组数据中，随机抽取n个数据作为新的数据集返回，类似于随机抽样。
    简单实现：
  ```javascript
    //获取范围内的随机数：
    function random(min, max) {
      if (max == null) {
        max = min;
        min = 0;
      }
      return Math.floor(min + Math.random()*( max - min));
    }
    
    function shuffle(arr, n) {
      var clone = arr.slice();
      var length = clone.length;
      n = Math.max(Math.min(n,length),0);
      for (var i = 0; i < n; i++ ){
        var shuffleIndex = random(i, n);
        //数据置换
        var target = clone[i],
        clone[i] = clone[shuffleIndex],
        clone[shuffleIndex] = target;
      }
      return clone;
    }
  ```
#### 函数组合
  所谓的函数组合，就是一个函数，参数是多个函数，执行时，对参数中的函数依次执行，当前执行的函数返回值是下个执行函数的参数。并返回最后一个执行函数的返回值。
  ```
    例子： 
      function fn1 (data){};
      function fn2 (data){};
      function fn3 (data){};
      function compose(){};
      compose(fn1, fn2, fn3)(4);
      
      执行顺序： fn3 -> fn2 -> fn1;
  ```
  简单来说， 函数组合有点类似于管道，将各个作为参数的函数连接起来。
  
  实现：
  ```
    function compose(){
      var fns = arguments;
      return function(){
        var args = [].slice.call(arguments);
        var start = fns.length - 1;
        var result = fns[start].apply(this, args);
        while(start--) {
          result = fns[start].call(this, result)
        }
        return result;
      }
    }
  ```