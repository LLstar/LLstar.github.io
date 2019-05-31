### 配置文件

#### npm包安装

```javascript
$ npm i -D eslint prettier eslint-plugin-prettier eslint-config-prettier husky lint-staged
$ yarn add eslint prettier eslint-plugin-prettier eslint-config-prettier husky lint-staged -D
```

##### 1、在前端应用中的package.json中新增如下文件：

- 在scripts中直接写“precommit”会报错，解决方案是：


```javascript
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.js": [
      "eslint --fix --ext .js",
      "prettier --write",
      "git add"
    ]
  },
  "devDependencies": {
    "eslint": "^5.0.0",
    "eslint-config-ali": "^2.0.1",
    "eslint-plugin-import": "^2.6.0",
    "eslint-plugin-react": "^7.1.0",
    "husky": "^0.14.2",
    "babel-eslint": "^8.1.1",
    "lint-staged": "^4.0.0",
    "prettier":"^1.16.4",
    "eslint-plugin-prettier":"^3.0.1",
    "eslint-config-prettier":"^4.0.0"
  },
}

备注：
{
    "*.js": "工程下所有的 js 文件",
    "**/*.js": "工程下所有的 js 文件",
    "src/*.js": "src 目录中所有的 js 文件",
    "src/**/*.js": "src 文件夹下所有的 js 文件",
    "src/**/*.{js,vue}": "src 文件夹下的所有的 js 和 vue文件"
}
```

##### 2、增加 .eslintrc.js 扫描规则：

```javascript
module.exports = {
  "extends": ["eslint-config-ali","prettier", "plugin:prettier/recommended"],
  "parser": "babel-eslint",
  "rules": {
    "prettier/prettier": "error",
    "strict": "off",
    "no-console": "off",
    "import/no-dynamic-require": "off",
    "global-require": "off",
    "require-yield": "off",
  },
  "plugins": ["prettier"],
  "globals": {
    "React": "readable"
  }
}

module.exports = {
  root: true,
  extends: ['airbnb-base', "prettier", 'plugin:vue/recommended'],
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: 'babel-eslint',
    sourceType: 'module',
    ecamaVersion: 2018,
  },
  env: {
    browser: true,
    node: true,
  },
  globals: {
    __static: true
  },
  plugins: [
    "vue",
    "prettier"
  ],
  'rules': {
    'import/no-unresolved': 0,
    "import/no-extraneous-dependencies":0,
    // allow paren-less arrow functions
    'arrow-parens': 0,
    // allow async-await
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0,
    "lines-between-class-members": ["error", "always", {
      exceptAfterSingleLine: false
    }],
    "arrow-body-style": 2,
    "comma-dangle": 0,
    "semi": [2, 'never'],
    "object-curly-newline": 0,
    "max-len": [0, {
      "code": 200
    }],
    "global-require": 0,
    "guard-for-in": 0,
    "no-console": 0,
    "no-unused-vars": 1,
    "no-param-reassign": [2, { "props": false }],
    "no-restricted-syntax":0,
    "no-restricted-globals":0,
    "no-underscore-dangle":0,
    "no-shadow":0,
    "no-lonely-if":0,
    "vue/require-valid-default-prop":0,
    "vue/no-unused-vars": 1,
    "vue/require-v-for-key": 1,
    "vue/max-attributes-per-line": 0, // 解决Attribute "bind" should be on a new line
    "vue/html-self-closing": 0,
    "vue/order-in-components": 0,
    'vue/attribute-hyphenation': 0,
    "vue/no-parsing-error": [2, {
      "x-invalid-end-tag": false
    }],
  }
}
```

##### 3、增加 .prettierrc.js 文件，用于在扫描通过后格式化代码(该步骤可选，如果不引入prettier的话，相应的在package和eslint中去除掉相应配置即可)

