---
layout:     post
title:      "前端工程化"
subtitle:   " \"前端工程化相关知识整理\""
date:       2019-04-20 22:00:00
author:     "iwalking11"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - 前端工程化
---



## 为什么需要工程化
随着近些年来前端技术的不断发展，越来越多复杂的业务放在了前端，前端不再是以前几个HTML + CSS + javascript就能解决的了。业务复杂了，需要维护的代码量就自然多了，如此一来，前端代码的可靠性，可维护性，可拓展性，以及前端web应用的性能，开发效率等等各方面就成了不得不考虑的问题。

于是我们就产生了前端工程化这个概念，来解决这些问题。

前端工程化有四个特点：模块化、组件化、自动化、规范化。  

## 模块化(低耦合)：
就是将一个大文件拆分成相互依赖的小文件，再进行统一的拼装和加载。只有这样，才有多人协助的可能。在工程化之前，一直是使用js、jquery、ajax，这没有模块概念，对于开发大型且复杂的系统会有一定的限制。

### 模块化打包工具
 整个项目存在一个或多个入口js文件，通过这个入口找到项目的所有依赖文件，通过loader，plugin进行处理后，打包生成对应的文件，输出到指定的output目录中。可以说是集模块化与工作流于一身的工具
 
 webpack 的最大特点是一切皆为模块，一切全包，最适和应用在SPA一站式应用场景。只有简单几个页面的情况下使用 webpack
 
 反而可能会增加不必要的配置成本，反而直接用gulp或者其他工具处理代码压缩，css 预处理之类的工作会更加快捷易用。 

### JS的模块化
在ES6之前，JavaScript一直没有模块系统，这对开发大型复杂的前端工程造成了巨大的障碍。对此社区制定了一些模块加载方案，如CommonJS、AMD和CMD等，某些框架也会有自己模块系统，比如Angular1.x。

现在ES6已经在语言层面上规定了模块系统，完全可以取代现有的CommonJS和AMD规范，而且使用起来相当简洁，并且有静态加载的特性。

### CSS的模块化
虽然SASS、LESS、Stylus等预处理器实现了CSS的文件拆分，但没有解决CSS模块化的一个重要问题：选择器的全局污染问题。

CSS in JS和CSS Modules

#### JS中直接使用style
CSS in JS是彻底抛弃CSS，使用JS或JSON来写样式。这种方法很激进，不能利用现有的CSS技术，而且处理伪类等问题比较困难；React-Style用的就是这种

缺点：

- 无法使用伪类
- 样式会大量重复
- 不能使用stylus、sass等成熟的预处理器

#### CSS Modules
仍然使用CSS，只是让JS来管理依赖。它能够最大化地结合CSS生态和JS模块化能力，目前来看是最好的解决方案。Vue的scoped style也算是一种。

webpack中css-loader开启了css模块功能

