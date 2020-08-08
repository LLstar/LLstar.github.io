---
title: vue的mixins和slot
date: 2019-08-24 10:09:19
tags:
- Vue
categories:
- Vue.js
copyright: true
---

### vue的混入mixins

#### 定义

官方定义：混入（mixins）提供了一种非常灵活的方式，来分发Vue组件中可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。

但是根据我自己的理解，我觉得mixins就是将vue的组件中的js部分提取出来，把各个组件中共用的数据（包括：data，methods，各种生命周期钩子函数等，只要你能在组件中操作的js都可以放在mixins中）统一放在一个文件中集中处理。

<!-- more -->

#### 用法

创建mixin.js文件

```js
var mixin = {
  data() {
    return {
      userList: [],
      mixinLatestVersion: '',
    }
  },
  computed: {
    versions() {
      let version = this.mixinLatestVersion
      const versions = []
      do {
        versions.unshift({
          label: `V${version}`,
          value: version,
        })
        version -= 1
      } while (version >= 1)
      return versions
    },
  },
  async created() {
    await this.getUserList()
  },
  methods: {
    // 获取用户列表
    async getUserList() {
      this.userList = await CrashFeedback.GET_USER_LIST()
    },
    async getNewVersion() {
      const id = sessionStorage.getItem('projectId')
      const result = await PROJECT_GET(id)
      sessionStorage.setItem('version', result.version)
      this.mixinLatestVersion = result.version
    },
  },
}

export default mixin
```

##### 局部混入

在需要用到的页面中引入该文件：

```js
import mixin from 'xxx/xxx/mixin'

export default { // 使用
    mixins: [mixin]
}
```

当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。生命周期中是先调用混入的再调用当前组件的，如果存在同名的数据则以当前组件中定义的数据为主。

##### 全局混入

在main.js文件中：

```js
import mixin from 'xxx/xxx/mixin'
Vue.mixin(mixin)
```

注意：全局混入会影响之后创建的每一个vue实例，需要谨慎使用。

### Vue的插槽slot

#### 单个slot

子组件中写入slot标签，父组件在引用时在相应的子组件中写入要显示在插槽中的元素内容，同时还可以在插槽中填写默认内容。代码如下：

parent.vue

```js
<template>
    <div class="parent">
        <child>
            <p>
                这是父组件的slot替代内容！
            </p>
        </child>
    </div>
</template>

<script>
import child from './child.vue'
export default {
    components:{
        child
    }
}
</script>
```

child.vue

```js
<template>
    <div class="child">
        <h2>这是子组件childComponment!</h2>
        <slot>
            <span style="color: red;">父组件没有插入的内容时，显示的默认内容！</span>
        </slot>
    </div>
</template>
```

#### 具名插槽

parent.vue

```js
<template>
    <div class="parent">
        <child>
        	<h1 slot="content">cotnent的内容</h1>
			<h2 slot="title">title的内容</h2>
            <p>无具名插槽的内容</p>
        </child>
    </div>
</template>

<script>
import child from './child.vue'
export default {
    components:{
        child
    }
}
</script>
```

child.vue

```js
<template>
    <div class="child">
        <h2>这是子组件childComponment!</h2>
		<slot name="content">子组件中content的内容</slot>
		<slot name="title">子组件中title的内容</slot>
        <slot>
            <span style="color: red;">父组件没有插入的内容时，显示的默认内容！</span>
        </slot>
    </div>
</template>
```

#### 作用域插槽

我认为作用域插槽即可以在插槽中传入参数

parent.vue

```js
<template>
    <div class="parent">
        <child>
        	<template slot-scope="slotProps">
                <!-- 这里显示子组件传来的数据 -->
                <p>{{slotProps}}</p>
            </template>
        </child>
    </div>
</template>

<script>
import child from './child.vue'
export default {
    components:{
        child
    }
}
</script>
```

child.vue

```js
<template>
    <div class="child">
        <slot msg="子组件信息" slotData="子组件数据"></slot>
    </div>
</template>
```

vue中默认将slot-scope的插槽对象命名为slotProps，也可以自定义名称，slotProps中得到的时候所有slot中的数据。
