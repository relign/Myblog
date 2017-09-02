---
title: 【总结】调研实现高性能动画
date: 2016-10-25 14:49:22
tags: [Animation, JavaScript]
category: Animation
---

> 本文是调研如何实现高性能动画,提升用户体验的总结,文章内容来源于对看过的相关技术文章的总结,相关技术文章已列到文章末尾,如有遗漏,敬请谅解.

快速响应和高度交互的页面往往能够吸引大量的用户群体.相反,如果页面存在性能低下的动画,动画不流畅,动画过程中页面闪烁等等,如此粗糙的交互体验必然丧失用户量.      

对于大多数的设备而言,屏幕以 60 次每秒的频率刷新,即`60HZ`.如果一个动画中的某些帧超过了这个时间,就会导致浏览器的刷新频率跟不上设备的刷新频率（跳帧现象）,出现页面闪烁.因此,高性能的动画都应该保持在`60fps`左右.    

接下来我们看几种动画的实现方式.

### 基于`setTimeout`或者`setInterval`实现的动画    

#### 基于帧算法实现的动画
<!-- more -->
<iframe height='471' scrolling='no' title='rWeGoX' src='//codepen.io/relign/embed/rWeGoX/?height=471&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/rWeGoX/'>rWeGoX</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

这是一个基于帧算法实现的JavaScript动画,这里设置的每秒钟更新60次,即`60fps`.大家可以看到现在的动画还是非常流畅的.动画的帧率也在60附近.    

但是由于JavaScript运行时需要耗费时间,而JavaScript又是单线程的,所以如果一个定时器如果比较耗时的话,是会阻塞下一个定时器的执行.所以即使你这里设置了`1000 / 60`每秒`60帧`的帧率,在不同的浏览器平台的差异也会导致实际上你的没有`60fps`的帧率.    

所以上面代码在一个手机上执行的时候可能有`60fps`的帧率,在另外一个手机上可能就只有`30fps`,更甚可能只有`10fps`.   

我们去模拟一下这几个帧率下的动画:

<iframe height='710' scrolling='no' title='xRVXyG' src='//codepen.io/relign/embed/xRVXyG/?height=710&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/xRVXyG/'>xRVXyG</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

很明显产生的交互效果是不符合预期的.导致这种情况的原因很简单,因为我们计算和绘制每个`div`位置的时候是在每帧更新,每帧移动`2px`.在`60fps`的情况下,我们1秒钟会执行`60帧`,所以小块每秒钟会移动`60 * 2 = 120px`;如果是`30fps`,小块每秒就移动`30 * 2 = 60px`,以此类推`10fps`就是每秒移动`20px`.三个小块在单位时间内移动的距离不一样.    

#### 基于时间算法实现的动画    

针对于这种情况,我们对其作出改进.我们不再以帧为基准来更新方块的位置,而是以时间为单位更新.也就是说,我们之前是`px/frame`,现在换成`px/ms`.    

<iframe height='602' scrolling='no' title='XNdVEE' src='//codepen.io/relign/embed/XNdVEE/?height=602&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/XNdVEE/'>XNdVEE</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>     

在这里,我们先确定一个固定更新的时间片,如固定为`60fps`时一帧的时间:`1000 / 60 = 0.167ms`.然后积累过去的时间,然后根据固定时间片分片进行更新.也就说,即使这一帧和上一帧相差过去了`100ms`,我也会把这100ms分成很多个`0.167ms`来执行`update`函数.这样做有两个好处:

* 固定的时间片足够小，更新的时候可以减少动画失帧
* 不同帧率,不管你是`60`,`30`,还是`10fps`,也是根据固定时间片来执行update函数,所以即使有损失,不同帧率之间的损失是一样的.那么我们三个方块就可以达到同步移动的效果的了!

#### 基于`setTimeout`或者`setInterval`实现动画存在的问题

使用`setTimeout`和`setInterval`来绘制动画,计算延时的精确度是不够的.    

延时的计算依靠的是浏览器的内置时钟,而时钟的精确度又取决于时钟更新的频率.不同版本的浏览器,这个频率是不一样的:IE8及其之前的IE版本更新间隔为15.6毫秒,最新版的Chrome与IE9+浏览器的更新频率都为4ms.而且如果你使用的是笔记本电脑,并且在使用电池而非电源的模式下,为了节省资源,浏览器会将更新频率切换至于系统时间相同,更新频率更低.    

