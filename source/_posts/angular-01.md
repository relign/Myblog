---
title: angular源码工具函数总结
desc: 'Lorem ipsum dolor sit amet, consectetur.'
date: 2016-09-21 23:16:01
tags: angular
---
在了解angular启动过程之前,首先了解几个必要的angular的工具函数:
- isFunction    
  ```javascript
  /**
   * @ngdoc function
   * @name angular.isFunction
   * @module ng
   * @kind function
   *
   * @description
   * Determines if a reference is a `Function`.
   *
   * @param {*} value Reference to check.
   * @returns {boolean} True if `value` is a `Function`.
   */
  function isFunction(value){return typeof value === 'function';}
  ```

  不多解释，判断是否是函数类型。

  <!-- more -->

- isArray

  ```javascript
  /**
   * @ngdoc function
   * @name angular.isArray
   * @module ng
   * @kind function
   *
   * @description
   * Determines if a reference is an `Array`.
   *
   * @param {*} value Reference to check.
   * @returns {boolean} True if `value` is an `Array`.
   */
  var isArray = (function() {
    if (!isFunction(Array.isArray)) {
      return function(value) {
        return toString.call(value) === '[object Array]';
      };
    }
    return Array.isArray;
  })();

  ```
  判断是否是数组类型，代码关键在于`toString.call(value)`返回的值.
  这里需要注意`Object.prototype.toString.call(this)`和`Object.prototype.toString(this)`的区别。举个例子：    

  ```javascript
  function foo(){
    console.log(this);      
    console.log(Object.prototype.toString.call(this));
    console.log(Object.prototype.toString(this));
  }
  foo.call("hello");
  //输出结果：
  //String { 0="h",  1="e",  2="l", 3="l" ,4="o"}
  //[object String]
  //[object Object]
  ```

  由于在使用call/apply时会进行基本类型到包装类型的装换(一般在非严格模式下),所以当用`foo.call('hello')`以后,this指向的实际是`new String('hello')`一个String对象.     
  而通过`Object.prototype.toString.apply/call`可以间接拿到对象的内部`[[class]]`标签,对于String对象会返回`[object String]`.
  直接调用`Object.prototype.toString`返回`[object Object]`是因为this指针指向的是`Object.prototype`.    
  那么下面这段代码输出什么就显而易见了:

  ```javascript
  var aa = 123;
  console.log(aa.toString());
  console.log(toString.call(aa));
  ```

- forEach

  ```javascript   
    /**
     * @ngdoc function
     * @name angular.forEach
     * @module ng
     * @kind function
     *
     * @description
     * Invokes the `iterator` function once for each item in `obj` collection, which can be either an
     * object or an array. The `iterator` function is invoked with `iterator(value, key)`, where `value`
     * is the value of an object property or an array element and `key` is the object property key or
     * array element index. Specifying a `context` for the function is optional.
     *
     * It is worth noting that `.forEach` does not iterate over inherited properties because it filters
     * using the `hasOwnProperty` method.
     *
       js
         var values = {name: 'misko', gender: 'male'};
         var log = [];
         angular.forEach(values, function(value, key) {
           this.push(key + ': ' + value);
         }, log);
         expect(log).toEqual(['name: misko', 'gender: male']);

     *
     * @param {Object|Array} obj Object to iterate over.
     * @param {Function} iterator Iterator function.
     * @param {Object=} context Object to become context (`this`) for the iterator function.
     * @returns {Object|Array} Reference to `obj`.
     */
    function forEach(obj, iterator, context) {
      var key;
      if (obj) {
        if (isFunction(obj)) {
          for (key in obj) {
            // Need to check if hasOwnProperty exists,
            // as on IE8 the result of querySelectorAll is an object without a hasOwnProperty function
            if (key != 'prototype' && key != 'length' && key != 'name' && (!obj.hasOwnProperty || obj.hasOwnProperty(key))) {
              iterator.call(context, obj[key], key);
            }
          }
        } else if (isArray(obj) || isArrayLike(obj)) {
          for (key = 0; key < obj.length; key++) {
            iterator.call(context, obj[key], key);
          }
        } else if (obj.forEach && obj.forEach !== forEach) {
            obj.forEach(iterator, context);
        } else {
          for (key in obj) {
            if (obj.hasOwnProperty(key)) {
              iterator.call(context, obj[key], key);
            }
          }
        }
      }
      return obj;
    }

    function sortedKeys(obj) {
      var keys = [];
      for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
          keys.push(key);
        }
      }
      return keys.sort();
    }

    function forEachSorted(obj, iterator, context) {
      var keys = sortedKeys(obj);
      for ( var i = 0; i < keys.length; i++) {
        iterator.call(context, obj[keys[i]], keys[i]);
      }
      return keys;
    }
  ```
  在读forEach函数的时候,有个地方是我所不理解的:    

  ```javascript
    if (isFunction(obj)) {
      for (key in obj) {
        // Need to check if hasOwnProperty exists,
        // as on IE8 the result of querySelectorAll is an object without a hasOwnProperty function
        if (key != 'prototype' && key != 'length' && key != 'name' && (!obj.hasOwnProperty || obj.hasOwnProperty(key))) {
          iterator.call(context, obj[key], key);
        }
      }
    }
  ```
  forEach函数首先会传进来的obj做一次判断,是否为函数,接着它居然对函数进行`for (key in obj)`的操作,注意这时的key是undefined.后来在百度大神黄子毅的解释下,写了一个demo:

  ```javascript
   function sum() {

   }
   Function.prototype.ss = '123';
   for(var key in sum){
     console.log(key);
   }
   //输出: ss
   //es6
   class a {};
   for (let key in a){
     console.log(key);
   }
   Function.prototype.xxx = 1;
   for (let key in a){
     console.log(key);//xxx
   }
  ```
  有木有觉得很神奇?类,class的typeof本质是function,当你定义```Function.prototype```了以后,对函数进行```for in```操作时,它的```prototype```居然可以被得到.
