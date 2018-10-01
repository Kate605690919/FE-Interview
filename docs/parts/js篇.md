## js动画和css3动画

**JS动画**

缺点：(1)JavaScript在浏览器的主线程中运行，而主线程中还有其它需要运行的JavaScript脚本、样式计算、布局、绘制任务等,对其干扰导致线程可能出现阻塞，从而造成丢帧的情况。

​        (2)代码的复杂度高于CSS动画

优点：(1)JavaScript动画控制能力很强, 可以在动画播放过程中对动画进行控制：开始、暂停、回放、终止、取消都是可以做到的。

​        (2)动画效果比css3动画丰富,有些动画效果，比如曲线运动,冲击闪烁,视差滚动效果，只有JavaScript动画才能完成

​        (3)CSS3有兼容性问题，而JS大多时候没有兼容性问题

**CSS动画**

缺点：

   (1)运行过程控制较弱,无法附加事件绑定回调函数。CSS动画只能暂停,不能在动画中寻找一个特定的时间点，不能在半路反转动画，不能变换时间尺度，不能在特定的位置添加回调函数或是绑定回放事件,无进度报告

   (2)代码冗长。想用 CSS 实现稍微复杂一点动画,最后CSS代码都会变得非常笨重。

优点： (1)浏览器可以对动画进行优化。

-  浏览器使用与 requestAnimationFrame 类似的机制，requestAnimationFrame比起setTimeout，setInterval设置动画的优势主要是:1)requestAnimationFrame 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成,并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率,一般来说,这个频率为每秒60帧。2)在隐藏或不可见的元素中requestAnimationFrame不会进行重绘或回流，这当然就意味着更少的的cpu，gpu和内存使用量。
- 强制使用硬件加速 （通过 GPU 来提高动画性能）

**CSS动画流畅的原因**

渲染线程分为main thread(主线程)和compositor thread(合成器线程)。
如果CSS动画只是改变`transform`和`opacity`，这时整个CSS动画得以在compositor thread完成（而JS动画则会在main thread执行，然后触发compositor进行下一步操作）
在JS执行一些昂贵的任务时，main thread繁忙，CSS动画由于使用了compositor thread可以保持流畅，

> 在主线程中，维护了一棵Layer树（LayerTreeHost），管理了TiledLayer，在compositor thread，维护了同样一颗LayerTreeHostImpl，管理了LayerImpl，这两棵树的内容是拷贝关系。因此可以彼此不干扰，当Javascript在main thread操作LayerTreeHost的同时，compositor thread可以用LayerTreeHostImpl做渲染。当Javascript繁忙导致主线程卡住时，合成到屏幕的过程也是流畅的。
> 为了实现防假死，鼠标键盘消息会被首先分发到compositor thread，然后再到main thread。这样，当main thread繁忙时，compositor thread还是能够响应一部分消息，例如，鼠标滚动时，加入main thread繁忙，compositor thread也会处理滚动消息，滚动已经被提交的页面部分（未被提交的部分将被刷白）。

**CSS动画比JS流畅的前提**

- JS在执行一些昂贵的任务
- 同时CSS动画不触发layout或paint
  在CSS动画或JS动画触发了paint或layout时，需要main thread进行Layer树的重计算，这时CSS动画或JS动画都会阻塞后续操作。

​     只有如下属性的修改才符合“仅触发Composite，不触发layout或paint”：

- backface-visibility
- opacity
- perspective
- perspective-origin
- transfrom

所以只有用上了3D加速或修改opacity时，css3动画的优势才会体现出来。

​      (2)代码相对简单,性能调优方向固定

​     (3)对于帧速表现不好的低版本浏览器，CSS3可以做到自然降级，而JS则需要撰写额外代码

**结论**

 如果动画只是简单的状态切换，不需要中间过程控制，在这种情况下，css动画是优选方案。它可以让你将动画逻辑放在样式文件里面，而不会让你的页面充斥 Javascript 库。然而如果你在设计很复杂的富客户端界面或者在开发一个有着复杂UI状态的 APP。那么你应该使用js动画，这样你的动画可以保持高效，并且你的工作流也更可控。所以，在实现一些小的交互动效的时候，就多考虑考虑CSS动画。对于一些复杂控制的动画，使用javascript比较可靠。

## async与generator

generator语法糖，*替换成async，yield替换成await。

相对于generator的改进：

1. 内置执行器，不需要想generator那样调用next方法或者co模块才能执行。
2. 语义化
3. co模块 yield后面只能是thunk函数或promise对象，await命令后面，可以是promise对象和原始类型的值（此时相当于同步操作）
4. 返回值时promise。generator函数的返回值是Iterator对象，await命令就是内部then命令的语法糖。

**错误监控**

如果有多个await，那么只要一个await后面的promise变成了reject，整个async函数都会中断执行。（此外，还有一个缺点时，await会阻塞代码，也许后面的一部代码不依赖前者，但仍需要等待前者完成，解决方案参照第三条）

1. async的函数里面，将多个await命令统一放在try catch结构中。
2. 在async函数返回的promise对象后加上.catch。
3. 多个await命令后面的一步操作，如果不存在继发关系，最好同时触发，用promise.all。

**实现原理**

async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。

```
async function fn(args) {
  // ...
}

// 等同于

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
```

所有的`async`函数都可以写成上面的第二种形式，其中的`spawn`函数就是自动执行器。

下面给出`spawn`函数的实现，基本就是前文自动执行器的翻版。

```
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```

## Object.create和new

首先了解下Object.create的实现方式

```js
Object.create =  function (o) {
    var F = function () {};
    F.prototype = o;
    return new F();
};
```

new操作符会做一下几个事情：

- 创建一个新对象
- 将构造函数的作用域赋给新对象(因此构造函数的this指向了这个新对象)
- 执行构造函数中的代码（为新创建的对象添加属性）
- 返回新对象

其中被new的函数就叫构造函数。

## 点击穿透

由于移动端双击缩放或者双击滚动，导致存在300ms点击延时。

在移动端手指点击一个元素，会按以下顺序触发事件。

​	touchstart—>touchmove—>touchend—》click

click从上手触摸开始、且手指没有在屏幕上移动（满足某个位移值），且触摸和离开屏幕之间的间隔较短（满足某个小的时间间隔）

**解决方案**

- 禁用缩放

  <pre><code><meta name="viewport" content="user-scalable=no">
  <meta name="viewport" content="initial-scale=1,maximum-scale=1"></code></pre>







## 模拟元素拖拽

