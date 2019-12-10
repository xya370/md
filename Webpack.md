### webPack
#### 前置
  node环境安装，配置
  注意： 
      1. 不死记 
      2. 不试图学习所有功能
##### 是什么 && 安装
  1. 为社么需要：
      * 前端工程化： 系统化，模块化，规范化的过程，提高系统生产效率
  2. 主要功能：
      * 编译，js && css
      * 文件的压缩，打包，合并，公共模块提取。。。
      * 图片等资源的处理，如压缩，合并雪碧图等。。
      * Tree-shaking等js优化工具
      * webpack-dev-server,..等帮助开发调试的工具
  3. 安装：
      npm install webpack webpack-cli -g  全局安装
##### 打包：
    新建文件夹-> npm init 初始化package.json 文件
    ---> 创建编写一个配置文件 --> 命令行打包
#### 前端模块化
##### 规范：
  1. COMMONJS
  2. AMD/CMD/UMD 需外部引用文件
  3. ES6 Module 兼容问题
  webpack支持： COMMONJ(配置), (业务逻辑)AMD, ES6Moudle
#### webpack核心概念
  1. 配置文件： 打包的依据，webpack的使用一般编写，修改配置文件
  2. 核心概念：
    * Entry && output
      entry: 打包入口，代码从这里开始编译
      module.exports = {
        // 单入口形式
        entry: "./app.js",
        entry： ["./app.js"."./app2.js"] //多文件打包为一个入口
        // 打包为多入口形式
        entry: {
          app: "./app.js",
          app2: "./app2.js"
        }
        // webpack打包出口，打包结果根据output的定义输出，影响资源的路径
        output: {
          // name -- 对应入口的名字， hash: 名字对应生成的哈希字符
          filename: "[name].[hash].js" //指定打包文件的名字（必填）
          // 可对打包结果文件路径进行指定
          filename: "./js/[name].[hash].js" //指定打包文件的名字（必填）
          //__dirname: 当前项目的绝对路径;
          // path：只接受绝对路径          
          path: __dirname+'src/bound/'
        }
      }
    * loader
      webpack的编译方法，本质是一个方法，需要额外安装
      module:{
        rules: [
          {
            test: /\.js$/ // 正则，定义处理什么类型的文件
            use:[{}] // 用什么进行处理
          }
        ]
      }
      常用：
        + css-loader style-loader
        + url-loader image-loader 等资源处理
        + less-loader sass-loader babel-loader
    * plugin： 插件-webpack的额外扩展，帮助webpack优化代码，提供功能
      plugin: [
        new webpack.HotModuleReplacementPlugin()
      ]
      常用：
        + commonsChunkPlugin, uglifyjsWebpackPlugin,PurifyCSS等优化文件体积
        + HtmlWebpackPlugin, HotModuleReplacemenPlugin等额外插件
#### webpack 实战
##### 打包：
  1. 不通过配置文件：
    直接指定出入口
    webpack-cli --entry <entery> --output <output>
  2. 通过配置文件
    * webpack : 在目录下寻找webpack.config.js作为配置文件进行打包
    * webpack --config configfile ： 指定一个文件作为配置文件进行打包
##### 全局webpack && 局部webpack
  1. 全局：
    通过npm intall webpack -g 安装的webpack
    作用：
      必须安装，
  2. 局部：
    当前项目下安装的webpack，不同的项目webpack可能会有不同
    局部安装：
      npm install webpack@v --save
    局部打包：
      package.json配置
      script: {
        "build":
      }
      运行package.json中的命令 npm run build