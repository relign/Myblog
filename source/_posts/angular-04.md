---
title: Angular源码解读之不同版本的Extend
date: 2016-09-27 21:31:53
tags: [Angular,extend]
category: Angular
---
>此文章比较各个版本之间Angular的Extend方法

### 版本1.2.26
此时的Angular Extend精简至极,只进行了浅拷贝.

```javascript
function extend(dst) {
  var h = dst.$$hashKey;
  forEach(arguments, function(obj) {
    if (obj !== dst) { // 做判断去除同类的继承
      forEach(obj, function(value, key) {
        dst[key] = value; // 对传入参数进行循环,将一个或者多个的可枚举属性复制到dst对象中
      });
    }
  });

  setHashKey(dst,h);// 为对象增加hash值,方便后续遍历过滤处理操作
  return dst;
}
```

上面的代码多的仅仅是将传入的参数中可枚举的属性复制到dst对象,并返回这个新的对象,传入参数中靠后的argument会覆盖调靠前的.
列举一个简单的例子:

<!--more-->

```javascript
var consData = {
    name: 'zhangsan',
    age: '12',
    say: {
        sex: 'man'
    },
    cc: ['1','2','3']
};
var bbData = {
    name: 'lisi',
    nam: '123',
    say: {
        sex: 'woman',
        mm: 'wang'
    },
    cc: ['a','b','c']
};
console.log(angular.extend(consData, bbData));
```
最终输出结果是

```json
Object{
	"age": "12",
	"cc": ["a","b","c"],
	"nam": "123",
	"name": "lisi",
	"say": {
		"mm": "wang",
		"sex": "woman"
	}
}

```
由此可见`extend`依次会将传入参数的第一层属性赋值给第一个参数的第一层属性,如果属性相同,则后面的值会覆盖前面的,如果是对象或者数组,则会引用同一个,靠后的优先级高,并返回第一个参数对象.

### 版本1.3.9

```javascript
function extend(dst) {
  var h = dst.$$hashKey;

  for (var i = 1, ii = arguments.length; i < ii; i++) {
    var obj = arguments[i];
    if (obj) { // 这里不再做同类的判断
      var keys = Object.keys(obj);
      for (var j = 0, jj = keys.length; j < jj; j++) {
        var key = keys[j];
        dst[key] = obj[key];
      }
    }
  }

  setHashKey(dst, h);
  return dst;
}
```

这个版本的`extend`脱离了Angular自己定义的`forEach`方法,减少了代码的耦合性.     

#### `Object.keys()`    

由于Angular在1.2.4版本后就放弃了对IE8的支持,所以这里用了`Object.keys()`去获取传入对象的可枚举属性.
关于`Object.keys()`看下面这个例子:

```javascript
var consData = {
    name: 'zhangsan',
    age: '12',
    say: {
        sex: 'man'
    }
};
console.log(Object.keys(consData));

```
输出结果是`['name','age','say']`,由此也可以看出`angular.extend()`并不支持深拷贝.
关于`Object.keys()`你还要需要注意:

```javascript
Object.keys("foo");
// TypeError: "foo" is not an object (ES5 code)

Object.keys("foo");
// ["0", "1", "2"]                   (ES6 code)
```

### 版本1.4.0

`Angular.extend()` 方法发生重大变化:

```javascript
function extend(dst) {
  return baseExtend(dst, slice.call(arguments, 1), false);
}
```

`extend`方法返回一个闭包函数`baseExtend`,在这里你可能会对`slice.call(arguments, 1)`有疑惑.这里首先在Angular源码244行附近你会找到`var slice = [].slice;` 那么在`baseExtend`方法中传入的第二个参数实际是`[].slice.call(argumens, 1)`,其实也就是`Array.prototype.slice.call()`.我们只道`arguments`实际是一个带有`length`属性的对象,而`Array.prototype.slice.call()`这个方法可以具有length属性的对象转成数组，除了IE下的节点集合(因为ie下的dom对象是以com对象的形式实现的，js对象与com对象不能进行转换);

```javascript
var a = {0:'first',1:'second',2:'third'};
console.log([].slice.call(a)); // []
var b = {0:'first',1:'second',2:'third',length:3};
console.log([].slice.call(b)); // ['first','second','third']
```
> 关于`Array.prototype.slice.call()`实现过程的猜测见肥杜的Blog [Array.prototype.slice.call(arguments)](http://www.cnblogs.com/littledu/archive/2012/05/19/2508672.html)

下面我们看看`baseExtend`干了什么:

```javascript
/**
 * Set or clear the hashkey for an object.
 * @param obj object
 * @param h the hashkey (!truthy to delete the hashkey)
 */
function setHashKey(obj, h) {
  if (h) {
    obj.$$hashKey = h;
  } else {
    delete obj.$$hashKey;
  }
}


function baseExtend(dst, objs, deep) {
  var h = dst.$$hashKey;

  for (var i = 0, ii = objs.length; i < ii; ++i) {
    var obj = objs[i];
    if (!isObject(obj) && !isFunction(obj)) continue;
    var keys = Object.keys(obj);
    for (var j = 0, jj = keys.length; j < jj; j++) {
      var key = keys[j];
      var src = obj[key];

      if (deep && isObject(src)) {
        if (!isObject(dst[key])) dst[key] = isArray(src) ? [] : {};
        baseExtend(dst[key], [src], true);
      } else {
        dst[key] = src;
      }
    }
  }

  setHashKey(dst, h);
  return dst;
}
```