[webpack项目轻松混用css module](https://juejin.im/post/5b195fcde51d4506b62cb0d3)

### 资源模块化
WebPack可以看做是模块打包机：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其打包为合适的格式以供浏览器使用。

Webpack的强大之处不仅仅在于它统一了JS的各种模块系统，取代了Browserify、RequireJS、SeaJS的工作。更重要的是它的万能模块加载理念，即所有的资源都可以且也应该模块化。

#### 资源模块化后，有三个好处：
- 依赖关系单一化。所有CSS和图片等资源的依赖关系统一走JS路线，无需额外处理CSS预处理器的依赖关系，也不需处理代码迁移时的图片合并、字体图片等路径问题；
- 资源处理集成化。现在可以用loader对各种资源做各种事情，比如复杂的vue-loader等等。
- 项目结构清晰化。使用Webpack后，你的项目结构总可以表示成这样的函数： dest = webpack(src, config)


## 组件化(高内聚)（React、Vue）
组件化≠模块化。模板化只是在文件层面上，对代码和资源的拆分；组件化是在设计层面上，对于UI的拆分。

从UI上拆分下来的每一个包模板（html）+样式（CSS）+逻辑（JS）功能完备的结构单元，称之为组件。

页面上所有的东西都可以看成组件，页面是个大型组件，可以拆成若干个中型组件，然后中型组件还可以再拆，拆成若干个小型组件，小型组件也可以再拆，直到拆成DOM元素为止。DOM元素可以看成是浏览器自身的组件，作为组件的基本单元。

传统前端框架/类库的思想是先组织DOM，然后把某些可复用的逻辑封装成组件来操作DOM，是DOM优先，而组件化框架/类库的思想是先来构思组件，然后用DOM这种基本单元结合相应逻辑来实现组件，是组件优先。这是两者最本质的区别。

目前市场上的组件化框架最多，主要的有Vue，React，Angular2。

## 自动化（Webpack）
“简单重复的工作交给机器来做”，自动化也就是有很多自动化工具代替我们来完成，例如持续集成、自动化构建、自动化部署、自动化测试等等。

Webpack自动化构建压缩打包

Webstorm自动化部署

## 规范化（至关重要的一环）

在项目规划初期制定的好坏对于后期的开发有一定影响。包括的规范有：
- 目录结构的制定
- 编码规范 eslint 代码检查
- 前后端接口规范
- 文档规范
- 组件管理
- Git分支管理
- Commit描述规范
- 定期codeReview
- 视觉图标规范

## 提升开发效率

### webpack-dev-server 热加载

以前，我们的日常前端开发的流程是这样的： 修改代码 -> 切换IDE到浏览器 -> 刷新浏览器查看效果（有时候还需要清除缓存） -> 修改代码 ....。

这套流程，尤其是刷新浏览器这个过程，无疑是相当低效繁琐枯燥的。 而webpack-dev-server 替我们解决了这个问题

- 它有两种模式，两种模式，一种是 watch，HMR（热更新） 模式，功能是你修改代码，自动帮你刷新页面，无需手动刷新；

- 另一种更加强大，基于 websocket（热替换） 全双工通信技术，直接无刷新帮你把修改的代码替换掉。 从而极大程度上提高了开发效率。

### 数据mock

在后端接口还没提供的时候，前后端制定好共同的接口协议，开发时前端可以使用mock模拟数据，与后端彻底分离，并行开发。面向接口编程，尽可能减少前后端沟通成本。

webpack-dev-server是我们开发vue、react时必备的工具，通过webpack-dev-server的before钩子，可以在webpack-dev-server上添加我们需要的mock server功能，而不需要另行搭建服务器

### 使用IDE

强大的 IDE 可以让开发过程如丝般顺滑
- VSCode: https://code.visualstudio.com/
- Atom: https://atom.io/
- Sublime: https://www.sublimetext.com/
- webstorm: https://www.jetbrains.com/webstorm/

更重要的是，这些 IDE 提供了插件机制，想要的功能通过插件大部分能够实现。

### 错误与性能监控

对页面错误和性能的监控是必要的，随着团队规模的不同，监控系统接入的难度不同。

其中，效率最低、准确性最差的是手动添加监控代码。这种我们放弃。

自动添加监控代码包括：
- 错误、性能监控脚本能够自动化添加到线上页面
- 错误、性能监控能够自动化获取

监控代码部署后，就通过各种手段展现、通知责任人，查看优化前后效果对比等。

性能监控系统需要的功能
- 数据可视化
- 可以查看：全网/业务/页面 的数据表现
- 可以查询任意时间段的数据
- 可以查看表现变化情况（趋势）
- 秒开率计算算法符合逻辑
- 展示首屏网络传输各个时刻的状态（转场、查询缓存、dns 解析、建立 tcp 连接等）
- 找出瓶颈页面
- 20%、80% 的页面处于哪个时间区间
- 找到处于某个时间区间的有哪些页面，以及各种网络信息
- 消息通知
- 线上性能表现差时，通过消息通知责任人优化，精确到页面

工具推荐：

自动获取前端首屏时间：https://github.com/hoperyy/auto-compute-first-screen-time

## 参考
[前端工程化](https://github.com/hoperyy/front-end-engineering)
[web前端工程化/构建自动化](http://www.fly63.com/article/detial/380)
[谈谈前端工程化是个啥？](http://www.fly63.com/article/detial/1504)


