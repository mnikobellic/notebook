## CSS

### Navigator

1. [Flex入坑指南](#flex入坑指南)
2. [纯CSS实现的tabbar切换](#纯css实现的tabbar切换)
3. [一个标准的九宫格实现](#一个标准的九宫格实现)
4. [纯CSS实现文本颜色与背景为反色](#纯css实现文本颜色与背景为反色)
5. [纯CSS实现的多层级菜单栏](#纯css实现的多层级菜单栏)

### Notes

#### Flex入坑指南

> [源码](./html/flex)  
> [Blog](https://blog.jiasm.org/2018/06/03/Flex入坑指南/)

`Flex`相关属性的基本用法，以及一些需要注意的小细节（`grow`的取值、`align-items`与`align-content`的区别）  

#### 纯CSS实现的tabbar切换

> （`:target`与`label + input`两种实现方式）  
> [源码](./dist/css/navigator-bar.scss)  
> [Live Demo](https://blog.jiasm.org/notebook/html/pure-css-tab-bar.html)

我们要实现一个纯CSS打造的tab切换，首先我们需要有这样一个结构的元素：
```html
<div class="nav-wrap">
  <div class="nav">
    Nav 1
  </div>
  <div class="nav">
    Nav 2
  </div>
</div>
<div class="container">
  <div class="content" data-index="1">
    Content 1
  </div>
  <div class="content" data-index="2">
    Content 2
  </div>
</div>
```
上边的元素是`tab`选项卡的按钮，下边的为选项卡对应的内容。  
以及对应的CSS样式大致如下：
```css
.nav-wrap {
  display: flex;
}

.nav {
  flex: 1 1 auto;
}

/* 默认隐藏所有的内容 */
.content {
  display: none;
}
```

##### label+input的实现方式

这里说的`input`是指`<input type="radio" />`，`radio`是单选框，所以比较符合我们的需求。  
所以我们要对上边的结构进行如下修改；
首先，是在`tab`选项卡按钮上添加`label`元素，以便可以在点击的时候选中对应的`input`。  
*P.S. 不直接在nav里边写input的原因是，nav通过CSS选择器的方式无法与下边的content产生联系*  

所以修改后的结构如下：  
```html
<input id="tab-1" type="radio" />
<input id="tab-2" type="radio" />
<div class="nav-wrap">
  <div class="nav" data-index="1">
    <label for="tab-1">Nav 1</label>
  </div>
  <div class="nav" data-index="2">
    <label for="tab-2">Nav 2</label>
  </div>
</div>
<div class="container">
  <div class="content" data-index="1">
    Content 1
  </div>
  <div class="content" data-index="2">
    Content 2
  </div>
</div>
```

然后我们就可以在CSS中这样写选择器了：
```css
#tab-1:checked ~ .nav-wrap .nav[data-index="1"] {
  background-color: gray;
  color: #fff;
}

#tab-1:checked ~ .container .content[data-index="1"] {
  display: block;
}
```

如果我们要实现第一个`tab`默认选中，仅需要将第一个`tab`对应的`input`设为`checked`即可`<input type="radio" checked />`  

##### :target的实现方式

`:target`是一个新的伪选择器，这个值代表了当前URL中的`hash`，也就是说，如果URL为`baidu.com/#name`  
则当前`:target`表示为`#name`，也就是我们熟悉的ID选择器了。  

所以，与上边的`label+input`版本的区别就在于，我们将`input`替换为一个空的携带ID的元素，或者直在对应的`content`元素上设置ID。  
*我们这里采用的是直接使用content元素+ID的方式，所以对结构进行如下修改*  

```html
<div class="container">
  <div class="content" id="tab-2">
    Content 2
  </div>
  <div class="content" id="tab-1">
    Content 1
  </div>
  <div class="nav-wrap">
    <div class="nav" data-index="1">
      <a href="#tab-1">Nav 1</a>
    </div>
    <div class="nav" data-index="2">
      <a href="#tab-2">Nav 2</a>
    </div>
  </div>
</div>
```

因为包含ID的元素必然要写在DOM树的前边，因为只有`~`选择器，并没有反向的。  
但是在用户看来，一定还是要是正常的，所以这里刚好用到了一个`flex`的属性。
这样会将所有的元素进行反排。
```css
.container {
  flex-direction: column-reverse;
}
```

同样是因为只有`~`选择器，所以我们在没有`:target`的时候要显示第一个`content`，但这时如果URL中的hash进行了改变，如何禁用掉这个默认值就成为了一个问题。

所以，我们还需要将所用的`content`进行倒序输出，这是为了能够填充默认值，默认显示第一个`content`。  

这样我们在实际DOM中的默认勾选`content`为最后一个，而只要`:target`后边可以匹配到`content`，都可以在这里禁用掉默认值的处理。  
代码如下：

```css
.container {
  display: flex;
  flex-direction: column-reverse;
}

/* default value */
.content:first-of-type {
  display: block;
}

.content:first-of-type ~ .nav-wrap .nav[data-index="1"] {
  background-color: gray;
  color: #fff;
}

/* disable default value */
.content:target ~ .content:first-of-type {
  display: none;
}

.content:target ~ .nav-wrap .nav[data-index="1"] {
  background-color: inherit;
  color: inherit;
}
```

![](/dist/img/example-pure-css-tabbar.gif)

#### 一个标准的九宫格实现

> [源码](./html/flex/examples/sudoku.html)
> [Live Demo](https://blog.jiasm.org/notebook/html/flex/examples/sudoku)

使用`flex`相关属性来实现一个响应式的九宫格样式。  

首先，需要如下结构的`html`：  
```html
<div class="container">
  <div class="item">
    <div class="item-content">Item 1</div>
  </div>
  <!-- ... -->
  <div class="item">
    <div class="item-content">Item 9</div>
  </div>
</div>
```

然后是要实现的需求：
1. 九宫格，每行三个
2. 每格的宽高一致，均为正方形
3. 单元格之间的间隙相等，为`10px`

首先为了实现每行三个元素，我们使用`flex + flex-wrap`。  
```css
.container {
  display: flex;
  flex-wrap: wrap;
}
.item {
  width: 33.33%;
}
```

上边的设置已经实现了每行填充三个，但是这时候我们并没有对`item`设置高度。  
这里就用到了`padding-top`，在文档中定义了，`padding`属性的百分比取值为父容器的`width`。  
所以，我们使用`padding-top: 100%`就可以变相的获取到宽度，并用之撑开容器，得到一个正方形。  

```css
.item::before {
  content: '';
  display: block;
  padding-top: 100%;
}
```

这时的元素依然会很诡异，因为`::before`在撑开元素的同时，也在元素中占据了一部分空间。  
导致我们的内容被挤到了下边，为了解决这个问题，我们需要将内容元素设置为`absolute`并悬浮在`item`上。  
```css
.container {
  position: relative; /* 避免子元素absolute定位脱离容器 */
}
.item .item-content {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #8BC34A;
}
```

这样我们就得到了九个正方形，每行三个。  
以及只剩下最后一个问题，如何实现单元格的间隙相等，保证为`10px`？  
因为我们上边为了保证一行三个元素，宽度写死了`33.33%`，这个是不能修改的。  
所以我们要用一个取巧的方式。
```css
.item .item-content {
  $gap: 10px;

  top: ($gap / 2);
  left: ($gap / 2);
  right: ($gap / 2);
  bottom: ($gap / 2);
}
```
对每个元素都设置`5px`的间隙。  
这样确实在元素中间都能过出现`10px`的间隙，但是在容器的外层一圈却只有`5px`间隙。  
所以我们还要对容器进行如下修改：  
```css
.container {
  $gap: 10px;

  padding: ($gap / 2);
}
```
在容器层也添加了`5px`的间隙，这样就能保证`item`所有的间隔均为`10px`了，一个九宫格，done。  
![](./dist/img/example-sudoku.png)

### Just Demos

#### 纯CSS实现文本颜色与背景为反色

> [源码](./html/invert-background-color-2-text-color.html)  
> [Live Demo](https://blog.jiasm.org/notebook/html/invert-background-color-2-text-color.html)

#### 纯CSS实现的多层级悬浮菜单栏

> [源码](./html/deep-child-menu.html)  
> [Live Demo](https://blog.jiasm.org/notebook/html/deep-child-menu.html)
