---
title: 利用EditorConfig定义代码格式,统一代码风格
date: 2016-09-25 12:22:30
tags: [EditorConfig]
category: 前端工具
---

> 在项目开发过程中,不同开发人员会使用不同的编辑器,常见的如:WebStorm,Atom,Sublime等等.不同编辑器对于缩进风格,默认的编码格式以及空白字符的设置等各有风格,因此在项目中做代码统一规范让开发人员尤为痛苦.    

### 作用

**EditorConfig** 帮助开发人员统一和维护不同编辑器之间的代码风格,有助于项目做`Eslint`等的代码检查和代码规范的统一.`EditorConfig`通过在项目中构建一个配置文件来让编辑器插件进行格式化代码.`EditorConfig`配置文件简单易懂，并且能够很好的和版本控制器协同工作.     

> 版本控制系统(Version Control Systems ,简称VCS)也称为源代码控制系统,它是跟踪并存储系统各对象源代码及文档的开发历史和变更资料,并提供了系统故障恢复及开发进程管理的功能.    

<!--more-->

### 配置文件     

在下面的例子中,`.editorconfig` 配置文件设置了end-of-line（行末换行符选择）和indentation styles（代码缩进风格),该配置文件将规范项目中Html,css,SCSS和JavaScript等文件的代码风格.

``` ruby
# EditorConfig is awesome: http://EditorConfig.org
# top-most EditorConfig file
# 最顶层的配置文件
root = true
# Unix-style newlines with a newline ending every file
# 行末应该采用什么符号来进行换行
[*]
end_of_line = lf
# 文件以空白行结尾
insert_final_newline = true
# Matches multiple files with brace expansion notation
# Set default charset
# 设置默认的编码格式
[*]
charset = utf-8
# 4 space indentation
[*.{js,html}]
# 缩进样式是 tab  还是 space
indent_style = space
# 每次缩进相当于多少列
indent_size = 4
# 去除代码行末的任意空白字符
trim_trailing_whitespace = true
# Indentation override for all JS under lib directory
[*.scss]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2
```    

这里给出一些项目中使用`EditorConfig`的[实例](https://github.com/editorconfig/editorconfig/wiki/Projects-Using-EditorConfig).     

### 配置文件详解    

> 转载

当打开一个文件时,`EditorConfig`插件会遍历文件的目录和其每一级父目录查找`.editorconfig`文件,直到有一个配置文件`root=true`.

`EditorConfig`配置文件从上往下读取,并且路径最近的文件最后被读取.匹配的配置属性按照属性应用在代码上,所以最接近代码文件的属性优先级最高.

**Windows 用户:** 在资源管理器创建`.editorconfig`文件,可以先创建`.editorconfig.`文件,系统会自动重名为`.editorconfig`.

### 文件格式详情

`EditorConfig`文件使用INI格式(*译注：请参考维基百科*),目的是可以与[Python ConfigParser Library](https://docs.python.org/2/library/configparser.html)兼容,但是允许在分段名（译注：原文是section names）中使用“and”.分段名是全局的文件路径,格式类似于[gitignore](http://manpages.ubuntu.com/manpages/intrepid/man5/gitignore.5.html#contenttoc2)。斜杠(`/`)作为路径分隔符，`#`或者`;`作为注释。注释应该单独占一行。`EditorConfig`文件使用`UTF-8`格式、`CRLF`或`LF`作为换行符。

下面将详细解释文件路径匹配模式和EditorConfig 目前支持的配置属性。



#### Wildcard Patterns


| \*	          | **除路径分隔符`\`外匹配所有字符串字符** |
| ------------- |         :-------------           |
| \**           | **除路径分隔符`\`外匹配所有字符串字符** |
| ?             | **匹配所有单个字符**                 |
| [name]        | **匹配name字符**                    |
| [!name]       | **匹配非name字符**                  |
| {s1, s2, s3}  | **匹配任意给定的字符串（通过逗号分隔）** |

特殊字符需要通过转移符进行转义，使得特殊字符不会被当做通配符解析。




**EditorConfig支持的配置属性**

**注意**：并不是所有的编辑器插件都支持以下全部配置属性，[项目Wiki](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties)上有一份兼容配置属性的表格供参考。

- `indent_style`: set to tab or space to use hard tabs or soft tabs respectively.

  缩进样式：可以设置为`tab`和`space`两个值。

- `indent_size`: a whole number defining the number of columns used for each indentation level and the width of soft tabs (when supported). When set to tab, the value of **tab_width** (if specified) will be used.

  当缩进样式设置为`space`时，`indent_size`用来设置每次缩进相当于多少列。

- `tab_width`: a whole number defining the number of columns used to represent a tab character. This defaults to the value of **indent_size** and doesn't usually need to be specified.

  当缩进设置为`tab`时，`tab_width` 用来设置每次缩进相当于多少列代码。

- `end_of_line`: set to lf, cr, or crlf to control how line breaks are represented.

  `end_of_line`用来定义行末采用什么符号来进行换行，可以的选择包括`lf` `cr` 和`crlf`.

- `charset`: set to latin1, utf-8, utf-8-bom, utf-16beor utf-16le to control the character set. Use of utf-8-bom is discouraged.

  编码格式，支持`latin1`、`utf-8`、`utf-8-bom`、`utf-16be`和`utf-16le`，不建议使用`uft-8-bom`。

- `trim_trailing_whitespace`: set to true to remove any whitespace characters preceding newline characters and false to ensure it doesn't.

  设为`true`表示会除去代码行末的任意空白字符，`false`反之。

- `insert_final_newline`: set to true ensure file ends with a newline when saving and false to ensure it doesn't.

  设为`true`表明使文件以一个空白行结尾，`false`反之。

- `root`: special property that should be specified at the top of the file outside of any sections. Set to true to stop `.editorconfig` files search on current file.

  表明是最顶层的配置文件，发现设为`true`时，才会停止查找`.editorconfig`文件。



目前所有的属性名和属性值都是大小写不敏感的。编译时都会将其转为小写。通常，如果没有明确指定某个属性，则会使用编辑器的配置，而`EditorConfig`不会处理。

推荐不要指定某些`EditorConfig`属性。比如，`tab_width`不需要特别指定，除非它与`indent_size`不同。同样的，当`indent_style`设为`tab`时，不需要配置`indent_size`，这样才方便阅读者使用他们习惯的缩进格式。另外，如果某些属性并没有规范化（比如`end_of_line`），就最好不要设置它。

### 不需要插件的编辑器

1. BBEdit
2. C lion
3. GitHub
4. InfelliJIDEA
5. RubyMine
6. Source Lair
7. WebStorm

### 需要下载插件的编辑器

1. AppCode
2. ATOM
3. Brackets
4. Coda
5. Code::Blocks
6. eclipse
7. Emacs
8. Geany
9. gedit
10. jEdit
11. Komodo
12. NetBeans
13. Nodepad++
14. PhpStorm
15. PyCharm
16. Sublime Text
17. textmate
18. Vim
19. Visual Stuido
20. Visual Studio Code
21. Xcode

**举个例子**

**Sublime Text**

如果要安装Sublime Text插件，直接在Package Control中搜索`EditorConfig`就行。

**WebStorm**

在Setting–>Plugins–>Browse JetBrains Plugins里搜索`EditorConfig`即可。

`IntellijIDEA`、`PhpStorm`、`PyCharm`等应该类似。

*注：*从9.X版本开始，`WebStorm`原生就内置了`EditorConfig`，不需要装任何插件。

*译注：*支持的编辑器/IDE很多，使用也都很简单，大家自己去[官网](http://editorconfig.org/#download)下吧。
