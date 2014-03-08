---
layout: post
title: "Web 前端开发小测验, part 1 之 CSS"
date: 2014-03-09 01:30
comments: true
categories: [web, css]
---

1)

```css
ul {
    MaRGin: 10px;
}
```
Are CSS property names case-sensitive?

CSS 属性名是大小写敏感的吗?

2) Does setting `margin-top` and `margin-bottom` have an affect on an inline element ?

`margin-top` 与 `margin-bottom` 对内联元素是否有效 ?

3) Does setting `padding-top` and `padding-bottom` on an inline element add to its dimensions ?

`padding-top` 与 `padding-bottom` 会增加内联元素的大小吗 ?

4) If you have a `<p>` element with `font-size: 10rem`, will the text be responsive when the user resizes / drags the browser window?

当拖动浏览器窗口大小时, 有 `font-size: 10rem` 属性的 `<p>` 元素中的文本内容是否会有反应 ?

5) The pseudo class `:checked` will select inputs with type radio or checkbox, but not `<option>` elements.

伪类 `:checked` 会作用域 `radio` 或者 `checkbox`, 但是不会作用于 `<option>`.

6) In a HTML document, the pseudo class `:root` always refers to the `<html>` element.

在一个 HTML 文档中, 伪类 `:root` 总是代表 `<html>` 元素.

7) The `translate()` function can move the position of an element on the z-axis.

函数 `translate()` 可以在 z 轴上移动元素的位置.

8) HTML:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

CSS:

```css
ul {
    color: red;
}
li {
    color: blue;
}
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

9) HTML:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

CSS:

```css
ul li {
    color: red;
}
#must-buy {
    color: blue;
}
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?


10) Given the HTML below:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

```css
.shopping-list .favorite {
    color: red;
}
#must-buy {
    color: blue;
}
```

11) Given the HTML below:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

```css
ul#awesome {
    color: red;
}
ul.shopping-list li.favorite span {
    color: blue;
}
```

12) Given the HTML below:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

```css
ul#awesome #must-buy {
    color: red;
}
.favorite span {
    color: blue!important;
}
```

13) Given the HTML below:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

```css
ul.shopping-list li .highlight {
    color: red;
}
ul.shopping-list li .highlight:nth-of-type(odd) {
    color: blue;
}
```

14) Given the HTML below:

```html
<ul class="shopping-list" id="awesome">
    <li><span>Milk</span></li>
    <li class="favorite" id="must-buy"><span class="highlight">Sausage</span></li>
</ul>
```

What is the color of the text Sausage ?

Sausage 的颜色是 ?

```css
#awesome .favorite:not(#awesome) .highlight {
    color: red;
}
#awesome .highlight:nth-of-type(1):nth-last-of-type(1) {
    color: blue;
}
```

15) HTML:

```html
<p id="example">Hello</p>
```

CSS:

```css
#example {
    margin-bottom: -5px;
}
```

What will happen to the position of `#example` ?

`#example` 的位置会有何变化 ?

16) HTML:

```html
<p id="example">Hello</p>
```

CSS:

```css
#example {
    margin-left: -5px;
}
```

What will happen to the position of `#example` ?

`#example` 的位置会有何变化 ?

17. HTML:

```html
<div id="test1">
    <span id="test2"></span>
</div>
```

CSS:

```css
#i-am-useless {
    background-image: url('mypic.jpg');
}
```

Are unused style resources still downloaded by the browser?

浏览器会下载未使用的样式资源吗 ?


18) HTML:

```html
<div id="test1">
    <span id="test2"></span>
</div>
```

CSS:

```css
#test2 {
    background-image: url('mypic.jpg');
    display: none;
}
```

On page load, will `mypic.jpg` get downloaded by the browser ?

页面加载时, 浏览器会下载图片 `mypic.jpg` 吗 ?

19) HTML:

```html
<div id="test1">
    <span id="test2"></span>
</div>
```

CSS:

```css
#test1 {
    display: none;
}
#test2 {
    background-image: url('mypic.jpg');
    visibility: hidden;
}
```

On page load, will `mypic.jpg` get downloaded by the browser ?

页面加载时, 浏览器会下载图片 `mypic.jpg` 吗 ?

20) CSS:

```css
@media only screen and (max-width: 1024px) {
    margin: 0;
}
```

What is the use of the `only` selector ?

`only` 选择器的作用是什么 ?


21) HTML:

```html
<div>
    <p>I am floated</p>
    <p>So am I</p>
</div>
```

CSS:

```css
div {
    overflow: hidden;
}
p {
    float: left;
}
```

Does `overflow: hidden` create a new block formatting context?

`overflow: hidden` 会创建一个新的块级格式化上下文吗 ?

22) CSS:

```css
@media only screen and (max-width: 1024px) {
    margin: 0;
}
```

Does the `screen` keyword apply to the device's physical screen or the browser's viewport ?

关键字 `screen` 指的是设备的物理屏幕, 还是指的浏览器的 viewport ?
