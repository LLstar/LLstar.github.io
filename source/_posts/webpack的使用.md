---
title: webpack的使用
date: 2019-06-21 16:12:40
tags: webpack
---

#### 一、前端环境搭建

使用npm或yarn来安装webpack，在webpack3中，webpack本身和它的cli是在同一个包中，但在webpack4中，已经将两者分来来更好地管理它们。

```shell
$ npm install webpack webpack-cli -g
# 或者
$ yarn global add webpack webpack-cli

$ npm init -y // -y 默认所有的配置
```

<!-- more -->

#### 二、部署webpack

在上面搭建好的环境项目中，我们来到package.json里配置scripts

```javascript
"scripts": {
    //我们在这里配置，就可以使用npm run build 启动我们的webpack
    "build": "webpack --mode production" 
  },
  "devDependencies": {
    "webpack": "^4.16.0",
    "webpack-cli": "^3.0.8"
  }
```

#### 三、npm run build发生了什么

在我们的根目录下新建一个src目录。在src目录下新建一个index.js文件。在里面写入任意代码。在终端运行npm run build命令后，你会发现新增了一个dist目录，里面存放着webpack打包好的main.js文件。类似于vue-cli中的main.js文件。

#### 四、webpack配置

```javascript
const path = require('path');
module.exports = {
  entry: "./app/entry", // string | object | array
  // Webpack打包的入口
  output: {  // 定义webpack如何输出的选项
    path: path.resolve(__dirname, "dist"), // string
    // 所有输出文件的目标路径
    filename: "[chunkhash].js", // string
    // 「入口(entry chunk)」文件命名模版
    publicPath: "/assets/", // string
    // 构建文件的输出目录
    /* 其它高级配置 */
  },
  module: {  // 模块相关配置
    rules: [ // 配置模块loaders，解析规则
      {
        test: /\.jsx?$/,  // RegExp | string
        include: [ // 和test一样，必须匹配选项
          path.resolve(__dirname, "app")
        ],
        exclude: [ // 必不匹配选项（优先级高于test和include）
          path.resolve(__dirname, "app/demo-files")
        ],
        loader: "babel-loader", // 模块上下文解析
        options: { // loader的可选项
          presets: ["es2015"]
        },
      },
  },
  resolve: { //  解析模块的可选项
    modules: [ // 模块的查找目录
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    extensions: [".js", ".json", ".jsx", ".css"], // 用到的文件的扩展
    alias: { // 模块别名列表
      "module": "new-module"
	  },
  },
  devtool: "source-map", // enum
  // 为浏览器开发者工具添加元数据增强调试
  plugins: [ // 附加插件列表
    // ...
  ],
}
```

* Entry：指定webpack开始构建的入口模块，从改模块开始构建并计算出直接或间接依赖的模块或库
* Output：告诉webpack如何命名输出的文件以及输出的目录
* Loaders：由于webpack只能处理JavaScript，所以我们需要对一些非js文件处理成webpack能够处理的模块，比如sass文件
* Plugins：Loaders将各类型的文件处理成webpack能够处理的模块，plugins有着很强的能力。插件的范围包括：从打包优化到压缩，一直都重新定义环境中的变量。但也是最复杂的一个。比如对js文件进行压缩优化的UglifyPlugin插件。
* Chunk：coding split的产物，我们可以对一些代码打包成一个单独的chunk，比如某些公共模块，去重，更好的利用缓存。或者按需加载默写功能模块，优化加载时间。在webpack3及以前我们都利用CommonsChunkPlugin将一些公共代码分割成一个chunk，实现单独加载。在webpack4中CommonsChunkPlugin被废弃，使用SplitChunksPlugin。

#### webpack配置详解：

##### 1、引入我们需要的npm模块

```javascript
 //node 原生path模块
var path = require('path');

 // webpack
var webpack = require('webpack');

 // glob模块，用于读取webpack入口目录文件
var glob = require('glob');

//webpack插件,用于分离项目中的css文件
var ExtractTextPlugin = require('extract-text-webpack-plugin'); 

 //webpack插件，用于生成HTML文件
var HtmlWebpackPlugin = require('html-webpack-plugin');

//webpack插件，用于webpack加载之后打开一个新的浏览器
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

//webpack插件，用于清除目录文件
var CleanPlugin = require('clean-webpack-plugin');

//处理chunk，用于提取第三方库和公共模块
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
```

##### 2、读取入口文件

利用glob读取模块的入口文件。然后将配置封装在modules.exports，定义入口entry字段，entry可以为字符串、对象或者数组，对应单页面和多页面应用。

##### 3、定义资源输出

