# 1.`HTML5`中的`data-*`的用法和作用（自定义属性）（×）

### 作用

> 使用`data-*`属性**来嵌入自定义数据，用于储存私有的自定义数据，可以让我们在所有`HTML`元素上增加自定义`data`属性，存储的`data`属性能被`JavaScript`调用**。`HTML 5`之前，需要使用其他的数据时，是非常糟糕的。为了使一切正常有效，你不得不将数据填充到`rel`或`class`属性。 有些开发者，甚至创建自己的自定义属性。 自从`HTML5`自定义`data`属性出现，你可以存储任意数据，通过一种简单，**符合标准**的方式。

### 使用规则

本质：一个保存数据的自定义属性

用法：`data-`作为前缀，后面跟着描述性的单词（只允许小写字母和连字符`-`，所以都**用连字符分隔单词**），一个元素可以有任意多个`data`属性，属性值不能是**对象类型**，如果真的想要保存对象，可通过**对象序列化**。

语法：`<element data-*="somevalue">`

### 修改、获取`data-*`存储的值（`JS`）

```javascript
elem.getAttribute("data-name");	//获取data-name属性值
elem.setAttribute("data-name","orange");
elem.getAttribute("data-first-name");//获取data-first-name属性值

elem.dataset.name	//获取data-name属性值
elem.dataset.firstName //获取data-first-name属性值，需要转换为驼峰形式
//data-first-name -> dataset.firstName
```

### 修改、获取`data-*`存储的值（`CSS`）

```css

/*
获取值:
attr() 理论上能用于所有的CSS属性但目前支持的仅有伪元素的 content 属性，其他的属性和高级特性目前是实验性的
*/
element::before{
    content: attr(data-name);
}

/*
改变值:
*/
element[data-name="apple"]{
    width="100px;"
}
element[data-name="banana"]{
    width="100px;"
}
```

# 2.DOM 中 Property 和 Attribute 的区别

- property是DOM中的属性，是JavaScript里的对象属性；

- attribute是HTML标签上的特性（一旦确定就不能改变），它的值只能够是字符串；

  

### 第一个问题: 什么是 attribute & 什么是 property ?

**attribute** 是我们在 **html** 代码中经常看到的键值对, 例如:

```html
<input id="the-input" type="text" value="Name:" />
```

上面代码中的 input 节点有三个 attribute:

- id : the-input
- type : text
- value : Name:

**property** 是 attribute 对应的 DOM 节点的 对象属性 (Object field), 例如:

```js
HTMLInputElement.id === 'the-input'
HTMLInputElement.type === 'text'
HTMLInputElement.value === 'Name:'
```

### 第二个问题:

从上面的例子来看, 似乎 attribute 和 property 是相同的, 那么他们有什么区别呢?

让我们来看另一段代码:

```html
<input id="the-input" type="typo" value="Name:" /> // 在页面加载后,
我们在这个input中输入 "Jack"
```

注意这段代码中的 type 属性，我们给的值是 **typo**，这并不属于 input 支持的 type 种类。

让我们来看看上面这个 input 节点的 attribute 和 property：

```js
// attribute still remains the original value
input.getAttribute('id') // the-input
input.getAttribute('type') // typo
input.getAttribute('value') // Name:

// property is a different story
input.id // the-input
input.type //  text
input.value // Jack
```

可以看到，在 attribute 中，值仍然是 html 代码中的值。而在 property 中，type 被自动修正为了 **text**，而 value 随着用户改变 input 的输入，也变更为了 **Jack**

这就是 attribute 和 Property 间的区别：

**attribute 会始终保持 html 代码中的初始值, 而 Property 是有可能变化的。**

其实, 我们从这两个单词的名称也能看出些端倪：

**attribute** 从语义上, 更倾向于不可变更的

而 **property** 从语义上更倾向于在其生命周期中是可变的

## Attribute or Property 可以自定义吗?

先说结论: **attribute 可以** **property 不行**

我们可以尝试在 html 中自定义 attribute:

```
<input value="customInput" customeAttr="custome attribute value" />
```

然后我们尝试获取自定义的属性:

```
input.getAttribute('customAttr') // custome attribute value
input.customAttr // undefined
```

可以看到, 我们能够成功的获取自定义的 attribute, 但是无法获取 property.

其实不难理解, DOM 节点在初始化的时候会将**html 规范**中定义的 attribute 赋值到 property 上, 而自定义的 attribute 并不属于这个氛围内, 自然生成的 DOM 节点就没有这个 property.

# 3.视口

## 视口概念

视口分为：`布局视口`、`视觉视口`、`理想视口`

<img src="https://github.com/NoAlligator/pico/blob/main/img/202203271744431.png" alt="image-20210613201019760" style="zoom:50%;" />

<img src="https://github.com/NoAlligator/pico/blob/main/img/202203271744476.png" alt="image-20210613200954030" style="zoom:50%;" />

<img src="https://github.com/NoAlligator/pico/blob/main/img/202203271744514.png" alt="image-20210613201111287" style="zoom:50%;" />

## 设置视口

```html
<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
```

| key           | mean             | example                                                      |
| ------------- | ---------------- | ------------------------------------------------------------ |
| width         | 视口的宽度       | width=device-width 指缩放为 100% 时以 CSS 像素计量的屏幕宽度 |
| initial-scale | 初始化缩放比例   | initial-scale=1.0 初始化不进行缩放                           |
| maximum-scale | 用户最大缩放比例 | maximum-scale=1.0 不允许用户缩放                             |

[参考](https://juejin.cn/post/6844904094495145992)

[详细参考](https://segmentfault.com/a/1190000017784801)

### 布局视口

由HTML外部容器大小决定

```javascript
document.documentElement.clientWidth
```

### 视觉视口

由

```javascript
window.innerWidth
```

### 理想视口

由设备决定

```
screen.width
```

# HTML5

- 语义化标签
- audio，video，source
- 新的表单类型、表单property
- progress\meter
- document
- web存储
- 拖放
- canvas
- svg

# 行内/块级/块级行内

## block

div族、ul、li、h族、p、ol、dl、dt、dd

## inline

span、a、b、i

## inline-block

input、img、button
