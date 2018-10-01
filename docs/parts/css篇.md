### width、height、margin、padding、top百分比时相对于谁

width百分比时的规则同top、left等基本相同，absolute相对于非static，fixed相对于body。

height没有这些问题，但是父元素的直系子元素的height设置百分比时，只在父元素设置了height时才有用。



margin和padding百分比均是相对于父元素的宽度



top、left无论百分比还是数值，都是子元素的margin外边缘相对于父元素padding外边缘定位。（同时参照第一条规则，决定相对于的父元素）

### 水平垂直居中

父元素：flex、align-items（垂直center）、justify-content（水平center（space-around、space-between））

子元素：absolute、transform（translate）

父元素：text-align、子元素：line-height为父元素高度

### BFC

内部垂直margin折叠，外部margin相加，可以清楚浮动。

**创建BFC的方式**

1.display为非block、非inline

2.position为absolute、fixed

3.float非none

4.body根元素

5.overflow为非visible

6.IE种也可以设置zoom为1（缩放）

### 优先级

!important

内联    1000

id选择器     100

class、属性、伪类     10

元素、伪元素      1

通配、>、+         0

### 行标签特点

高宽不起作用，左右margin无效。

存在一些可替换内联标签，比如img、input、textarea、object等等，就是可以设置这些的。



inline和inline-block都会因为换行引起4px到8px的水平间隙，可以通过负的margin去掉（直接设置为-4px或者-8px，因为非bfc等margin会折叠）

### margin折叠

margin父子折叠、兄弟折叠

### css3新特性

css3选择器

```css
Body > .mainTabContainer  div  > span[5]{}
```



### 选择器优先级

1，2，3优先级递增

1. **类型选择器**（type selectors）（例如, h1）和 **伪元素**（pseudo-elements）（例如, ::before）1
2. **类选择器**（class selectors） (例如,`.example`)，**属性选择器**（attributes selectors）（例如, `[type="radio"]`），**伪类**（pseudo-classes）（例如, :hover）10
3. **ID选择器**（例如, #example）100
4. 内联样式 1000

注意：通用选择器（*），子选择器（>）和相邻同胞选择器（+）并不在这四个等级中，所以他们的权值都为0。

### 测试题

### **行标签**都有哪些？有什么特点？

span,a,img,button,

高宽不起作用，padding都有用，上下margin没用，靠内容撑开高度。

但是img就可以设置这些。具体来说，

img元素是**可替换内联元素**，可替换就是浏览器根据元素的标签和属性，来决定元素的具体显示形式，img、input、textarea、select、object都是替换元素，替换元素一般有内在尺寸，所以具有width/height，可以设定，当不指定img的width和height的时候，就按照其内在尺寸显示，也就是图片被保存的时候的宽度和高度

### href、src差别

href和src是有区别的，而且是不能相互替换的。我们在可替换的元素上使用src，然而把href用于在涉及的文档和外部资源之间建立一个关系。

### 可替换元素和不可替换元素

- 替换元素：浏览器根据元素的标签和属性，决定具体显示的内容。

比如根据img的src来显示，input根据type来显示不同的输入框。如果查看html代码，看不到实际的内容。

<img>、<input>、<textarea>、<select>、<object>都是替换元素。这些元素往往没有实际的内容，即是一个空元素。

- 不可替换元素：(X)HTML 的大多数元素是不可替换元素，即其内容直接表现给用户端（例如浏览器）。

**替换元素一般有内在尺寸，所以具有width和height，可以设定。例如你不指定img的width和height时，就按其内在尺寸显示，也就是图片被保存的时候的宽度和高度。**

**对于表单元素，浏览器也有默认的样式，包括宽度和高度。**



### 按块级元素和行内元素来分类。

- 块级元素：默认在横向充满其父元素的内容区域，而且在其左右两边没有其他元素，即块级元素默认是独占一行的。
- 行内元素：不形成新内容块，即在其左右可以有其他元素，通过设置float、overflow：hidden等等可以形成块级格式化上下文。

**行级元素中不能插入块级元素。**，这是说在HTML中是不合法的，即使在css中将display反过来，这样也不行（并不会报什么错，就是实际渲染出来，行级还是不会包住块级）。如果是块级中放个行级，即使css中将display反过来，也还是可以。



