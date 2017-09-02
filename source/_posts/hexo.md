---
title: 利用Hexo框架在Github上搭建个人博客
desc: 'Lorem ipsum dolor sit amet, consectetur.'
date: 2016-07-10 21:55:22
tags: hexo
category: hexo
---
### 导语:
[hexo](https://hexo.io) 是一款基于Node.js的静态博客框架，是一款快速、简洁且高效的博客框架。
>它具有以下几个特点：
  + 超快速度
    Node.js 所带来的超快生成速度，让上百个页面在几秒内瞬间完成渲染。
  + 支持Markdown
    Hexo 支持 GitHub Flavored Markdown 的所有功能，甚至可以整合 Octopress 的大多数插件。
  + 一键部署
    只需一条指令即可部署到 GitHub Pages, Heroku 或其他网站。
  + 丰富的插件
    Hexo 拥有强大的插件系统，安装插件可以让 Hexo 支持 Jade, CoffeeScript。  

<!-- more -->


### 环境安装:
1. [安装git](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)
2. [安装node.js](http://jingyan.baidu.com/article/fd8044faf2e8af5030137a64.html)
### 安装hexo
在你的指定文件夹中打开git或者cmd执行以下命令进行安装：
```
$ npm install hexo-cli -g  //安装hexo框架
$ hexo init blog  //初始化你的博客项目到/blog(你也可以指定自己的目录)目录下
$ cd blog  //进入博客目录
$ npm install //安装blog项目所依赖的各种插件(最新版的hexo会在初始化项目时自动安装)
$ hexo server //启动一个本地服务器,预览初始化后的博客 (默认为本机地址:http://localhost:4000/)
```

在浏览器中打开 http://localhost:4000/，这时可以看到Hexo已为你生成了一篇blog。

### 在Github上创建个人博客仓库
登陆你自己的[github](https://github.com/),在[创建仓库](https://github.com/new)页面创建自己的个人博客项目管理仓库,注意仓库名臣格式必须是username.github.io(uesername为你自己定义的项目名)
> github会为username.github.io命名规则的仓库开启site(站点)功能即 GitHub Pages 功能，site默认地址为：https://usename.github.io/

![alt text](/images/hexo/2016-07-10_233220.png)
![alt text](/images/hexo/2016-07-10_233422.png)
### 生成静态资源，部署到Github
1. 配置本地_config.yml
在你的博客项目根目录下找到`_config.yml`文件,在最后找到`deploy`选项,修改如下：
```
deploy:
  type: git //部署类型，我采用的是git上传
  repo: git@github.com:relign/relign.github.io.git //部署的github仓库地址，此处必须是刚刚部署的仓库的SSH地址
  branch: master //部署仓库的分支(默认为master分支)
```
![alt text](/images/hexo/2016-07-11_001900.png)

更多上传方式参见官网deploy参数设置:https://hexo.io/zh-cn/docs/deployment.html
git上传有问题的，建议通读廖雪峰的 [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

2. 生成静态资源
在博客项目根目录地址下打开git,运行命令`$ hexo generate` (可简写为`$ hexo g`) 即可生成网站的静态资源，存储在项目根目录下的 public 文件夹下。
3. 部署到Github
执行命令`$ hexo deploy` (可简写为`$ hexo d`) 即可将本地的静态资源上传到Github仓库中。

最后打开浏览器输入网址：https://username.github.io/ (username为你之前设置的仓库名称) 即可查阅你的博客站点。
