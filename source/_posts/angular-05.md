---
title: 关于ng-table v0.3.3的那些事 (一)
date: 2016-09-30 18:15:03
tags: [Angular,ng-table]
category: Angular
---

>入职一周左右发现项目中使用`ngTable`版本是v0.3.3,而github上`ngTable`版本已经更新到v1.0.0,找到一篇v0.3.3的英文api进行翻译.同时抽出时间看看官方给出的例子总结了一下v0.3.3,希望可以帮到后续入职的新同学快速上手,老司机请绕路哈.

### ngTable指令Config简介

`ngTable`指令可以美化你的`table`.它支持表格数据排序,过滤筛选和分页.当表格初始化完成后会自动带有头部和过滤器.    
假如你想让`ngTable`在IE9以下运行,你需要`jQuery`:

```html
<!--[if lt IE 9]>
  <script src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
<![endif]-->
```

`ngTable`指令通过在`table`标签上挂在`ng-table`属性来启动.而`ng-table`属性的值对应你挂载到`scope`上的一个`ngTableParams`实例化以后的对象.举个例子:    

HTML:

```html
<table ng-table="tableParams">
	...
</table>
```
<!-- more -->

Controller:    

```javascript
$scope.tableParams = new ngTableParams(parameters, settings)
```
当对`ngTableParams`进行实例化的时候,这个函数会接收两个参数`parameters`和`settings`.下面我们详细介绍一下这两个参数.
### parameters

`parameters`对象应该定义一些初始化的配置.更多复杂的配置应该在`setting`对象中进行.下面这些属性可以被设置到`parameters`对象中.

#### `page`
类型: `Number`,默认值:`1`    
表格展示第几页    
#### `count`
类型: `Number`,默认值:`1`    
表格每一页展示多少数据
#### `filter`
类型: `Object`,默认值:`{}`    
定义表格列的过滤器.不区分大小写.举个例子,绑定一个下拉框作为表格的过滤器:    
in view:

```html
<select name="status" ng-model="filters.status" ...>
```
in controller:

```javascript
$scope.filters = {
  status: ''
};
```
in ngTableParams:    

```javascript
$scope.tableParams = new ngTableParams(
  {
    ...
    // Attach filters to ng-table.
    filter: $scope.filters
  },
  {
    ...
  }
);
```
#### `sorting`    
类型: `Object`,默认值:`{}`   
定义表格的列排序.举个例子,首先给出数据:    

```javascript
var data = [
	{"name": "Moroni", "age": 50},
	{"name": "Tiancum", "age": 43}
];    
```

你可以通过设置`sorting`定义一个表格名字的升序排列:

```javascript
{
	"name": "asc"
}
```
### settings    
#### `total`    
类型: `Number`,默认值: `0`    
定义表格数据的总数,通常为你需要渲染的Array数据的length.    
#### `counts`
类型: `Array`,默认值: `[10,25,50,100]`    
定义表格每页显示数据数量的选项卡,可以通过设置一个空的数组来禁用这个功能.    
#### `defaultSort`
类型: `String`, 默认值: `desc`, 可选项:`["asc","desc"]`    
定义一个默认的排列顺序,当你点击一个之前没有被排序的列数据时,按照默认的排列顺序进行排序.     
#### `groupBy`    
类型: `String | Function`     
定义表格数据分组.你可以设置`groupBy`对数据进行分组:

```javascript
var data = [
  {name: "Moroni", age: 50},
  {name: "Tiancum", age: 43}
];
...
groupBy: 'name'
```
或者通过传入回调函数设置你的分组规则:

```javascript
// Group by first letter of name + age
// The item parameter is provided internally by ngTable
groupBy: function (item) {
  return item.name[0] + item.age;
}
```
> 关于`groupBy`的说明并不太懂,之后会上手几个demo看看

#### `filterDelay`    
类型: `Number`, 默认值: `750`    
定义一个表格过滤器参数变化时,数据被显示在表格中的延迟时间.例如,你先想要按键过滤器的延迟效果.

#### `data`    
类型: `Array`, 默认值: `[]`    
最初的数据集,由 `{colname:value}`键值对组成的`object`对象.

#### `getData`    

类型: `Function`    
定义一个方法处理要展示在`ngTable`中的数据.`ngTable`可以通过`getData`传入的两个参数`$defer`和`params`进行回调.举个例子:

```javascript
getData: function ($defer, params) {
  ...
}
```
`$defer` 是一个`Promise`对象.通过调用`$defer`的`resolve`方法传递`data`数据到`ngTable`.    
举个例子,给出如下的`data`:

```javascript
var data = [
  {name: "Moroni", age: 50},
  {name: "Tiancum", age: 43}
];
```
通过这个最基础的方法,你可以让`ngTable`获得`data`数据:

```javascript
getData: function ($defer, params) {
  $defer.resolve(data);
}
```
`params`对应的是你在`parameters`对象中的配置(否则你再`parameters`中也没有设置,就会使用`ngTable`的默认配置);     
举个例子,检索现在`parameters`的`filter`和`sorting`的配置:

```javascript
getData: function ($defer, params) {
  ...
  var filter = params.filter();
  var sorting = params.sorting();
  ...
}
```
设置`paramters`中的`total`,你可以这样引用变量`vm.tableParams.total(value)`.举个例子,在`ngTable`中使用`ajax`请求数据:

```javascript
this.tableParams = new ngTableParams(
  {page: 1, count: 10},
  {
    total: 0,
    getData: function ($defer, params) {
      var filter = params.filter();
      var sorting = params.sorting();
      var count = params.count();
      var page = params.page();
      myService.query(page, count, filter, sorting).success(function (result) {
        vm.tableParams.total(result.total);
        $defer.resolve(result.data);
      });
    }
  }
);
```