```javascript
module.exports = {
  printWidth: 80, // 设置prettier单行输出(不拆行)的(最大)长度，默认为80
  tabWidth: 2, // 设置工具每一个水平缩进的空格数
  useTabs: false, // 使用tab(制表位)缩进而非空格
  semi: false,  // 使用分号结尾，默认为true
  singleQuote: true, // 使用单引号，在jsx语法中，所有引号均为双引号，改设置在jsx中被自动忽略
  trailingComma: 'none',  // 行尾逗号，默认none,可选none|es5|all    
    // none 无尾逗号
    // es5 添加es5中被支持的尾逗号
    // all 所有可能的地方都被添加尾逗号
  bracketSpacing: true, // 在对象字面量声明所使用的花括号后({)和前(})输出空格  true -- { foo: bar }   false -- {foo:bar}
  jsxBracketSameLine: false, // 在多行jsx元素最后一行的末尾添加 > 而使 > 单独一行 (不适用于自闭和元素)
  arrowParens: 'avoid', // 箭头函数参数括号 默认avoid 可选 avoid| always
    // avoid 能省略括号的时候就省略 例如 x => x
    // always 总是有括号
  requirePragma: false, // 严格按照文件顶部的一些特殊的注释格式化代码
  insertPragma: false, // 在文件的顶部插入一个@format的特殊注释，以表明该文件已经被Prettier格式化过了。
  proseWrap: 'preserve' // 文件折行的方式 可选参数：always|never|perserve
    // always 当超出print width时就折行
    // never 不折行
    // perserve 按照文件原样折行
}
```

### 实现原理

##### 1、执行流程

达到上述效果，执行的流程如下：

- 待提交的代码git add添加到暂存区；
- 执行git commit;
- husky注册在git pre-commit的钩子函数被调用，执行lint-staged;
- lint-staged取得所有被提交的文件依次执行写好的任务(ESlint 和 Prettier);
- 如果有错误(没通过ESlint检查) 则停止任务，同时打印错误信息，等待修复后再执行commit；
- 成功commit，可push到远程。

在上述流程中，有这样几个核心点：

- husky注册git的钩子函数保证在git 执行commit时调用代码扫描的动作；
- eslint完成按照配置的规则进行扫描；
- Lint-staged保证只对当前add到git stage区的文件进行扫描操作，这样做的原因在于，如果对全工程的文件进行扫描的话，并且之前的前端工程并未注重代码规则的检测的话，很大可能性会出现成百上千的error，基本上心里是崩溃的。因此，只对当前add的文件进行检测，达到及时止损的目的，历史代码可以切到新的分支进行修复后再进行合并。

##### 2、插件说明

- ###### eslint

  eslint是一个插件化的JavaScript代码检测工具。

  eslint的优点如下：

  1、可配置的代码检测工具；

  2、确保应用的线上质量，不满足代码规范的代码都不能push到远程分支上；

  3、更高的可读性，eslint会对代码质量进行扫描，并且一般结合prettier使用的话，在通过代码规范后可以对代码进行格式化，可以保证代码可读性；

  4、避免低级的错误，通过eslint --fix可以根据规范对代码的部分低级错误进行更正。

- ###### husky

  确保本地的代码已经通过检查才能push到远程，这样才能从一定程度上确保应用的线上质量，同时能够避免lint的反馈流程过长。

- ###### lint-staged

  每次只对当前修改后的文件进行扫描，即进行git add加入到stage区的文件进行扫描即可，完成对增量代码进行检查。如何实现呢？这里就需要使用到lint-staged工具来识别被加入到stage区文件。

  ```javascript
  "scripts": {
    "precommit": "lint-staged"
  },
  "lint-staged": {
    "src/**/*.js": [
      "eslint --fix --ext .js",
      "prettier --write",
      "git add"
    ]
  }
  ```

  在进行git commit的时候会触发到git hook进而执行precommit，而precommit脚本引用了lint-staged配置表明只对git add到stage区的文件进行扫描，具体lint-staged做了三件事：

  1、执行eslint --fix操作，进行扫描，若发现工具可修复的问题进行fix;

  2、执行prettier脚本，这是对代码进行格式化的；

  3、上述两项任务完成后对代码重新add。

- ###### prettier

  prettier工具主要用来统一代码格式，eslint也会对代码进行一定程度的格式校验，但主要用来对代码规范的扫描，而prettier则是专门用来对代码进行格式化，两个工具各司其职，为代码质量保驾护航。它的主要原理是将格式化前的代码和格式化后的代码进行对比，如果发现不一样，prettier就会对其进行标记并按照指定的格式化规范进行修复。

### 附加内容

##### editorconfig介绍

当多人共同开发一个项目时，每个人使用的编辑器各不相同，为了对编辑器统一代码风格，引入了editorconfig。

用editorconfg解决上述问题的步骤：

- 在项目根目录下创建一个名为 .editorconfig 的文件。该文件的内容定义改项目的编码规范。editorconfig支持的编码规范在后文会有详细的介绍。
- 安装与编辑器对应的editorconfig插件。

其工作原理是：当你再编码的时候，editorconfig插件会去查找当前编辑文件的所在文件夹或其上级文件夹中是否有 .editorconfig 文件。如果有，则编辑器的行为会与 .editorconfig 文件中定义的一致，并且其优先级高于编辑器自身的设置。