---
layout: post
title: "Web 前端开发小测验, part 1 之 CSS"
date: 2014-03-09 01:30
comments: true
categories: [web, css]
---

这是 [@devqin](http://www.devqin.com/) 在 [NADbb](http://bb.ijser.cn/topic/46/%E7%9C%8B%E7%9C%8B%E4%BD%A0%E8%83%BD%E5%BE%97%E5%88%B0%E5%A4%9A%E5%B0%91%E5%88%86-%E8%87%AA%E6%88%91%E6%84%9F%E8%A7%89%E8%89%AF%E5%A5%BD%E8%80%85%E8%8E%AB%E5%85%A5-%E5%9B%A0%E4%B8%BA-%E5%AE%98%E6%96%B9%E8%AF%B4%E7%9A%84%E6%98%AF-maight-hurt-your-feelings) 上发的一个找虐的测试. (原作者有个提示: Warning: might hurt your feelings). 

我来挨个找证据, 今天是 part 1, CSS 部分.


####1)

```css
ul {
    MaRGin: 10px;
}
```
Are CSS property names case-sensitive?

CSS 属性名是大小写敏感的吗 ?

**答**: 不敏感. [Cascading Style Sheets, level 1][css1] 的 [7.1 Forward-compatible parsing][css1.71] 最后有这么一段话, 

> All CSS style sheets are case-insensitive, except for parts that are not under the control of CSS. I.e., in CSS1, font family names and URLs can be case-sensitive. Also, the case-sensitivity of the CLASS and ID attributes is under the control of HTML. 

因此, 只有不受 CSS 控制的, 如 `font family` 的名字, `url`, 以及受 `HTML` 控制的 `ID` 和 `class` 大小写敏感, 其他受 CSS 控制的内容都是大小写不敏感的.


####2) Does setting `margin-top` and `margin-bottom` have an affect on an inline element ?

`margin-top` 与 `margin-bottom` 对内联元素是否有效 ?

**答**: 没效果, [Cascading Style Sheets, level 1][css1] 的 [4.2 Inline elements][css1.42] 中有如下描述:

> If the inline element has margins, borders, padding or text decorations attached, these will have no effect where the element is broken. 

CSS2.1 中, [8.3 Margin properties: 'margin-top', 'margin-right', 'margin-bottom', 'margin-left', and 'margin'][css21.83] 节关于 `margin-top` 和 `margin-bottom` 有如下描述:

> These properties have no effect on non-replaced inline elements.

关于 non-replaced 与 replaced element 的定义可以参考 CSS2.1 中的 [Replaced element][css21.31.re]


####3) Does setting `padding-top` and `padding-bottom` on an inline element add to its dimensions ?

<!-- more -->
`padding-top` 与 `padding-bottom` 会增加内联元素的大小吗 ?

**答**: 不会, [Cascading Style Sheets, level 1][css1] 的 [4.2 Inline elements][css1.42] 中有如下描述:

> If the inline element has margins, borders, padding or text decorations attached, these will have no effect where the element is broken.
 
CSS2.1 中, [10.6.1 Inline, non-replaced elements][css21.1061] 节, 有如下描述:
 
> The vertical padding, border and margin of an inline, non-replaced box start at the top and bottom of the content area, and has nothing to do with the 'line-height'. But only the 'line-height' is used when calculating the height of the line box.

因此, 只有 `line-height` 属性才能改变内联元素的高度.


####4) If you have a `<p>` element with `font-size: 10rem`, will the text be responsive when the user resizes / drags the browser window?

当拖动浏览器窗口大小时, 有 `font-size: 10rem` 属性的 `<p>` 元素中的文本内容是否会有反应 ?

**答**: 不会有反应. CSS3 中, 对 [rem unit][css3rem] 有如下描述:
 
> Equal to the computed value of ‘font-size’ on the root element. When specified on the ‘font-size’ property of the root element, the ‘rem’ units refer to the property's initial value.
 
因此 `<p>` 元素中的文本内容只与根元素的字体大小有关, (主流的浏览器, 默认根元素的 `font-size` 为 `16px`), 而拖动窗口大小是不会改变根元素的 `font-size` 属性值的. 所以不会有反应.


####5) The pseudo class `:checked` will select inputs with type radio or checkbox, but not `<option>` elements.

伪类 `:checked` 会作用域 `radio` 或者 `checkbox`, 但是不会作用于 `<option>`.

**答**: CSS3 中, [6.6.4.2. The :checked pseudo-class][css3.664] 中有如下描述:
 
> Radio and checkbox elements can be toggled by the user. Some menu items are "checked" when the user selects them. When such elements are toggled "on" the :checked pseudo-class applies. While the :checked pseudo-class is dynamic in nature, and can altered by user action, since it can also be based on the presence of semantic attributes in the document, it applies to all media. For example, the :checked pseudo-class initially applies to such elements that have the HTML4 selected and checked attributes as described in Section 17.2.1 of HTML4, but of course the user can toggle "off" such elements in which case the :checked pseudo-class would no longer apply.
 
从上面的描述中可以看出, 除了 radio 和 checkbox 之外, 伪类 `:checked` 也可以作用于 HTML 标准中含有 `selected` 和 `checked` 属性的元素,  在 HTML4 [17.2.1 Control types][html4.1721] 一节的描述中包含 option, 因此 `:checked` 可以作用于 `<option>`.


####6) In a HTML document, the pseudo class `:root` always refers to the `<html>` element.

在一个 HTML 文档中, 伪类 `:root` 总是代表 `<html>` 元素.

**答**: 是. CSS3 [6.6.5.1. :root pseudo-class][css3.6651] 一节, 有如下描述: 

> The :root pseudo-class represents an element that is the root of the document. In HTML 4, this is always the HTML element.

而 HTML5 [4.1 The root element][html5.41] 一节, 也有如下描述:

> The html element represents the root of an HTML document.

所以, `:root` 就是 `<html>` 元素.


####7) The `translate()` function can move the position of an element on the z-axis.

函数 `translate()` 可以在 z 轴上移动元素的位置.

**答**: 错误. CSS Transforms Module Level 1 中 [translate][css3.translate] 的定义如下: 
 
`translate() = translate( <translation-value> [, <translation-value> ]? )`
 
> specifies a 2D translation by the vector [tx, ty], where tx is the first translation-value parameter and ty is the optional second translation-value parameter. If <ty> is not provided, ty has zero as a value.

所以 `translate()` 函数是一个 2D 变换的函数, 仅仅能改变 x 与 y 轴的位置, 不能改变 z 轴的位置.


####8)

HTML:

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

**答**: 蓝色.

首先来看一下标准中的描述, [Cascading Style Sheets, level 1][css1] 中 [3.2 Cascading order][css1.32] 一节, 有如下描述.
 
> Conflicting rules are intrinsic to the CSS mechanism. To find the value for an element/property combination, the following algorithm must be followed:
>
>  1.  Find all declarations that apply to the element/property in question. Declarations apply if the selector matches the element in question. If no declarations apply, the inherited value is used. If there is no inherited value (this is the case for the 'HTML' element and for properties that do not inherit), the initial value is used.
>
>  2. Sort the declarations by explicit weight: declarations marked '!important' carry more weight than unmarked (normal) declarations.
>
>  3. Sort by origin: the author's style sheets override the reader's style sheet which override the UA's default values. An imported style sheet has the same origin as the style sheet from which it is imported.
>
>  4. Sort by specificity of selector: more specific selectors will override more general ones. To find the specificity, count the number of ID attributes in the selector (a), the number of CLASS attributes in the selector (b), and the number of tag names in the selector \(c\). Concatenating the three numbers (in a number system with a large base) gives the specificity.
>
>  5. Sort by order specified: if two rules have the same weight, the latter specified wins. Rules in imported style sheets are considered to be before any rules in the style sheet itself.

第 4 条关于权重的细节跳过. 

根据第 1 条, 显然 `li` 比 `ul` 元素匹配更精确, 因此选择 `li` 蓝色.


####9)

HTML:

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

**答**: 蓝色.

根据标准中的描述:

1. 元素选择的准确程度, 两个一致, 都选择到了 `span` 的父元素 `li`.
2. 显式声明的权重也一致.
3. 这里不用考虑 CSS 文件, 用户, 以及浏览器默认的样式覆盖问题.
4. 根据选择器的权重, `ID` 获胜.

因此答案是: 蓝色.


####10)

Given the HTML below:

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

**答**: 蓝色.

根据标准中的描述:

1. 元素选择的准确程度, 两个一致, 都选择到了 `span` 的父元素 `li`.
2. 显式声明的权重也一致.
3. 这里不用考虑 CSS 文件, 用户, 以及浏览器默认的样式覆盖问题.
4. 根据选择器的权重, `ID` 获胜.

因此答案是: 蓝色.


####11)

Given the HTML below:

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

**答**: 蓝色.

根据标准中的描述:

1. 元素选择的准确程度不一致, 虽然 `ul#awesome` 的权重很高, 但是只选到了 `span` 的父元素 `li`.

因此答案是: 蓝色.


####12)

Given the HTML below:

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

**答**: 蓝色.

根据标准中的描述:

1. 元素选择的准确程度不一致, 虽然 `ul#awesome #must-buy` 的权重很高, 但是只选到了 `span` 的父元素 `li`.

因此答案是: 蓝色. (即使没有 `!important` 也是蓝色.


####13)

Given the HTML below:

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

**答**: 蓝色.

根据标准中的描述:

1. 元素选择的准确程度, 两个一致, 都选择到了 `span` 的父元素 `li`.
2. 显式声明的权重也一致.
3. 这里不用考虑 CSS 文件, 用户, 以及浏览器默认的样式覆盖问题.
4. `ul.shopping-list li .highlight:nth-of-type(odd)` 多了一个伪类, 权重多 10.

因此答案是: 蓝色.


####14)

Given the HTML below:

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

根据标准中的描述:

1. 元素选择的准确程度, 两个一致, 都选择到了 `span` 的父元素 `li`.
2. 显式声明的权重也一致.
3. 这里不用考虑 CSS 文件, 用户, 以及浏览器默认的样式覆盖问题.
4. 这个题目是权重的题目, 但是 `#awesome .favorite:not(#awesome) .highlight` 的权重究竟是多少呢 ?

CSS3 selectors [9. Calculating a selector's specificity][css3] 一节中有如下描述,

> Selectors inside the negation pseudo-class are counted like any other, but the negation itself does not count as a pseudo-class.

所以, 很明显, `#awesome .favorite:not(#awesome) .highlight` 有两个 `ID` 两个 `class` 权重 220, 远远大于 `#awesome .highlight:nth-of-type(1):nth-last-of-type(1)` 的 130

因此答案是: 蓝色.


####15)

HTML:

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

**答**: 后面的元素都向上移动 5px.

CSS2.1 标准 [8.4 Padding properties][css21.84] 一节中有如下描述:

> Negative values for margin properties are allowed, but there may be implementation-specific limits.
 
这个题目是 browser specific 的, 不过大部分主流的浏览器实现方式都是一致的. 可以参考这里: http://coding.smashingmagazine.com/2009/07/27/the-definitive-guide-to-using-negative-margins/


####16)

HTML:

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

**答**: 像左←移动 5px.

CSS2.1 标准 [8.4 Padding properties][css21.84] 一节中有如下描述:

> Negative values for margin properties are allowed, but there may be implementation-specific limits.
 
这个题目是 browser specific 的, 不过大部分主流的浏览器实现方式都是一致的. 可以参考这里: http://coding.smashingmagazine.com/2009/07/27/the-definitive-guide-to-using-negative-margins/


####17)

HTML:

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

**答**: 不会. 这又是一个 browser specific 的题目. 不过主流的浏览器都会遵循 `lazy-loading` 的原则. 因此这个图片不会下载.


####18)

HTML:

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

**答**: 会. 虽然属性 `display` 的值为 `none`, 但是图片仍然会被下载, 因为 css 文件解析是自上而下的, 因此当解析到 `background-image` 时, 没有足够的信息表明 test2 将不会显示时, 因此图片会下载.


####19)
HTML:

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

**答**: 不会. 与上面的分析一致, 只是此时已有足够的信息表明 `test2` 将不会显示.


####20)

CSS:

```css
@media only screen and (max-width: 1024px) {
    margin: 0;
}
```

What is the use of the `only` selector ?

`only` 选择器的作用是什么 ?

**答**: Stops older browsers from parsing the remainder of the selector. 是阻止旧浏览器解析剩余的选择器的.

在 CSS3 [Media Queries][css3.mq] 中有如下描述:

> The keyword ‘only’ can also be used to hide style sheets from older user agents. User agents must process media queries starting with ‘only’ as if the ‘only’ keyword was not present.

所以, 关键字 `only` 就是为了不让旧浏览器正确解析的.


####21)

HTML:

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

Does `overflow: hidden` create a new block formatting context ?

`overflow: hidden` 会创建一个新的块级格式化上下文吗 ?

**答**: 会. CSS2.1 [9.4.1 Block formatting contexts][css21.941] 一节中有如下描述:
 
> Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.

因此, 只要 `overflow` 的值不是 `visible` 时, 都会创建一个新的块级格式化上下文.


####22)
CSS:

```css
@media only screen and (max-width: 1024px) {
    margin: 0;
}
```

Does the `screen` keyword apply to the device's physical screen or the browser's viewport ?

关键字 `screen` 指的是设备的物理屏幕, 还是指的浏览器的 viewport ?

**答**: CSS2 [Media types][css2.73] 一节, 有如下描述,

> **screen**
>
> Intended primarily for color computer screens.

因此, 答案应该是设备的物理屏幕, 但是原作者给的答案是浏览器的 viewport. (不知所措. o(>﹏<)o


`HTML`, 以及 `Javascript` 的后续部分, 敬请期待 ~O(∩_∩)O~


  [css1]: http://www.w3.org/TR/2008/REC-CSS1-20080411/
  [css1.32]: http://www.w3.org/TR/2008/REC-CSS1-20080411/#cascading-order
  [css1.42]: http://www.w3.org/TR/2008/REC-CSS1-20080411/#inline-elements
  [css1.71]: http://www.w3.org/TR/2008/REC-CSS1-20080411/#forward-compatible-parsing
  [css2.73]: http://www.w3.org/TR/CSS2/media.html#media-types 
  [css21.83]: http://www.w3.org/TR/CSS21/box.html#propdef-margin-bottom
  [css21.84]: http://www.w3.org/TR/CSS21/box.html#padding-properties
  [css21.31.re]: http://www.w3.org/TR/CSS21/conform.html#replaced-element
  [css21.1061]: http://www.w3.org/TR/CSS21/visudet.html#Computing_heights_and_margins
  [css3rem]: http://www.w3.org/TR/css3-values/#font-relative-lengths
  [css3.664]: http://www.w3.org/TR/css3-selectors/#UIstates
  [css3.6651]: http://www.w3.org/TR/css3-selectors/#root-pseudo
  [css3.translate]: http://www.w3.org/TR/css3-transforms/#funcdef-translate
  [css3.mq]: http://www.w3.org/TR/css3-mediaqueries/#media0
  [html4.1721]: http://www.w3.org/TR/REC-html40/interact/forms.html#h-17.2.1
  [html5.41]: http://www.w3.org/TR/html5/semantics.html#the-root-element
