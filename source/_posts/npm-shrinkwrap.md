---
title: 一场由npm shrinkwrap执行error引发的思考
date: 2016-10-13 17:00:48
tags: [npm, shrinkwrap]
category: node
---

> 做项目时执行 `npm shrinkwrap` 报错,并对这次错误求根问底,贴出来让大家看看,希望跟我出现一样错误的童鞋可以得到解答

## npm shrinkwrap 命令简介    

项目开发过程中,当你使用`npm install [package] --save`安装项目所依赖的包的时候,会在`package.json`文件中写入包的版本号:`"express": "^4.8.7"`(一般是这种版本号,`^`表示允许大版本迭代,即允许`4.8.7`到`5.0.0`之间版本的升级).但这样就会出现问题,当你下次执行`npm install`安装`package.json`中文件的时候,可能会出现版本不一致导致运行结果不同的情况.

<!-- more -->

`npm shrinkwrap`这个命令用来指定项目依赖包的安装版本,可以稳定项目的开发环境.它会在当前目录下创建`npm-shrinkwrap.json`文件,里面描述了项目所依赖的包,以及这些包内嵌套的包的确定版本号和包的下载位置:    

```json
"acorn-jsx": {
      "version": "3.0.1",
      "from": "acorn-jsx@>=3.0.0 <4.0.0",
      "resolved": "http://r.npm.sankuai.com/acorn-jsx/-/acorn-jsx-3.0.1.tgz",
      "dependencies": {
        "acorn": {
          "version": "3.2.0",
          "from": "acorn@>=3.0.4 <4.0.0",
          "resolved": "http://r.npm.sankuai.com/acorn/-/acorn-3.2.0.tgz"
        }
      }
    }

```

当项目中其他人进行开发执行`npm install`的时候就会去这个文件中找到包的下载位置和版本号去安装,从而实现项目依赖包锁定版本号.如果你想要更新`npm-shrinkwrap.json`中包的版本,你可以先`npm update [package]`更新包,再用`npm shrinkwrap`重新创建`npm-shrinkwrap.json`文件.    
## 执行npm shrinkwrap报错    
新人刚到公司,接手项目,偶然一次机会执行了`npm shrinkwrap`命令,发现它居然报错了,于是便追根溯源.报错信息如下:    

```dash
npm WARN shrinkwrap   underscore: '^1.8.3' }
npm ERR! Darwin 16.0.0
npm ERR! argv "/Users/relign/.nvm/versions/node/v4.4.6/bin/node" "/Users/relign/.nvm/versions/node/v4.4.6/bin/npm" "shrinkwrap"
npm ERR! node v4.4.6
npm ERR! npm  v3.10.2

npm ERR! Problems were encountered
npm ERR! Please correct and try again.
npm ERR! extraneous: babel-generator@6.11.4 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-generator
npm ERR! extraneous: lodash@4.14.0 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-generator/node_modules/lodash
npm ERR! extraneous: babel-plugin-transform-es2015-block-scoping@6.10.1 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-plugin-transform-es2015-block-scoping
npm ERR! extraneous: lodash@4.14.0 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-plugin-transform-es2015-block-scoping/node_modules/lodash
npm ERR! extraneous: babel-register@6.11.5 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register
npm ERR! extraneous: babel-core@6.11.4 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register/node_modules/babel-core
npm ERR! extraneous: babylon@6.8.4 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register/node_modules/babylon
npm ERR! extraneous: core-js@2.4.1 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register/node_modules/core-js
npm ERR! extraneous: lodash@4.14.0 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register/node_modules/lodash
npm ERR! extraneous: minimatch@3.0.2 /Users/relign/Documents/project/hotel-fe-tbms/node_modules/babel-register/node_modules/minimatch
npm ERR! missing: pmx@latest, required by pm25@0.143.6
npm ERR! missing: vizion@latest, required by pm25@0.143.6
npm ERR! missing: pm2-axon@latest, required by pm25@0.143.6
npm ERR! missing: pm2-deploy@latest, required by pm25@0.143.6
npm ERR!
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>

npm ERR! Please include the following file with any support request:
npm ERR!     /Users/relign/Documents/project/hotel-fe-tbms/npm-debug.log
```

