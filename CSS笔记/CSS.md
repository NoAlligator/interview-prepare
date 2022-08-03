CSS属性速查

[参考](http://code.ciaoca.com/style/css-cheat-sheet/)

# 1.@规则

[CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)[@规则](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule)，用于从其他样式表导入样式规则。这些规则必须先于所有其他类型的规则，[`@charset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@charset) 规则除外。

- [@namespace](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%40namespace) 告诉 CSS 引擎必须考虑XML**命名空间**。
- [@media](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%40media), **如果满足媒体查询的条件则条件规则组里的规则生效**。
- [@font-face](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%40font-face), 描述将下载的**外部的字体**。
- [@keyframes](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%40keyframes), 描述 **CSS 动画的关键帧**。

## @import

详见性能优化

## @support

[@supports](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%40supports) 用于查询特定的 CSS 是否生效，可以结合 not、and 和 or 操作符进行后续的操作。

# 2.选择器

## 基础选择器

- 标签选择器：`h1`
- 类选择器：`.checked`
- ID 选择器：`#picker`
- 通配选择器：`*`

## 组合选择器

- 相邻兄弟选择器：`A + B`
- 普通兄弟选择器：`A ~ B`
- 子选择器：`A > B`
- 后代选择器：`A B`

## **属性选择器**

- `[attr]`：指定属性的元素；
- `[attr=val]`：属性**等于指定值**的元素；
- `[attr*=val]`：属性**包含指定值**的元素；
- `[attr^=val]`：属性以指定值**开头**的元素；
- `[attr$=val]`：属性以指定值**结尾**的元素；
- `[attr~=val]`：属性包含指定值(完整单词)的元素(不推荐使用)；
- `[attr|=val]`：属性以指定值(完整单词)开头的元素(不推荐使用)；

## 伪类

### **条件伪类**

- `:lang()`：基于元素语言来匹配页面元素；
- `:dir()`：匹配特定文字书写方向的元素；
- `:has()`：匹配包含指定元素的元素；
- `:is()`：匹配指定选择器列表里的元素；
- `:not()`：用来匹配不符合一组选择器的元素；

### **行为伪类**

- **`:active`：鼠标激活的元素；**
- **`:hover`：	鼠标悬浮的元素**；
- **`::selection`：鼠标选中的元素；**

### **状态伪类**

- `:target`：当前锚点的元素；
- **`:link`：未访问的链接元素；**
- **`:visited`：已访问的链接元素；**
- **`:focus`：输入聚焦的表单元素；**
- `:required`：输入必填的表单元素；
- `:valid`：输入合法的表单元素；
- `:invalid`：输入非法的表单元素；
- `:in-range`：输入范围以内的表单元素；
- `:out-of-range`：输入范围以外的表单元素；
- **`:checked`：选项选中的表单元素；**
- `:optional`：选项可选的表单元素；
- `:enabled`：事件启用的表单元素；
- `:disabled`：事件禁用的表单元素；
- `:read-only`：只读的表单元素；
- `:read-write`：可读可写的表单元素；
- `:blank`：输入为空的表单元素；
- `:current()`：浏览中的元素；
- `:past()`：已浏览的元素；
- `:future()`：未浏览的元素；

### **结构伪类**

- `:root`：文档的根元素；
- `:empty`：无子元素的元素；
- `:first-letter`：元素的首字母；
- `:first-line`：元素的首行；
- **`:nth-child(n)`：元素中指定顺序索引的元素；**
- **`:nth-last-child(n)`：元素中指定逆序索引的元素；；**
- **`:first-child	`：元素中为首的元素；**
- **`:last-child`	：元素中为尾的元素；**
- `:only-child`：父元素仅有该元素的元素；
- `:nth-of-type(n)	`：标签中指定顺序索引的标签；
- `:nth-last-of-type(n)`：标签中指定逆序索引的标签；
- `:first-of-type`	：标签中为首的标签；
- `:last-of-type`：标签中为尾标签；
- `:only-of-type`：父元素仅有该标签的标签；

### 链接伪类顺序

```css
a:link {}
a:visited {}
a:hover {}
a:active {}
```

## 伪元素

| 选择器                                                       | 例子            | 例子描述                      |
| :----------------------------------------------------------- | :-------------- | :---------------------------- |
| [::after](https://www.w3school.com.cn/cssref/selector_after.asp) | p::after        | 在每个 <p> 元素之后插入内容。 |
| [::before](https://www.w3school.com.cn/cssref/selector_before.asp) | p::before       | 在每个 <p> 元素之前插入内容。 |
| [::first-letter](https://www.w3school.com.cn/cssref/selector_first-letter.asp) | p::first-letter | 选择每个 <p> 元素的首字母。   |
| [::first-line](https://www.w3school.com.cn/cssref/selector_first-line.asp) | p::first-line   | 选择每个 <p> 元素的首行。     |
| [::selection](https://www.w3school.com.cn/cssref/selector_selection.asp) | p::selection    | 选择用户选择的元素部分。      |

### `::before`和`::after`的使用

- ::before和::after下特有的content，用于在css渲染中向元素逻辑上的头部或尾部添加内容。
- ::before和::after**必须配合content属性来使用，content用来定义插入的内容，content必须有值，至少是空**。
- 这些添加不会出现在DOM中，**不会改变文档内容，不可复制，仅仅是在css渲染层加入**。所以不要用::before或::after展示有实际意义的内容，尽量使用它们**显示修饰性内容**。
- content属性可以设置**attr()**和**url()/uri()**

# 3.继承性

**不会影响到页面布局的属性往往存在继承性**，如下属性具有继承性：

- 字体相关：`font-family`、`font-style`、`font-size`、`font-weight` 等；
- 文本相关：`text-align`、`text-indent`、`text-decoration`、`text-shadow`、`letter-spacing`、`word-spacing`、`white-space`、`line-height`、`color` 等；
- 列表相关：`list-style`、`list-style-image`、`list-style-type`、`list-style-position` 等；
- 其他属性：`visibility`、`cursor` 等；

对于其他默认不继承的属性也可以**通过以下几个属性值来控制继承行为**：

- `inherit`：继承父元素对应属性的计算值；
- `initial`：应用该属性的默认值，比如 color 的默认值是 `#000`；
- `unset`：如果属性是默认可以继承的，则取 `inherit` 的效果，否则同 `initial`；
- `revert`：效果等同于 `unset`，兼容性差。

# 4.优先级

**!important** > **内联样式** > **ID 选择器** > **类选择器**、伪类选择器、属性选择器 > **元素选择器**、伪元素选择器 > **通配选择器**、后代选择器、兄弟选择器

# 5.层叠性

层叠性原则：

- 选择器选中了有冲突就比较权重；
- 选择器没有选中有冲突遵循就近原则；
- 以上都一样的话，写在后面的会覆盖写再前面的样式。

# 6.文档流

在 CSS 的世界中，会把内容按照**从左到右、从上到下的顺序进行排列显示**。正常情况下会把页面分割成一行一行的显示，而每行又可能由多列组成，所以从视觉上看起来就是从上到下从左到右，而这就是 CSS 中的流式布局，又叫文档流，特征如下：

- 文本空白折叠；
- 内联元素高矮不齐，底边对齐；
- **块级元素默认会占满整行，所以多个块级盒子之间是从上到下排列的；**
- **内联元素默认会在一行里一列一列的排布，当一行放不下的时候，会自动切换到下一行继续按照列排布；**

# 7.盒模型

在 CSS 中任何元素都可以看成是一个盒子，**而一个盒子是由 4 部分组成的：内容（content）、内边距（padding）、边框（border）和外边距（margin）。**盒模型有 2 种：标准盒模型和 IE 盒模型，本别是由 W3C 和 IExplore 制定的标准。

![img](http://img.smyhvae.com/2015-10-03-css-27.jpg)

![img](http://img.smyhvae.com/2015-10-03-css-30.jpg)

## 设置盒模型

```css
p {
	box-sizing: border-box | content-box;
}
```

# 8.视觉格式化模型

> 外在盒子是决定元素排列方式的盒子，即决定盒子具有块级特性还是内联特性的盒子。外在盒子负责结构布局。内在盒子是决定元素内部一些属性是否生效的盒子。内在盒子负责内容显示。如 `display: inline-table;` 外在盒子就是`inline`，内在盒子就是`table`。外在盒子决定了元素要像内联元素一样并排在一排显示，内在盒子则决定了元素可以设置宽高、垂直方向的margin等属性。

视觉格式化模型（Visual formatting model）是用来处理和在视觉媒体上显示文档时使用的计算规则。CSS 中一切皆盒子，而视觉格式化模型简单来理解就是规定这些盒子应该怎么样放置到页面中去，这个模型在计算的时候会依赖到很多的因素，比如：盒子尺寸、盒子类型、定位方案（是浮动还是定位）、兄弟元素或者子元素以及一些别的因素。

盒子类型由 display 属性决定，同时给一个元素设置 display 后，将会决定这个盒子的 2 个显示类型（display type）：

- **outer display type（对外显示）**：决定了该元素本身是如何布局的，即**参与何种格式化上下文**；
- **inner display type（对内显示）**：其实就相当于**把该元素当成了容器，规定了其内部子元素是如何布局的**，参与何种格式化上下文；

## ⭐对外显示类型（Outer display type）

对外显示方面，盒子类型可以**分成 2 类**：block-level box（块级盒子） 和 inline-level box（行内级盒子）：

- 块级盒子：**display 为 block、list-item、table、flex、grid、flow-root 等；**
- 行内级盒子：**display 为 inline、inline-block、inline-table 等；**

> 所有块级盒子都会参与 BFC，呈现垂直排列；而所有行内级盒子都参会 IFC，呈现水平排列。

### **block**

- 默认宽度占满一行，默认继承父元素的宽度；**多个块元素将从上到下进行排列**；
- 设置 width/height 将会生效；
- 设置 padding 和 margin 将会生效；

### **inline**

- 不会占满一行，宽度随着内容而变化；多个 inline 元素将按照从左到右的顺序在一行里排列显示，如果一行显示不下，则自动换行；
- 设置 width/height 将不会生效；
- 设置竖直方向上的 padding 和 margin 将不会生效；

### **inline-block**

- 是行内块元素，不单独占满一行，可以看成是能够在一行里进行左右排列的块元素；
- 设置 width/height 将会生效；
- 设置 padding 和 margin 将会生效；

## ⭐对内显示类型（Inner display type）

对内方面，其实就是把元素当成了容器，里面包裹着文本或者其他子元素。container box 的类型依据 display 的值不同，分为 4 种：

- block container：建立 BFC 或者 IFC；
- flex container：建立 FFC；
- grid container：建立 GFC;
- ruby container：接触不多，不做介绍。

> 如果把 img 这种替换元素（replaced element）申明为 block 是不会产生 container box 的，因为替换元素比如 img 设计的初衷就仅仅是通过 src 把内容替换成图片，完全没考虑过会把它当成容器。
>

# 9.格式化上下文（对内显示类型）

> BFC、IFC、FFC、GFC

格式化上下文（Formatting Context）是 CSS2.1 规范中的一个概念，大概说的是页面中的一块渲染区域，规定了渲染区域内部的子元素是如何排版以及相互作用的。

不同类型的盒子有不同格式化上下文，大概有这 4 类：

- BFC (Block Formatting Context) 块级格式化上下文；
- IFC (Inline Formatting Context) 行内格式化上下文；
- FFC (Flex Formatting Context) 弹性格式化上下文；
- GFC (Grid Formatting Context) 格栅格式化上下文；

> 其中 BFC 和 IFC 在 CSS 中扮演着非常重要的角色，因为它们直接影响了网页布局，所以需要深入理解其原理。

## ⭐BFC块级格式化上下文

![图来源于 yachen168](https://github.com/NoAlligator/pico/blob/main/img/4a73e2276d8b41f0a905361f151157e2~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

块格式化上下文，它是一个独立的渲染区域，只有块级盒子参与，它规定了内部的块级盒子如何布局，并且与这个区域外部毫不相干。

### **BFC 渲染规则**

> 垂直排列、外边距合并、边框抵住、互不干扰、浮动

- 垂直放置：内部的盒子会在垂直方向，一个接一个地放置；
- 垂直margin合并：盒子垂直方向的距离由 margin 决定，属于同一个 BFC 的两个相邻盒子的 **margin 会发生重叠**；
- 每个元素的 margin 的左边，与包含块 border 的左边相接触（对于从左往右的格式化，否则相反），即使存在浮动也是如此；
- **BFC 的区域不会与 float 盒子重叠；**
- BFC 就是页面上的一个隔离的独立容器，**容器里面的子元素不会影响到外面的元素。反之也如此。**
- 计算 BFC 的高度时，**浮动元素也参与计算。**

### 创建BFC

> 根元素、浮动、定位、行内块、overflow、表格、弹性盒

- 根元素（`<html>`）
- 浮动元素（`float`不为`none`）
- 绝对定位元素（`position`为`absolute`或`fixed`）
- 表格的标题和单元格（`display` 为 `table-caption`，`table-cell`）
- 匿名表格单元格元素（`display` 为 `table` 或 `inline-table`）
- 行内块元素（`display` 为 `inline-block`）
- `overflow` 的值不为 `visible` 的元素
- 弹性元素（`display` 为 `flex` 或 `inline-flex` 的元素的直接子元素）
- 网格元素（`display` 为 `grid` 或 `inline-grid` 的元素的直接子元素）

> 其中，最常见的就是`overflow:hidden`、`float:left/right`、`position:absolute`。也就是说，每次看到这些属性的时候，就代表了**该元素以及创建了一个BFC了**。

### BFC应用

[DEMO](https://codepen.io/bulandent/pen/eYBVpEm)

#### 自适应两栏布局

应用原理：BFC 的区域不会和浮动区域重叠，所以就可以把侧边栏固定宽度且左浮动，而对右侧内容触发 BFC，使得它的宽度自适应该行剩余宽度。

```html
<div class="layout">
    <div class="aside">aside</div>
    <div class="main">main</div>
</div>
```

```css
.aside {
    float: left;
    width: 100px;
}
.main {
    <!-- 触发 BFC -->
    overflow: auto;
}
```

#### 清除内部浮动

浮动造成的问题就是父元素高度坍塌，所以清除浮动需要解决的问题就是让父元素的高度恢复正常。而用 BFC 清除浮动的原理就是：计算 BFC 的高度时，浮动元素也参与计算。只要触发父元素的 BFC 即可。

```css
.parent {
    overflow: hidden;
}
```

#### 防止垂直 margin 合并

BFC 渲染原理之一：同一个 BFC 下的垂直 margin 会发生合并。所以如果让 2 个元素不在同一个 BFC 中即可阻止垂直 margin 合并。那如何让 2 个相邻的兄弟元素不在同一个 BFC 中呢？可以给其中一个元素外面包裹一层，然后触发其包裹层的 BFC，这样一来 2 个元素就不会在同一个 BFC 中了。

```html
<div class="layout">
    <div class="a">a</div>
    <div class="contain-b">
        <div class="b">b</div>
    </div>
</div>
```

```css
.demo3 .a,
.demo3 .b {
    border: 1px solid #999;
    margin: 10px;
}
.contain-b {
    overflow: hidden;
}
```

## ⭐IFC行内格式化上下文

![img](https://github.com/NoAlligator/pico/blob/main/img/5cee1281ae5f44a69abc94fb9fa760fd~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

> IFC 的形成条件非常简单，块级元素中仅包含内联级别元素，需要注意的是当IFC中有块级元素插入时，会产生两个匿名块将父元素分割开来，产生两个 IFC。

### IFC 渲染规则

- 子元素在**水平方向上一个接一个排列，在垂直方向上将以容器顶部开始向下排列**；
- 节点无法声明宽高，**其中 margin 和 padding 在水平方向有效在垂直方向无效**；
- 节点在**垂直方向上以不同形式对齐**；
- 能把在一行上的框都完全包含进去的一个矩形区域，被称为该行的线盒（line box）。**线盒的宽度是由包含块（containing box）和与其中的浮动来决定**；
- IFC 中的 line box 一般左右边贴紧其包含块，**但 float 元素会优先排列（表现为浮动元素遮蔽现象）**。
- IFC 中的 line box 高度**由 line-height 计算规则来确定**，同个 IFC 下的多个 line box 高度可能会不同；
- 当内联级盒子的总宽度少于包含它们的 line box 时，其水平渲染规则由 text-align 属性值来决定；
- **当一个内联盒子超过父元素的宽度时，它会被分割成多盒子，这些盒子分布在多个 line box 中。如果子元素未设置强制换行的情况下，inline box 将不可被分割，将会溢出父元素**。

```html
<p>It can get <strong>very complicated</storng> once you start looking into it.</p>
```

![img](https://github.com/NoAlligator/pico/blob/main/img/d357e140d61c4635a13771067758862b~tplv-k3u1fbpfcp-watermark.awebp?raw=true)



对应上面这样一串 HTML 分析如下：

- p 标签是一个 “block container”，对内将产生一个 IFC：
  - 由于一行没办法显示完全，所以产生了 2 个线盒（`line box`），线盒的宽度就继承了 p 的宽度，高度是由里面的内联盒子的 `line-height` 决定；
  - `“It can get”`：匿名的内联盒子；
  - `“very complicated”`：<strong/> 标签产生的内联盒子；
  - `“once you start”`：匿名的内联盒子；
  - `“looking into it”`：匿名的内联盒子。

### IFC应用场景

- 水平居中：当一个块要在环境中水平居中时，设置其为 `inline-block` 则会在外层产生 IFC，通过 `text-align`则可以使其水平居中。
- 垂直居中：创建一个 IFC，用其中一个元素撑开父元素的高度，然后设置其 vertical-align: middle，其他行内元素则可以在此父元素下垂直居中。

# 10.层叠上下文

> HTML 元素依据自己定义的属性的优先级在 Z 轴上按照一定的顺序排开，就是层叠上下文所要描述的东西。

我们对层叠上下文的第一印象可能要来源于 z-index，认为它的值越大，距离屏幕观察者就越近，那么层叠等级就越高，事实确实是这样的，但层叠上下文的内容远非仅仅如此：

- z-index 能够在层叠上下文中对元素的堆叠顺序其作用是**必须配合定位**才可以；
- 除了 z-index 之外，**一个元素在 Z 轴上的显示顺序还受层叠等级和层叠顺序影响**；

符合以下任一条件的元素都会产生层叠上下文：

- **<html/> 文档根元素**
- **声明 position: absolute/relative 且 z-index 值不为 auto 的元素；**
- **声明 position: fixed/sticky 的元素；**
- **flex 容器的子元素，且 z-index 值不为 auto；**
- **grid 容器的子元素，且 z-index 值不为 auto；**
- **opacity 属性值小于 1 的元素；**
- mix-blend-mode 属性值不为 normal 的元素；
- 以下任意属性值不为 none 的元素：
  - **transform**
  - filter
  - **perspective**
  - clip-path
  - mask / mask-image / mask-border
- isolation 属性值为 isolate 的元素；
- -webkit-overflow-scrolling 属性值为 touch 的元素；
- will-change 值设定了任一属性而该属性在 non-initial 值时会创建层叠上下文的元素；
- contain 属性值为 layout、paint 或包含它们其中之一的合成值（比如 contain: strict、contain: content）的元素。

## 层叠等级

层叠等级指节点在三维空间 Z 轴上的上下顺序。它分两种情况：

- 在同一个层叠上下文中，它描述定义的是该层叠上下文中的层叠上下文元素在 Z 轴上的上下顺序；
- 在其他普通元素中，它描述定义的是这些普通元素在 Z 轴上的上下顺序；

普通节点的层叠等级优先由其所在的层叠上下文决定，层叠等级的比较只有在当前层叠上下文中才有意义，脱离当前层叠上下文的比较就变得无意义了。

## 层叠顺序

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21043848687d42c6b46d6cf9c59c17ff~tplv-k3u1fbpfcp-watermark.awebp)

以下这个列表越往下层叠优先级越高，视觉上的效果就是越容易被用户看到（不会被其他元素覆盖）：

- 层叠上下文的 border 和 background
- z-index < 0 的子节点
- **标准流内块级非定位的子节点**
- 浮动非定位的子节点
- **标准流内行内非定位的子节点**
- z-index: auto/0 的子节点
- z-index > 0的子节点

## 如何比较两个元素的层叠等级？

- 在同一个层叠上下文中，比较两个元素就是按照上图的介绍的层叠顺序进行比较。
- 如果不在同一个层叠上下文中的时候，那就需要比较两个元素分别所处的层叠上下文的等级。
- 如果两个元素都在同一个层叠上下文，且层叠顺序相同，则在 HTML 中定义越后面的层叠等级越高。

# 11.像素

[参考](https://www.cnblogs.com/AhuntSun-blog/p/13581877.html)

> 在`CSS`中我们一般使用`px`作为单位，需要注意的是，`CSS`样式里面的`px`和物理像素并不是相等的。`CSS`中的像素只是一个抽象的单位，在不同的设备或不同的环境中，`CSS`中的`1px`所代表的物理像素是不同的。在`PC`端，`CSS`的`1px`一般对应着电脑屏幕的`1`个物理像素，但在移动端，`CSS`的`1px`等于几个物理像素。

## 分辨率

物理像素个数。

## 物理像素

物理像素又被称为设备像素、设备物理像素，它是显示器（电脑、手机屏幕）最小的物理显示单位，每个物理像素由颜色值和亮度值组成。所谓的一倍屏、二倍屏(`Retina`)、三倍屏，指的是设备以多少物理像素来显示一个`CSS`像素，也就是说，多倍屏以更多更精细的物理像素点来显示一个`CSS`像素点，在普通屏幕下1个`CSS`像素对应`1`个物理像素，而在`Retina`屏幕下，`1`个CSS像素对应的却是`4`个物理像素（参照下文田字示意图理解）。

## 逻辑像素

设备独立像素又被称为`CSS`像素，是我们写`CSS`时所用的像素，它是一个抽像的单位，主要使用在浏览器上，用来精确度量`Web`页面上的内容。

## 设备像素比（Device Pixel Ratio，DPR）

设备像素比 ＝ 物理像素 / 设备独立像素

## pt & px

> pt和px的关系就是1pt 里面有几个像素点，比如 1pt里面有1个px，也可以有2个，3个，分别对应@1x - 一倍图，@2x - 二倍图，@3x - 三倍图。（pt是逻辑像素、px是物理像素）。
>
> **移动端开发时， 设计师以iPhone6(750px * 1344px) 为模板进行设计的，这是是物理像素。但是前端开发者以逻辑像素为准，所以开发时需要将 单位除以2**

- `iPhone6`，物理像素 pt 是`375*667`，逻辑像素 px 是`750*1334`，px的宽高是pt的2倍，设计师出的图是根据`750*1334` 出的，**因此前端开发时需要将单位除以 2**（因为开发遵照物理像素，出图遵照真实像素，所以必选话按照几倍图进行换算）；
- `iPhone6sPlus` pt 和 px的比例是1:3，如果设计师iPhone6sPlus的尺寸出图，则前端开发时需要将单位除以 3。

​    ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c9ace1267d44416b5377dfea901a08d~tplv-k3u1fbpfcp-watermark.awebp)

## DPI(Dot Per Inch) & PPI(Pixel Per Inch)

PPI用于显示器，表示每英寸长度上有多少个**像素**，又叫像素数目。像素越多，代表画面更细腻更清晰。我们常说的视网膜屏幕（Retina），就是指PPI较普通屏幕要高。

DPI用于打印机，“每英寸墨点”。

# 12.相对尺寸

## em 

- **在 font-size 中使用em是相对于父元素的 font-size 大小**，比如父元素 font-size: 16px，当给子元素指定 font-size: 2em 的时候，经过计算后它的字体大小会是 32px；
- **在其他属性中使用em是相对于自身的 font-size 大小**，如 width/height/padding/margin 等；
- 总结：**使用em相对的是父元素font-size大小**

## rem

- rem 相对的是 HTML 的根元素 <html/>

## vw

- 1vw = 视口宽度均分成 100 份中 1 份的长度

## vh

- 1vh = 视口高度均分成 100 份中 1 份的长度

## vmin

- vmin：取 vw 和 vh 中值较小的

## vmax

- vmax：取 vw 和 vh 中值较大的

# 13.颜色

- 颜色关键字
- transparent 关键字
- currentColor 关键字
- RGB 颜色
- HSL 颜色

# 14.媒体查询

> 媒体查询是指针对不同的设备、特定的设备特征或者参数进行定制化的修改网站的样式。

你可以通过给 `<link>` 加上 `media` 属性来指定该样式文件只能对什么设备生效，不指定的话默认是 `all`，即对所有设备都生效：

```html
<link rel="stylesheet" src="styles.css" media="screen" />
<link rel="stylesheet" src="styles.css" media="print" />
```

## 常见设备类型：

- all：适用于所有设备；
- print：适用于在打印预览模式下在屏幕上查看的分页材料和文档；
- screen：主要用于屏幕；
- speech：主要用于语音合成器。

## 规则

除了通过 `<link>` 让指定设备生效外，还可以通过 `@media` 让 CSS 规则在特定的条件下才能生效。响应式页面就是使用了 @media 才让一个页面能够同时适配 PC、Pad 和手机端。

媒体查询支持逻辑操作符：

- and：查询条件都满足的时候才生效；
- not：查询条件取反；
- only：整个查询匹配的时候才生效，常用语兼容旧浏览器，使用时候必须指定媒体类型；
- 逗号或者 or：查询条件满足一项即可匹配；

媒体查询优先级：

`括号内媒体查询` > `and` > `not` > `or/,`

# 15.标签分类

## 按HTML分类

### 容器级

```html
<div/> <h/> <li/> <dt/> <dd/>
```

### 文本级

```html
<p/> <a/> <span/> <b/> <em/> <u/> <i/>
```

## 按闭合特征分类

### 单标签元素

单标签元素，也叫空标签或空元素，指没有内容的标签，在开始标签中自动闭合。
常见的空标签有：<br />、<hr />、 <img />、 <input />、<area />、 <base />、 <link />、 <meta />等。

### 双标签元素

双标签元素，也叫闭合标签元素，是由开始标签和结束标签组成的一对标签，它可以嵌套和承载内容。
例如：<div> </div>、<p> </p>、<html> </html>、<h1> </h1>、<span> </span>等。

## 按显示模式分类

### 行内元素

```html
<span> <a> <lable> <strong> <b> <small> <abbr> <button> <textarea> <select> <code> <br> <q> <i> <cite> <var> <kbd> <sub> <bdo>
```

### 块级元素

```html
 <div> <p> <li> <h1> <h2> <h3> <h4> <h5> <h6> <form> <header> <hr> <ol> <address> <article> <aside> <audio> <canvas> <dd> <dl> <fieldset> <section> <ul> <video>
```

### 行内块元素

```html
<img> <input>
```

### 区别

- 块级元素独占一行，行内元素不会独占一行且自动换行；
- 块级元素可以设置宽高（不设置的话默认宽度是父元素的100%），行内元素的宽度就是内容宽度，高度就是最高内容（行高）高度。
- 块级元素可以设置margin 和 padding。**行内元素只能设置水平方向的padding 和 margin-left,margin-right。**
- 块级元素可以包含行内元素和块级元素。行内元素不能包含块级元素。

## 可替换元素

*`html`中的`<img>`、`<input>`、`<textarea>`、`<select>`、`<object>`都是替换元素。这些元素往往没有实际的内容，即是一个空元素。*

# 16.盒模型属性

```css
div {
    width: 20px;
    height: 20px;
    padding: t r b l | t rl b | tb rl | trbl;
    margin: t r b l | t rl b | tb rl | trbl;
    border-width: t r b l | t rl b | tb rl | trbl;
    border-style: t r b l | t rl b | tb rl | trbl;
    border-color: t r b l | t rl b | tb rl | trbl;
    border-radius: t r b l | t rl b | tb rl | trbl;
    border-image: source slice width outset repeat;
}
```

# 17.阴影

```css
div {
	box-shadow: x偏移量 | y偏移量 | 阴影模糊半径 | 阴影扩散半径 | 阴影颜色;
}
```

# 18.轮廓属性

`outline` 与 `border` 相似，不同之处在于 `outline` 在整个元素周围画了一条线（`border外侧`）；**它不能像 `border` 那样，指定在元素的一个面上设置轮廓，也就是不能单独设置顶部轮廓、右侧轮廓、底部轮廓或左侧轮廓。**另外 `outline` 不会像`border`那样影响元素的尺寸或者位置，`outline`**不占据空间。**

```css
p {
	outline-color: red;
	outline-style: dotted;
	outline-width: 1px;
}
```

# 19.字体属性

```css
p {
    line-height: 30px;            /*行高*/
    font-size: 50px; 		      /*字体大小*/
    font-family: sans-serif;      /*字体*/
    font-style: italic ;	      /*字体样式*/
    font-weight: bold;	          /*字重*/
    font-variant: small-caps;     /*小写变大写*/
}
```

# 20.文本属性

## vertical-align

vertical-align属性可用于指定**行内元素**（inline）、**行内块元素**（inline-block）、**表格的单元格**（table-cell）的垂直对齐方式。主要是用于图片、表格、文本的对齐。[他与line-height属性的关系非常紧密。](https://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)

```css
p {
    vertical-align: baseline | top | middle | bottom | text-top | sub;
}
```

## line-height

行高用于设置多行元素的空间量，如多行文本的间距。对于块级元素，它指定元素行盒（line boxes）的最小高度。对于非[替代](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element)的 inline 元素，它用于计算行盒（line box）的高度。

```css
p {
    line-height: normal | 2.5 | 3em | 150% | 32px;
}
```

## text-align

定义行内内容（例如文字）如何相对它的块父元素对齐。`text-align` 并不控制块元素自己的对齐，只控制它的行内内容的对齐。

## text-indent

text-indent 属性规定文本块中首行文本的缩进。通过`text-indent`缩进的属性把文字赶走到视线以外的地方。这是做搜索引擎优化的一个重要的技巧。

## white-space

该属性用于处理空白（换行）符的行为。

normal（默认值）：合并空白字符 + 正常换行

nowrap：合并空表字符 + 不换行

pre：保留空白字符 + 不换行

pre-wrap：保留空白字符 + 换行

pre-line：仅保留空格和制表符 + 换行

| 换行符         | 空格和制表符 | 文字换行 | 行尾空格 |      |
| :------------- | :----------- | :------- | :------- | ---- |
| `normal`       | 合并         | 合并     | 换行     | 删除 |
| `nowrap`       | 合并         | 合并     | 不换行   | 删除 |
| `pre`          | 保留         | 保留     | 不换行   | 保留 |
| `pre-wrap`     | 保留         | 保留     | 换行     | 挂起 |
| `pre-line`     | 保留         | 合并     | 换行     | 删除 |
| `break-spaces` | 保留         | 保留     | 换行     | 换行 |

## word-break

### normal

使用默认的断行规则（CJK任意文本断行/非CJK默认断行规则）。

### break-all

对于non-CJK (CJK 指中文/日文/韩文) 文本，可在任意字符间断行。

### keep-all

CJK 文本不断行。 Non-CJK 文本表现同 normal。

### break-word 

他的效果是word-break: normal 和 overflow-wrap: anywhere  的合，不论 overflow-wrap的值是多少。

## overflow-wrap

[参考](https://www.zhangxinxu.com/wordpress/2020/03/css-overflow-wrap-anywhere/)

该属性用于处理单词/链接换行的行为。

```css
p {
	overflow-wrap: normal | break-word | anywhere;
}
```

## text-overflow

**设置在父容器。**该属性确定如何向用户发出未显示的溢出内容信号。默认值是clip。这个属性**只对那些在块级元素溢出的内容有效，但是必须要与块级元素内联(inline)方向一致（举个反例：内容在盒子的下方溢出。此时就不会生效）**。文本可能在以下情况下溢出：当其因为某种原因而无法换行(例子：设置了"white-space: nowrap")，或者一个单词因为太长而不能合理地被安置(fit)。这个属性并不会强制“溢出”事件的发生，因此为了能让"text-overflow"能够生效，程序员们必须要在元素上添加几个额外的属性，比如"将[`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 设置为hidden"。

```css
p {
    text-overflow: [ clip | ellipsis | <string> ]{1,2}
}
```

## 其他属性

```css
p {
    letter-spacing: 2px; /*字母间距*/
    word-spacing: 2px;   /*单词间距*/
    text-decoration: none; /*文本修饰*/
    color: red;
    text-transform: lowercase; /*文本大小写*/
    direction: rtl;
    text-shadow: h-shadow v-shadow blur color;
    
}
```

# 21.文本换行

## 文本溢出

默认情况下单词太长会溢出容器。![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40ade871ebd34a53a87b5fdac31190b8~tplv-k3u1fbpfcp-watermark.awebp)

## 长单词断行

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c5d1b30df0041258c0ae35a496d5e78~tplv-k3u1fbpfcp-watermark.awebp)

## 强制不换行

```css
p {
	white-space: pre;
}
```

## 英文单词任意断行

```css
p {
    word-break : break-all;
}
```

## 单行文本超出省略

```css
p {
    overflow: hidden;			/*BFC限制溢出*/
    text-overflow: ellipsis;	/*溢出显示...*/
    white-space: nowrap;		/*强制不换行*/
}
```

## 字符超出位置使用连字符

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f85b1c16a5c4dbfa549650794659553~tplv-k3u1fbpfcp-watermark.awebp)

## 多行文本自动省略功能

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88afb6a8f2824d4487267f6ff7f3e49c~tplv-k3u1fbpfcp-watermark.awebp)

[整块文本溢出解决方案](https://segmentfault.com/a/1190000039399159)  [可能是最全的 “文本溢出截断省略” 方案合集](https://www.zoo.team/article/text-overflow)

# 23.浮动 与 清除浮动

## 性质

- 浮动创建 BFC（根据层叠顺序，浮动元素会**遮挡标准文档流**）；
- 浮动元素互相贴靠；
- 浮动的元素有“字围”效果；
- 浮动元素自动收缩为内容宽度

## 清除浮动

### 1.给浮动元素祖先添加高度；

### 2.使用clear:both给受到浮动影响的元素清除浮动；

clear就是清除，both指的是左浮动、右浮动都要清除。`clear:both`的意思就是：不允许左侧和右侧有浮动对象。

### 3.隔墙法

隔一个div并设置`clear:both`

### 4.内墙法撑开高度

### 5.BFC清除浮动

使用`overflow:hiddern`生成BFC。

### 6.推荐：clearfix：BFC+伪元素

```css
.box::after {
    content: '.';
    height: 0;
    display: block;
    clear: both;
}
```

# 24.Margin塌陷

[参考](https://juejin.cn/post/6844903601718951949)

# 24.清除浏览器默认样式

```css
html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed, 
figure, figcaption, footer, header, hgroup, 
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
    margin: 0;
    padding: 0;
    border: 0;
    font-size: 100%;
    font: inherit;
    vertical-align: baseline;
}
/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure, 
footer, header, hgroup, menu, nav, section {
    display: block;
}
body {
    line-height: 1;
}
ol, ul {
    list-style: none;
}
blockquote, q {
    quotes: none;
}
blockquote:before, blockquote:after,
q:before, q:after {
    content: '';
    content: none;
}
table {
    border-collapse: collapse;
    border-spacing: 0;
}
```

[Normalize.css](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fnecolas%2Fnormalize.css)

# 25.定位属性

## 分类

- 绝对定位（absolute）
- 相对定位（relative）
- 固定定位（fixed）

> **使用固定定位时必须设置任意四个属性之一才可以生效**。使用相对定位时，两个范围的定位是“独立”的。

## 相对定位

基本作用：

- 不会脱离标准流，**元素依然会占据原位置，只是“影子的位移”**，让元素相对于自己原来的位置，进行位置调整（可用于盒子的位置微调）；
- 子元素实施“子绝父相”的参考元素。

## 绝对定位

基本作用

- 可以用于创建**BFC**；
- 脱离标准流，实施“子绝父相”相对父元素进行位移；

### 参考点

#### 有参考父元素时

子元素相对**父元素边框区内侧**定位。

#### 无参考父元素时

如果用**top描述**，那么参考点就是**页面的左上角**，而不是浏览器的左上角：

![image-20220214144927297](https://github.com/NoAlligator/pico/blob/main/img/image-20220214144927297.png)

如果用**bottom描述**，那么参考点就是**浏览器首屏窗口尺寸**：

![image-20220214144947656](https://github.com/NoAlligator/pico/blob/main/img/image-20220214144947656.png)

## 应用

- 利用子绝父相实现“压盖”效果；
- 利用子绝父相实现水平居中；

## 为什么会出现定位

[解答](https://juejin.cn/post/6844903601718951949)

# 26.滚动条相关属性

默认情况下内容超出容器后会溢出，通过控制`overflow`属性控制溢出还是显示滚动条（形成BFC）。

```css
div {
    overflow: visible;  /* 默认值。内容不会被修剪，会呈现在元素框之外 */
    overflow: hidden;   /* 内容会被修剪，并且其余内容不可见 */
    overflow: scroll;   /* 内容会被修剪，浏览器会显示滚动条以便查看其余内容 */
    overflow: auto;     /* 由浏览器定夺，如果内容被修剪，就会显示滚动条 */
    overflow: inherit;  /* 规定从父元素继承overflow属性的值 */
}
```

> 此外还有`overflow-x` 、`overflow-y`属性分别控制两个方向

# 27.鼠标控制属性

```css
p:hover{
    cursor: pointer;
}
```

# 28.滤镜属性

```css
img {
    filter: blur(5px);				/* 仅对当前图层生效 */
    backdrop-filter: blur(5px);		/* 对合成图层生效 */
}
```

# 29.背景属性

> 可以同时设置**多个背景图片**，并设置**多个背景属性**，以逗号分隔。
>
> ```css
> div {
> 	background-image: url('paper.gif');
> }
> ```

## background-attachment（scroll）

（支持多张）用于设置背景图片放置模式，默认scroll。

- `fixed`（和视口固定）：此关键属性值表示**背景相对于视口固定**。即使一个元素拥有滚动机制，背景也不会随着元素的内容滚动；
- `local`（背景**伪装成本地元素**，一起滚动）：此关键属性值表示背景**相对于元素的内容**固定。如果一个元素拥有滚动机制，背景将会随着元素的内容滚动， 并且背景的绘制区域和定位区域是相对于可滚动的区域而不是包含他们的边框；
- `scroll`（背景固定在内容上，所以内容会在背景上滚动）：此关键属性值表示背景**相对于元素本身固定**， 而不是随着它的内容滚动（对元素边框是有效的）。

## background-color

背景颜色。

## back-ground-image

（支持多张）背景图。

## background-position（0% 0%）

（支持多张）设置背景位置，默认在左上角，可以设置位置关键词、数值、百分比。

## background-repeat（repeat）

（支持多张）设置背景重复方式。

| **单值**    | **等价于双值**        |
| ----------- | --------------------- |
| `repeat-x`  | `repeat no-repeat`    |
| `repeat-y`  | `no-repeat repeat`    |
| `repeat`    | `repeat repeat`       |
| `space`     | `space space`         |
| `round`     | `round round`         |
| `no-repeat` | `no-repeat no-repeat` |

## background-clip（border-box）

设置背景图片被盒子裁切的起始点。

```css
div {
    background-clip: border-box;
    background-clip: padding-box;
    background-clip: content-box;
    background-clip: text;
}
```

## background-origin（padding-box）

> 当使用 [`background-attachment`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-attachment) 为fixed时，该属性将被忽略不起作用。

设置背景图片的参考起始点，默认为，参数和background-clip一致。

## background-size

1. `contain`
2. `cover`
3. 数值：单值 | 双值

# 30.可视相关

- `display:none`（真正意义上的消失）
- `opacity:0`（不透明度为0）
- `visibility:hidden`（被隐藏，但是实际上存在）

# 31.z-index

- 属性值大的位于上层，属性值小的位于下层；
- 如果大家都没有z-index值，或者z-index值一样，那么在HTML代码里写在后面，谁就在上面能压住别人。定位了的元素，永远能够压住没有定位的元素；
- 只有定位了的元素，才能有`z-index`值。也就是说，不管相对定位、绝对定位、固定定位，都可以使用z-index值。**而浮动的元素不能用**；
- 遵循子元素从父现象，不能跨层级。

# 32.flex布局

> 设为Flex布局以后，形成FFC，**其子元素的`float`、`clear`和`vertical-align`属性将失效。**

[flex布局](https://juejin.cn/post/7019075844664459278)

align-items 和 align-content 的差异可以总结为如下两点：

1. align-items 的上下文是**行内**，align-content 的上下文是**弹性盒子容器**；
2. align-items **控制成员的对齐行为**，align-content **控制所有行的对齐行为**。

[差别](https://zhuanlan.zhihu.com/p/87146411#:~:text=%E6%80%BB%E7%BB%93,%E6%89%80%E6%9C%89%E8%A1%8C%E7%9A%84%E5%AF%B9%E9%BD%90%E8%A1%8C%E4%B8%BA%E3%80%82)

# 33.CSS效果相关属性

- box-shadow：盒子的阴影
- text-shadow：文本的阴影
- border-radius
- background
- clip-path

# 34.水平垂直居中

让元素在父元素中呈现出水平居中的状态，无非就两种情况：

- 单行文本、inline、inline-block元素；
- 固定宽高的块级盒子；
- 不固定宽高的块级盒子。

### 块级元素最常用水平居中方法

```css
.child {
	margin: 0 auto;	
}
```



## 多行文本垂直居中

[参考](https://www.1024sou.com/article/298169.html)

## 单行文本、inline、inline-block元素居中

### 水平居中

```css
.parent {
	text-align: center
}
```

### 垂直居中

```css
.single-line-parent {
	padding: 10px 0;		/*通过设置单行父元素的内边距来达到居中的效果*/
}
```

```css
.single-line-parent {
    height: 100px;
    line-height: 100px	/*通过设置行高和父容器高度一致来达到居中*/
}
```

## 固定宽高的块级盒子

### 子绝父相 + 固定尺寸

```css
.parent {
	position: relative;	
}

.child {
    width: 100px;
    height: 100px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin: -50px 0 0 -50px;
}
```

### 子绝父相 + auto margin

```css
.parent {
	position: relative;	
}

.child {
    width: 100px;
    height: 100px;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
}
```

### 子绝父相 + 计算

```css
.parent {
	position: relative;	
}

.child {
    width: 100px;
    height: 100px;
    position: absolute;
    left: calc(50% - 50px);
    top: calc(50% - 50px);
}
```

## 不定宽高的块级盒子

### ⭐子绝父相 + transform

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e69e2c2e35f74ae6a0c38949d627798e~tplv-k3u1fbpfcp-watermark.awebp)

### inline-block + 行高法

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cca3c98ae284d1aa6db063739df6e2a~tplv-k3u1fbpfcp-watermark.awebp)

### table-cell法

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/317b0670c9fe4c90bb8b77f3f90d8c79~tplv-k3u1fbpfcp-watermark.awebp)

### flex法

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/070f1c7a61ec4cbd8aa2d9b000f8dcd4~tplv-k3u1fbpfcp-watermark.awebp)

### grid法

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b509172aabc4a3580c3eb57953755f5~tplv-k3u1fbpfcp-watermark.awebp)

### 参考：[链接](https://segmentfault.com/a/1190000016389031)

# 35.两栏布局（一侧定宽，两侧自适应）

### float + overflow BFC

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f90447388404ebd9a9c238317e81e23~tplv-k3u1fbpfcp-watermark.awebp)

### float + margin

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/544ac1c6ec884ff18582faa348b69c70~tplv-k3u1fbpfcp-watermark.awebp)

### flex

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8866bac770443b39c806150cdcc8a7c~tplv-k3u1fbpfcp-watermark.awebp)

### grid

### ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/688c3b7599614081b1df80a50ec2965f~tplv-k3u1fbpfcp-watermark.awebp)

# 35.三栏布局（两侧定宽主栏自适应）

### 圣杯布局（浮动）

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c3df7e23b8d4fbd96eeeccb4fc99215~tplv-k3u1fbpfcp-watermark.awebp)

### 双飞翼布局

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488a7974931f46d687075c30468286f1~tplv-k3u1fbpfcp-watermark.awebp)

### float + overflow（BFC 原理）

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62fb7453443344149fcf27d551e63075~tplv-k3u1fbpfcp-watermark.awebp)

### flex

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c00727559ed4833892b01d12193c621~tplv-k3u1fbpfcp-watermark.awebp)

### grid

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e8d2ef98c4c4d718a10f91984c57e71~tplv-k3u1fbpfcp-watermark.awebp)

# [36.多列等高布局](https://juejin.cn/post/6844903615182667789)

## padding + 负margin

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/012fee13397f46c7ab1b4826b9d58c08~tplv-k3u1fbpfcp-watermark.awebp)

## flex布局

```css
.Article {
  display: flex;
}

.Article>li {
  flex: 1;
  border-left: solid 5px currentColor;
  border-right: solid 5px currentColor;
  color: #fff;
  background: #4577dc;
}

.Article>li>p {
  padding: 10px;
}
```

## 模仿table布局

```css
.Article {
  display: table;
  width: 100%;
  table-layout: fixed;
}

.Article>li {
  display: table-cell;
  width: 200px;
  border-left: solid 5px currentColor;
  border-right: solid 5px currentColor;
  color: #fff;
  background: #4577dc;
}

.Article>li>p {
  padding: 10px;
}
```

## grid布局

```css
.Article {
  display: grid;
  grid-auto-flow: column;
  grid-gap: 20px;
}

.Article>li {
  border-left: solid 5px currentColor;
  border-right: solid 5px currentColor;
  color: #fff;
  background: #4577dc;
}

.Article>li>p {
  padding: 10px;
}
```

# 37.三行布局（头尾定高主栏自适应）

## 使用calc()

[DEMO](https://jsbin.com/limovec/edit)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8930dba26767406c9154b03160a5f931~tplv-k3u1fbpfcp-watermark.awebp)

## flex

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/689b14dbb11e4c72a983ef0cd127fb9a~tplv-k3u1fbpfcp-watermark.awebp)

## absolute

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3925987fb43245498483a9083fe6b4b4~tplv-k3u1fbpfcp-watermark.awebp)

## grid

## ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a60294b8dc943378f9a851fc0c63afa~tplv-k3u1fbpfcp-watermark.awebp)

# 38.CSS 关键字 initial、inherit 和 unset

## initial

`initial` 关键字用于设置 CSS 属性为它的默认值，可作用于任何 CSS 样式。（IE 不支持该关键字）

## inherit

每一个 CSS 属性都有一个特性就是，这个属性必然是默认继承的 (`inherited: Yes`) 或者是默认不继承的 (`inherited: no`)其中之一，我们可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference) 上通过这个索引查找，判断一个属性的是否继承特性。

譬如，以 `background-color` 为例，由下图所示，表明它并不会继承父元素的 `background-color`:

### 可继承属性

最后罗列一下默认为 `inherited: Yes` 的属性：

- 所有元素可继承：visibility 和 cursor
- 内联元素可继承：letter-spacing、word-spacing、white-space、line-height、color、font、 font-family、font-size、font-style、font-variant、font-weight、text- decoration、text-transform、direction
- 块状元素可继承：text-indent和text-align
- 列表元素可继承：list-style、list-style-type、list-style-position、list-style-image
- 表格元素可继承：border-collapse

还有一些 inherit 的妙用可以看看这里：[谈谈一些有趣的CSS题目（四）-- 从倒影说起，谈谈 CSS 继承 inherit](http://www.cnblogs.com/coco1s/p/5908120.html)，合理的运用 inherit 可以让我们的 CSS 代码更加符合 DRY（Don't Repeat Yourself）原则。

## unset

名如其意，`unset` 关键字我们可以简单理解为不设置。其实，它是关键字 `initial` 和 `inherit` 的组合。

什么意思呢？也就是当我们给一个 CSS 属性设置了 `unset` 的话：

1. **如果该属性是默认继承属性，该值等同于 `inherit`**
2. **如果该属性是非继承属性，该值等同于 `initial`**

举个例子，先列举一些 CSS 中默认继承父级样式的属性：

- 部分可继承样式: `font-size`, `font-family`, `color`, `text-indent`
- 部分不可继承样式: `border`, `padding`, `margin`, `width`, `height`

[参考](https://www.cnblogs.com/libin-1/p/6734751.html)

# 39.渐进增强和优雅降级

## 什么是渐进增强

在网页开发中，渐进增强认为应该专注于内容本身。一开始针对低版本的浏览器构建页面，满足最基本的功能，再针对高级浏 览器进行效果，交互，追加各种功能以达到更好用户体验,换句话说，就是以最低要求，实现最基础功能为基本，**向上兼容**。以css为例，以下这种写法就是渐进增强。

## 什么是优雅降级

在网页开发中，优雅降级指的是一开始针对一个高版本的浏览器构建页面，先完善所有的功能。然后针对各个不同的浏览器进行测试，修复，保证低级浏览器也有基本功能就好，低级浏览器被认为“简陋却无妨 (poor, but passable)” 可以做一些小的调整来适应某个特定的浏览器。但由于它们并非我们所关注的焦点，因此除了修复较 大的错误之外，其它的差异将被直接忽略。也就是以高要求，高版本为基准，**向下兼容**。

## 二者区别

- 如果你采用渐进增强的开发流程，先做一个基本功能版，然后针对各个浏览器进行渐进增加，增加各种功能。相对于优雅降级来说，开发周期长，初期投入资金大。 你想一下不可能拿个基本功能版给客户看呀，多寒酸，搞不好一气之下就不找你做了，然后就炸了。但是呢，也有好处，就是提供了较好的平台稳定性，维护起来资金小， 长期来说降低开发成本。
- 那采用优雅降级呢，这样可以在较短时间内开发出一个只用于一个浏览器的完整功能版，然后就可以拿给PM找客户谈呀，可以拿去测试，市场试水呀，对于功能尚未确定的 产品，优雅降级不失为一种节约成本的方法。

# 40.响应式布局步骤

> 移动端优先的页面小尺寸媒体查询在前，PC端优先大尺寸媒体查询在前。（如果是移动端优先，则用`min-width`；如果是PC端优先，则用`max-width`。）
>
> 页面头部必须有<meta>声明的`viewport`。

# 41.rem布局原理

[参考](https://github.com/amfe/article/issues/17)

三种方案：

- 媒体查询 + rem
- JS + rem
- vw + rem

# 42.felxible.js

> 基本思路依旧是`rem`布局，只是在获取设备的真实尺寸上采用了`JS`的方式来进行动态计算。
>
> - 通过获取`DPR`来设置<meta>控制画面画布大小；
> - 根据屏幕的尺寸动态设置根元素`font-size`值。

[参考](https://github.com/wanlixi/flexible.js-1)

# 43.vw + rem布局

[参考](https://jelly.jd.com/article/6006b1055b6c6a01506c8801)

