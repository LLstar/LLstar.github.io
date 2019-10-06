---
title: 利用hexo搭建自己的博客
date: 2019-5-29 15:19:48
categories: hexo
---
### 前言

作为一个程序员，不论功力高深与否，都要有一个属于自己的博客，来记录自己成长的轨迹以及心得。本教程针对的是不懂技术又想搭建个人博客的小白，简单快速的搭建一个属于自己的个人博客。正所谓站在巨人的肩膀上，本教程参考了别人搭建博客的经历，再加上自己实战经历。

### 快速开始

我采用的搭建博客的方式是使用github + hexo 的方式。

hexo是一个快速、简洁且高效的博客框架(官网介绍)。[hexo官网](https://hexo.io/zh-cn/docs/)。

<!-- more -->

#### 一、安装Git

- Windows：下载并安装 [git](https://git-scm.com/download/win).
- Mac：使用 [Homebrew](http://mxcl.github.com/homebrew/), [MacPorts](http://www.macports.org/) ：`brew install git`;或下载 [安装程序](http://sourceforge.net/projects/git-osx-installer/) 安装。

#### 二、安装 Node.js

- 安装 Node.js 的最佳方式是使用 [nvm](https://github.com/creationix/nvm)。 
- 或者您也可以下载 [安装程序](http://nodejs.org/) 来安装。 

#### 三、在GitHub上创建GitHub Pages项目

- 创建github仓库，命名格式为：github用户名+github.io（前提是要有github账号），GitHub会识别并自动将该仓库设为GitHub Pages。用户主页是唯一的，填其他的名称会被当做是普通项目。

- 克隆刚刚创建的项目仓库到本地，新建一个index.html页面，填入一些修改信息。

  ```bash
  $ cd 项目名
  $ echo 'Hello World' > index.html
  $ git add *
  $ git commit -m 'first commit'
  $ git push -u origin master
  ```

  这时候，打开了浏览器，输入https://username.github.io，就可以成功访问啦。（出现Hello World表示成功啦）。

#### 四、安装并启动hexo

```bash
$ npm install -g hexo 
$ yarn add global hexo // 全局安装hexo
// 手动创建一个文件夹，我的文件夹为Blog
$ cd Blog
$ hexo init 初始化项目，新建一个网站
$ hexo install 
$ hexo gengerate // 简写：hexo g 生成静态网页
$ hexo server // 简写：hexo s 启动本地服务，默认端口为4000
$ hexo s -p [port] // 修改端口号
```

#### 五、部署

- 修改根目录下的 _config.yml文件：

  ```javascript
  {
      deploy:
        type: git 
        repo: https://github.com/username/username.github.io.git // 博客的地址
        branch: master // 选择GitHubPages中设置的那个分支，而不是拉取这个项目的分支，一般为master
        message: Blog // 可自定义
  }
  ```

#### 六、写作和配置

```bash
$ hexo new [layout] <title> // 创建新的博文
```
- 菜单显示about链接，在主题的_config.yml设置中将menu中about前面的注释去掉即可。

```javascript
menu:
  home: /
  archives: /archives
  tags: /tags
  about: /about
```
- 创建页面

```bash
$ hexo new page "about" // "关于我"页面
$ hexo new page "categories" // "分类"页面
$ hexo new page "tags" // 创建标签云页面
```

- 设置侧边栏头像

  编辑主题的`_config.yml`文件，新增字段`avatar`，值设置成头像的链接地址。

  其中，头像的链接地址可以分为两种：

  1、完整的地址：例如：`https://avatars1.githubusercontent.com/u/32269?v=3&s=460 `

  2、站内地址：例如：`source/img/avatar.jpg`需要将你的图片放置在主题的`source/img`文件夹下

- 设置加载更多

  要实现加载更多，只需要在文章开头写好摘要后，另起一行添加<!-- more -->即可

  ```
  ---
  这是摘要
  <!-- more -->
  这是正文
  ```

  如果想要修改加载更多的样式，打开主题中的_config.yml文件：

  ```javascript
  excerpt_link: 阅读更多 // 默认为more
  ```


#### 七、踩坑经历

##### 1、不蒜子网站统计报错

采用spfk这个主题，发现他利用不蒜子网站统计不显示，看到不蒜子的官方网站发出了这样的消息：

![](https://i.loli.net/2019/06/20/5d0b51950f72e26880.png)

进入hexo博客项目的`themes`目录下，在`spfk/layout/share/after-footer.ejs`文件，将

```javascript
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

更改为：

```javascript
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js">
</script>
```