```js
            class Drag {
                constructor(el) {
                    this.el = document.querySelector(el)
                    this.params = {
                        flag: false,
                        left: this.getCss(this.el, 'left'),
                        top: this.getCss(this.el, 'height'),
                        currentX: 0,
                        currentY: 0
                    }
                }
                // 获取css样式（包含兼容性）,currentStyle是IE自带，IE9之后才支持getComputedStyle，且不只是伪元素
                getCss(el, style) {
                    return el.currentStyle ? el.currentStyle[style] : window.getComputedStyle(el, null)[style]
                }
                init() {
                    const { el, params } = this
                    const that = this;
                    el.addEventListener('mousedown', function (e) {
                        params.flag = true
                        // 阻止拖动的默认事件，比如图片在新窗口打开等等
                        if (e.preventDefault) e.preventDefault()
                        else e.returnValue = false

                        params.currentX = e.clientX
                        params.currentY = e.clientY
                    })
                    // 要将mousemove和mouseup绑在document上，因为元素移动很可能跟不上鼠标移动的速度，那么mousemove和mouseup就不在el上了
                    document.addEventListener('mousemove', function (e) {
                        // var e = event ? event : window.event;


                        if (params.flag) {
                            el.style.left = parseInt(params.left) + e.clientX - params.currentX + 'px'
                            el.style.top = parseInt(params.top) + e.clientY - params.currentY + 'px'
                        }
                    })
                    document.addEventListener('mouseup', function(e) {
                        params.flag = false
                        params.left = that.getCss(el, 'left')
                        params.top = that.getCss(el, 'top')
                    })
                }
            }
        let drag = new Drag('#drag')
        drag.init()
```
## 深拷贝和浅拷贝（对象和数组）

数组浅拷贝：直接赋值，slice()或slice(0)，concat()，for遍历（单纯的for...）,Object.assign()(两参数时，因为如果多参数的话，后面的会覆盖前面的，因为都是同样的key值).

```js
Object.assign(b, [1,2,3,4,5], [2,3,4])
// (5) [2, 3, 4, 4, 5]
```

对象浅拷贝：map，Object.assign(target, source1, source2, source3...)。

深拷贝：递归。（T_T，不会优化）

```js
function deepCopy(des, src) {
  des = des || new src.constructor;// 避免最开始des是undefined
  for(var index in src) {
    if(typeof src[index] !== 'object') {
      des[index] = src[index];
    } else {
      // des[index] = Array.isArray(src[index]) ? [] : {};
      des[index] = new src[index].constructor;
      arguments.callee(des[index], src[index]); // 避免因函数名改变造成错误
    }
  }
  return des;
}
```

## 递归的优化（尾递归：减少内存空间）

尾递归的标准是，函数运行到最后一步调用自身（而不是在最后一行调用自身），这样的话，不用再保存外层函数的信息，避免了栈溢出的风险。

将递归改成迭代，相当于循环代码的执行效率。

## 递归的优化（记忆）（看一下你不知道的js的递归那一块）

就是每进行一次递归，都把上一次递归结果缓存到一个数组中，下一次递归可以直接调用这个数组的值，减少寄存器的调用。



各种高宽

## 一.js获取页面各种距离（空）