资源打包输出的配置在output内，主要包括path、filename、chunkFilename以及publicPath。path是资源输出路径，filename是资源命名规则，chunkFilename是公共js打包后输出的命名，publicPath是静态资源的公共路径，比如线上CDN地址等，开发环境可以不设置，这样css中的相对路径就不会包括publicPath。在output输出的时候可以根据开发环境或者生产环境选择不同的文件命名方法，因为一般来说，线上的资源都是要经过压缩的。

```javascript
output: {
        path: path.resolve(__dirname, prod ? "./dist" : "./build"),
        filename: prod ? "js/[name].min.js" : "js/[name].js",
        chunkFilename: 'js/[name].chunk.js',
        publicPath: prod ? "http:cdn.mydomain.com" : ""
    },
```

[name]的值是根据入口entry显示的文件名。比如index.js这个入口文件，对应的output的[name]值就应该是'index'。

##### 4、定义resolve

为了开发方便，我们可以定义自己的别名，以便很快捷地引用不同的模块，别名(alias)的定义是在resolve对象之中。比如：

```javascript
resolve: {
    // 模块别名
    alias:{
        xyz: "/absolute/path/to/file.js" 
    }，
    // 配置项，设置可以忽略的文件后缀
    extensions: ['', '.js', '.less', '.css', '.png', '.jpg']
}
```

当我们在代码中require('xyz')的时候，实际上我们是引入'、absolute/path/to/file.js'这个文件。还可以配置extensions对象，使得开发过程中文件资源的处理可以忽略后缀。

##### 5、Loader

Loader就是资源转换器。由于在webpack里，所有的资源都是模块，不同资源都最终转化为js去处理。针对不同形式的资源采用不同的Loader去编译，这就是Loader的意义。Loader在使用之前必须先通过npm安装，然后在config里面通过module配置才能使用。举个例子：

```javascript
loaders: [{
    test: /\.(png|jpg|jpeg|gif)$/,
    loader: 'url-loader',
    query:{
        limit:'10000',
        name:'images/[name].[ext]'
    }
}]
```

上述配置中，test的作用是正则匹配，匹配到png或jpg或gif结尾的文件就采用url-loader来做对应的编译。由于loader都是默认以-loader后缀结尾的，所以可以直接省略'-loader'，直接写成url。limit表示10000B以下的图片直接压缩成base64编码，超时10000B的图片输出到'images/文件名.拓展名'。

配置中常用的loader：

- 处理样式，转成css，如：less-loader，sass-loader。
- 图片处理，如：url-loader，file-loader。两个都必须用上。否则超过大小限制的图片无法生成到目标文件夹中。
- 处理js，将es6或跟高级的代码转成es5的代码。如：babel-loader，babel-preset-es2015，babel-preset-react
- 将js模块暴露到全局，使用expose-loader

##### 6、Plugin

插件的引入和loader差不多，只是插件是以对象的形式引入。像静态资源路径的替换这种功能就能通过插件来处理。比如共用模块打包到chunk的插件：

```javascript
var chunks = Object.keys(entries);
plugins: [
    new  webpack.optimize.CommonsChunkPlugin({
        name: 'vendors', // 将公共模块提取，生成名为`vendors`的chunk
        chunks: chunks,
        minChunks: chunks.length // 提取所有entry共同依赖的模块
    })
],
```

配置中常用的plugin：

- 代码热替换，HotModuleReplacementPlugin
- 生成html文件，HtmlWebpackPlugin
- 将css生成文件，而非内联，ExtractTextPlugin
- 报错但不退出webpack进程，NoErrorsPlugin
- 代码丑化，UglifyJsPlugin，开发过程不建议打开
- 多个html共用一个js文件(chunk)，可用CommonsChunkPlugin
- 清理文件夹，CleanWebpackPlugin
- 调用模块的别名ProvidePlugin，例如想在js中用'$'，如果通过webpack加载，需要将$与JQuery对应起来

##### 7、定义webpack-dev-server

webpack-dev-server是webpack提供的静态资源服务器，它的存在使得开发可以脱离代理服务器工作。开发调试静态资源不再需要搭建本地服务器。webpack-dev-server有多种配置形式，这里采用的是写死在config的方式，这种方式的特点是方便开发，缺点是不灵活。

```
 devServer = {
        port: 8080,
        contentBase: './build', //定义静态服务器的基路径
        hot: true,
        historyApiFallback: true,
        publicPath: "",
        stats: {
            colors: true
        },
        plugins: [
        new webpack.HotModuleReplacementPlugin()
        ]
    }
```



##### webpack的机制详解，参考文章：<https://juejin.im/post/5aa3d2056fb9a028c36868aa#heading-1> 