而另外一个问题,使用`setTimeout`和`setInterval`,需要面临异步队列问题.因为异步关系,`setTimeout`和`setInterval`中回调函数并非立即执行.而是需要加入等待队列中.但问题是,如果在等待延迟触发的过程中,有新的同步脚本需要执行,那么同步脚本不会排在回调之后,而是立即执行.    


例如:

<iframe height='391' scrolling='no' title='jVqYXw' src='//codepen.io/relign/embed/jVqYXw/?height=391&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/jVqYXw/'>jVqYXw</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

很显然,这样的动画交互体验是不可控的.


### 基于`requestAnimationFrame`实现的动画  

针对`setTimeout`和`setInterval`实现动画存在的缺陷,`Mozilla`首先推出了`mozRequestAnimationFrame`,通过它告诉浏览器某些JavaScript代码将要执行动画,这样浏览器可以在运行某些代码后进行适当的优化.之后,`Chrome`和`IE10+`也都给出了自己的实现,`webkitRequestAnimationFrame`和`msRequestAnimationFrame`.后来随着`HTML5`新的API发布,`requestAnimationFrame`被正式推出.

官方定义:
> window.requestAnimationFrame()这个方法是用来在页面重绘之前,通知浏览器调用一个指定的函数,以满足开发者操作动画的需求.这个方法接受一个函数为参,该函数会在重绘前调用.

注意: 如果想得到连贯的逐帧动画,函数中必须重新调用 `requestAnimationFrame()`.