![](https://images.cnblogs.com/cnblogs_com/52linux/437309/o_client-offset-scroll.jpg)

offset是从子元素的margin外边缘算到offsetparent的content边缘。

client为content+padding所占据空间（scroll的部分不算在内）。

scroll为content+padding被隐藏的部分。

浏览器窗口大小：

```
var height = window.innerHeight // IE不支持
    || document.documentElement.clientHeight
    || document.body.clientHeight;

```

事实上后两种方式获取的高度和`window.innerHeight`是不一样的，这3个属性的值逐个变小。 具体说来，`window.innerHeight`包括整个DOM：内容、边框以及滚动条。

- `documentElement.clientHeight`不包括整个文档的滚动条，但包括`<html>`元素的边框。

- `body.clientHeight`不包括整个文档的滚动条，也不包括`<html>`元素的边框，也不包括`<body>`的边框和滚动条。

  1. 屏幕可视窗口大小

  ```
          **原生方法**：
              window.innerHeight 标准浏览器及IE9+ ||
              document.documentElement.clientHeight 标准浏览器及低版本IE标准模式 ||
              document.body.clientHeight  低版本混杂模式
          **jQuery方法**： 
              $(window).height();

  ```

  1. 浏览器窗口顶部与文档顶部之间的距离，也就是滚动条滚动的距离：

  ```
          **原生方法**：
                window.pagYoffset 标准浏览器及IE9+ ||
                document.documentElement.scrollTop 兼容ie低版本的标准模式 ||
                document.body.scrollTop 兼容混杂模式；
          **jQuery方法**：
                $(document).scrollTop();

  ```

  1. 获取元素的尺寸

  ```
  $(o).width() = o.style.width;
  $(o).innerWidth() = o.style.width+o.style.padding;
  $(o).outerWidth() = o.offsetWidth = o.style.width+o.style.padding+o.style.border；
  $(o).outerWidth(true) = o.style.width+o.style.padding+o.style.border+o.style.margin；

  ```

  ```
  **注意**
  要使用原生的style.xxx方法获取属性，这个元素必须已经有内嵌的样式，如`<div style="...."></div>`；

  如果原先是通过外部或内部样式表定义css样式，必须使用`o.currentStyle[xxx] || document.defaultView.getComputedStyle(0)[xxx]`来获取样式值。

  ```

  1. 获取元素的位置信息
     **jQuery**：
     `$(o).offset().top`元素距离文档顶的距离
     `$(o).offset().left`元素距离文档左边缘的距离。
     **原生**：`getoffsetTop();`
     顺便提一下返回元素相对于第一个以定位的父元素的偏移距离，注意与上面偏移距的区别；
     `jQuery：position()`返回一个对象
     `$(o).position().left = o.style.left;`
     `$(o).position().top = o.style.top；`

**1.screen系列**

- `screen.width`:屏幕的宽度
- `screen.height`:屏幕的高度

**2.style系列（必须是行内设置样式才有效）**

- `element.style.width`：当前对象的宽度。
- `element.style.height`：当前对象的高度。
- `element.style.left`：当前对象的left值。
- `element.style.top`：当前对象的top值。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>demo</title>
<style type="text/css">
*{padding: 0;margin: 0;}
html,body{height: 100%; margin:0; padding:0;}
.wrap{height: 1900px;background: #000;}
</style>
</head>
<body>
<!--设置百分百宽度-->
<div class="wrap" id="wrap" style="width: 80%"></div>
<script type="text/javascript">
    window.onload=function(){
        var wrap=document.getElementById("wrap");
        var styleW=wrap.style.width;
        var styleH=wrap.style.height;
        var oW=wrap.offsetWidth;
        console.log(styleW);//返回80%
        console.log(styleH);//没有在行内样式设置，无效，返回空值
        console.log(oW);//返回的是数字
    }
</script>
</body>
</html>

```

**3.offset系列**

- `element.offsetParent`：当前对象的上级层对象。
- `element.offsetWidth`：当前对象的宽度。（width+padding+border）
- `element.offsetHeight`：当前对象的高度。（Height+padding+border）
- `element.offsetLeft`：当前对象到其上级层左边的距离 。
- `element.offsetTop`：当前对象到其上级层上边的距离  。
- `element.offsetWidth`和`style.width`区别：
  1.`style.width`返回值除了数字外还带有单位px；
  2.如对象的宽度设定值为百分比宽度,则无论页面变大还是变小，`style.width`都返回此百分比,而`offsetWidth`则返回在不同页面中对象的宽度值而不是百分比值；
  3.如果没有给 HTML 元素指定过 width样式，则 `style.width` 返回的是空字符串；
  4.`style.width`不包含`borde`r和`padding`。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>demo</title>
<style type="text/css">
*{padding: 0;margin: 0;}
html,body{height: 100%; margin:0; padding:0;}
.wrap{width: 600px;height: 600px;background: #000;border: 3px solid green;position: relative;}
.box{width:300px;height: 400px; border: 5px solid #fff;padding:5px;margin: 30px}
</style>
</head>
<body>
<div class="wrap" id="wrap">
    <div class="box" id="box"></div>
</div>
<script type="text/javascript">
    window.onload=function(){
        var wrap=document.getElementById("wrap");
        var box=document.getElementById("box");
        var oW=box.offsetWidth;
        var oT=box.offsetTop;
        console.log(oW);//返回320,包含padding+border+width
        console.log(oT);//返回30,不包含父级的border
    }
</script>

```

**4.client系列**

- `element.clientWidth`: 获取对象可见内容的宽度，不包括滚动条，不包括边框；
- `element.clientHeight`: 获取对象可见内容的高度，不包括滚动条，不包括边框；
- `element.clientLeft`: 获取对象的border宽度
- `element.clientTop`：获取对象的border高度

**5.scroll系列**

- `element.scrollWidth`:获取对象的滚动宽度 。
- `element.scrollHeight`: 获取对象的滚动高度。
- `element.scrollLeft`:设置或获取位于对象左边界和对象中目前可见内容的最左端之间的距离(width+padding为一体)
- `element.scrollTop`:设置或获取位于对象最顶端和对象中可见内容的最顶端之间的距离；(height+padding为一体)

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>demo</title>
<style type="text/css">
*{padding: 0;margin: 0;}
html,body{height: 100%; margin:0; padding:0;}
.wrap{width: 600px;height: 600px;border: 3px solid green;position: relative;}
.box{width:300px;height: 400px; border: 3px solid red;padding:5px;margin: 30px;overflow: auto;}
.box p{font-size: 20px;line-height: 36px;}
</style>
</head>
<body>
<!-- 设置父级 -->
<div class="wrap" id="wrap">
    <div class="box" id="box">
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
        <p>测试</p>
    </div>
</div>
<script type="text/javascript">
    window.onload=function(){
        var wrap=document.getElementById("wrap");
        var box=document.getElementById("box");
        var oW=box.offsetWidth;
        var oH=box.offsetHeight;
        var oT=box.offsetTop;
        var cW=box.clientWidth;
        var cH=box.clientHeight;
        var cT=box.clientTop;
        console.log('box.offsetWidth:'+oW);//返回box.offsetWidth:316
        console.log('box.offsetHeight:'+oH);//返回box.offsetHeight:416
        console.log('box.offsetTop:'+oT);//返回box.offsetTop:30
        console.log('box.clientWidth:'+cW);//返回box.clientWidth:293
        console.log('box.clientHeight:'+cH);//返回box.clientHeight:410
        console.log('box.clientTop:'+cT);//返回box.clientTop:3
        console.log('box.scrollWidth:'+sW);//返回box.scrollWidth:293
        console.log('box.scrollHeight:'+sH);//返回box.scrollHeight:1018
    }
</script>
</body>
</html>

```

**document.body和document.documentElement兼容问题**

- body是DOM对象里的body子节点，即 <body> 标签；documentElement 是整个节点树的根节点root，即<html> 标签；
- 整个文档的一个根就是,在DOM中可以使用document.documentElement来访问它，它就是整个节点树的根节点。而body是子节点，要访问到body标签，在脚本中可以写：document.body。
- 参考链接 [clientHeight , scrollHeight , offsetHeight之间的区别及兼容方案](https://link.jianshu.com?t=http://www.cnblogs.com/nanshanlaoyao/p/5964730.html)
- JS中完美兼容各大浏览器的scrolltop方法

```
var scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;

```

## 二.js获取页面坐标数据（空）

**1.client系列**

- `event.clientX` :设置或获取鼠标指针位置相对于当前窗口的 x 坐标，其中客户区域不包括窗口自身的控件和滚动条。
- `event.clientY`: 设置或获取鼠标指针位置相对于当前窗口的 y 坐标，其中客户区域不包括窗口自身的控件和滚动条。

**2.screen系列**

- `event.screenX`： 设置或获取获取鼠标指针位置相对于用户屏幕的 x 坐标。
- `event.screenY`： 设置或获取鼠标指针位置相对于用户屏幕的 y 坐标。

**3.offset系列**

- `event.offsetX`：,鼠标相比较于触发事件的元素的X位置,以元素盒子模型的内容区域的左上角为参考点,如果有boder,可能出现负值。
- `event.offsetY`：同上，Y位置。

**4.layer系列（IE8以及以下版本没有）**

- `event.layerX`：鼠标相比较于当前坐标系的X位置,即如果触发元素没有设置绝对定位或相对定位,以页面为参考点,如果有,将改变参考坐标系,从触发元素盒子模型的border区域的左上角为参考点
- `event.layerY`：同上，Y位置。

**5.XY系列（FF没有）**

- `event.x`：相对可视区域的X坐标
- `event.y`：相对可视区域的Y坐标

**6.page系列 （IE8以及以下版本没有）**

- 类似于`event.clientX`、`event.clientY`，但它们使用的是文档坐标而非窗口坐标。。
- `event.pageX`：设置或获取鼠标指针位置相对于当前文档的 x 坐标
- `event.pageY`：设置或获取鼠标指针位置相对于当前文档的 y 坐标

```
document.onclick = function(e){
        e = e || window.event;
        console.log('e.clientX:',e.clientX);
        console.log('e.screenX:',e.screenX);
        console.log('e.offsetX:',e.offsetX);
        console.log('e.layerX:',e.layerX);
        console.log('e.pageX:',e.pageX);
        console.log('e.x:',e.x);
    }
```

## js面向对象编程

一般具有继承、封装、多态，模块化。但是js有些不太一样的地方。

继承，通过原型链方式继承（object.create，加上设置constructor）。

多态，不同类中可以有名称相同的方法。（不是父子类）（

多态的定义如下：同一个操作作用于不同的对象上，可以产生不同的解释和不同的执行结果。

）

重载：是函数或者方法有相同的名称，但是参数列表不相同的情形。目前还实现不了。

命名空间，一个包含方法属性，对象的 对象。

封装，类中，不需要知道某方法具体如何实现的，就可以使用这种方法。

##         作用域与执行环境

执行环境定义了变量或函数是否有权访问其他数据，决定了他们各自的行为。

每个执行环境都有一个与之关联的**变量对象**和**作用域链**。

环境中定义的所有变量和函数都保存在 **变量对象**中。执行环境分两种：全局执行环境、函数执行环境。

- **全局执行**环境是最外围的一个执行环境，变量对象就是全局活动对象（window），全局执行环境在应用程序退出（关闭网页或浏览器时）才会被销毁。

- **函数执行环境**

  每个函数都有自己的执行环境。当执行流进入一个函数时，函数环境就会被推入一个环境栈中，执行完后，栈将其环境弹出，把控制权返回给之前的执行环境。函数执行环境的变量对象是该函数的活动对象。

- **作用域链**

  代码再某个环境中执行时，会从执行环境开始创建变量对象的一个作用域链。保证对执行环境有权访问的所有变量的所有变量和函数的有序访问。顶端是当前执行代码所在环境的变量对象，下一个是其外部环境的变量对象，以此类推直到全局执行环境中的window。

ES5只有函数作用域和全局作用域，ES6还有块级作用区域。

## 事件流

**事件冒泡**

IE的事件流就叫事件冒泡，由嵌套层次最深的节点，逐级向上传播直到html，再到document对象，再到window对象。（所以IE事件流只有冒泡没有捕获）

**事件捕获**

同事件冒泡相反，顶层节点先收到，嵌套层次深的节点最后收到。

捕获就是指在事件到达预定目标之前，先捕获下来，所以是这么个顺序。

**事件流的三个阶段**

捕获阶段（为捕获创造机会）、目标阶段（实际触发事件的元素对事件做出响应，并堪称冒泡阶段的一部分）、冒泡阶段（对事件做出响应）。

标准规定在捕获阶段不涉及目标，但现代浏览器都会在捕获阶段触发事件对象上的事件，所以有两个机会在目标对象上操作事件。

**事件处理程序**

相应某个事件的函数，又叫事件监听器。

事件处理程序执行时有权访问 **全局作用域** 和 **局部变量event**（事件对象），内部的this值等于event.target（或者给事件监听器bind一下，改变this和参数，，，）。

DOM2中有两个事件处理程序，addEventListener和removeEventListener。

IE9、 Firefox、 Safari、 Chrome 和 Opera 支持 DOM2 级事件处理程序 。

**IE** 实现了与 DOM 中类似的两个方法： attachEvent()和 detachEvent()。这两个方法接受相同
的两个参数：事件处理程序名称与事件处理程序函数。由于 IE8 及更早版本只支持事件冒泡，所以通过attachEvent()添加的事件处理程序都会被添加到冒泡阶段 。

在编写**跨浏览器的代码**时，牢记**attachEvent**添加的监听器里面 **this === window** 非常重要。 

**addEventListener**监听器里面的this为当前触发事件的DOM元素，**this === e.target**。

```js
var EventUtil = {
    addHandler: function(element,type,handler) {
        if (element.addEventListener) {
            element.addEventListener(type,handler,false);
        }
        else if (element.attachEvent) {
            element.attachEvent('on'+type,handler);
        }
        else {
            element['on'+type] = handler;
        }
    },

    removeHandler: function(element,type,handler) {
        if (element.removeEventListener)
        {
            element.removeEventListener(type,handler,false);
        }
        else(element.detachEvent) {
            element.detachEvent('on' +type,handler);
        }
        else {
            element['on'+type] = null;
        }
    }
}
```

**给事件监听器传参**

根据上面可以看到，只能访问全局作用域和event，所以传参的话，可以利用闭包的形式。

```js
    function btnClick(a, b) {
        console.log(a, b);
        console.log(this);
        return function(e) {
            console.log('闭包', a, b);
            console.log(this, e); // this === e.target
        }
    }
    button.addEventListener('click', btnClick(1, 2), false);
```

**addEventListener**（4个参数，第三个参数有两种形式）

1. type 事件类型 string

2. listener 事件监听器 function或是实现了eventlistener接口的object

3. options 可选 object

   同listener属性相关的可选参数对象

   ```js
   {
     capture: false, // 捕获还是冒泡
     once: false, // 最多调用一次listener
     passive: false, // 永远不会调用preventDefault
   }

   ```

4.  useCapture boolean（仅在现代浏览器最近几个版本中是可选的，为了更广泛的支持，应该提供这个参数。）

5. `wantsUntrusted` ：如果为 `true `, 则事件处理程序会接收网页自定义的事件。此参数只适用于 Gecko。

false为冒泡，true为捕获。

但是调用的时候是三个参数，就是说第三个参数可为options，可为useCapture

```js
addEventListener(type, listener[, useCapture ])
addEventListener(type, listener[, options ])
```

**removeEventListener**

同add......大致相同，但是值得注意的是，add时是捕获还是冒泡 ，则remove时需和add时相同，因为监听器注册的阶段不同。

## 数据类型

**5种基本数据类型**

Undefined、Null、Boolean、Number、String。（按值传递，new。。。创建时按引用传递）

**1种复杂数据类型**

Object（按引用传递，但是函数中传参只能是按值，其实就是简单的赋了个值，引用的还是同一对象，所以才有了深复制和潜复制两种）。

按值传递是说，实际上函数中是创建了一个局部变量，指向了外部作用域中obj的地址，但是改变这个变量的值切断与那个地址之间的联系，外部的obj不会变。在函数内部重写obj时（不是说对指向的那个地址里的值进行修改），这个变量引用的就是一个局部对象了，会在函数执行完后销毁。

## new 和 Object.create

首先了解下Object.create的实现方式

```js
Object.create =  function (o) {
    var F = function () {};
    F.prototype = o;
    return new F();
};
```

new操作符会做一下几个事情：

- 创建一个新对象

- 将构造函数的作用域赋给新对象(因此this指向了这个新对象)

- 执行构造函数中的代码（为新创建的对象添加属性）

- 返回新对象

其中被new的函数就叫构造函数。

## 变量声明 var let const

**1. var**

不支持块级作用域

存在变量提升

**变量提升**：引擎在解释js代码之前会首先对其进行编译，编译的一部分工作就是找到所有的声明，并用合适的作用域将它们关联起来。

但是只有**声明**本身会被提升，而**赋值或其它运行逻辑**会留在原地。（提升是指在每个作用于块中）

当引擎执行LHS查询（找到该变量的声明？）时，如果找不到，就会在顶层（全局作用域）中创建一个具有该名称的变量，当然，这是在非严格模式下，否则会同RHS（找到变量的源值）一样，抛出一个Reference异常。

**注：**记得区分LHS和RHS查询，LHS找不到变量会创建一个声明，而RHS会报Reference错误。

**2. let**

- 块级作用域

- 同一作用域中不允许重复声明

- 暂存死区（TDZ）

  变量不会被提升的，在let声明语句之前调用，会报错，因为此时处于暂存死区的状态

  ```js
  let x = 10;

  function foo(){
    console.log(x);  //ReferenceError: x is not defined
    console.log(typeof x);  //ReferenceError: x is not defined
    let x = 20;  //如果这一句改成 var 会怎样？
    return x * x;
  }

  console.log(foo());
  ```

  如果let换成var，则打印出undefined，不报错。

  ​

- 循环中的let作用域

- 浏览器的兼容性

**3. const**

- const 声明的标识符所绑定的值不可以再次改变。o

```js
const obj = {a: 2}
obj = {a: 3}
console.log(obj) // TypeError: Assignment to constant variable.

const obj = {a: 2};
obj.a = 3;
console.log(obj); // {a: 3}
```

**const只是不允许修改变量的引用，修改内部的具体内容是可以的。**



- ES6 const 的其他行为和 let 一样。
- ES6中，const、let、class命令声明的全局变量，不再属于window的属性，从ES6开始，全局变量逐步与顶层对象的属性脱钩。

## 函数

## this指向

通常是由函数求值时的调用者决定的。

匿名函数指向window。

当然这是说不用call、apply、bind时的情况。

## call、bind、apply

func.call(thisObj，arg1，arg2...)、func.apply(thisObj，[obj1,obj2...])

第一个参数改变当前的this，第二个参数为函数传入的参数。

call和apply的差别，仅在于后续参数是数组还是多个写开。



func.bind(thisObj, arg1, arg2)

基本与call相同，差别在于bind返回的是绑定对象和参数后的函数，而非像call和apply那样立即执行，bind之后要赋给一个新的变量，再执行。

执行是还可以接着传参。

```js
function foo(arg1, arg2) {
    console.log(arg1, arg2); // 1, 2
}
var bar = foo.bind(null, 1);
bar(2,3); // 1, 2
```

bind第一个参数为null，表示不改变函数this指向，这么写可以达到先传参但不立即执行的效果。

而且通过bind也可以部分传参，如上面的，可以先传一个参数进去。

如果全部参数传进去，则可以实现延时调用的效果，比如setTimeout就可以先将参数传进去。

## ajax

原理：

核心是**XMLHttpRequest**，可以根据不同状态执行不同操作，其实它就是XMLHttpRequest的方法onreadystatechange，它在每次状态发生改变时都会触发。那么是谁取得的返回信息呢？它就是XMLHttpRequest的另一个方法responseText（当然还有responseXML）。(⊙o⊙)哦，我们还没有说发送给谁以及执行发送操作，这两个就是XMLHttpRequest的open和send方法。

  onreadystatechange  每次状态改变所触发事件的事件处理程序。
  responseText     从服务器进程返回数据的字符串形式。
  responseXML   从服务器进程返回的DOM兼容的文档数据对象。
  status                 从服务器返回的数字代码，比如常见的404（未找到）和200（已就绪）
  status Text         伴随状态码的字符串信息
  readyState         对象状态值，0—未初始化 1—正在加载  2—加载完毕 3—交互 4—完成。

**基本流程**

1. **创建xhr对象。**

```js
var xhr = new XMLHttpRequest(); // IE7+、 Firefox、 Opera、 Chrome 和 Safari 都支持原生的 XHR 对象

// IE6 检测原生 XHR 对象是否存在，如果存在则返回它的新实例。如果原生对象不存在，则检测 ActiveX 对象。
function createXHR(){
  if (typeof XMLHttpRequest != "undefined"){
    return new XMLHttpRequest();
  } else if (typeof ActiveXObject != "undefined"){
    if (typeof arguments.callee.activeXString != "string"){
      var versions = [ "MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
      "MSXML2.XMLHttp"], i, len;
      for (i=0,len=versions.length; i < len; i++){
      	try {
        	new ActiveXObject(versions[i]);
        	arguments.callee.activeXString = versions[i];
        	break;
      	} catch (ex){
        //跳过
        }
      }
    }
    return new ActiveXObject(arguments.callee.activeXString);
  } else {
  	throw new Error("No XHR object available.");
  }
}

var xhr = createXHR();
```

2. **发送请求**

   ```js
   xhr.open('get', url, boolean// 是否异步发送)
   xhr.send(null);
   ```

   收到响应后，响应数据会自动填充XHR对象的属性，相关属性如下：

   **responseText**：作为响应主体被返回的文本。
   **responseXML**：如果响应的内容类型是"text/xml"或"application/xml"，这个属性中将保存包含着响应数据的 XML DOM 文档。
   **status**：响应的 HTTP 状态。
   **statusText**： HTTP 状态的说明。 

3. **接受响应**

   ```js
   // 同步请求
   xhr.open("get", "example.txt", false);
   xhr.send(null);
   if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
   	alert(xhr.responseText);
   } else {
   	alert("Request was unsuccessful: " + xhr.status);
   }
   // 异步请求 
   // 必须在调用open之前指定onreadystatechange事件处理程序
   var xhr = createXHR();
   xhr.onreadystatechange = function(){
     if (xhr.readyState == 4){
       if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
       	alert(xhr.responseText);
       } else {
       	alert("Request was unsuccessful: " + xhr.status);
       }
     }
   };
   xhr.open("get", "example.txt", true);
   xhr.send(null);
   ```

   如果是2xx的状态码，或者是304表示可以直接使用缓存中的资源。

   ​

   异步的XHR可以检测XHR对象的 **readyState**属性。

   0：未初始化。还没调用open的时候。

   1：启动。调用open还没调用send。

   2：发送。调用了send还没接收到响应。

   3：接收。已经接受到部分响应数据。

   4：完成。已经接收到全部响应数据，并且已经可以在客户端使用了。

   每一次状态变化，都会触发一次onreadystatechange事件，所以要判断一下，一般只用判断是否为4。

4. 如果要发送**自定义请求头**，需在open和send之间设置

   ```js
   xhr.open("get", "example.php", true);
   xhr.setRequestHeader("MyHeader", "MyValue");
   xhr.send(null);
   ```

   否则只会发出以下这些基本的请求头。

   Accept：浏览器能够处理的内容类型。
   Accept-Charset：浏览器能够显示的字符集。
   Accept-Encoding：浏览器能够处理的压缩编码。
   Accept-Language：浏览器当前设置的语言。
   Connection：浏览器与服务器之间连接的类型。
   Cookie：当前页面设置的任何 Cookie。
   Host：发出请求的页面所在的域 。
   Referer：发出请求的页面的 URI。注意， HTTP 规范将这个头部字段拼写错了，而为保证与规范一致，也只能将错就错了。（这个英文单词的正确拼法应该是 referrer。）
   User-Agent：浏览器的用户代理字符串。 

5. 接收响应头。

   var myHeader = xhr.getResponseHeader("MyHeader");
   var allHeaders = xhr.getAllResponseHeaders(); 

**get和post**

get常用于向服务器查询某些信息。url末尾追加查询字符串参数。

但是使用get的话，查询字符串必须用 encodeURIComponent 进行编码。

```js
function addURLParam(url, name, value) {
  url += (url.indexOf("?") == -1 ? "?" : "&");
  url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
  return url;
}
```

post通常向服务器发送应该被保存的数据。

把数据作为请求的主体提交，可以使用xhr来模仿表单提交。

```js
function submitData(){
  var xhr = createXHR();
  xhr.onreadystatechange = function(){
    if (xhr.readyState == 4){
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
          alert(xhr.responseText);
      } else {
          alert("Request was unsuccessful: " + xhr.status);
      }
    }
  };
  xhr.open("post", "postexample.php", true);
  xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
  var form = document.getElementById("user-info");
  xhr.send(serialize(form));
}
```

xhr2 可以使用formData，这样不用自定义请求头。

**进度事件**

loadstart：在接收到响应数据的第一个字节时触发。
progress：在接收响应期间持续不断地触发。
error：在请求发生错误时触发。
abort：在因为调用 abort()方法而终止连接时触发。
load：在接收到完整的响应数据时触发。
loadend：在通信完成或者触发 error、 abort 或 load 事件后触发。 

```js
xhr.onprogress = function(event){
  var divStatus = document.getElementById("status");
  if (event.lengthComputable){
    divStatus.innerHTML = "Received " + event.position + " of " +
    event.totalSize +" bytes";
  }
};
```



## Promise和Async

写一个返回promise对象的函数a。

```js
(async ()=>{
  await a();
})()
```

**promise原理**

**极简promise雏形**

```js
function Promise(fn) {
    var value = null,
        callbacks = [];  //callbacks为数组，因为可能同时有很多个回调


    this.then = function (onFulfilled) {
      callbacks.push(onFulfilled);
      return this; // 链式调用
  	};

    function resolve(value) {
      setTimeout(function() {
          callbacks.forEach(function (callback) {
              callback(value);
          });
      }, 0)
  	} 

    fn(resolve);
}
```

new Promise(fn),传fn进去，fn为执行的异步函数，这个函数可以接受一个resolve参数，resolve是一个有一个value参数的函数，调用promise实例的then属性，就可以将then里面传的函数 放入回调，并返回实例，以便链式调用。

fn异步操作成功时，会执行resolve(value)（fn内部一般是这么写的），然后就会遍历callbacks数组，依次执行。

resolve里面的setTimeout延时函数：第二个参数设为0，可以保证里面的函数在普通函数，这里指then之后再执行。因为浏览器会先执行当前任务队列中的任务，再执行setTimeout队列中的任务。这样就可以保证在then注册完回调之后，再resolve。

**引入状态机制**

pending、fulfilled、rejected（后两个都只能由第一个转化而来，而且只能转化一次，不能再变了）

```js
function Promise(fn) {
    var state = 'pending',
        value = null,
        callbacks = [];

    this.then = function (onFulfilled) {
        if (state === 'pending') {
            callbacks.push(onFulfilled);
            return this;
        }
        onFulfilled(value);
        return this;
    };

    function resolve(newValue) {
        value = newValue;
        state = 'fulfilled';
        setTimeout(function () {
            callbacks.forEach(function (callback) {
                callback(value);
            });
        }, 0);
    }

    fn(resolve);
}
```

这样的话，在resolve之前的注册的，都会在异步返回之后执行，在resolve之后注册的，都会立即执行。

**链式Promise**

当前promise达到fulfilled之后，即开始进行下一个promise（后邻promise），要衔接前后promise。

所以在then里面return一个promise。

```js
this.then = function (onFulfilled) {
  return new Promise(function (resolve) {
    handle({
      onFulfilled: onFulfilled || null,
      resolve: resolve
    });
  });
};
function handle(callback) {
  if (state === 'pending') {
    callbacks.push(callback);
    return;
  }
  //如果then中没有传递任何东西
  if(!callback.onFulfilled) {
    callback.resolve(value);
    return;
  }

  var ret = callback.onFulfilled(value);
  callback.resolve(ret);
}
function resolve(newValue) {
  if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
    var then = newValue.then;
    if (typeof then === 'function') {
      then.call(newValue, resolve);
      return;
    }
  }
  state = 'fulfilled';
  value = newValue;
  setTimeout(function () {
    callbacks.forEach(function (callback) {
      handle(callback);
    });
  }, 0);
}
```

then返回新的promise，串行基础，并支持链式调用。

内部方法handle，传入onFilfilled及新promise实例创建时传入的resolve均被push到当前promise的callbacks中，衔接当前promise和后面的promise。

第一次resolve的时候，值是正常的数字，所以跳过if，执行handle，在回调中取得了另一个异步方法，生成新的primise。

```js
function Promise(fn) {
    var state = 'pending',
        value = null,
        callbacks = [];

    this.then = function (onFulfilled, onRejected) {
        return new Promise(function (resolve, reject) {
            handle({
                onFulfilled: onFulfilled || null,
                onRejected: onRejected || null,
                resolve: resolve,
                reject: reject
            });
        });
    };

    function handle(callback) {
      if (state === 'pending') {
          callbacks.push(callback);
          return;
      }

      var cb = state === 'fulfilled' ? callback.onFulfilled : callback.onRejected,
          ret;
      if (cb === null) {
          cb = state === 'fulfilled' ? callback.resolve : callback.reject;
          cb(value);
          return;
      }
      try {
          ret = cb(value);
          callback.resolve(ret);
      } catch (e) {
          callback.reject(e);
      } 
  }

    function resolve(newValue) {
        if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
            var then = newValue.then;
            if (typeof then === 'function') {
                then.call(newValue, resolve, reject);
                return;
            }
        }
        state = 'fulfilled';
        value = newValue;
        execute();
    }

    function reject(reason) {
        state = 'rejected';
        value = reason;
        execute();
    }

    function execute() {
        setTimeout(function () {
            callbacks.forEach(function (callback) {
                handle(callback);
            });
        }, 0);
    }

    fn(resolve, reject);
}
```

总结：

1. 通过Promise.prototype.then和Promise.prototype.catch方法将观察者方法注册到被观察者Promise对象中，同时返回一个新的Promise对象，以便可以链式调用。
2. 被观察者管理内部pending、fulfilled和rejected的状态转变，同时通过构造函数中传递的resolve和reject方法以主动触发状态转变和通知观察者。

## promise.all()和Promise.race()

**promise.all()** 

并行执行任务

```js
// 同时执行p1和p2，并在它们都完成后执行then:
Promise.all([p1, p2]).then(function (results) {
    console.log(results); // 获得一个Array: ['P1', 'P2']
});
```

**Promise.race()**

获得两个并行执行的先返回的那个结果，丢弃另一个执行结果。

比如，同时向两个URL读取用户的个人信息，只需要获得先返回的结果即可。

```js
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
Promise.race([p1, p2]).then(function (result) {
    console.log(result); // 'P1'
});
```



## 字符串API

charAt(num) // 得到指定索引位置的单字符

charCodeAt(num) // 得到指定索引位置字符的Unicode值 (ascii为其子集)

concat(str01,str02) // 连接俩字符~

indexOf("str") // 取str第一次出现的索引

lastIndexOf("str") // 取str最后一次出现的索引

以上这两个方法都接受第二个参数，表示从那个位置开始往前或往后搜索。



slice( start , end ) // 其对象可以是字符串or数组 , 记得其范围不包括end

slice/substr/substring都不改变原数组，第一个参数指定子字符串开始位置，第二个参数表示到哪里结束，substr()第二个参数是返回的字符个数，一个参数时，都是返回从指定位置到字符串末尾。

参数有负值时，slice将负值与长度相加，substr（）第一个参数是负值加length，第二个是转换为0，substring会把所有负值参数转换为0，所以只有一个负值参数时，返回整个字符串，第二个参数为负数，返回空字符串。

trim（）删除前缀和后面的空格，不改变原字符串。

## 数组API

**length (改变原数组)**

并不是只读，通过设置这个属性，可以从数组的末尾移除项或向数组中添加项。

```js
var arr1 = [1,2,3,4];
arr1.length = 2;
console.log(arr1); // [1,2]
```

直接这样子新增的话是新增undefined项。

如下面方式新增可以利用length在末尾增加非undefined的项。

```js
arr1[arr1.length] = 5;
```

**push,pop, shift, unshift (改变原数组)**

**concat()（不改变原数组）**

不传参时，会创建一个副本。（注：同样是浅拷贝，深层结构还是保存了相同的引用）

**slice（不改变原数组）**

slice基于当前数组中的一或多个项创建一个新数组。

- 1参数时，返回从指定位置开始到当前数组末尾的所有项。
- 2参数时，返回从起始和结束位置之间的项（不包括结束位置）。
- 如果有负数，那么用length加上这个负数当做正数。如果这个负数加上length还是负数，那么当做0处理。

**splice（改变原始数组）**

返回的是删除的项。

- 删除：指定两个参数，要删除的第一项的位置，和要删除的项数。

  `arr.splice(0,2)`会删除数组中的前两项。

- 插入：向指定位置插入任意数量的项。三个及三个以上参数，起始位置、0（要删除的项数）、要插入的项。

  ```js
  arr = [1,2,3,4] // (4) [1, 2, 3, 4]
  arr.splice(1, 0 , 1.5, 1.6) // [] (返回的始终是从原始数组中删除的项)
  arr // (6) [1, 1.5, 1.6, 2, 3, 4]
  ```

-  替换：显而易见，三个及三个以上参数，起始位置、1、要插入的项。

  ```js
  arr = [1,2,3,4] // (4) [1, 2, 3, 4]
  arr.splice(1, 1, 1.5, 1.6) // [2]
  arr // (5) [1, 1.5, 1.6, 3, 4]
  ```

  ​

**instanceof和Array.isArray()**

**instanceof**是假定了只有一个全局执行环境，如果网页上包含多个框架，那么可能存在两个以上不同版本的Array构造函数。如果从一个框架向另一个框架传入数组，那么构造函数不同，这个方法可能不起作用。

`value instanceof Array`

**Array.isArray()**: 确定这个值到底是不是数组。不用考虑在那个全局执行环境创建的（第一个是判断是不是Array的实例）。

**toLocaleString()、toString()、valueOf() （不改变原数组）**

**toString**: 返回数组中每个值的字符串形式拼接成的一个以逗号分隔的字符串。

注意：实际上为创建这个字符串，会调用数组的**每一项的toString()方法**。

toLocaleString: 与toString()基本相同，只是一个是调用每一项的toString()方法，一个是toLocaleString()，通常来说返回的结果会相同。

```js
var person1 = {
  toLocaleString: function () {
    return 'Ni';
  }
  toString: function () {
  	return 'Hi';
  }
}
var person2 = {
  toLocaleString: function () {
    return 'Li';
  }
  toString: function () {
  	return 'Mi';
  }
}

