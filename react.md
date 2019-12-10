## React
#### 代码规范
  1. standard.js 规范
  2. bem css规范

#### react && vue区别和共性
  共性：都比较支持组件化开发，都使用了虚拟DOM节点
  区别：
  1. vue：是一个前端框架， 主要是MVVM模式，数据驱动页面更改，
    在vue的编写中，vue更加贴近常规的html写法，
    vue的设计是基于数据可变的情况下，双向数据流，响应式开发，运行速度更快，对数据改变更加敏感。
    vue 将html, css, js进行分离，各自具有自己的一套处理方法。
  2. react:是一个前端组件优化库， MVC模式，js开发，触发更新后，先移除原有节点，再创建一个新的节点，不是完全的数据驱动，
    在写法上，react则是支持JSX（javascript的语法扩展）进行书写，html和js进行混写，利用js对节点，样式，通过js来生成html, all in js
    感觉上更加偏向于后台思维
    react则是默认数据不可变，单项数据流，需要手动的对节点进行重新渲染。

#### 虚拟DOM 和 JSX
  虚拟DOM： 原始dom节点渲染，每次改变dom时，都会对全部的dom进行重新绘制，对性能的消耗较大，于是就有了虚拟DOM，
  在每次更新时，虚拟DOM进行对比，然后对有更改的节点进行修改，以达到预期的结果

  JSX: javascript的语法扩展，建议react中使用，并不强求在react中使用，
#### 生命周期
  1. 创建期：
    * initial render
    * constructor
    * componentWillMount
    * render
    * componentDidMount
  2. 存在期
    * 父组件 render
    * componentReceiveProps
    * componentShouldUpdate
    * componentWillUpdate
    * render
    * componentDidUpdate
  3. 销毁期 (清理状态，取消绑定)
    * render
    * componentWillUnmount
  react组件挂载顺序;
  render自上而下，由父组件深入子组件内部，
  componentDidMount自下而上，由子组件延伸至父组件

#### 组件通信
  * 父组件-->子组件
    这是最常见的一种情况，父组件通过props对子组件进行传值，注意子组件中的props是不可更改的;

      ```
          import React, {Component} from "react";
          class Parent extends Component {
            render(){
              return <div><Child name = "lisi"/></div>
            }
          }
          class Child extends Component {
            render() {
              return <div> My name is {this.props.name}</div>
            }
          }
      ```
  * 子组件 --> 父组件
    父组件通过将回调方法或自定义方法以props的形式传递给子组件,子组件则在合适的时候调用方法，将值传递给父组件

    ```
        import React, {Component} from "react";
        class Parent extends Component {
          constructor(props){
            super(props);
            this.state({
              result: "",
            })
          }
          render(){
            return <div>
              <Child fn={(msg)=>{
                var result = msg;
                this.setState(result);
              }}/>
              <span>{this.state.result}</span>
            </div>
          }
        }

        class Child extends Component {
          render(){
            return (
              <div>
                <input type="text" onChange={(e)=>{
                  var value = e.target.value;
                  this.props.fn(value)
                }}>
              </div>
            )
          }
        }

    ```
  * 兄弟组件之间（子组件 --> 父组件 -->子组件）
    兄弟组件之间通信，以共同的父组件为媒介进行传值通信
  * 跨组件通信 （父组件-->子组件的子组件， context）
    跨组件通信，父组件与子组件的子组件通信，在没有很多层次的时候可以选择props传递，
    但在层次过多的时候props就显得很累赘了，在react中可以使用context来进行跨组件的通信

    ```
        import React, {Component} from 'react'
        const MainContext = React.createContext({
          title: "mainContext",
        })

        class Parent extends Component {
          render() {
            return <MainContext.Provider value = {{title: "I'm not title"}}
              <Child />
            </MainContext.Provider>
          }
        }

        class Child extends Component {
          render() {
            return <GrandSon />
          }
        }

        class GrandSon extends Component {
          render() {
            return  <MainContext.Consumer>{
              context=>(
                <h1>{context.title}</h1>
              )
            }</MainContext.Consumer>
          }
        }
    ```

#### context的简单使用。
  之前简单了解使用了context,下面来继续了解一下：

  1. 是什么:
    context提供了一种无需每层手动添加props,就可以在组件树间进行数据传递的方法
  2. 何时使用:
    context的目的是为了共享那些对组件来说是全局使用的数据，如用户，主题等，context的主要应用场景在于
    不同层次的组件需要访问同样的一些数据
  3. api:
     * React.createContext(); 创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，
        这个组件会从组件树中离自身最近的那个匹配的 Provider 中读取到当前的 context 值。
     * Context.Provider
        每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。
        Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。
        当 Provider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 shouldComponentUpdate 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。
     * Context.Consumer
        这里，React 组件也可以订阅到 context 变更。这能让你在函数式组件中完成订阅 context。
        这需要函数作为子元素（function as a child）这种做法。这个函数接收当前的 context 值，返回一个 React 节点。传递给函数的 value 值等同于往上组件树离这个 context 最近的 Provider 提供的 value 值。如果没有对应的 Provider，value 参数等同于传递给 createContext() 的 defaultValue。

      ```
        1. React.createContext()【创建，并设定默认值】
        2. Context.Provider[父组件,对context进行赋值]
        3. Context.Consumer [子组件,获取context的值, 如果之前父组件未赋值，则获取到默认值]
      ```