### float和display：inline-block

都可以设置高宽。

**float**：脱离文档流。呈环绕装排列（除非清楚浮动），如遇上行有空白，而当前元素大小可以挤进去，这个元素会在上行补位排列。默认是顶部对齐。

没有清楚浮动的话，外部块级元素不能包裹浮动元素。

**inline-block**：不脱离文档流。水平排列一行，即使元素高度不一，也会以高度最大的元素高度为行高（父元素的content高度），即使高度小的元素周围留空，也不回有第二行元素上浮补位。可以设置默认的垂直对齐基线（vertical-align: baseline;）。

CSS中使用display:inline-block;来样式化，在Firefox, Safari, Google Chrome 和 IE 8及以上是有效的。但是在早期的IE，比如IE 7，就要做一些改变才能适应。
​    /* For IE 7 */
​    zoom: 1;
​    *display: inline;

inline和inline-block的元素之间经常会有空白符引起4px的间隔。

行级元素之间的换行符会当成空格处理。（是由于标签换行）

也可以使用负的margin，：IE8-9、Firefox、Safari等浏览器下4px，换句话说，这种现像只有在这几种浏览器中才会出现。下面我们就来说说解决这个“4px”(Chrome下是8px)

```html
<li>1</li><li>2</li><li>3</li>
<li>4</li>
```

会显示出 123 4。

### 清除浮动的方式

- 浮动元素后面加空div标签 clear:both ；（div沾满一行，默认在浮动块的下面，空div高度为0，所以可以撑开父元素的高度）

- 父级div定义 伪类:after 和 zoom ；

  ```css
  .clearfloat:：after{
    display:block;
    clear:both;
    content:"";
    visibility:hidden;
    height:0} /* IE8以下不支持after微元素 */
  .clearfloat{zoom:1} /* 激活IE中的BFC 原来是放大缩小的意思 */
  ```

  ```html	
  <div class="div1 clearfloat"> 
  	<div class="left">Left</div> 
  	<div class="right">Right</div> 
  </div> 

  ```

- 创建BFC。

### 用css实现一个自适应正方形容器

1. ​

```css
.abc {
  width: 100px;
  height: 200px;
  background: orange;
}
.rect {
  width: 100px;
  height: 0;
  background: purple;
  padding-bottom: 100%; /* 设置margin的话，background就不起作用了 */
}
```

```html
<div class="abc">
  <div class="rect">
    111111111111111111111112222222222222222222222
    2222222222222222222222222222222222222222222222222222222222
    33333333333333333333333333333333333333333333333333333333
    44444444444444444444444444444444444444444444444444444444444444444444
  </div>
</div>
```

margin、padding设置为百分数时，是**父元素的宽度**决定的。再将height设置为0，就可以实现自适应的正方形，但是，还会存在一个问题，height设为0了，max-height会失效（因为本身height就是0），min-height依然有效，height立刻增加至设置值。

**PS：**顺带一提，background的有效范围（border以内，包括border，因为border颜色设置为transparent时，还是background设置的颜色）。

### margin-college

包括父子折叠和兄弟折叠，都是垂直方向上的折叠。

父子折叠：两个嵌套的盒子中间没有非空内容间隔，则垂直margin会合并。

兄弟折叠：两个普通流元素（非浮动非定位）的margin也会折叠。

**计算**：都是正的，取最大的；都是负的，取绝对值最大的；有正有负，取正绝对值最大和负绝对值最大相加。

如同时存在父子兄弟嵌套，是一起计算（找多个中最大的那个），而不是各算各的再一起，不能先分步再合并。

解决方式：创造一个BFC。（或者根据定义，中间放一个不为空的盒子？）

**BFC的特点和创建方式**

特点：

- 内部（垂直margin折叠）margin-college折叠，外部相加。（也不存在父子折叠）
- 内部不影响外部。
- 不被浮动元素覆盖，清除浮动。
- 高度会包含其内的浮动元素。

创建方式：

- overflow: hidden
- position: relative和fixed
- float: 非none
- body根元素
- display为非块级（block）的块容器（比如flex、inline-block、table-cell等等）

