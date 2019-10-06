---
title: electron-vue利用webpack打包实现多页面的入口文件
date: 2019-06-21 16:11:00
tags: webpack electron
---

项目需要在electron的项目中新打开一个窗口，利用webpack作为静态资源打包器，发现在webpack中可以设置多页面的入口，今天来讲一下我在electron中利用webpack建立多页面入口的踩坑经验。

#### 1、webpack的核心概念

- Entry：入口，Webpack执行构建的第一步从Entry开始；
- Module：模块，在Webpack里一切皆模块，一个模块对应着一个文件。Webpack会从配置的Entry开始递归找出所有依赖的模块。
- Chunk：代码块，一个Chunk由多个模块组合而成，用于代码合并与分割。
- Loader：模块转换器，用于把模块原内容按照需求转换成新内容。
- Plugin：扩展插件，在Webpack构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
- Output：输出结果，在Webpack经过一系列处理并得出最终想要的代码后输出结果。

<!-- more -->

#### 2、配置多页面的入口文件

electron构建后的项目目录如下：

![](https://i.loli.net/2019/06/20/5d0aeee31efa696976.png)

- ##### 创建新的页面

  vue-cli生成的项目中只有一个main.js主入口的js文件来处理所有的vue页面，我们创建多个页面需要生成一个这个页面相对应的js文件，保存该页面中包含的内容。

- ##### 配置多页面的入口文件

  electron-vue创建的项目中有三个webpack的配置，我主要是在webpack.renderer.config中配置多个入口，生成多页面的入口文件，代码如下：

  ![](https://i.loli.net/2019/06/20/5d0aef394308586575.png)

webpack中的HtmlWebpackPlugin插件是用来简单创建HTML文件，用于服务器访问。必须在新建HtmlWebpackPlugin中写chunks,不然无法识别，页面加载不出来

- electron中新建窗口，访问新生成的页面

  electron中src的main文件中的index.js为主进程，在该页面中新建窗口，调用新生成的HTML文件，代码如下：

  ```
  const dialpadUrl = process.env.NODE_ENV === 'development'
    ? `http://localhost:9080/dialpad.html`
    : `file://${__dirname}/dialpad.html`
  ```

  创建新窗口打开页面的地址。electron的win.loadURL(url[, options])中的加载的文件方式包含：

  * httpReferrer：一个HTTP Referrer url
  * userAgent 发起请求的 userAgent
  * extraHeaders：用”\n“分割的额外标题
  * baseURLForDataURL：要加载的数据文件的根URL(带有路径分隔符)，只有当指定的url是一个数据url并需要加载其他文件时，才需要这样做

  其实我也没太懂这些都是什么，反正据我理解，url加载的只能是远程地址(如：http://)或是本地的HTML文件路径(file://)

  参考文章：<https://segmentfault.com/a/1190000014984842#articleHeader0> 

- ##### 打包报错

  上述就是我在electron-vue中利用webpack实现多页面入口的全过程，不过最后打包时出现了错误，上网搜了一下，说是node内存溢出的问题，在package.json中手动设置node的内存大小就可以啦


```
"scripts": {
    "buildAll": "node --max-old-space-size=4096 .electron-vue/build.js && electron-builder",
    "build": "node --max-old-space-size=4096 .electron-vue/build.js && electron-builder --win --   ia32 --publish always",
    "build:dir": "node --max-old-space-size=4096 .electron-vue/build.js && electron-builder --	win --ia32 --dir",
    "build:clean": "cross-env BUILD_TARGET=clean node .electron-vue/build.js",
    "build:web": "cross-env BUILD_TARGET=web node .electron-vue/build.js",
    "dev": "node --max-old-space-size=4096 .electron-vue/dev-runner.js",
    "lint": "eslint --ext .js,.vue -f ./node_modules/eslint-friendly-formatter src",
    "lint:fix": "eslint --ext .js,.vue -f ./node_modules/eslint-friendly-formatter --fix src",
    "pack": "npm run pack:main && npm run pack:renderer",
    "pack:main": "cross-env NODE_ENV=production webpack --progress --colors --config .electron-   vue/webpack.main.config.js",
    "pack:renderer": "cross-env NODE_ENV=production webpack --progress --colors --config           .electron-vue/webpack.renderer.config.js"
},
```

参考文章：<https://segmentfault.com/a/1190000010437948> 

#### 3、多个窗口之间可以利用vuex-electron插件实现数据共享

​	多个窗口之间本来使用ipc通信来传递数据，发现这样的做法太麻烦啦，就发现了vuex-electron插件可以实现多个窗口之间共享同一个vuex数据。我来说一下它的使用方法：

在src/renderer/store/index.js中引入createSharedMutations和createPersistedState，如果有时候状态没有发生变化，可以将createSharedMutations注释掉(为什么注释掉，暂时还未找到原因)

```
import Vue from 'vue'
import Vuex from 'vuex'

import { createPersistedState, createSharedMutations } from 'vuex-electron'

import modules from './modules'

Vue.use(Vuex)

export default new Vuex.Store({
  modules,
  plugins: [
    createPersistedState(),  // 创建数据持久化，不需要时可以删除
    createSharedMutations()  // 创建共享的mutations
  ]
})
```

在多个窗口之间就可以用共用同一个vuex中的数据，这样就省去了繁琐的ipc通信，获取数据更方便。

这就是我的分享，有什么不对的地方欢迎指正。