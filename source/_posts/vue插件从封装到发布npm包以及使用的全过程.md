---
title: vue插件从封装到发布npm包以及使用的全过程
date: 2019-06-22 17:38:19
tags:
- Vue
- npm
categories:
- npm
copyright: true
---

#### 封装vue插件

在开发过程中我们经常会遇到一个组件到处都可以重复使用，如果每次都把代码复制过去就太复杂了，封装为一个插件是最好的选择，哪里需要哪里就引入即可。

利用`vue init webpack-simple 文件名`创建我们的项目目录，在写文件名之前最好先去npm官网查一下有没有相同的名称，因为创建的文件名就是之后我们要发包的文件名（虽然发包时文件名称也可以修改，但还是直接创建一个不重名的项目比较好）。

<!-- more -->

创建好项目之后，项目目录如下所示（这是我修改过的文件目录，创建好的文件目录是一个vue项目）：

![](https://i.loli.net/2019/06/22/5d0de5ffd653d79995.png)

在src下新建一个packages文件夹，来存放我们将要创建的子组件。我创建了一个Test的文件夹，来存放我创建的test组件。我再test目录下放了一个src的文件夹和index.js文件（参考element组件的封装，估计是为了结构更清晰吧。）

src目录下的index.vue文件：

```html
<template>
  <div>
    这是一个npm包测试的demo
  </div>
</template>

<script>
export default {
  name: 'Test',  // 这个组件的名称必须要写，在之后我们会用到
  data() {
    return {
    }
  },
  props: {},
  methods: {},
  created() {},
  components: {},
  watch: {},
  computed: {},
}
</script>

<style scoped lang='scss'>
</style>

```

Test目录下的index.js文件：

```javascript
// Test是对应组件的名字，要记得在index.vue文件中的name属性
import Test from './src/index.vue'

Test.install = Vue => {
  Vue.component(Test.name, Test)
}

export default Test
```

install是vue中挂载组件的方法，有了它我们就可以在外部use一个插件。

在src目录下有一个index.js文件，这就是我们再webpack中配置的入口文件，后面会说到，先看一下该文件的配置：

```javascript
import Test from './packages/Test'
// 如果还有其他组件的话可以继续添加

const components = [Test] // 还有的话可以继续添加，是一个数组

const install = vue => {
  components.map(component => {
    vue.component(component.name, component)
  })
}
// 每个组件都有一个install方法，同上面的install方法相同，这个防范的第一个参数是Vue构造器,第二个参数是一个可选的选项对象。

if(typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}

export default {
  Test
}
```

package.json文件的配置：

```javascript
{
  "name": "module-test-demo",
  "description": "npm测试包",
  "version": "1.0.8",
  "author": "XXX",
  "license": "MIT",
  "private": false, //如果你发布的共有包，这个参数必须为false
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
    "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
  },
  "main": "dist/module-test-demo.min.js" // 发包之后的主文件，跟webpack中的配置相同
}
```

webpack.config.js文件配置：

```javascript
entry: './src/index.js', // 项目的主入口文件
output: {
   path: path.resolve(__dirname, './dist'),
   publicPath: '/dist/',
   filename: 'module-test-demo.min.js', // 打包完之后文件的名称
   library: 'ModuleTestDemo', // 将library暴露为一个名为"ModuleTestDemo"的变量
   libraryTarget: 'umd', // 这是一种可以将你的library能够在所有的模块定义下都可运行的方式
   umdNamedDefine: true
}
```

当你在import引入模块时，这可以讲你的library bundle暴露为名为ModuleTestDemo的全局变量，为了让library和其他环境兼容，还需要在配置文件中添加`libraryTarget`属性，这是可以控制library如何以不同方式暴露的选项。

libraryTarget选项（默认值为：var）：

- 变量：作为一个全局变量，通过`script`标签来访问 { `libraryTarget: 'var'` }
- this：通过`this`对象来访问 { `libraryTarget: 'this'` }
- window：通过`window`对象访问，在浏览器中 { `libraryTarget: 'window'` }
- umd：在AMD或CommonJS的`require`之后可访问 { `libraryTarget: 'umd'` }

#### 发布npm包

- 初始化刚刚的项目

```bash
$ npm init // 发包之前必须要执行这一步，我就是没执行这一步，导致发布的包在引入之后不能用，如果不执行这一步，发包之后在包目录下并没有生成dist文件夹，因此发包之后你的东西并没有引入进入，导致你引入的包找不到
```

- 登录npm

```bash
$ npm login // 输入用户名和密码，以及当时注册的邮箱，邮箱必须要认证，不然发包时会报错
```

- 登录成功后就可以发包了，发包之前必须使用npm官方镜像，不然发包会报错

```bash
$ nrm use npm // 使用官方的镜像
$ npm version patch // 修改版本
$ npm publish // 发布
```

npm version参数说明：

- patch：小变动，比如修复bug等，版本号变动  v1.0.0 -> v1.0.1
- minor：增加新功能，不影响现有版本的功能，版本号变动  v1.0.0 -> v1.1.0
- major：破坏模块对其向后的兼容性，一般这个版本号的变动意味着不向下兼容，版本号变动  v1.0.0 -> v2.0.0

发布时，可能会遇到下面这个问题，这就是你的npm包的名称与现有的包名称冲突，你需要更改package.json中的name，重新发布。

![](https://i.loli.net/2019/06/22/5d0df56722d6030815.png)

运行之后看到如下图，说明已经发包成功，在npm官网搜索就可以看到了。

![](https://i.loli.net/2019/06/22/5d0df5e7883a342589.png)

这就是我从封装插件到npm发包的过程，中间因为没有执行`npm init`命令，导致我发包只有引入一直报错，我找了很久才发现，所以在发包之前一定要执行该命令，不然会被坑，记一次我踩坑的经历。