#### 受控组件与非受控组件
  react中的受控组件与非受控组件都是就要表单来讨论的，一般而言，像input, select或是富文本编辑器，如果我们设置了value,
  在react中是无法被修改的，除非使用setState对值进行重新更新，以达到修改其value的目标，对于这样的一些组件，我们称之为受控组件
  简单概括：

  1. 受控组件： 自身的value值与state中的值进行绑定，无法通过自身的修改更改value,只能通过修改state中的值来达到修改value的目标。

  ```
      import React, {Component} from 'react';
      class InputCom extends Component {
        constructor(props) {
          super(props);
          this.state = {
            value: "1234",
          }
        }
        render() {
          return (
            <div><input type = "text" value = {this.state.value} onChange = {
              (e) => {
                var value = e.target.value;
                this.setState(value)
              }
            } /></div>
          )
        }
      }
  ```

  2. 非受控组件： 了解了受控组件，那么相对的非受控组件就比较好了解了，那就是自身value值未被绑定，可以自己修改自身的value.

  ```
    render() {
      return <div><input type = "text" /></div>
    }
  ```
  ok，了解到了受控组件与非受控组件的基础，那么就来看下他们的应用场景吧。
  受控组件：因为绑定了初始值，比较适合需要展示初始值的情况下，如编辑等情况
  而非受控组件，未设定初始值，比较适合需要用户输入的表单创建等情况。

#### 类的继承与组件组合
##### 组件的组合
  在vue中组件之间的组合已经是司空惯见的事情了，对于不确定组件内部的地方，也可以使用slot插槽的方式。
  而在react中似乎没有很好的方式来进行这种常见组合。但是可以使用props.children来进行传递，组合
  ```
    class Child extends Component {
      render(){
        return <div>{this.props.children}</div>
      }
    }

    class Parent extends Component {
      render() {
        return (
          <div>
            <Child>
              <span> from Parent </span>
            </Child>
          </div>
        )
      }
    }
  ```
  这样的话，我们似乎可以实现slot,我们需要了解的一点是，任何在child标签之间的内容，都会作为props.children传入。
  而当我们的代码中需要多处插槽时，我们也可以将所需要的内容通过props传递使用，而不局限于children,写出属于自己的使用规范。

  ```
    class Panel(){
      render(){
        return (
          <div className = "panel">
            <div className = "panel_left">{this.props.left}</div>
            <div className = "panel_right">{this.props.right}</div>
          </div>
        )
      }
    }
    class ConLeft(){
      render(){
        return (
          <div>Left</div>
        )
      }
    }
    class ConRight(){
      render(){
        return (
          <div>Right</div>
        )
      }
    }
    function App(){
      return (
        <Panel left={<ConLeft />} right = {<ConRight />}></Panel>
      )
    }
    ReactDOM.render(<App/>, document.getElementById('root'))
  ```
  通过这种方法，在react中也可以实现这种组件组合了。

##### 类的继承
  在一些相似的组件中，可能有着类似的方法，但是，众所周知，程序员都很懒，我并不想每个组件里面都重新写一遍方法
  基本方式是将这些方法提取出来，引入需要的文件中，再进行使用，
  然而，子类也可以继承父类的方法，感觉更像是一种对父类的扩展
  ```
    class ComA extends Component {
      sayHello(){
        console.log("hello world")
      }
    }
    class Demo extends ComA {
      componentDidMount(){
        this.sayHello()
      }
      render(){
        return <div></div>
      }
    }
  ```
  虽然在react的文档中说，props和组合可以清晰且安全的定制外观和行为，如果需要在组件中复用
  非ui的功能，建议提取为一个单独的javascript模块，使用import引入直接使用，但是所用的一切工具都需要
  根据使用场景来判断，不需要一昧拒绝某种方式。

#### proxy(拦截，设置转发代理)
  proxy:拦截器

#### 装饰器

