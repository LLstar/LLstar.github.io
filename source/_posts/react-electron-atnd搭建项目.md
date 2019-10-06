---
title: react+electron+atnd搭建项目
date: 2019-08-06 16:19:05
tags: react electron
---

#### 按需引入antd组件

1、引入

```javascript
yarn add antd react-app-rewired babel-plugin-import 
// antd 样式组件
// react-app-rewired 自定义配置react项目的工具
// babel-plugin-import 一个按需加载代码、样式的babel插件

yarn customize-cra less-loader less
// 由于新的react-app-rewired@2.x版本的关系，需要安装customize-cra来配置override文件
```

2、修改根目录下的package.json文件：

```javascript
# /package.json

"scripts": {
-    "start": "react-scripts start",
-    "build": "react-scripts build",
-    "test": "react-scripts test --env=jsdom",
-	 "eject": "react-scripts eject",
+    "start": "react-app-rewired start",
+    "build": "react-app-rewired build",
+    "test": "react-app-rewired test --env=jsdom",
+	 "eject": "react-app-rewired eject"
}
```

<!-- more -->

3、在根目录下创建一个config-overrides.js文件，用于书写自定义配置：

```javascript
# /config-overrides.js

const { override, fixBabelImports, addLessLoader} = require("customize-cra")

module.exports = override(
  fixBabelImports("import", {
    libraryName: "antd", libraryDirectory: "es", style: true
  }),
  addLessLoader({
    javascriptEnabled: true,
    modifyVars: { "@primary-color": "#1DA57A" }
  })
)
```

#### 将启动react的命令和启动electron的命令合并为一个命令

1、引入

```javascript
yarn add concurrently cross-env wait-on --dev
// concurrently同时运行多个命令
// 运行react-app-rewired start时，不打开浏览器，需要传入BROWSER=none,所以需要安装cross-env做传值兼容
// 同时运行时，electron打开时，服务器还没运行好，所以需要使用wait-on
```

2、在package.json中添加命令

```javascript
{
    "electron-start": "cross-env NODE_ENV=development electron .",
    "electron-dev": "concurrently \"cross-env BROWSER=none yarn start\" \"wait-on 				http://localhost:3000 && electron . \""
}
```

3、运行命令 yarn electron-dev就可以看到项目启动起来了。

#### 利用electron-builder打包项目

1、引入electron-builder，electron-builder只能在开发环境中使用，所以必须下载在devdependencies下

```javascript
yarn add electron-builder -D
```

2、配置package.json文件

```javascript
{
  "version": "0.1.0", // 版本号
  "author": "XXX", // 作者
  "description": "XXX", // 描述
  "main": "./public/electron.js", // 主入口文件必须放入到public文件下(在react项目中，打包时会有报错提示，可以根据提示将electron.js文件放入该放入的文件夹中)
  "build": { // electron-builder配置
    "productName": "XXX", // 项目名称，即打包后的*.exe文件名
    "copyright": "XXX", // 版权
    "appId": "XXX", // 应用id
    "compression": "maximum", // 打包压缩情况 <store|normal|maximum> store相对较快
    "asar": false, // asar打包
    "directories": {
      "output": "build" // 输出文件夹
    },
    "files": [
      "dist/**/*",
      "package.json"
    ],
    "win": {
      "target": "nsis", // 打包为nsis安装文件
      "icon": "./public/favicon.ico" // 应用的图标，改图标必须使用256*256像素的才可以
    },
    "nsis": { // nsis配置
      "allowToChangeInstallationDirectory": true, // 允许用户选择安装位置
      "oneClick": false, // 是否一键安装
      "allowElevation": false // 允许请求提升。如果为false，则用户必须使用提升的权限重新启动安装程序
    }
  },
  "scripts": {
    "build": "react-app-rewired build",
    "pack": "electron-builder --dir", // 打包为目录文件
    "dist": "electron-builder --win --ia32 --publish always" // 打包为.exe文件
}
```

3、运行打包命令

```javascript
yarn build // 打包react项目
yarn dist // 打包electron
```

#### react-router的使用

```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { HashRouter as Router, Route, Link, Redirect, Switch } from 'react-router-dom';
import Home from './Home';
import Profile from './Profile';
import User from './User';

export default class App extends Component {
  render() {
    return (
      <Router>
        <div>
          <div>
            <Link to="/home">首页</Link>
            <Link to="/profile">个人中心</Link>
            <Link to="/user">用户</Link>
          </div>
          <div>
            <Switch>
              <Route path="/home" exact component={Home}>
                  <Route path="/profile" component={Profile}></Route>
                  <Route path="/user" component={User}></Route>
              </Route>
            </Switch>
          </div>
        </div>
      </Router>
    )
  }
}
render(<App></App>, document.querySelector('#root'));
```

react-router4之前的写法， 路由嵌套时只要这样写就可以，但是如果是react-router4之后写法就跟这种写法不一样啦，需要在其父页面中引入路由，在其父页面中添加使用。

```javascript
import React from 'react'
import { Route } from 'react-router-dom'
import Profile from './Profile';
import User from './User';

class Home extends React.Component {
  render() {
    return (
      <div className="home">
        <Route path='/profile' component = { Profile } />
        <Route path='/user' component = { User } />
      </div>
    )
  }
}
```

#### react中使用mobx装饰器报错

在react项目中使用mobx，例如：

```js
class MyState {
  @observable num = 0;
  @action addNum = () => {
    this.num++;
  };
}
```

启动后会收到这样的错误：

```js
SyntaxError: D:\lianxi\redux-mobx-test\src\App.js: Support for the experimental syntax 'decorators-legacy' isn't currently enabled (7:3):

   5 |
   6 | class MyState {
>  7 |   @observable num = 0;
     |   ^
   8 |   @action addNum = () => {
   9 |     this.num++;
  10 |   };
```

需要在package.json中配置babel

```bash
$ yarn add @babel/plugin-proposal-decorators -D
```

```js
"babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "@babel/plugin-proposal-decorators",
        {
          "legacy": true
        }
      ]
    ]
  },
```

之后需要执行：

```bash
$ yarn eject 
```

执行完之后可能会报错，找不到某些模块，按需添加这些模块就可以，执行完命令之后，babel的这个配置会被覆盖，重新再讲上述代码替换到，重新运行就可以了。