重点在于`Problems were encountered`,于是上万能的`Google`去搜索,搜到如下[结果](https://github.com/SaltwaterC/aws2js/issues/58):    

```json
If you want to make shrinkwrap work, you have to have only modules installed via (dev)Dependencies in package.json.

If you use solutions like napa or if you manually installed some modules in node_modules, it won't work.

npm prune removes these extra modules and it should work juster after that.
```

如果你想要`shrinkwrap`起作用,你就只能安装`package.json`中`(dev)Dependencies`字段名下的模块.
如果你使用了`napa`或者你在`node_modules`手动安装了一些模块,`shrinkwrap`不会起作用.
`npm prune` 这个命令可以删除这些模块,然后`shrinkwrap`就会起作用了.

## npm prune命令简介    
`npm prune`,用于清除多余的包资源.在项目`package.json`所在路径下运行这个命令,会移除`package.json`中没有包含的`node_modules`的包,如果要移除`devDependencies`里面的包,命令中加入`--production`.    

## 新的错误
执行完`npm prune`后,我继续执行`npm shrinkwrap`出现了新的错误:             

```json
npm WARN shrinkwrap   underscore: '^1.8.3' }
npm ERR! Darwin 16.0.0
npm ERR! argv "/Users/relign/.nvm/versions/node/v4.4.6/bin/node" "/Users/relign/.nvm/versions/node/v4.4.6/bin/npm" "shrinkwrap"
npm ERR! node v4.4.6
npm ERR! npm  v3.10.2

npm ERR! Problems were encountered
npm ERR! Please correct and try again.
npm ERR! missing: pmx@latest, required by pm25@0.143.6
npm ERR! missing: vizion@latest, required by pm25@0.143.6
npm ERR! missing: pm2-axon@latest, required by pm25@0.143.6
npm ERR! missing: pm2-deploy@latest, required by pm25@0.143.6
npm ERR!
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>

npm ERR! Please include the following file with any support request:
npm ERR!     /Users/relign/Documents/project/hotel-fe-tbms/npm-debug.log
```

在错误日志中可以看到关于`extraneous`相关的错误已经解决了,还剩下`missing`相关的错误.紧接着好到了先关问题的[探讨](https://github.com/JamieMason/shrinkpack/issues/14):    

```dash
npm install
npm prune
npm dedupe
npm install
npm shrinkwrap --dev
```

关键在于`npm deduqe`命令.  
## 神奇的npm deduqe
### 令人头疼的npm的依赖   
用过npm1或者npm2的人,尤其是windows的用户,相信大家都遇到过`node_modules`太多太深的情况,当你复制粘贴删除就会报错.      

而使用npm3的童鞋会发现自己的`node_modules`下面出现了很多你没有`npm install`安装过的模块.    

究其原因是由于npm的依赖方式引起的.   

假如项目开发过程中我们写了个模块app,app模块依赖两个包`a@1`和`b@1`,其中`a@1`依赖包`c@1`,`b@1`依赖包`c@2`.    

npm1或者npm2安装后是这样的:    

```json
app
+-- a@1 <-- depends on c@1
|   `-- c@1
`-- b@1 <-- depends on c@2
    `-- c@2
```

npm1或者npm2安装模块,不会去理会包之间的公共依赖关系,依赖模块会一层一层继续安装下去,这样`node_modules`很容易就会超过windows资源管理器能处理的最长路径长度了,也就是你在windows下复制粘贴删除`node_modules`会报错的原因了.    

npm3安装后是这样的:

```json
app
+-- a@1 <-- depends on c@1
+-- c@1
`-- b@1 <-- depends on c@2
    `-- c@2`
```

npm3会按照`package.json`中依赖模块的顺序依赖模块,安装`a@1`的时候发现`c@1`没有安装过,在现有的情况下也没有其他版本的`c`模块冲突,它会把`c@1`便会放在第一层目录,而`c@2`为了避免和`c@1`冲突,还是继续放在`b@1`下面.    

**npm3在一定程度上处理了包之间的公共依赖关系**.    
这里为什么要说是一定程度呢？现在app又需要安装一个`d@1`,`d@1`依赖`c@2`,npm3安装后会是这样:    

```json
app
+-- a@1 <-- depends on c@1
+-- c@1
+-- b@1 <-- depends on c@2
    `-- c@2`
`-- d@1 <-- depends on c@2
    `-- c@2`    
```

因为之前安装`a@1`的时候,依赖的`c@1`已经被放到第一层目录,后续安装其他依赖包的时候,如果安装包依赖`c`,只要版本不是`1`,为了避免和`c@1`冲突,都会安装到相应的包下面.    

接下来app又依赖`e@1`,而`e@1`又依赖`c@1`,但是`c@1`已经安装过了,并且不会版本冲突,那么npm3安装后就会变成这样:    

```json
app
+-- a@1 <-- depends on c@1
+-- c@1
+-- b@1 <-- depends on c@2
    `-- c@2`
+-- d@1 <-- depends on c@2
    `-- c@2`
`-- e@1    
```

紧接着问题来了,app要升级了,`a@1`要升成`a@2`,而`a@2`依赖`c@2`,
`e@1`升级`e@2`,`e@2`依赖`c@2`,这时`c@1`不再被依赖,`npm`安装时就会卸载`c@1`,npm3安装后就会变成这样:   

```json
app
+-- a@2 <-- depends on c@2
    `-- c@2`
+-- c@2
+-- b@1 <-- depends on c@2
    `-- c@2`
+-- d@1 <-- depends on c@2
    `-- c@2`
`-- e@2    
```

预期结果跟大家想的不一样吧,模块出现了冗余,既然模块都依赖`c@2`,为什么还需要安装这么多呢?这时,神奇的`npm dedupe`就出现了.    

### npm dedupe简介    

项目开发过程中,npm的树状依赖结构很大几率会导致重复的模块和代码量的臃肿,`npm dedupe`可以尽量去压平依赖树.   

在npm的[文档](https://docs.npmjs.com/cli/dedupe)中对它是这么解释的:    

```
Searches the local package tree and attempts to      
simplify the overall structure by moving dependencies      
further up the tree, where they can be more effectively shared by multiple dependent packages.
```

这个命令会去搜索本地的`node_modules`中的包,并且通过移动相同的依赖包到外层目录去尽量简化这种依赖树的结构,让公用包更加有效被引用.   

那么上面的例子执行一次`npm dedupe`,就会变成这样:    

```json
+-- a@2 <-- depends on c@2
+-- c@2
+-- b@1 <-- depends on c@2
+-- d@1 <-- depends on c@2
`-- e@2    
```

那么现在这个结构就是最优结构了.同时解决了Windows上令人头疼的问题.    



**参考资料**     

* [npm-dedupe](https://docs.npmjs.com/cli/dedupe)
* [玩转npm](http://www.jianshu.com/p/8a114304dd6e)
* [npm、bower、jamjs 等包管理器，哪个比较好用](https://www.zhihu.com/question/24414899)
* [npm 重点小结](http://www.cnblogs.com/JuFoFu/p/5643471.html)
* [npm shrinkwrap extraneous module](http://stackoverflow.com/questions/35982100/npm-shrinkwrap-extraneous-module)
