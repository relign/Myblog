---
title: Angular启动过程解读（一）
desc: 'Lorem ipsum dolor sit amet, consectetur.'
date: 2016-09-21 23:31:28
tags: Angular
category: Angular
---

### 找到代码代入口点    

  ```javascript
  if (window.angular.bootstrap) {
    //AngularJS is already loaded, so we can return here...
    console.log('WARNING: Tried to load angular more than once.');
    return;
  }
  //首先判断angular是否是多次加载,如果多次加载,输出警告信息;


  //try to bind to jquery now so that one can write angular.element().read()
  //but we will rebind on bootstrap again.
  bindJQuery();//该方法会去绑定jquery,内部会判断jQuery有没有加载,有就是用jQuery,没有则调用自带的jqLite

  publishExternalAPI(angular);//暴露angular对象,加载angular的一系列api方法

  jqLite(document).ready(function() {
    angularInit(document, bootstrap);//等待dom加载完后启动angular.
  });
  ```
  <!-- more -->

### bindJQuery()    

  ```javascript
  function bindJQuery() {
    // bind to jQuery if present;
    jQuery = window.jQuery;
    // Use jQuery if it exists with proper functionality, otherwise default to us.
    // Angular 1.2+ requires jQuery 1.7.1+ for on()/off() support.
    if (jQuery && jQuery.fn.on) {
      jqLite = jQuery;
      extend(jQuery.fn, {
        scope: JQLitePrototype.scope,
        isolateScope: JQLitePrototype.isolateScope,
        controller: JQLitePrototype.controller,
        injector: JQLitePrototype.injector,
        inheritedData: JQLitePrototype.inheritedData
      });
      // Method signature:
      //     jqLitePatchJQueryRemove(name, dispatchThis, filterElems, getterIfNoArguments)
      jqLitePatchJQueryRemove('remove', true, true, false);
      jqLitePatchJQueryRemove('empty', false, false, false);
      jqLitePatchJQueryRemove('html', false, false, true);
    } else {
      jqLite = JQLite;
    }
    angular.element = jqLite;
  }
  ```
这个方法是用来绑定jQuery的,如果发现window已经绑定了jQuery,就将jQueryt通过extend方法绑定到jqLite上,同时扩展了scope,isolateScope,controller,injector,inheritedData.否则,就使用angular自带的jQLite,最终都赋给angular.element.这也就是我们可以通过angular.element去调用jQuery的方法.    
下面看一下这个extend方法:    

