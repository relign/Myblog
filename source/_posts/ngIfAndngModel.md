---
title: Angular小坑之ngIf和ngModel
date: 2016-12-12 22:11:05
tags: [Angular]
category: Angular
---
Angular项目需求开发中偶遇小坑,发现利用`ngModel`绑定值后,监测不到值的变化.
于是简单写了个demo复现了这种情况:

<iframe height='437' scrolling='no' title='WogMpK' src='//codepen.io/relign/embed/WogMpK/?height=437&theme-id=0&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/WogMpK/'>WogMpK</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

这种情况发生在`ng-if`中嵌套一个`ng-model`,`ng-model`就会无效.

AngularJS中`ng-show`,`ng-hide`,`ng-if`用来控制DOM元素的显示和隐藏,
不同的是`ng-show`和`ng-hide`是通过计算表达式的值来控制增加还是删除DOM元素的类名(`ng-hide`),详细见源码:
```javascript
// ngHide
var ngHideDirective = ['$animate', function($animate) {
  return function(scope, element, attr) {
    scope.$watch(attr.ngHide, function ngHideWatchAction(value){
      $animate[toBoolean(value) ? 'addClass' : 'removeClass'](element, 'ng-hide');
    });
  };
}];
// ngShow
var ngShowDirective = ['$animate', function($animate) {
  return function(scope, element, attr) {
    scope.$watch(attr.ngShow, function ngShowWatchAction(value){
      $animate[toBoolean(value) ? 'removeClass' : 'addClass'](element, 'ng-hide');
    });
  };
}];
```
`ng-hide`类名是控制CSS的`display`属性值.

<!-- more -->

然而,`ng-if`指令是根据表达式的值增加或者移除一个DOM元素:
```javascript
var ngIfDirective = ['$animate', function($animate) {
  return {
    transclude: 'element',
    priority: 600,
    terminal: true,
    restrict: 'A',
    $$tlb: true,
    link: function ($scope, $element, $attr, ctrl, $transclude) {
        var block, childScope, previousElements;
        $scope.$watch($attr.ngIf, function ngIfWatchAction(value) {

          if (toBoolean(value)) {
            if (!childScope) {
              childScope = $scope.$new();
              $transclude(childScope, function (clone) {
                clone[clone.length++] = document.createComment(' end ngIf: ' + $attr.ngIf + ' ');
                // Note: We only need the first/last node of the cloned nodes.
                // However, we need to keep the reference to the jqlite wrapper as it might be changed later
                // by a directive with templateUrl when its template arrives.
                block = {
                  clone: clone
                };
                $animate.enter(clone, $element.parent(), $element);
              });
            }
          } else {
            if(previousElements) {
              previousElements.remove();
              previousElements = null;
            }
            if(childScope) {
              childScope.$destroy();
              childScope = null;
            }
            if(block) {
              previousElements = getBlockElements(block.clone);
              $animate.leave(previousElements, function() {
                previousElements = null;
              });
              block = null;
            }
          }
        });
    }
  };
}];
```
当表达式为`false`的时候,`ng-if`指令会移除创建好的元素,**并且销毁与之相关的`childScope`**.
当表达式为`true`的时候,`ng-if`指令会重新创建DOM元素,**并且会从它的父作用域上继承一个新的子作用域**.
这就意味着`ng-if`会新建新的作用域.这一点我们可以从`JavaScript`原型来理解:
**对于从原型对象继承而来的成员,其读和写具有内在的不对等性.**
举个例子:
```javascript
function Student () {

}
Student.prototype.name = 'zhangsan';
Student.prototype.info = {'sex': 'man'};

var a = new Student();
var b = new Student();

a.name = 'lisi';
a.info.sex = 'woman';

console.log(a.name); // lisi
console.log(b.name); // zhangsan
console.log(a.info.sex); // woman
console.log(b.info.sex); // woman

```
在上述例子中可以发现,当改变`a`的`name`时,不会改变原型中的属性.当改变`a`中`info`对象的属性时,由于对象是引用的,所以也改变了引用对象的属性.

由此我们可以对这个小坑提出解决方法,可以在`ng-if`下`ng-model`赋值的属性,提供一个外部对象的引用:

<iframe height='427' scrolling='no' title='woEmQv' src='//codepen.io/relign/embed/woEmQv/?height=427&theme-id=0&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/woEmQv/'>woEmQv</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

上面还提到一种解决方式,通过`$parent`引用到外部作用域的属性,也可以解决这个小坑.不过当你`ng-if`嵌套多层时,这种方法就显然不合适了.