#### async, await, promise
  promise 应该是比较熟悉的了，用来解决异步问题的，那么什么是async,await呢？
  async/await 是es7推出的解决异步问题的方法

  1. 简单使用
  ```
    function getData(){
      return "data"
    }
    // 在需要使用异步函数的函数前添加说明符async表示当前函数内部调用了异步函数
    async function test(){
      // 在调用异步函数前添加await说明，执行时就会等待异步函数加载完，再向下执行
      let data = await getData();
      console.log(data)
    }
  ```
  值得一提的是， __async返回的是一个promise对象，如果标记的函数不是promise,则会包装成promise__。
  而await必须写在async函数内部
  2. 执行顺序
  我们都知道js的赋值是从右向左进行赋值，那么await呢，是先执行await后面的函数，再检测到await标志符，还是先遇见await阻塞呢？
  基于这个问题，不如看下下面的代码，看下打印结果
  ```
    function getData(){
      console.log("await")
    }
    async function fn(){
      await getData()
      console.log("end");
    }
    fn();
    console.log("script")
  ```
  从打印结果来看，先执行了getData方法，打印出"await"，然后再打印了"script"。
  基于这一点可以看出，__await是从右向左执行，等待右侧函数的执行完毕__
  3. await
  对于await右侧的函数有两种情况，1. 普通函数 2. promise函数,那么这两种有什么区别呢？
  ```
    // 1. 普通函数
    {
      function getData(){
        console.log("common")
      }
      async function fn(){
        console.log("start")
        await getData()
        console.log("end")
      }
      fn();
      console.log("script")
    }

    // promise
    {
      function getData(){
        return new Promise((resolve)=>{
          console.log("promise");
          resolve("1234")
        }).then((res)=>{
          console.log("promise2");
          return res
        })
      }
      async function fn(){
        console.log("start")
       var ida = await getData();
        console.log("end", ida)
      }
      fn();
      console.log("script")
    }
  ```
  * 普通函数
  await会阻塞后面的执行，先执行async外部的同步代码，等待外部代码执行完毕，
  再回到async内部，将await右侧代码执行完毕后获取返回值作为await表达式结果。
  ```
    1. asyncFunction 调用执行 -->
    2. 遇见await，进行阻塞 -->
    3. 跳转asyncFunction外部，执行async外部同步代码 -->
    4.同步代码执行完毕，跳转async内部 -->
    5. 获取await右侧代码返回值值作为结果，继续执行asyncFunction
  ```
  * Promise函数
  await也会阻塞async内部后面的执行，然后跳转到外部执行async同步代码，然后等待执行完毕，转回内部，继续执行
  由于是个promise函数，await会获取resolve的参数作为表达式的运算结果，如果promise有执行then方法，则会获取then方法的返回值作为结果。
  ```
    1. asyncFunction 调用执行 -->
    2. 遇见await，进行阻塞 -->
    3. 跳转asyncFunction外部，执行async外部同步代码 -->
    4.同步代码执行完毕，跳转async内部 -->
    5. 获取await右侧resolve参数作为返回值，继续执行asyncFunction
  ```
  4. 错误捕捉
  promise如果失败的也会继续执行，并不会抛出错误，因此我们并不能很好的查看错误信息，对于async来说，可以使用try..catch来进行错误捕捉。
  只不过如果将信息打印在控制台，不留神，就会被控制台打印的大堆信息所淹没，这个是需要注意的
  ```
    function getDate(){
      console.log("awaitFunction");
      return new Promise((resolve, reject)=>{
        reject("error")
      })
    }
    async function fn(){
      console.log("start")
      try{
        var iad = await getDate();
        console.log(iad);
      } catch(error){
        console.log(error)
      }

      console.log("end")
    }
    fn();
    console.log("script")
  ```
  5. 其他
  关于async/await的使用感觉，就我个人而言，似乎是对Promise的一种封装提升，可以就async/await的写法改写成Promise
  ```
    //async/await
    {
      function getDate(){
        console.log("awaitFunction");
        return 1234
      }
      async function fn(){
        console.log("start")
        var iad = await getDate();
        console.log(iad);
        console.log("end")
      }
      fn();
      console.log("script")
    }

    //Promise
    {
      function getDate(){
        console.log("awaitFunction");
        return 1234
      }
      function fn(){
        console.log("start")
        return new Promise((resolve)=>{
          resolve(getDate())
        }).then((res)=>{
          console.log(res);
          console.log("end")
        })
      }
      fn();
      console.log("script")
    }
  ```
  ok，这下有了比较直观的感觉了，从上面的代码来看，async的写法，以一种同步的方法来写异步，代码阅读性体验较好，
  比较适合在有多重异步套用的时候，当然Promise在简单使用的时候还是很直观的，这需要我们根据不同的场景进行使用。[](https://segmentfault.com/a/1190000017224799)
  参考文章：[https://segmentfault.com/a/1190000017224799](https://segmentfault.com/a/1190000017224799)

#### 组件部分
  ant-design(参考)

#### react props设置类型检测 && 添加默认值

#### react 高阶组件
##### 属性渲染 (解决有状态组件可复用性)