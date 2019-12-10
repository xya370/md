### WEBGL 
#### 初始
#### 绘制一个点：
  1. 着色器相关： 
    + 顶点着色器: 设置顶点位置
    + 片源着色器: 设置渲染颜色
  2. 获取上下文
  
    ```
      // 创建一个webgl的上下文环境
      //experimental-webgl || webgl
      var gl = document.getElementById("canvas").getContent("experimental-webgl")
      
      // clearColor :清除绘图区，使用指定颜色填充
      // params: [red, blue, green, alpha] 色值在0-1之间
      gl.clearColor(1.0, 1.0, 1.0, 1.0)
    ```
    
  3. 着色器初始化
  
    ```
      // 着色器设置

      /**
       * 顶点着色器： 描述顶点特性
       * gl_Position: 必填， vec4 ，四维坐标矢量（x, y , z, w）
       * w: 深度，一般设为1.0；
       * gl_PosintSize： Float, 顶点尺寸；
       */
      
      var VSHADER_SOURCE = `
        void mian () {
          gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
          gl_PointSize = 10.0;
        }
      `
      
      /**
       *  片元着色器：
       * gl_FragColor: vec4, (r, g, b, a) 范围值在 0-1之间；
       */
      var FSHADER_SOURCE = `
        void main() {
          gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
        }
      `
      function initSource(gl, Vshader, Fshader) {
        var vertexShader = gl.createShader(gl.VERTEX_SHADER);
        var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
        
        //引入着色器
        gl.shaderSource(vertexShader, Vshader);
        gl.shaderSource(fragmentShader, Fshader);
        
        // 着色器编译
        gl.compileShader(vertexShader);
        gl.compileShader(fragmentShader);
        
        //创建程序对象
        var program = gl.createProgram();
        
        //附着着色器
        gl.attachShader(program, vertexShader);
        gl.attachShader(program, fragmentShader);
        
        // 链接程序
        gl.linkProgram(program);
        gl.useProgram(program);
        return program;
      }
    ```
    
  4. 绘制
   
    ```
      var program = initSource(gl, VSHADER_SOURCE, FSHADER_SOURCE);
      
      //绘制
      /**
       * gl.drawArrays()
       *
       */
      gl.drawArrays(gl.POINTS, 0, 1);
      
    ```
    
#### attribute变量的使用。
  通过attribute变量来动态的设置绘制物的坐标
  1. 代码如下：
   
    ```
      //顶点着色器设置
      var VSHADER_SOURCE = `
        attribute vec4 a_Position;
        void main(){
          gl_Position = a_Position;  //设置a_Position变量接收动态坐标
          gl_PointSize = 3.0;
        }
      `
      
      //片元着色器
      var FSHADER_SOURCE = `
        void main(){
          gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
        }
      `
      var gl = document.getElementById("canvas").getContent("experimental-webgl")
      var program = initSource(gl, VSHADER_SOURCE, FSHADER_SOURCE); // 着色器初始化
      
      // 获取attribute变量存储位置
      var a_Position = gl.getAttribLocation(program, "a_Position");
      // 将顶点位置进行传输给attribute变量
      gl.vertexAttribPointer(a_Position, 0.0,0.0,0.0,1.0);
      
      gl.drawArrays(gl.POINTS, 0,1)
    ```
    
#### uniform变量修改点的颜色
    之前通过attribute变量，对点的位置进行了动态传入，也可以使用uniform变量对图形的颜色进行动态修改
    
  1. 代码如下：
  
    ```
      var VSHADER_SOURCE = `
        attribute vec4 a_Position;
        void main(){
          gl_Position = a_Position;
          gl_PointSize = 3.0;
        }
      `
      //片元着色器, 设置颜色
      var FSHADER_SOURCE = `
        precision mediump float; //指定变量范围和精度
        uniform vec4 u_FragColor;
        void main() {
          gl_FragColor = u_FragColor;
        }
      `
      var gl = document.getElementById("canvas").getContent("experimental-webgl")
      var program = initSource(gl, VSHADER_SOURCE, FSHADER_SOURCE); // 着色器初始化
      
      // 获取attribute变量存储位置
      var a_Position = gl.getAttribLocation(program, "a_Position");
      // 将顶点位置进行传输给attribute变量
      gl.vertexAttribPointer(a_Position, 0.0,0.0,0.0,1.0);
      
      // 获取u_FragColor变量存储位置
      var u_FragColor = gl.getUniformLocation(program, "u_FragColor");
      // 将颜色值传入变量中
      gl.uniform4f(u_FragColor,1.0,0.8,0.6,1.0)
      
      gl.drawArrays(gl.POINTS, 0,1)
    ```
  
#### 缓冲区的使用
  webGl缓冲区：允许一次性向着色器传递多个顶点的值，将数据保存在缓冲区域内，然后传递给着色器
  1. 代码：
 
  ```
    var VSHADER_SOURCE = `
      void mian () {
        gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
        gl_PointSize = 10.0;
      }
    `
    var FSHADER_SOURCE = `
      void main() {
        gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
      }
    `
    var gl = document.getElementById("canvas").getContent("experimental-webgl")
    var program = initSource(gl, VSHADER_SOURCE, FSHADER_SOURCE);
    
    var data = [0.0,0.5,-0.5,-0.5,0.5,-0.5]
    var n = initVertexBuffers(gl, program, data);
    gl.drawArrays(gl.POINTS,0,n)
    
    function initVertexBuffers(gl,program, data,) {
      var vertices = new Float32Array(data);
      
      var n = 3 // 需要绘制的点的个数
      
      // 创建一个缓冲区
      var vertexBuffer = gl.createBuffer();
      
      //缓冲区对象绑定到目标上
      gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
      
      //将数据写入缓冲区
      gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW)
      
      //获取a_Position变量存储位置
      var a_Position = gl.getAttribLocation(program, "a_Position");
      
      //将缓冲区对象赋值给a_Position变量
      gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0,0);
      
      //链接a_Position变量和缓冲区对象
      gl.enableVertexAttribArray(a_Position);
      return n ;
    } 
  ```
#### GL: 平移
  有点类似于css中的translate， 在初始点的位置上再加上需要移动的距离，形成最终平移后的点
  1. 代码如下：
   
      ```
        var VSHADER_SOURCE = `
          attribute vec4 a_Position;
          uniform vec4 u_translate
          void main (){
            gl_Position = a_Position + u_translate;
            gl_PointSize = 3.0;
          }
        `
        var gl = document.getElementById("canvas").getContent("experimental-webgl")
        var program = initSource(gl, VSHADER_SOURCE, FSHADER_SOURCE); // 着色器初始化
        
        var Tx = 0.3, Ty = 0.3, Tz = 0.0 // 定义三个方向的移动距离
        // 获取u_translate的位置；
        var u_translate = gl.getUniformLocation(program, "u_translate");
        // 对变量赋值
        gl.uniform4f(u_translation, Tx, Ty, Tz, 0.0) 
        
        ...
        //绘制
      ```
      
#### GL: 旋转 和 缩放；
##### 缩放：
  之前说了 webGL的顶点着色器，其中gl_Position 是个4维坐标矢量，（x, y, z, w）； 而其中的w就是控制绘制大小的，
  w，深度值， 我个人理解就是以屏幕为人眼，绘制图片距离人眼的距离，具体值的设置遵循近大远小， 可以参考css的3维坐标。