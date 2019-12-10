#### 函数化编程
  百科定义：
  函数化编程： 又称泛函数编程，是一种编程泛式，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。
  简单理解，以函数为单元，对复杂逻辑进行拆分，将复杂逻辑转化为多个简单函数逻辑，同时通过对函数进行层层调用，来达到最终目的。
  特点： 函数可以作为参数传入，也可以作为返回值返回。
##### 1. 纯函数 && 非纯函数
  定义： 输入一个x产生输出一个唯一y值
  特点： 
      1. 输入相同的值时，输出也是一样的，不受外部环境影响
      2. 运行时，无副作用，不对外部环境产生影响
```
  例子： 
      Array.prototype.slice;
      function add(a) {
        return a+1
      }
      add(1)
      add(1) 
```
  那么非纯函数则是正好相反：易受外部环境影响，提高了系统复杂性。
```
    var arr = [1, 3, 5, 6];
    arr.splice(0,1) ==> [1]
    arr.splice(0,1) ==> [3]
    var b = 34;
    function add(a) {
      return a + b;
    }
```
  纯函数是函数编程的基础，那么如何将非纯函数转化为纯函数呢？
  
##### 2. 函数柯里化（curry）
  简单的定义就是：将一个低阶函数转化为高阶函数的过程被称之为柯里化。
  这样说未免有点不太清楚，就是将一个需要传入多个参数的函数转化为多个只需要传入一个参数的函数。
  
```
  如： 
      function (arg1, arg2, arg3) ==> function(arg1)(arg2)(arg3)
      
      function add(a, b){
        return a+b
      }
      add(1,2)
      ==> 
      function add(a) {
        return function(b){
          return a+b;
        }
      }
      add(1)(2)
```
  
  这样一看是不是就清楚的多了，其中有提到一个概念<font  color="#D86658">**高阶函数**</font>，那么什么是高阶函数呢？
  <font color="#D86658">**高阶函数： "Higher-order function",js中的函数都是指向某个变量的，那么同样可以指向某个函数，同理，也可以将某个函数作为返回值，返回。**</font>
  所谓的高阶函数，就是可以接受一个函数为参数或返回一个函数的函数。
  
```
  例子： 
      var arr = [1,2,34]
      arr.map(function(item, index){
        return item*2
      })
      
      function add(a, b, fn){
        return fn(a) + fn(b)
      }
```
  
##### 3. 声明式语句 && 命令式语句
  顾名思义，命令式语句是，一步一步的指令，告诉你要怎么做，而声明式语句，则是只需要说要什么，具体怎么做，你自己看着办
```
  命令式：
    var arr = [1, 2, 4, 5],result = [];
    for(var i = 0; i< arr.length; i++) {
      result.push(arr[i])
    }
  
  声明式：
    var result = arr.map(function(item){return item})
  从这两个例子，就可以清楚的看到命令式语句和声明式语句的区别了
```
  
  通过上面的内容，已经对js函数化编程有了一个较为基础的浅显的认识，至于更深层次的认识，就需要看看其他大佬的文章了。