### :nth-of-type(*n*) 

选择器匹配属于父元素的特定类型的第 n 个子元素的每个元素.



### 单行文本溢出显示 ...

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
width: 20px; 
```

### 多行文本溢出显示 ...

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```

-webkit-line-clamp用来限制在一个块元素显示的文本的行数。 为了实现该效果，它需要组合其他的WebKit属性。常见结合属性：

display: -webkit-box; 必须结合的属性 ，将对象作为弹性伸缩盒子模型显示 。

-webkit-box-orient 必须结合的属性 ，设置或检索伸缩盒对象的子元素的排列方式 。

### flex

```css
justify-content：center，space-between，space-around; /*水平居中*/
align-items：center; /*垂直居中*/
```

### animation

```css
@-webkit-keyframes anim1 { 
   0% { 
       Opacity: 0; 
Font-size: 12px; 
   } 
   100% { 
       Opacity: 1; 
Font-size: 24px; 
   } 
} 
.anim1Div { 
   -webkit-animation-name: anim1 ; 
   -webkit-animation-duration: 1.5s; /*动画持续时间*/
   -webkit-animation-iteration-count: 4; /*动画重复次数*/
   -webkit-animation-direction: alternate; /*动画执行完一次后方向的变化方式（如第一次从右向左，第二次则从左向右）alternate为轮流*/
   -webkit-animation-timing-function: ease-in-out; /*变化模式 s形？ease-in: slow at the beginning, fast/abrupt at the end
ease-out: fast/abrupt at the beginning, slow at the end*/
}
```



## 布局

### 居中

1.  绝对定位

```css
position：absolute;
top/left: 50%;
transform: translate(-50%, -50%);
```

2. flex

```css
display: flex;
justify-content: center
```

3. text-align

   需设置在块级元素，表格单元格，行内块元素 上，使内部文本或元素居中。

4. display：table-cell+text-align: center

### 两栏式布局

```css
// float
.box{
  width: 300px;
  height: 300px;
}
.left {
  width: 100px;
  float: left;
  height: 300px;
  background: #000000;
}
.right {
  margin-left: 100px;
  height: 300px;
  background: green;
}
```

```css
// flex
.box{
  width: 300px;
  height: 300px;
  display: flex;
}
.left {
  width: 100px;
  height: 300px;
  background: #000000;
}
.right {
  flex-grow: 1;
  height: 300px;
  background: green;
}
```

```html
    <style>
#header, #footer{ 
    height: 100px;
    background: red;
 }
#content .main{
    height: 200px;
    background: green;
    overflow: auto;
}
#content .aside{
    height: 200px;
    width: 100px;
    background: blue;
    float: left;
}
</style>
<body>
<div id="header"></div>
<div id="content">
<div class="aside"></div>
<div class="main">
    main main main main main main
</div>
</div>
<div id="footer"></div>
```

### 页脚自适应

父级设置min-height 100vh，flex-direction：column，中间内容设置flex-grow为1。

## 盒模型

**ie盒子模型**: content的宽度和高度包括了border和padding。

**标准w3c盒子模型**: 不包含。（可设置box-sizing切换（border-box和content-box））。



## BFC

可以包裹浮动元素（内部有浮动元素）。也可以清除浮动（不被浮动元素覆盖 ）（周围有浮动元素）

1. 里面的不会影响到外边的。
2. 内部元素margin折叠，外部元素相加。
3. 不被浮动元素覆盖 ，清除浮动

**创建BFC**

- body 根元素
- 浮动元素：float 除 none 以外的值
- 绝对定位元素：position (absolute、fixed)
- display 为 inline-block、table-cells（没有margin）、flex等等（非block、inline）
- overflow 除了 visible 以外的值 (hidden、auto、scroll)

##　优先级

1. 通用选择器 *

2. 元素（类型）选择器

3. 类选择器

4. 属性选择器

5. 伪类

6. ID 选择器

7. 内联样式

   ​

   内联样式表的权值最高 1000。

   `ID` 选择器的权值为 100。

   `Class` 类选择器的权值为 10。

   `HTML` 标签（类型）选择器的权值为 1。