```javascript
/**
 * @ngdoc function
 * @name angular.extend
 * @module ng
 * @kind function
 *
 * @description
 * Extends the destination object `dst` by copying own enumerable properties from the `src` object(s)
 * to `dst`. You can specify multiple `src` objects.
 * 将一个或多个 src 的可枚举属性复制到dst对象中
 *
 * @param {Object} dst Destination object.
 * @param {...Object} src Source object(s).
 * @returns {Object} Reference to `dst`.
 */
function extend(dst) {
  var h = dst.$$hashKey;
  forEach(arguments, function(obj) {
    if (obj !== dst) { //去除同类的继承
      forEach(obj, function(value, key) {
        dst[key] = value;
      });
    }
  });

  setHashKey(dst,h);
  return dst;
}

/**
 * Set or clear the hashkey for an object.
 * @param obj object
 * @param h the hashkey (!truthy to delete the hashkey)
 */
function setHashKey(obj, h) {
  if (h) {
    obj.$$hashKey = h;
  }
  else {
    delete obj.$$hashKey;
  }
}

```
关于此处使用hashKey可以看看angularJs git的[提交记录](https://gitcandy.com/Repository/Commit/angular.js/016e1e675e717ab851759eac5be640bd2f238331):    

```
fix(angular): do not copy $$hashKey in copy/extend functions.
//修复angular在copy/extend函数中不能复制 $$hashKey
Copying the $$hashKey as part of copy/extend operations makes little
sense since hashkey is used primarily as an object id, especially in
the context of the ngRepeat directive. This change maintains the
existing $$hashKey of an object that is being copied into (likewise for
extend).
It is not uncommon to take an item in a collection, copy it,
and then append it to the collection. By copying the $$hashKey, this
leads to duplicate object errors with the current ngRepeat.

Closes #1875
```
在绑定jQuery的时候,通过`jqLitePatchJQueryRemove`方法将jQuery原生的`remove,empty,html`多做一层处理,处理完后再调用jQuery原生的方法.

```javascript
	/////////////////////////////////////////////
// jQuery mutation patch
//
// In conjunction with bindJQuery intercepts all jQuery's DOM destruction apis and fires a
// $destroy event on all DOM nodes being removed.
//
/////////////////////////////////////////////

function jqLitePatchJQueryRemove(name, dispatchThis, filterElems, getterIfNoArguments) {
  var originalJqFn = jQuery.fn[name]; //获取jQuery原生的方法
  originalJqFn = originalJqFn.$original || originalJqFn;//未知 $original 有什么用
  removePatch.$original = originalJqFn; //未知 绑上去有什么用
  jQuery.fn[name] = removePatch; //重写jQuery的remove,html,empty方法

  function removePatch(param) {
    // jshint -W040

    var list = filterElems && param ? [this.filter(param)] : [this],//this指向jQuery选择对象,当传入的filterElems为true并且jQuery原生方法传入参数不为空的时候,list才去取this.filter(param).
        fireEvent = dispatchThis,
        set, setIndex, setLength,
        element, childIndex, childLength, children;
    if (!getterIfNoArguments || param != null) {//这个判断remove,empty都会走这里,当html('')传入参数不为空也会走这里.
      while(list.length) {
        set = list.shift();
        for(setIndex = 0, setLength = set.length; setIndex < setLength; setIndex++) {
          element = jqLite(set[setIndex]);
          if (fireEvent) {
            element.triggerHandler('$destroy');
          } else {
            fireEvent = !fireEvent;
          }
          for(childIndex = 0, childLength = (children = element.children()).length;
              childIndex < childLength;
              childIndex++) {
            list.push(jQuery(children[childIndex]));
          }
        }
      }
    }
    return originalJqFn.apply(this, arguments);
  }
}
```
需要注意的是这一个三元表达式:
```javascript
var list = filterElems && param ? [this.filter(param)] : [this]
```
当传入的filterElems为true并且jQuery原生方法传入参数不为空的时候,list才去取this.filter(param).这种情况会发生在:
`angular.elment('div').remove('.test')`这种情况下,因为list是始终要得到你选择的那个DOM元素,所以进行这样一个三元判断.    
>这里备注一个从[参考资料](http://www.sparrowjang.com/2014/01/11/jquery-detach-on-angular-directive/)上了解的内容:
在这里`element.triggerHandler('$destroy')`会告知angular底下的`transcludedScope `,这个element已经被摧毁了,所以会导致`$element.detach`的时候,`transcludedScope `无法被更新.`element.triggerHanler('$destroy')`的监听event可以参考`createBoundTranscludeFn`这个function,会有下面这段code去监听:    
>
```javascript
clone.on('$destroy', bind(transcludedScope, transcludedScope.$destroy));
```
>在`jqLitePatchJQueryRemove`只替换`remove`、`empty`、`html`,而没有替换`detach`,使用`detach`却会影响`scope`,真正原因要找到jquery的source code.
因为detach间接呼叫了remove:    
>
```javasript
detach: function( selector ) {
	return this.remove( selector, true );
}
```
>Example(1.2.7):
>    
>* [directive在ready中呼叫$element.deatch](http://jsbin.com/olISoGu/1/edit?html,js,output):此时`transclude`里面的scope将会失效.    
>
>* [directive在ready之前呼叫$element.deatch](http://jsbin.com/olISoGu/3/edit?html,js,output):`tansclude`尚未被建立,自然而然就不会处理`$destroy`的event,`transclude`里的scope仍然会work.     
>
由此可知,只要scope尚未ready时,呼叫`detach`就不会中断scope的更新.若要让`directive`的scope正常运作,可用以下方式:    
>
>* 	[不使用transclude](http://jsbin.com/olISoGu/4/edit?html,js,output)
>* [不在scope ready中,使用deatch](http://jsbin.com/olISoGu/3/edit?html,js,output)
>* [使用原生api](http://jsbin.com/olISoGu/5/edit?html,js,output)
>* 最后一种方式,jquery在angular之后include    

> 如果不是这么需要使用到jquery，建议就不要include了，当然jquery也提供很多方便的功能，依照project需求自己评估吧！

### publishExternalAPI()