var people = [person1, person2];
alert(people) // alert的内容要是字符串，所以会隐式调用每一项的toString方法。 Hi,Mi
```
**重排序方法 reverse()和sort()（不改变原数组）**

**reverse()**逆序：

```js
// 如果没有reverse() O(1)
var arr = [];
var source = [1,2,3,4,5];

source.forEach((item) => {
  arr.unshift(item);
});

console.log(arr); // 5,4,3,2,1 
```

**sort()**: 默认会先调用每一项的toString()方法，再比较排序。

数组长度小于等于 22 的用插入排序 InsertionSort，比22大的数组则使用快速排序 QuickSort



可以接受一个比较函数作为参数，以便我们指定哪个值应该放在哪个值的前面。

function campare( val1, val2 ) { ... }

如果第一个参数应该位于第二个参数之前，返回负数，。。。。

即负数表示不变，正数表示交换位置。

0的话，位置不变。

```js
function compare(val1, val2) {
  return val1-val2;
}
[1,2,3].sort(compare);
```



## 数组转化字符串(不改变原数组)

1. `var str = String(str)` 。
2. `var str = arr.join('自定义分隔符')` 。
3. 默认用逗号连接的话，还可以使用toString和toLocaleString。

## 连接子数组（不改变原数组）

1. `var newArr = arr.concat(arg1, arg2, arr2, ...)`（可以是元素也可以是数组）

**ps**: 当Object.assign应用于数组时，会出现后面的来替代前面的。（毕竟数组也是对象的一种）

```js
var arr = Object.assign([1,2,3,5],[2,3,4]);
console.log(arr); // (4) [2, 3, 4, 5]
```

因为数组时按照索引对应的，想象成这样：

```js
var arr1 = {
  0: 1,
  1: 2,
  2: 3,
  3: 4
}
var arr2 = {
  0: 2,
  1: 3,
  2: 4,
}
var arr = Object.assign(arr1, arr2); // [2,3,4,5]
console.log(arr1); // [2,3,4,5]
```

要注意的一点，Object.assign会改变源对象。

## 正则（空）

## js语言特性

**高阶函数**

可以将函数作为参数，返回一个函数，可以进行过程抽象，函数式编程。

**动态类型**

静态类型的缺点：

- 必须提前声明意图，这常常会导致灵活性降低。例如，更改一个 Java 类就会更改类的类型，因而必须重新编译。对比之下，Ruby 允许开放的类，但更改一个 Java 类还是会更改类的类型。
- 要实现相同的功能，必须输入更多的代码。例如，必须用参数形式包括进类型信息，必须用函数形式返回值和所有变量的类型。另外，还必须声明所有变量并显式地转化类型。
- 静态语言的编译-部署周期要比动态语言的部署周期长，尽管一些工具可被用来在某种程度上缓解这一问题。

静态类型更适合用于构建中间件或操作系统的语言中。UI 开发常常需要更高的效率和灵活性，所以更适合采用动态类型。

对象原型继承

## jQeury

**闭包结构**

```js
// 用一个函数域包起来，就是所谓的沙箱
// 在这里边 var 定义的变量，属于这个函数域内的局部变量，避免污染全局
// 把当前沙箱需要的外部变量通过函数参数引入进来
// 只要保证参数对内提供的接口的一致性，你还可以随意替换传进来的这个参数
(function(window, undefined) {
   // jQuery 代码
})(window);
```

undefinded: 避免开发者错误的给undefined变量赋值。

这种写法还有一种好处，可以针对性的压缩代码。

```js
// 压缩策略
// w -> windwow , u -> undefined
(function(w, u) {
 // jQuery代码
})(window);
```

**无new构造**

```js
(function(window, undefined) {
    var
    // ...
    jQuery = function(selector, context) {
        // The jQuery object is actually just the init constructor 'enhanced'
        // 看这里，实例化方法 jQuery() 实际上是调用了其拓展的原型方法 jQuery.fn.init
        return new jQuery.fn.init(selector, context, rootjQuery);
    },
 
    // jQuery.prototype 即是 jQuery 的原型，挂载在上面的方法，即可让所有生成的 jQuery 对象使用
    jQuery.fn = jQuery.prototype = {
        // 实例化化方法，这个方法可以称作 jQuery 对象构造器
        init: function(selector, context, rootjQuery) {
            // ...
        }
    }
    // 这一句很关键，也很绕
    // jQuery 没有使用 new 运算符将 jQuery 实例化，而是直接调用其函数
    // 要实现这样,那么 jQuery 就要看成一个类，且返回一个正确的实例
    // 且实例还要能正确访问 jQuery 类原型上的属性与方法
    // jQuery 的方式是通过原型传递解决问题，把 jQuery 的原型传递给jQuery.prototype.init.prototype
    // 所以通过这个方法生成的实例 this 所指向的仍然是 jQuery.fn，所以能正确访问 jQuery 类原型上的属性与方法
    jQuery.fn.init.prototype = jQuery.fn;
 
})(window);
```

jQuery.fn.init.prototype = jQuery.prototype= jQuery.fn;

也就是说，new jQuery.fn.init() 相当于 new jQuery() ;他们具有相同的原型链，构造函数都是执行init里的内容‘’，直接执行jquery返回的new...，所以可以无new实例化jQuery对象。

**jquery的重载**

比如attr一个参数是get，两个参数是set，$(...)内部有9种重载场景。

## 排序算法比较表格

| 排序算法     | 平均时间复杂度   | 最坏时间复杂度   | 空间复杂度         | 是否稳定 |
| ------------ | ---------------- | ---------------- | ------------------ | -------- |
| 冒泡排序     | O（n2）O（n2）   | O（n2）O（n2）   | O（1）O（1）       | 是       |
| 选择排序     | O（n2）O（n2）   | O（n2）O（n2）   | O（1）O（1）       | 不是     |
| 直接插入排序 | O（n2）O（n2）   | O（n2）O（n2）   | O（1）O（1）       | 是       |
| 归并排序     | O(nlogn)O(nlogn) | O(nlogn)O(nlogn) | O（n）O（n）       | 是       |
| 快速排序     | O(nlogn)O(nlogn) | O（n2）O（n2）   | O（logn）O（logn） | 不是     |
| 堆排序       | O(nlogn)O(nlogn) | O(nlogn)O(nlogn) | O（1）O（1）       | 不是     |
| 希尔排序     | O(nlogn)O(nlogn) | O（ns）O（ns）   | O（1）O（1）       | 不是     |
| 计数排序     | O(n+k)O(n+k)     | O(n+k)O(n+k)     | O(n+k)O(n+k)       | 是       |
| 基数排序     | O(N∗M)O(N∗M)     | O(N∗M)O(N∗M)     | O(M)O(M)           | 是       |

## 快排 *O(*nlog2n)

大致分三步：

1、找基准（一般是以中间项为基准）

2、遍历数组，小于基准的放在left，大于基准的放在right

3、递归

```js
function quickSort(arr){
  //如果数组<=1,则直接返回
  if(arr.length<=1){return arr;}
  var pivotIndex=Math.floor(arr.length/2);
  //找基准，并把基准从原数组删除
  var pivot=arr.splice(pivotIndex,1)[0];
  //定义左右数组
  var left=[];
  var right=[];

  //比基准小的放在left，比基准大的放在right
  for(var i=0;i<arr.length;i++){
    if(arr[i]<=pivot){
      left.push(arr[i]);
    }
    else{
      right.push(arr[i]);
    }
  }
  //递归（将这三个拼接起来）
  return quickSort(left).concat([pivot],quickSort(right));
}                
```



## 去重



[...new Set(arr)]

map

## 两数和（空）