<iframe height='371' scrolling='no' title='qqZxqW' src='//codepen.io/relign/embed/qqZxqW/?height=371&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/relign/pen/qqZxqW/'>qqZxqW</a> by songruigang (<a href='http://codepen.io/relign'>@relign</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>  

`requestAnimationFrame`最大的好处在于可以可以避免浏览器不必要的重绘.想要理解这个好处,我们首先需要简单了解一下浏览器的渲染过程.

### 浏览器渲染过程    

要实现一个高性能的动画,首选我们必须对浏览器的渲染机制有所了解:

> 更加详细的渲染过程解读详见[浏览器的工作原理：新式网络浏览器幕后揭秘](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)

Chrome渲染过程:
![Chrome渲染过程](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/webkitflow.png)    


从图中可以看出,浏览在渲染页面过程中依次经历了:      
1.  HTML Parse(html解析)
2.  Calculate Style(计算样式)
3.  Layout(布局)
4.  Rasterizer(光栅化)
5.  Paint(绘制)
6.  Composite Layers(渲染层合并)


####  HTML Parser

发送`http`请求,获取请求内容,然后解析HTML的过程.更加详细的可以看这里[What happens when...](https://github.com/alex/what-happens-when)以及对应的翻译文档[当···时发生了什么?](https://github.com/skyline75489/what-happens-when-zh_CN)

#### Calculate Style

即计算样式.      

Calculate被触发的时候做的事情就是处理JavaScript给元素设置的样式.此时Recalculate Style会计算Render树(渲染树),然后从根节点开始进行页面渲染,将CSS附加到DOM上的过程.   

这个过程是根据CSS选择器,对每个DOM元素匹配对应的CSS样式.这一步结束之后,就确定了每个DOM元素上应该应用什么CSS样式规则.

**任何企图改变元素样式的操作都会触发Recalculate(重新计算样式)**.同Layout一样,它们都是JavaScript执行完后才触发的.

#### Layout    

计算页面上的布局,即元素在文档中的位置及大小.正如前面所述,Layout计算的是布局位置信息.

上一步确定了每个DOM元素的样式规则,这一步就是具体计算每个DOM元素最终在屏幕上显示的大小和位置.

需要注意的是:在页面解析完成后,任何有可能改变元素位置或大小的样式都会触发这个Layout事件.    

常见影响布局的CSS属性有:
* `width`
* `height`
* `padding`
* `margin`
* `display`
* `border-width`
* `border`
* `top`
* `position`
* `font-size`
* `float`
* `text-align`
* `overflow-y`
* `font-weight`
* `overflow`
* `left`
* `font-family`
* `line-height`
* `vertical-align`
* `right`
* `clear`
* `white-space`
* `bottom`
* `min-height`

等等,更多触发Layout事件的属性,可以在[CSS Triggers](https://csstriggers.com/)网站查阅.    

#### Rasterizer

光栅化,一般的安卓手机都会进行光栅化,光栅主要是针对图形的一个栅格化过程.低端手机在这部分耗时还是蛮多的.

#### Paint

本质上就是填充像素的过程.包括绘制文字、颜色、图像、边框和阴影等,也就是一个DOM元素所有的可视效果.一般来说,这个绘制过程是在多个层上完成的.

Paint的工作就是把文档中用户可见的那一部分展现给用户.Paint是把Layout和Calculate的计算的结果直接在浏览器视窗上绘制出来,它并不实现具体的元素计算.     


同样,页面解析完成后,改变某些样式也会引起RePaint(重绘).

常见引起RePaint(重绘)的样式:
* `color`
* `border-style`
* `visibility`
* `background`
* `text-decoration`
* `background-image`
* `background-position`
* `background-repeat`
* `outline-color`
* `outline`
* `outline-style`
* `border-radius`
* `outline-width`
* `box-shadow`
* `background-size`

如果你在元素中对以上的属性设置动画,那么将会引起重绘,并且元素所属的图层将提交给GPU进行处理.      
对于移动端设备来说,这代价是非常昂贵的,因为它们的CPU的处理能力明显弱于桌面端.这意味着,任务将用更长的时间来完成;并且CPU和GPU之间的带宽是有限的,所以数据的上传需要花费很长的一段时间.    

#### Composite Layers

最后合并图层,输出页面到屏幕.浏览器在渲染过程中会将一些含有特殊样式的DOM结构绘制于其他图层,有点类似于`PhotoShop`的图层概念.一张图片在`PotoShop`是由多个图层组合而成,而浏览器最终显示的页面实际也是有多个图层构成的.

在每个层上完成绘制过程之后,浏览器会将所有层按照合理的顺序合并成一个图层,然后在屏幕上呈现.对于有位置重叠的元素的页面,这个过程尤其重要,因为一旦图层的合并顺序出错,将会导致元素显示异常.

常见的导致新图层创建的因素有:
  * 进行3D或者透视变换的CSS属性
  * 使用硬件加速视频解码的`<video>`元素
  * 具有3D(WebGL)上下文或者硬件加速的2D上下文的`<canvas>`元素
  * 组合型插件(即Flash)
  * 具有有CSS透明度动画或者使用动画式Webkit变换的元素
  * 具有硬件加速的CSS滤镜的元素


### 影响动画渲染性能的因素

上述流程可以归纳为五个关键步骤:    

![浏览器渲染过程](http://cdn2.w3cplus.com/cdn/farfuture/usATdiAyFY--i9MqBmjcTjmP8DVlKngOBmJiJlPoNIs/mtime:1475068062/sites/default/files/blogs/2016/1609/css-animation-4.jpg)    

这也是我们在实现动画过程中有可能会触发的五个步骤,搞清楚我们实现动画的代码在哪一步,有助于我们实现高性能流畅的动画.
在上面的流程中,我们需要注意两个概念 **重排(也就是回流)** 和 **重绘**.这两个概念与上述流程中的Layout和Paint都有关系,而Layout和Paint又对动画渲染的性能至关重要.

#### 重排

`Reflow`(重排)指的是计算页面布局(Layout).某个节点`Reflow`时会重新计算节点的尺寸和位置,而且还有可能触发其后代节点`Reflow`.在这之后再次触发一次`Repaint`(重绘).

当Render Tree中的一部分(或全部)因为元素的尺寸、布局、隐藏等改变而需要重新构建.这就称为回流,每个页面至少需要一次回流,就是页面第一次加载的时候.   

在Web页面中,很多状况下会导致回流:

  * 调整窗口大小
  * 改变字体
  * 增加或者移除样式表
  * 内容变化
  * 激活CSS伪类
  * 操作CSS属性
  * JavaScript操作DOM
  * 计算`offsetWidth`和`offsetHeight`
  * 设置`style`属性的值
  * CSS3 Animation或Transition

#### 重绘

`Repaint`(重绘)或者`Redraw`遍历所有节点,检测节点的可见性、颜色、轮廓等可见的样式属性,然后根据检测的结果更新页面的响应部分.
当Render Tree中的一些元素需要更新属性,而这些属性只是影响元素的外观、风格、而不会影响布局的.就是重绘.

将重排和重绘的介绍结合起来,不难发现:**重绘(Repaint)不一定会引起回流(Reflow重排),但回流必将引起重绘(Repaint)**.

由此可见,重排和重绘很容易被触发,而他们对动画渲染的性能影响非常大.我们需要做的是尽量不去触发重绘和重排.

### 动画渲染性能优化

**过早进行性能优化是大忌**,如果我们实现的动画并没有性能方面的问题,就没有必要将时间成本浪费在性能优化上.

#### 在`Composite`这步优化动画

在实现用户交互动画的过程中,我们尽量避免重绘和重排.现在浏览器可以利用`transform`和`opacity`绘制很好的动画.因为这些属性只会影响
浏览器渲染的最后一步`Composite`过程.

共有四个让动画更好的属性:
* 位置(Position): `transform: translateX(n) translateY(n) translateZ(n)`
* 缩放(Scale): `transform: scale(n)`
* 旋转(Rotation): `transform: rotate(ndeg)`
* 透明度(Opacity): `opacity: n`

#### 在GPU上运行动画

在CSS中提供了一个新的CSS特性:`will-change`.其主要作用就是 **提前告诉浏览器我这里将会进行一些变动,请分配资源(告诉浏览器要分配资源给我).** 因此浏览器不需要考虑容器布局的渲染或绘制.

> `will-change`属性,允许作者提前告知浏览器的默认样式,那他们可能会做出一个元素.它允许对浏览器默认样式的优化如何提前处理因素,在动画实际开始之前,为准备动画执行潜在昂贵的工作.有关于`will-change`更详细的介绍可以[点击这里](http://www.w3cplus.com/css3/introduction-css-will-change-property.html).

在使用`will-change`一定要注意方式方法,比如常见的错误方法是直接在`:hover`是使用,并没有告诉浏览器分配资源:
```css
.element:hover {
    will-change: transform;
    transition: transform 2s;
    transform: rotate(30deg) scale(1.5);
}
```
其正确使用的方法是,在进入父元素的时候就告诉浏览器,你该分配一定的资源:
```css
.element {
    transition: opacity .3s linear;
}
/* declare changes on the element when the mouse enters / hovers its ancestor */
.ancestor:hover .element {
    will-change: opacity;
}
/* apply change when element is hovered */
.element:hover {
    opacity: .5;
}
```
另外在应用变化之后,取消`will-change`的资源分配:
```javascript
var el = document.getElementById('demo');
el.addEventListener('animationEnd', removeHint);

function removeHint() {
    this.style.willChange = 'auto';
}
```
在使用`will-change`时,还需注意:
* 不要将`will-change`应用到太多元素上:浏览器已经尽力尝试去优化一切可以优化的东西了.有一些更强力的优化,如果与`will-change`结合在一起的话,有可能会消耗很多机器资源,如果过度使用的话,可能导致页面响应缓慢或者消耗非常多的资源.
* 有节制地使用:通常,当元素恢复到初始状态时,浏览器会丢弃掉之前做的优化工作.但是如果直接在样式表中显式声明了`will-change`属性,则表示目标元素可能会经常变化,浏览器会将优化工作保存得比之前更久.所以最佳实践是当元素变化之前和之后通过脚本来切换`will-change`的值.
* 不要过早应用`will-change`优化:如果你的页面在性能方面没什么问题,则不要添加`will-change`属性来榨取一丁点的速度.`will-change`的设计初衷是作为最后的优化手段,用来尝试解决现有的性能问题.它不应该被用来预防性能问题.过度使用`will-change`会导致大量的内存占用,并会导致更复杂的渲染过程,因为浏览器会试图准备可能存在的变化过程.这会导致更严重的性能问题.
* 给它足够的工作时间:这个属性是用来让页面开发者告知浏览器哪些属性可能会变化的.然后浏览器可以选择在变化发生前提前去做一些优化工作.所以给浏览器一点时间去真正做这些优化工作是非常重要的.使用时需要尝试去找到一些方法提前一定时间获知元素可能发生的变化,然后为它加上 `will-change`属性.

### 参考资料
* [Javascript高性能动画与页面渲染](http://www.infoq.com/cn/articles/javascript-high-performance-animation-and-page-rendering/)
* [使用CSS3实现60FPS动画](http://www.w3cplus.com/animation/how-to-achieve-60-fps-animations-with-css3.html)
* [High Performance Animations](https://www.html5rocks.com/zh/tutorials/speed/high-performance-animations/)
* [使用 FLIP 来提高 Web 动画的性能](http://bubkoo.com/2016/03/31/high-performance-animations/)
* [浏览器的工作原理：新式网络浏览器幕后揭秘](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)
* [CSS Animation性能优化](http://www.w3cplus.com/animation/animation-performance.html)
