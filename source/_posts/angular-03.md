---
title: Angular源码解读之minErr
date: 2016-09-26 22:48:41
tags: [Angular]
category: Angular
---

 > 错误异常是面向对象开发中的记录提示程序执行问题的一种重要机制，在程序执行发生问题的条件下，异常会在中断程序执行，同时会沿着代码的执行路径一步一步的向上抛出异常，最终会由顶层抛出异常信息。而与异常同时出现的往往是日志，而日志往往需要记录具体发生异常的模块、编码、详细的错误信息、执行堆栈等，方便问题的快速定位分析。      
  ___摘自无风听海的博客[angular代码分析之异常日志设计](http://www.cnblogs.com/wufengtinghai/p/5428753.html)___

 打开Angular源码首先看到的就是这个`minErr`函数,这个函数是angular对异常信息的处理函数。

```javascript
/**
 * @description
 *
 * This object provides a utility for producing rich Error messages within
 * Angular. It can be called as follows:
 *
 * var exampleMinErr = minErr('example');
 * throw exampleMinErr('one', 'This {0} is {1}', foo, bar);
 *
 * The above creates an instance of minErr in the example namespace. The
 * resulting error will have a namespaced error code of example.one.  The
 * resulting error will replace {0} with the value of foo, and {1} with the
 * value of bar. The object is not restricted in the number of arguments it can
 * take.
 *
 * If fewer arguments are specified than necessary for interpolation, the extra
 * interpolation markers will be preserved in the final string.
 *
 * Since data will be parsed statically during a build step, some restrictions
 * are applied with respect to how minErr instances are created and called.
 * Instances should have names of the form namespaceMinErr for a minErr created
 * using minErr('namespace') . Error codes, namespaces and template strings
 * should all be static strings, not variables or general expressions.
 *
 * @param {string} module The namespace to use for the new minErr instance.
 * @returns {function(code:string, template:string, ...templateArgs): Error} minErr instance
 */

function minErr(module) {
  return function () {
    var code = arguments[0],
      prefix = '[' + (module ? module + ':' : '') + code + '] ', //构造日志前缀
      template = arguments[1],
      templateArgs = arguments,
      stringify = function (obj) {
        if (typeof obj === 'function') {
          return obj.toString().replace(/ \{[\s\S]*$/, '');
        } else if (typeof obj === 'undefined') {
          return 'undefined';
        } else if (typeof obj !== 'string') {
          return JSON.stringify(obj);
        }
        return obj;
      },
      message, i;
	//模板信息替换
    message = prefix + template.replace(/\{\d+\}/g, function (match) {
    //获取占位符索引,转化为数字
      var index = +match.slice(1, -1), arg;
	//获取替换占位符的传入参数对象的字符串表示形式
      if (index + 2 < templateArgs.length) {
        arg = templateArgs[index + 2];
        // 将传入的对象转化为便于调试的字符串形式
        if (typeof arg === 'function') {
          return arg.toString().replace(/ ?\{[\s\S]*$/, '');// 如果是函数则清空函数体
        } else if (typeof arg === 'undefined') {
          return 'undefined';
        } else if (typeof arg !== 'string') {
          return toJson(arg);
        }
        return arg;
      }
      return match;
    });

    message = message + '\nhttp://errors.angularjs.org/1.2.26/' +
      (module ? module + '/' : '') + code;
    for (i = 2; i < arguments.length; i++) {
      message = message + (i == 2 ? '?' : '&') + 'p' + (i-2) + '=' +
        encodeURIComponent(stringify(arguments[i]));
    }

    return new Error(message);
  };
}
```

<!-- more -->

结合注释给出的例子来看:

```javascript
var foo = '1234', bar = '3456';
var exampleMinErr = minErr('example');
throw exampleMinErr('one', 'This {0} is {1}', foo, bar);

```
首先angular通过`minErr('example')` 传入`example`(module参数,类似的还有:`minErr(ng)`,`minErr('$injector')`等等),并在返回的闭包函数中引用module.接着传入四个参数,初始化一些变量:

```javascript
	var code = arguments[0], // 'one' 日志编码
      prefix = '[' + (module ? module + ':' : '') + code + '] ', // [example:one]   构造日志前缀
      template = arguments[1], // 'This {0} is {1}'  日志模板
      templateArgs = arguments, // ["one", "This {0} is {1}", "1234", "3456"] 传入参数
      stringify = function (obj) {
        if (typeof obj === 'function') {
          return obj.toString().replace(/ \{[\s\S]*$/, '');
        } else if (typeof obj === 'undefined') {
          return 'undefined';
        } else if (typeof obj !== 'string') {
          return JSON.stringify(obj);
        }
        return obj;
      },// 定义一个方法 先留着 不解释
      message, i;

```
