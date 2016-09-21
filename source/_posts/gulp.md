---
title: gulp之学习教程
desc: 'Lorem ipsum dolor sit amet, consectetur.'
date: 2016-07-11 23:24:26
tags: gulp
---
### 导语
gulp是基于node环境的一款前端自动化构建工具，是自动化构建项目的神器。利用它我们可以抛弃前端项目开发中很多重复的工作,比如：压缩文件(css,js,img)，编译文件(es6、JSX、coffee),文件测试、合并、格式化,浏览器自动刷新,部署文件生成等等,极大的开发了我们的工作效率。
gulp与grunt非常像，但是grunt频繁进行IO操作，用插件做太多的事情，插件没有遵守单一责任原则，落后流程控制直接导致其性能滞后。而gulp有以下几个特点:
> * 易于使用
通过代码优于配置的策略，Gulp 让简单的任务简单，复杂的任务可管理。
* 构建快速
利用 Node.js 流的威力，你可以快速构建项目并减少频繁的 IO 操作。
* 插件高质
Gulp 严格的插件指南确保插件如你期望的那样简洁高质得工作。
* 易于学习
通过最少的 API，掌握 Gulp 毫不费力，构建工作尽在掌握：如同一系列流管道。

<!-- more -->

### 安装gulp
1.由于gulp基于node环境，所以首先你需要[安装node](http://jingyan.baidu.com/article/fd8044faf2e8af5030137a64.html);
2.打开命令行工具(CMD或者git),安装gulp(全局安装);
` $ npm install gulp -g `
### 运行一个简单的任务
我们通过构建一个简单的编译sass文件任务,来了解gulp;
1.新建一个文件夹(我的是gulp),进入这个文件夹;
`$ cd gulp`
2.初始化这个新项目,生成package.json;
`$ npm init `
>package.json文件描述了一个NPM包的所有相关信息，包括作者、简介、包依赖、构建等信息,通过它我们可以了解到项目开发中所依赖的各种插件信息。
当然它远不止这么简单，你可以参考这篇文章去更加详细的了解它:http://ju.outofmemory.cn/entry/130809。

3.安装示例所需要的npm包;
你可以将gulp当作项目依赖安装
`$ npm install gulp --save-dev`
安装编译sass文件的gulp插件 gulp-sass
`$ npm install gulp-sass --save-dev`
安装gulp-sass依赖的 node-sass
`$ npm install node-sass --save-dev`
>如果安装失败,可尝试用淘宝镜像源cnpm去安装,80%的几率你会安装失败
`$ npm install -g cnpm --registry=https://registry.npm.taobao.org`

4.建立gulpfile.js文件
在项目根目录下创建一个名为 `gulpfile.js` 的文件作为gulp的入口文件。
接着创立一个scss文件 index.scss
```css
@mixin center($width,$height){
  width: $width;
  height: $height;
  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -($height) / 2;
  margin-left: -($width) / 2;
}
.box-center{
  @include center(500px,300px);
}
```

在gulpfile.js中编写代码去管理sass编译:
```javascript
var gulp = require('gulp');
var sass = require('gulp-sass');
gulp.task('sass',function () {
  return gulp.src('./index.scss')
  .pipe(sass())
  .pipe(gulp.dest('./build'));
})
```

打开命令行(CMD或者git)执行命令
`$ gulp sass`
查看编译出的文件
```css
.box-center {
  width: 500px;
  height: 300px;
  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -150px;
  margin-left: -250px; }
```

编译成功！
