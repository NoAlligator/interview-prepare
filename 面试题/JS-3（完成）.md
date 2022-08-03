# 76.如何有效阻止内存泄漏？

## 防止隐式全局变量

> 如果一个变量被绑定在了全局对象上而没有被及时地解除引用，那么在`GC`阶段会检测到它始终是可达的，尽管它不再会被使用，`GC`也不会去主动回收它。常见的情形有：在非严格模式下滥用`var`声明变量，在非严格模式下未经声明使用变量，通过默认绑定的全局对象`this`将变量存储在了全对象上。

为了避免这些失误的发生，添加 `“use strict”;` 行到你的`JavaScript`文件的头部。这将开启`JavaScript`解析器的严格模式，从而**阻止创建意外的全局变量**。

函数中的**局部变量在函数执行结束后这些变量已经不再被需要，所以垃圾回收器会识别并释放它们**。但是对于**全局变量**，垃圾回收器很难判断这些变量什么时候才不被需要，所以全局变量通常不会被回收。只有需要临时存储和处理非常大的数据时才会考虑使用全局变量**。**如果你一定要使用全局变量存储很多数据的话，请确保在你完成一些需要这些数据的操作后为这个全局变量赋一个`null`值或者重新分配一个引用。一个最常见的会增加内存消耗的东西就是“缓存”，缓存数据是指那些会重复使用到的数据，为了有效的使用，缓存必须拥有峰值边界限制，如果不加约束的使用缓存技术，必定会由于他们的内容无法被回收而造成非常高的内存的消耗。

减少全局变量的使用，这一类变量不仅难以被垃圾回收，并且容易污染全局作用域。

## 防止游离的`DOM`引用

当我们使用变量缓存`DOM`节点引用后删除了节点，如果不将缓存引用的变量置为`null`，那么它这个`DOM`节点始终是可达的，即使它并不处在`DOM`树上，所以无法被`GC`回收。假如我们将父节点置空，但是被删除的父节点其子节点引用也缓存在变量里，那么就会导致整个父 `DOM `节点树下整个游离节点树均无法清理，还是会出现内存泄漏，解决办法就是将引用子节点的变量也置空。

## 清除定时器

在 `setInterval` 没有结束前，**回调函数里引用的闭包以及回调函数本身都无法被回收**，`setTiemout` 也会有同样的问题。当不需要 `interval` 或者 `timeout` 时，最好调用 `clearInterval` 或者 `clearTimeout`来清除，另外，浏览器中的 `requestAnimationFrame` 也存在这个问题，我们需要在不需要的时候用 `cancelAnimationFrame` API 来取消使用

## 遗忘的事件监听器

当事件监听器在组件内挂载相关的事件处理函数，而**在组件销毁时不主动将其清除时，其中引用的变量或者函数都被认为是需要的而不会进行回收**，如果内部引用的变量存储了大量数据，可能会引起页面占用内存过高，这样就造成意外的内存泄漏。所以，在事件监听器不再生效之前，务必记得取消`DOM`事件监听：

```javascript
target.removeEventListener(type, listener[, options]);
target.removeEventListener(type, listener[, useCapture]);
```

## 遗忘的订阅模式

当我们实现了监听者模式并在组件内挂载相关的事件处理函数，而在组件销毁时不主动将其清除时，其中引用的变量或者函数都被认为是需要的而不会进行回收，如果内部引用的变量存储了大量数据，可能会引起页面占用内存过高，这样也会造成意外的内存泄漏。例如`Vue`中的`EventBus.on`，在生命周期`beforeDestory`中应当取消所有的监听。

## 遗忘的`Map`、`Set`对象

当使用 `Map` 或 `Set` 存储对象时，对于存储的引用类型是强引用，如果不将其主动清除引用，其同样会造成内存不自动进行回收。

- 如果使用 `Map` ，对于键为对象的情况，可以采用 `WeakMap`，`WeakMap` 对象同样用来保存键值对，对于键是弱引用（注：`WeakMap` 只对于键是弱引用），且必须为一个对象，而值可以是任意的对象或者原始值，由于是对于对象的弱引用，不会干扰 `Js` 的垃圾回收。
- 如果需要使用 `Set` 引用对象，可以采用 `WeakSet`，`WeakSet` 对象允许存储对象弱引用的唯一值，`WeakSet` 对象中的值同样不会重复，且只能保存对象的弱引用，同样由于是对于对象的弱引用，不会干扰 `Js` 的垃圾回收。

## 未清理的`console`输出

我们之所以在控制台能看到数据输出，是因为浏览器保存了我们输出对象的信息数据引用，也正是因此未清理的 `console` 如果输出了对象也会造成内存泄漏。**开发环境下我们可以使用控制台输出来便于我们调试，但是在生产环境下，一定要及时清理掉输出。**

## 正确使用闭包

在使用闭包对函数内部的数据产生引用关系时，在使用结束后要及时进行内存的释放。否则闭包会一直伴随着闭包函数的引用而间接存在。

```javascript
function closure(){
  let gcTarget = new Array(1000).fill('garbage')
  return function(){
    return gcTarget
  }
}
let gbg = closure()
gbg()		//执行完之后，因为gbg还持有对gcTarget的引用，所以gcTarget内存不会被释放
gbg = null	//主动释放，让GC进行回收
```

```javascript
function foo() { 
    var a = {}; 
    function bar() { 
        console.log(a); 
    }; 
    a.fn = bar; 
    return bar; 
};
//因为 a.fn = bar 这一句（a维持了对bar的引用，相应的bar也就维持了对a的引用，并导致闭包不被释放），bar作为一个闭包，即使它内部什么都没有，foo中的所有变量都还是隐式地被 bar 所引用。 即使 bar 内什么都没有还是造成了循环引用，不要将 a.fn = bar
```

## 减少内存泄露的核心策略

- 关注全局变量，减少全局变量，可能的话尽量使用块级作用域；
- 时刻警惕使用闭包；
- 牢记解绑事件侦听器、订阅模式、清除定时器；
- 牢记强引用数据结构的内存释放；
- 牢记游离`DOM`的危害。

## 内存三大件

`内存泄漏` ：对象已经不再使用但没有被回收，内存没有被释放，即内存泄漏，那想要避免就避免让无用数据还存在引用关系，也就是多注意我们上面说的常见的几种内存泄漏的情况。

`内存膨胀` ：即在短时间内内存占用极速上升到达一个峰值，想要避免需要使用技术手段减少对内存的占用。

`频繁 GC` ：GC工作时应用程序是停止的，GC频繁工作且过长会导致应用假死，用户就会感觉到卡顿，所以应当把握好触发CG的尺度。

# 77.说一说和`DOM`相关的操作？

## 注意区分`node`和`element`

> `Element`继承于`Node`，具有`Node`的方法，同时又拓展了很多自己的特有方法。
>
> `Element`在`HTML`中有对应的具体的标签的`DOM`节点，而`Node`既包括拥有具体标签的`DOM`节点，也包括没有具体对应的`HTML`标签的文本节点等。故两者是包含关系。
>
> 也就是说`Element`指的是具体的标签元素，`Node`可以是`HTML`中的任意一个节点。
>
> ![img](https://github.com/NoAlligator/pico/blob/main/img/v2-8dc1ccc0553dd4e99c699755031d9ab4_r.jpg?raw=true)

```js
// 父
node.parentNode
element.parentElement

//子
node.childNodes
element.children

node.firstChild
element.firstElementChild

node.lastChild
element.lastElementChild

//兄

node.nextSibling
element.nextElementSibling

node.previousSibling
element.previousElementSibling

node.nodeName		//节点名称
node.nodeType		//节点类型
node.nodeValue		//节点值
node.textContent	//节点内容
node.isConncted		//是否在DOM树上
node.ownerDocument	//获取根document
node.baseURL
```

## 创建`DOM`节点

```js
document.createDocumentFragment()    			//创建一个DOM片段
document.createElement(tagName)					//创建一个元素节点
document.createTextNode(data)   				//创建一个文本节点
let attr = document.createAttribute(name)		//创建一个属性节点
attr.value = 'value'
```

## 添加、移除、替换、插入、克隆、比较、查询`DOM`

```js
parentNode.appendChild(node) 		//尾部添加（原本就存在于文档数的话相当于移动位置）
document.cloneNode(node)				//克隆
parentNode.removeChild(node)		//移除
parentNode.replaceChild(newNode, referenceNode)	//替换（原本就存在于文档数的话相当于移动位置）
parentNode.insertBefore(newNode, referenceNode)	//指定添加（原本就存在于文档数的话相当于移动位置）

node.compareDocumentPosition(otherNode) 	//位置比较
node.contains(otherNode)					//包含关系或本身
node.getRootNode(node)					    //获取游离节点的子节点
node.hasChildNodes							//是否是叶子节点
node.isEqualNode(otherNode)					//比较是否是同一个node
node.normalize()							//规范化（去空文本节点、合并文本节点）

element.closest(selector)
```

## 查找`DOM`

```js
document.getElementById();
document.getElementsByName();
document.getElementsByTagName();
document.getElementsByClassName();
document.querySelector();
document.querySelectorAll();
```

## `Element`添加`attribute`

```js
element.getAttribute(attrName);			 //获取元素对应属性名的属性值
element.getAttibuteNode(attrName);		 //获取元素对应属性名的值
element.getAttributeNames();			//获取所有元素所有属性名

element.hasAttribute(attrName);			//确认是否存在指定属性

element.setAttribute(name, value);		 //以k-v的方式设置属性
element.setAttributeNode(attributeNode); //以属性节点的方式设置元素属性

element.removeAttribute(attributeName);
element.removeAttributeNode(attributeNode);
```

## `node`属性

```javascript
node.parentNode
   
node.childNodes
node.firstChild
node.lastChild

node.nextSibling
node.previousSibing

node.nodeName		//节点名称
node.nodeType		//节点类型
node.nodeValue		//节点值。Text节点返回null；text,comment,CDATA节点返回该节点的文本内容；attribute节点返回该属性的属性值.
node.textContent
node.isConnected	//查询是否连接到DOM树上
node.ownerDocument	//获取document
node.baseURL		//获取当前URI
```

## 其他属性

```javascript
element.className
element.classList
element.attributes
element.children
element.id
element.tagName
element.innerHTML


element.className
element.classList

element.clientHeight
element.clientLeft
element.clientTop
element.clientWidth

element.scrollTop
element.scrollWidth

HTMLElement.innerText
HTMLElement.hidden
HTMLElement.lang
```

## 关于节点的移动注意事项

**<u>一个节点不可能同时出现在文档的不同位置。</u>**所以，如果**<u>某个节点已经拥有父节点，在被传递给此方法后，它首先会被移除，再被插入到新的位置。</u>**若要保留已在文档中的节点，可以**<u>先使用 [`Node.cloneNode()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/cloneNode) 方法来为它创建一个副本，再将副本附加到目标父节点下。</u>**请注意，用 `cloneNode` 制作的副本不会自动保持同步。

# 78.谈谈`createElement`与`createDocumentFragment`的区别？

`createDocumentFragment`是 `DOM` 节点。 它们不是主 `DOM` 树的一部分。通常的用例是创建**文档片段**，将元素附加到文档片段，然后将文档片段附加到`DOM`树。在`DOM`树中，文档片段被其所有的子元素所代替。因为文档片段存在于内存中，并不在`DOM`树中，所以**将子元素插入到文档片段时不会引起页面回流**（对元素位置和几何上的计算）。因此，使用文档片段通常会带来更好的性能。

| 方法                                | 描述                        |
| ----------------------------------- | --------------------------- |
| `document.createElement(tagname)`   | 创建标签名为`tagname`的节点 |
| `document.createDocumentFragment()` | 创建文档碎片节点            |

## 与`createElement`的主要区别

`nodeType`

- `createElement` 创建的是元素节点，节点类型(`nodeType`)为 **1**。
- `createDocumentFragment` 创建的是文档碎片，节点类型(`nodeType`)为 **11**。

`innerHTML`

- `createElement` 创建的元素可以直接使用 `innerHTML` 添加子元素。
- `createDocumentFragment` 创建的文档碎片不能直接使用 `innerHTML` 添加子元素，只会变成一个普通属性。

`appendChild`

- `createElement` 使用 `appendChild` 追加子元素时，如果**将被插入的节点已经存在于当前文档的文档树中，那么 `appendChild` 只会将它从原先的位置移动到新的位置**（不需要事先移除要移动的节点）；如果把`Element`追加进页面中，则插入的是`Element`本身和它的所有子孙节点；即便`Element`已经添加进了页面，我们依旧能继续重复操作，因为我们维持了对`DOM`节点的引用。
- `createDocumentFragment` 使用 `appendChild` 追加子元素时，**会把页面中的原来存在的元素删除**；如果**把`DocumentFragment`追加进页面中，则插入的不是 `DocumentFragment` 自身，而是它的所有子孙节点**；**如果`DocumentFragment`已经添加进了页面，我们就不能继续操作，它属于一次性操作。**

## 返回值

二者使用 `appendChild` 作为子元素追加到`DOM`后返回值都是新添加的子元素。可以借助返回值通过 `innerHTML` 间接给 `createDocumentFragment` 创建的文档碎片直接添加子元素。

## 何时使用`createDocumentFragment`

不希望进行重复的编辑`DOM`操作引发重绘、希望创建的`DOM`片段外层没有多余容器包裹。

# 79.`nodeType`、`nodeValue`、`nodeName`

# 80.获取、修改元素样式

```javascript
window.getComputedStyle(element[, pseudoElt])
//window.getComputedStyle()方法给出应用活动样式表后的元素的所有CSS属性的值，并解析这些值可能包含的任何基本计算（会触发重排）

element.getBoundingClientRect();
//getBoundingClientRect()用来返回元素的大小以及相对于浏览器可视窗口的位置

element.style.color = 'red'
//1.直接修改样式

element.style.cssText = 'color: red'
//2.通过cssText修改样式

element.setAttribute('class', element.className + ' ' + 'newClass')
//通过class修改样式
```

# 81.`Array.prototype.indexOf`，`Array.prototype.includes`以及`for-loop`之间的比较

- `indexOf`：使用的是`=== + while`
- `includes`：使用的是`Object.is() + while`

# 82.遍历数组的性能比较

![image-20211130113107537](https://github.com/NoAlligator/pico/blob/main/img/image-20211130113107537.png?raw=true)

# 83.说一说`document`对象上的可以直接获取的`DOM`标签？

`document.body`：获取<body/>

`document.documentElement`：获取<html/>

`document.head`：获取<head/>

`document.title`：获取<title/>

`document.doctype`：获取<DOCTYPE>

# 84.说一说`Object.defineProperty`

## 语法

> `Object.defineProperty(obj, prop, descriptor)`

## `descriptor`详解

### `descriptor`描述符有两种类型

- **数据描述符**
- **存取描述符**

### 两种类型的`descriptor`分别可以设置的值

#### 数据描述符

```js
{
	configurable: [false | true],
	enumberable: [false | true],
	writable: [false | true],
	value: any
}
```

#### 存取描述符

```js
{
	configurable: [false | true],
	enumberable: [false | true],
	get(){
		/*...*/
		return any
	},
	set(newValue){
		/*...*/
	}
}
```

#### 描述符属性详解

`writable`：是否只读；

`configurable`：是否不可配置（属性**不能被删除**、不能修改 `configurable` 标志、能修改 `enumerable` 标志、不能将 `writable: false` 修改为 `true`（反过来则可以修改）、能修改访问者属性的 `get/set`（如果没有的话可以分配它们））`"configurable: false"` 的用途是防止更改和删除属性标志，但是允许更改对象的值；

`enumberable`：是否可枚举；

`value`：属性对应的具体值；

`get`：取值函数；

`set`：存值函数。

#### 描述符默认值

`configurable`： `false`

`writable`： `false`

`enumberable`： `false`

`value`：`undefined` 

`get`：`undefined`

`set`：`undefined`

### 同时定义多个属性

> `Object.defineProperties(obj, props)`

```javascript
var obj = {};
Object.defineProperties(obj, {
  'property1': {
    value: true,
    writable: true
  },
  'property2': {
    value: 'Hello',
    writable: false
  }
});
```

### 获取属性描述符

```javascript
Object.getOwnPropertyDescriptor(obj, prop)
Object.getOwnPropertyDescriptors(obj)
```

> `Object.defineProperty`，顾名思义针对的是对象属性的定义，它能够提前部署并拦截对象指定属性的读写操作，但是无法获知整个对象上发生了怎样的变化，他的能力仅限于对象上的某个属性的变更；而`Proxy`是对整个对象的代理，针对对象任何一个属性的操作都会被代理代理对象拦截，所以这两者是有很大区别的。

# 85.一些特殊操作符

## 1. 数值分割符 _

`ES2021`引入了数值分割符 `_`，在数值组之间提供分隔，使一个长数值读起来更容易。`Chrome`已经提供了对数值分割符的支持，可以在浏览器里试起来。

```javascript
let number = 100_0000_0000_0000 // 0太多了不用数值分割符眼睛看花了
console.log(number)             // 输出 100000000000000
```

此外，十进制的小数部分也可以使用数值分割符，二进制、十六进制里也可以使用数值分割符。

```javascript
0x11_1 === 0x111   // true 十六进制
0.11_1 === 0.111   // true 十进制的小数
0b11_1 === 0b111   // true 二进制
```

## 2. 逗号运算符

```javascript
function reverse(arr) {
    return [arr[0], arr[1]]=[arr[1], arr[0]], arr[0] + arr[1]
}
const list = [1, 2]
reverse(list)   // 返回 3，此时 list 为[2, 1]
```

逗号操作符对它的每个操作数求值（从左到右），并返回**最后**一个操作数的值。

```javascript
expr1, expr2, expr3...
```

会返回最后一个表达式 `expr3` 的结果，其他的表达式只会进行求值。

## 3. 零合并操作符 ??

零合并操作符 `??` 是一个逻辑操作符，当左侧的操作数为 `null` 或者 `undefined` 时，返回右侧操作数，否则返回左侧操作数。

```javascript
expr1 ?? expr2
```

空值合并操作符一般用来为常量提供默认值，保证常量不为 `null` 或者 `undefined`，以前一般使用 `||` 来做这件事 `variable = variable || 'bar'`。然而，由于 `||` 是一个布尔逻辑运算符，左侧的操作数会被强制转换成布尔值用于求值。任何假值（`0`， `''`， `NaN`， `null`， `undefined`）都不会被返回。这导致如果你使用 `0`、`''`、`NaN` 作为有效值，就会出现不可预料的后果。

正因为 `||` 存在这样的问题，而 `??` 的出现就是解决了这些问题，`??` 只会在左侧为 `undefined`、`null` 时才返回后者，`??` 可以理解为是 `||` 的完善解决方案。

可以在浏览器中执行下面的代码感受一下：

```javascript
undefined || 'default' // 'default'
null || 'default'      // 'default'
false || 'default'     // 'default'
0 || 'default'         // 'default'

undefined ?? 'default' // 'default'
null ?? 'default'      // 'default'
false ?? 'default'     // 'false'
0 ?? 'default'         // 0
```

另外在赋值的时候，可以运用赋值运算符的简写 `??=`

```javascript
let a = {b: null, c: 10}
a.b ??= 20
a.c ??= 20
console.log(a)     // 输出 { b: 20, c: 10 }
```

## 4. 可选链操作符 ?.

可选链操作符 `?.` 允许读取位于连接对象链深处的属性的值，而不必验证链中的每个引用是否有效。`?.` 操作符的功能类似于 `.` 链式操作符，不同之处在于，在引用为 `null` 或者 `undefined` 的情况下不会引起错误，该表达式短路返回值是 `undefined`。当尝试访问可能不存在的对象属性时，可选链操作符将会使表达式更短、更简明。

```javascript
const obj = {
  a: 'foo',
  b: {
    c: 'bar'
  }
}

console.log(obj.b?.c)      // 输出 bar
console.log(obj.d?.c)      // 输出 undefined
console.log(obj.func?.())  // 不报错，输出 undefined
```

以前可能会通过 `obj && obj.a && obj.a.b` 来获取一个深度嵌套的子属性，现在可以直接 `obj?.a?.b` 即可。

可选链除了可以用在获取对象的属性，还可以用在数组的索引 `arr?.[index]`，也可以用在函数的判断 `func?.(args)`，当尝试调用一个可能不存在的方法时也可以使用可选链。

调用一个对象上可能不存在的方法时（版本原因或者当前用户的设备不支持该功能的场景下），使用可选链可以使得表达式在函数不存在时返回 `undefined` 而不是直接抛异常。

```javascript
const result = someInterface.customFunc?.()
```

## 5. 私有方法/属性

在一个类里面可以给属性前面增加 `#` 私有标记的方式来标记为私有，除了属性可以被标记为私有外，`getter/setter` 也可以标记为私有，方法也可以标为私有。

```javascript
class Person {
  getDesc(){ 
    return this.#name +' '+ this.#getAge()
  }
  
  #getAge(){ return this.#age } // 私有方法

  get #name(){ return 'foo' } // 私有访问器
  #age = 23                   // 私有属性
}
const a = new Person()
console.log(a.age)       // undefined 直接访问不到
console.log(a.getDesc()) // foo 23
```

## 6. 双位运算符 `~~`

可以使用双位操作符来替代正数的 `Math.floor( )`，替代负数的 `Math.ceil( )`。双否定位操作符的优势在于它执行相同的操作运行速度更快。

```javascript
Math.floor(4.9) === 4      // true
// 简写为：
~~4.9 === 4      // true
```

不过要注意，对正数来说 `~~` 运算结果与 `Math.floor( )` 运算结果相同，而对于负数来说与 `Math.ceil( )` 的运算结果相同：

```javascript
~~4.5                // 4
Math.floor(4.5)      // 4
Math.ceil(4.5)       // 5
 
~~-4.5               // -4
Math.floor(-4.5)     // -5
Math.ceil(-4.5)      // -4
```

## 7. `void` 运算符

```
void` 运算符 对给定的表达式进行求值，然后返回 undefined
```

可以用来给在使用立即调用的函数表达式（`IIFE`）时，可以利用 `void` 运算符让`JS`引擎把一个 `function` 关键字识别成函数表达式而不是函数声明。

```javascript
function iife() { console.log('foo') }()       // 报错，因为JS引擎把IIFE识别为了函数声明
void function iife() { console.log('foo') }()  // 正常调用
~function iife() { console.log('foo') }()      // 也可以使用一个位操作符
(function iife() { console.log('foo') })()     // 或者干脆用括号括起来表示为整体的表达式
```

还可以用在箭头函数中避免传值泄漏，箭头函数，允许在函数体不使用括号来直接返回值。这个特性给用户带来了很多便利，但有时候也带来了不必要的麻烦，如果右侧调用了一个原本没有返回值的函数，其返回值改变后，会导致非预期的副作用。

```javascript
const func = () => void customMethod()   // 特别是给一个事件或者回调函数传一个函数时
```

安全起见，当不希望函数返回值是除了空值以外其他值，应该使用 `void` 来确保返回 `undefined`，这样，当 `customMethod`返回值发生改变时，也不会影响箭头函数的行为。

# 86.说一说`Proxy`？

> `Proxy`是`ES6`引入的一个内置类，实现了对象的访问代理， `Proxy` 实例对象用于包装另一个对象并**拦截**诸如读取属性、写入属性、删除属性和其他操作，可以选择**拦截并处理**它们，或者透明地**允许该对象**处理它们。`Proxy`是语言层面上针对对象实现的代理模式。可以利用`Proxy`覆写针对当前对象的对象原生操作方法。

## 1.语法

- `target` —— 是要包装的对象，可以是任何东西，包括函数。
- `handler` —— 代理配置：带有“捕捉器”的对象。比如 `get` 捕捉器用于读取 `target` 的属性，`set` 捕捉器用于写入 `target` 的属性，等等。

```javascript
const proxy = new Proxy(target, handler)
```

## 2.没有捕捉器的`Proxy`

**如果没有指定任何捕捉器（`handler`），所有对 `proxy` 实例的操作（写入、读取、迭代等）都会直接转发给 `target`。**

## 3.`Proxy`可以通过捕捉器拦截什么？

| 内部方法                | `Handler`方法              | 何时触发                                                     |
| :---------------------- | :------------------------- | :----------------------------------------------------------- |
| `[[Get]]`               | `get`                      | **读取**属性                                                 |
| `[[Set]]`               | `set`                      | **写入**属性                                                 |
| `[[HasProperty]]`       | `has`                      | `in` 操作符                                                  |
| `[[Delete]]`            | `deleteProperty`           | `delete` 操作符                                              |
| `[[Call]]`              | `apply`                    | 函数调用                                                     |
| `[[Construct]]`         | `construct`                | `new` 操作符                                                 |
| `[[GetPrototypeOf]]`    | `getPrototypeOf`           | [Object.getPrototypeOf](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) |
| `[[SetPrototypeOf]]`    | `setPrototypeOf`           | [Object.setPrototypeOf](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) |
| `[[IsExtensible]]`      | `isExtensible`             | [Object.isExtensible](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible) |
| `[[PreventExtensions]]` | `preventExtensions`        | [Object.preventExtensions](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions) |
| `[[DefineOwnProperty]]` | `defineProperty`           | [Object.defineProperty](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty), [Object.defineProperties](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) |
| `[[GetOwnProperty]]`    | `getOwnPropertyDescriptor` | [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor), `for..in`, `Object.keys/values/entries` |
| `[[OwnPropertyKeys]]`   | `ownKeys`                  | [Object.getOwnPropertyNames](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames), [Object.getOwnPropertySymbols](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols), `for..in`, `Object/keys/values/entries` |

<img src="https://github.com/NoAlligator/pico/blob/main/img/image-20211207092518881.png" alt="image-20211207092518881" style="zoom: 67%;" />

## 4.`get`捕捉器拦截访问（`.`操作、`[]`操作）

- `target` —— 是目标对象，该对象被作为第一个参数传递给 `new Proxy`，
- `property` —— 目标属性名，
- `receiver` —— 如果目标属性是一个`getter`访问器属性，则 `receiver` 就是本次读取属性所在的 `this` 对象。通常，这就是 `proxy` 对象本身（或者，如果我们` proxy`继承，则是从该`proxy`继承的对象）。现在我们不需要此参数，因此稍后我们将对其进行详细介绍。

> 如果违背了以下的约束，proxy会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
>
> - 如果要访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同。

```javascript
//将常规数组包装到代理中，以捕获读取操作，并在没有要读取的属性的时返回0
let numbers = [0, 1, 2];
numbers = new Proxy(numbers, {
  get(target, prop) {
    if (prop in target) {
      return target[prop];
    } else {
      return 0; // 默认值
    }
  }
});
alert( numbers[1] ); // 1
alert( numbers[123] ); // 0（没有这个数组项）
```

## 5.`set`捕捉器拦截赋值

- `target` —— 是目标对象，该对象被作为第一个参数传递给 `new Proxy`，
- `property` —— 目标属性名称，
- `value` —— 目标属性的值，
- `receiver` —— 与 `get` 捕捉器类似，仅与 setter 访问器属性相关。

> 如果违背以下的约束条件，proxy 会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常：
>
> - 若目标属性是一个**不可写及不可配置**的数据属性，则**不能改变它的值**。
> - 如果目标属性**没有配置存储方法，即 `[[Set]]` 属性的是 `undefined`，则不能设置它的值**。
> - 如果 **`set()` 方法返回 `false`，那么也会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常**。
>
> 总结：如果写入操作成功，`set` 捕捉器应该返回 `true`，否则返回 `false`（触发 `TypeError`）。

```javascript

let numbers = [];

numbers = new Proxy(numbers, { // (*)
  set(target, prop, val) { // 拦截写入属性操作
    if (typeof val == 'number') {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  }
});

numbers.push(1); // 添加成功
numbers.push(2); // 添加成功
alert("Length is: " + numbers.length); // 2

numbers.push("test"); // TypeError（proxy 的 'set' 返回 false）

alert("This line is never reached (error in the line above)");
```

## 6.使用`ownKeys`和`getOwnPropertyDescriptor`捕捉器拦截键的遍历

`Object.keys`，`for..in` 循环和大多数其他遍历对象属性的方法都使用内部方法 `[[OwnPropertyKeys]]`（由 `ownKeys` 捕捉器拦截) 来获取属性列表。

会被`ownKeys`拦截的方法如下：

- `Object.getOwnPropertyNames(obj)` 返回非 Symbol 键。
- `Object.getOwnPropertySymbols(obj)` 返回 Symbol 键。
- `Object.keys/values()` 返回带有 `enumerable` 标志的非 Symbol 键/值（属性标志在 [属性标志和属性描述符](https://zh.javascript.info/property-descriptors) 一章有详细讲解)。
- `for..in` 循环遍历所有带有 `enumerable` 标志的非 Symbol 键，以及原型对象的键。

> 如果违反了下面的约束，`proxy`将抛出错误 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
>
> - `ownKeys` 的结果**必须是一个数组.**
> - 数组的元素类型**要么是一个 [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) ，要么是一个 [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol).**
> - 结果列表**必须包含目标对象的所有不可配置**（`non-configurable` ）、自有（`own`）属性的key.
> - 如果目标对象**不可扩展**，那么结果列表**必须包含目标对象的所有自有（`own`）属性的`key`，不能有其它值**.
>
> 如果下列不变量被违反，代理将抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)：
>
> - `getOwnPropertyDescriptor` 必须**返回一个 `object`或 `undefined`。**
> - 如果属性作为目标对象的**不可配置的属性存在，则该属性无法报告为不存在**。
> - 如果属性**作为目标对象的属性存在，并且目标对象不可扩展，则该属性无法报告为不存在**。
> - 如果属性**不存在作为目标对象的属性，并且目标对象不可扩展，则不能将其报告为存在。**
> - 属性**不能被报告为不可配置，如果它不作为目标对象的自身属性存在，或者作为目标对象的可配置的属性存在。**
> - `Object.getOwnPropertyDescriptor（target）`的结果可以**使用`Object.defineProperty`应用于目标对象，也不会抛出异常。**

```javascript
//使用 ownKeys 捕捉器拦截 for..in 对 user 的遍历，并使用 Object.keys 和 Object.values 来跳过以下划线 _ 开头的属性
let user = {
  name: "John",
  age: 30,
  _password: "***"
};

user = new Proxy(user, {
  ownKeys(target) {
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
});

// "ownKeys" 过滤掉了 _password
for(let key in user) alert(key); // name，然后是 age

// 对这些方法的效果相同：
alert( Object.keys(user) ); // name,age
alert( Object.values(user) ); // John,30
```

如果我们返回对象中不存在的键，`Object.keys` 并不会列出这些键：

```javascript
let user = { };

user = new Proxy(user, {
  ownKeys(target) {
    return ['a', 'b', 'c'];
  }
});

alert( Object.keys(user) ); // <empty>
```

原因很简单：`Object.keys` 仅返回带有 `enumerable` 标志的属性。为了检查它，该方法会对每个属性调用内部方法 `[[GetOwnProperty]]` 来获取 它的描述符（`descriptor`）。在这里，由于没有属性，其描述符为空，没有 `enumerable` 标志，因此它被略过。为了让 `Object.keys` 返回一个属性，我们需要它要么存在于带有 `enumerable` 标志的对象，要么我们可以拦截对 `[[GetOwnProperty]]` 的调用（捕捉器 `getOwnPropertyDescriptor` 可以做到这一点)，并返回带有 `enumerable: true` 的描述符。

```javascript
let user = { };

user = new Proxy(user, {
  ownKeys(target) { // 一旦要获取属性列表就会被调用
    return ['a', 'b', 'c'];
  },

  getOwnPropertyDescriptor(target, prop) { // 被每个属性调用
    return {
      enumerable: true,
      configurable: true
      /* ...其他标志，可能是 "value:..." */
    };
  }

});

alert( Object.keys(user) ); // a, b, c
```

如果该属性在对象中不存在，那么我们**只需要拦截** `[[GetOwnProperty]]`。

## 7.使用`deleteProperty`捕捉器拦截删除操作

- `target` —— 是目标对象，该对象被作为第一个参数传递给 `new Proxy`
- `property` —— 待删除属性名称

> 如果违背了以下不变量，`proxy`将会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
>
> - 如果目标对象的属性是不可配置的，那么该属性不能被删除。
>
> 如果删除操作成功，`set` 捕捉器应该返回 `true`，否则返回 `false`（任何`falsy`都将触发 `TypeError`）。

```javascript
const person = new Proxy({name: 'ycp'}, {
  deleteProperty: function(target, prop) {
    if(!(prop in target)) return false
    console.log('delete: ' + prop);
    return true;
  }
});
Reflect.deleteProperty(person, 'name')
```

## 8.使用`has`捕捉器拦截`in`操作符

- `target` —— 是目标对象，被作为第一个参数传递给 `new Proxy`，
- `property` —— 属性名称。

> 如果违反了下面这些规则,  `proxy`将会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
>
> - 如果目标对象的某一属性本身**不可被配置**，则该属性**不能够被代理隐藏**.
> - 如果目标对象为**不可扩展对象**，则该对象的属性**不能够被代理隐藏**
>
> `has` 方法返回一个`boolean`属性的值.

```javascript
//使用in操作符来检查一个数字是否在range范围内
let range = {
  start: 1,
  end: 10
};

range = new Proxy(range, {
  has(target, prop) {
    return prop >= target.start && prop <= target.end;
  }
});
```

## 9.其他捕捉器

- **`defineProperty()`** 用于拦截对对象的 [`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 操作。

  - `defineProperty` 方法必须以一个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean) 返回，表示定义该属性的操作成功与否。

  - > 如果违背了以下的不变量，`proxy`会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
    >
    > - 如果目标对象**不可扩展， 将不能添加属性**。
    > - 不能**添加或者修改一个为不可配置的属性**，如果**它不作为一个目标对象的不可配置的属性存在的话。**
    > - 如果目标对象**存在一个对应的可配置属性，这个属性可能不会是不可配置的**。
    > - 如果一个**属性在目标对象中存在对应的属性，那么 `Object.defineProperty(target, prop, descriptor)` 将不会抛出异常**。
    > - 在严格模式下， `false` 作为` handler.defineProperty` 方法的返回值的话将会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常.

  - ```javascript
    var p = new Proxy({}, {
      defineProperty: function(target, prop, descriptor) {
        console.log('called: ' + prop);
        return true;
      }
    });
    
    var desc = { configurable: true, enumerable: true, value: 10 };
    Object.defineProperty(p, 'a', desc);
    ```

- **`setPrototypeOf()`** 方法主要用来拦截 [`Object.setPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf).

  - 如果成功修改了`[[Prototype]]`, `setPrototypeOf` 方法返回 `true`,否则返回 `false`.

  - > 如果违反了下列规则，则`proxy`将抛出一个[`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
    >
    > - `如果 target` 不可扩展, 原型参数必须与`Object.getPrototypeOf(target)` 的值相同.

  - ```javascript
    var handlerReturnsFalse = {
        setPrototypeOf(target, newProto) {
            return false;
        }
    };
    
    var newProto = {}, target = {};
    
    var p1 = new Proxy(target, handlerReturnsFalse);
    Object.setPrototypeOf(p1, newProto); // throws a TypeError
    Reflect.setPrototypeOf(p1, newProto); // returns false
    ```

- **`getPrototypeOf()`** 是一个代理（Proxy）方法，当读取代理对象的原型时，该方法就会被调用。

  - `getPrototypeOf` 方法的返回值必须是一个对象或者 `null`。

  - > 如果遇到了下面两种情况，`JS` 引擎会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常：
    >
    > - `getPrototypeOf()` 方法**返回的不是对象也不是 `null。**`
    > - 目标对象**是不可扩展的**，且 `getPrototypeOf()` 方法返回的**原型不是目标对象本身的原型。**

  - ```javascript
    var obj = {};
    var proto = {};
    var handler = {
        getPrototypeOf(target) {
            console.log(target === obj);   // true
            console.log(this === handler); // true
            return proto;
        }
    };
    
    var p = new Proxy(obj, handler);
    console.log(Object.getPrototypeOf(p) === proto);    // true
    ```

- `construct()` 方法用于拦截[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 操作符. 为了使`new`操作符在生成的`Proxy`对象上生效，用于初始化代理的目标对象自身必须具有`[[Construct]]`内部方法（即 `new target` 必须是有效的）。

  - `construct` 方法必须返回一个对象。

  - > 如果违反以下约定，代理将会抛出错误 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
    >
    > - 必须返回一个对象.

  - ```javascript
    var p = new Proxy(function() {}, {
      construct: function(target, argumentsList, newTarget) {
        console.log('called: ' + argumentsList.join(', '));
        return { value: argumentsList[0] * 10 };
      }
    });
    
    console.log(new p(1).value);
    ```

- `isExtensible()`方法用于拦截对对象的`Object.isExtensible()`。

  - `isExtensible`方法必须返回一个 Boolean值或可转换成Boolean的值。

  - > 如果违背了以下的约束，proxy会抛出 TypeError:
    >
    > - `Object.isExtensible(proxy)` 必须同`Object.isExtensible(target)`返回相同值。也就是必须返回`true`或者为`true`的值,返回`false`和为`false`的值都会报错。

  - ```javascript
    var p = new Proxy({}, {
      isExtensible: function(target) {
        console.log('called');
        return true;//也可以return 1;等表示为true的值
      }
    });
    
    console.log(Object.isExtensible(p)); // "called"
                                         // true
    ```

- **`preventExtensions()`** 方法用于设置对[`Object.preventExtensions()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)的拦截

  - `preventExtensions` 方法返回一个布尔值.

  - > 如果违反了下列规则, proxy则会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError):
    >
    > - 如果目标对象是可扩展的，那么只能返回 `false`

  - ```javascript
    var p = new Proxy({}, {
      preventExtensions: function(target) {
        console.log('called');
        Object.preventExtensions(target);
        return true;
      }
    });
    
    console.log(Object.preventExtensions(p)); // "called"
                                              // false
    ```

- **`apply()`** 方法用于拦截函数的调用（`call`、`apply`、`proxy()`）。

  - `apply`方法可以返回任何值。

  - > 如果违反了以下约束，代理将抛出一个`TypeError`：
    >
    > `target`必须是可被调用的。也就是说，它必须是一个函数对象。

  - ```javascript
    var p = new Proxy(function() {}, {
      apply: function(target, thisArg, argumentsList) {
        console.log('called: ' + argumentsList.join(', '));
        return argumentsList[0] + argumentsList[1] + argumentsList[2];
      }
    });
    
    console.log(p(1, 2, 3)); // "called: 1, 2, 3"
                             // 6
    ```

## 10.使用`apply`进行代理的好处

案例：调用 `delay(f, ms)` 会返回一个函数，该函数会在 `ms` 毫秒后把所有调用转发给 `f`。

```javascript
function delay(f, ms) {
  // 返回一个包装器（wrapper），该包装器将在时间到了的时候将调用转发给函数 f
  return function() { // (*)
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

// 在进行这个包装后，sayHi 函数会被延迟 3 秒后被调用
sayHi = delay(sayHi, 3000);

sayHi("John"); // Hello, John! (after 3 seconds)
```

包装函数缺点：但是包装函数不会转发属性读取/写入操作或者任何其他操作。进行包装后，就失去了对原始函数属性的访问，例如 `name`，`length` 和其他属性：

```javascript
function delay(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

alert(sayHi.length); // 1（函数的 length 是函数声明中的参数个数）

sayHi = delay(sayHi, 3000);

alert(sayHi.length); // 0（在包装器声明中，参数个数为 0)
```

`Proxy` 的功能要强大得多，因为它可以将所有东西转发到目标对象。

```javascript
function delay(f, ms) {
  return new Proxy(f, {
    apply(target, thisArg, args) {
      setTimeout(() => target.apply(thisArg, args), ms);
    }
  });
}

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

sayHi = delay(sayHi, 3000);

alert(sayHi.length); // 1 (*) proxy 将“获取 length”的操作转发给目标对象

sayHi("John"); // Hello, John!（3 秒后）
```

结果是相同的，但现在不仅仅调用，而且代理上的所有操作都能被转发到原始函数。所以在 `(*)` 行包装后的 `sayHi.length` 会返回正确的结果。我们得到了一个“更丰富”的包装器。

## 11.`receiver`参数的详解（`receiver`）

我们有一个带有 `_name` 属性和`getter`的对象 `user`。这是对 `user` 对象对一个代理（`proxy`）：

```javascript
let user = {
  _name: "Guest",
  get name() {
    return this._name;
  }
};

let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    return target[prop];
  }
});

alert(userProxy.name); // Guest
```

另一个对象 `admin` 从 `user` 继承后，我们可以观察到错误的行为：

```javascript
let user = {
  _name: "Guest",
  get name() {
    return this._name;
  }
};

let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    return target[prop]; // (*) target = user
  }
});

let admin = {
  __proto__: userProxy,
  _name: "Admin"
};

// 期望输出：Admin
alert(admin.name); // 输出：Guest (?!?)
```

读取 `admin.name` 应该返回 `"Admin"`，而不是 `"Guest"`！发生了什么？或许我们在继承方面做错了什么？问题实际上出在代理中，在 `(*)` 行。

1. 当我们读取 `admin.name` 时，由于 `admin` 对象自身没有对应的的属性，搜索将转到其原型。

2. 原型是 `userProxy`。

3. 从代理读取 `name` 属性时，`get` 捕捉器会被触发，并从原始对象返回 `target[prop]` 属性，在 `(*)` 行。

   当调用 `target[prop]` 时，若 `prop` 是一个 getter，它将在 `this=target` 上下文中运行其代码。因此，结果是来自原始对象 `target` 的 `this._name`，即来自 `user`。

为了解决这种情况，我们需要 `get` 捕捉器的第三个参数 `receiver`。它保证将正确的 `this` 传递给`getter`。在我们的例子中是 `admin`。对于一个常规函数，我们可以使用 `call/apply`，但这是一个`getter`，它不能“被调用”，只能被访问。以下代码将陷入死循环（无限递归）。

```javascript
let user = {
  _name: "Guest",
  get name() {
    return this._name;
  }
};

let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    return receiver[prop]; // (*) target = user
  }
});

let admin = {
  __proto__: userProxy,
  _name: "Admin"
};

// 期望输出：Admin
alert(admin.name); // 输出：Guest (?!?)
```

`Reflect.get` 可以做到，如果我们使用它，一切都会正常运行。

```javascript
let user = {
  _name: "Guest",
  get name() {
    return this._name;
  }
};

let userProxy = new Proxy(user, {
  get(target, prop, receiver) { // receiver = admin
    return Reflect.get(target, prop, receiver); // (*)
  }
});


let admin = {
  __proto__: userProxy,
  _name: "Admin"
};

alert(admin.name); // Admin
```

现在 `receiver` 保留了对正确 `this` 的引用（即 `admin`），该引用是在 `(*)` 行中被通过 `Reflect.get` 传递给`getter`的。我们可以把捕捉器重写得更短：

```javascript
get(target, prop, receiver) {
  return Reflect.get(...arguments);
}
```

`Reflect` 调用的命名与捕捉器的命名完全相同，并且接受相同的参数。它们是以这种方式专门设计的。因此，`return Reflect...` 提供了一个安全的方式，可以轻松地转发操作，并确保我们不会忘记与此相关的任何内容。

## 12.使用代理的注意点

代理应当在所有地方都完全替代目标对象。目标对象被代理后，任何人都不应该再引用目标对象，否则容易导致错误。

```javascript
dictionary = new Proxy(dictionary, handlers)
```

# 87.说一说`Reflect`?

### 定义

`Reflect`对象与`Proxy`对象一样，也是`ES6`为了操作对象而提供的新`API`。`Reflect`对象的设计目的是：

1.将将一些明显属于**语言内部的方法转移到`Reflect`对象**上，未来**新的针对对象的方法将只在`Reflect`上部署**，也就是说，从`Reflect`对象上可以获得语言内部的方法。

2.修改某些`Object`方法的返回结果，让其**变得更合理**。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会**抛出一个错误**，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。

3.让部分针对对象的操作变成了**函数行为**（例如`Reflect.has`替代`in`操作符，`Reflect.deleteProperty`替代`delete`操作符）。

4.`Reflect`对象的方法与`Proxy`对象的方法的handler是**一一对应**的，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，**完成默认行为，作为修改行为的基础**。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为来作为fallback，只针对指定的情况进行劫持。

## `Reflect`对象一共有 13 个静态方法

```javascript
Reflect.apply(target, thisArg, args)
Reflect.construct(target, args)
Reflect.get(target, name, receiver)
Reflect.set(target, name, value, receiver)
Reflect.defineProperty(target, name, desc)
Reflect.deleteProperty(target, name)
Reflect.has(target, name)
Reflect.ownKeys(target)
Reflect.isExtensible(target)
Reflect.preventExtensions(target)
Reflect.getOwnPropertyDescriptor(target, name)
Reflect.getPrototypeOf(target)
Reflect.setPrototypeOf(target, prototype)
```

# 88.说一说`Reference Type`？

```javascript
let user = {
  name: "John",
  hi() { alert(this.name); }
}

// 把获取方法和调用方法拆成两行
let hi = user.hi;
hi(); // 报错了，因为 this 的值是 undefined
```

**为确保 `user.hi()` 调用正常运行， 点 `'.'` 返回的不是一个函数，而是一个特殊的`Reference Type`的值。**`Reference Type`是`ECMA`中的一个**“规范类型”**。我们不能直接使用它，但它被用在`JavaScript`语言内部。`Reference Type`的值是一个三个值的组合 `(base, name, strict)`，其中：

- `base` 是对象。
- `name` 是属性名。
- `strict` 在 `use strict` 模式下为 true。

所以，对属性 `user.hi` 访问的结果不是一个函数，而是一个`Reference Type`的值。对于 `user.hi`，在严格模式下如以下代码所示：

```javascript
// Reference Type 的值
(user, "hi", true)
```

> 当 `()` 被在`Reference Type`上调用时，它们**会接收到关于对象和对象的方法的完整信息，然后可以设置正确的 `this`**（在此处 `this = user`）。`Reference Type`是一个**特殊的“中间人”内部类型**，目的是从 `.` 传递信息给 `()` 调用。任何例如赋值 `hi = user.hi` 等其他的操作，都会将`Reference Type`作为一个整体丢弃掉，而会取 `user.hi`（一个函数）的值并继续传递。所以任何后续操作都“丢失”了 调用者`this`。因此，`this` 的值仅在函数直接被通过点符号 `obj.method()` 或方括号 `obj['method']()` 语法（此处它们作用相同）调用时才被正确传递。还有很多种解决这个问题的方式，例如 `func.bind()`。
>

## 注意点

```javascript
let user = {
    name: "John",
    go: function() { alert(this.name) }
}

void (user.go)()
```

`(user.go)()`就等同于`user.go()`

# 89.说一说浏览器事件和相应的`api`?

## 定义

> **事件** 是某事发生的信号。所有的`DOM`节点都生成这样的信号（但事件不仅限于`DOM`）。

## 事件处理程序

> 为了对事件作出响应，我们可以分配一个 **处理程序（`handler`）**—— 一个在事件发生时运行的函数。处理程序是在发生用户行为（`action`）时运行`JavaScript`代码的一种方式。有三种分配处理程序的方法：
>
> 1. `HTML` 特性（`attribute`）：`onclick="..."`。
> 2. `DOM`属性（`property`）：`elem.onclick = function`。
> 3. 方法（`method`）：`elem.addEventListener(event, handler[, options | useCapture])` 用于添加，`removeEventListener(type, listener[, options | useCapture])` 用于移除。

## 事件处理函数里的`this`

> 处理程序中的 `this` 的值是对应的元素。就是处理程序所在的那个元素。

### `addeventListner`

```javascript
element.addEventListener(event, handler[, options]);
element.removeEventListener(event, handler[, options]);
//要移除处理程序，我们需要传入与分配的函数完全相同的函数
```

`options`具有以下属性的附加可选对象：

- `once`：如果为 `true`，那么会在**被触发后自动删除监听器**。
- `capture`：**事件处理的阶段**。由于历史原因，`options` 也可以是 `false/true`，它与 `{capture: false/true}` 相同。
- `passive`：如果为 `true`，告诉浏览器处理程序**将明确不会调用 `preventDefault()`。**

## `addEventListner`和`onclick`的区别

`onclick`**不能直接为一个元素的同一事件分配多个事件处理函数**（只能变通的使用`AOP`），`addEventListner`**可以为一个元素的同一个事件分发多个事件处理函数**（在事件处理阶段按照定义顺序执行）。对于某些事件，**只能通过 `addEventListener` 设置处理程序，有些事件无法通过`DOM`属性进行分配**。只能使用 `addEventListener`。例如，`DOMContentLoaded` 事件，该事件在文档加载完成并且 DOM 构建完成时触发。所以 `addEventListener` 更通用。

```javascript
// 无效的
document.onDOMContentLoaded = function() {
    alert("DOM built");
};
// 这种方式可以运行
document.addEventListener("DOMContentLoaded", function() {
    alert("DOM built");
});
```

## `event`对象上的有用属性

`event.type`：事件类型；

`event.currentTarget`：**处理事件**的元素；

`event.target`：**触发事件**的元素。

# 90.说一说冒泡和捕获？

## 事件传播

**`DOM`事件标准**描述了事件传播的`3`个阶段：

1. 事件冒泡：事件从`事件目标元素`开始，从`事件目标`元素`往祖先元素冒泡`，如果没有中途被`stopPropagation()`拦截的话事件会一直传播到页面`window`；
2. 事件捕获：事件从`祖先元素`往`子元素`查找，直到捕获到事件目标；
3. 目标阶段：不是**冒泡阶段**，也不是**捕获阶段**，捕获目标事件之后会`按照目标定义事件回调函数的顺序（无论是冒泡还是捕获）`执行回调函数。

<img src="https://github.com/NoAlligator/pico/blob/main/img/image-20211207103214351.png" alt="image-20211207103214351" style="zoom:50%;" />

## 阻止事件继续传播

用于阻止事件继续传播的方法是 `event.stopPropagation()`。用于阻止事件继续传播（**和阻止当前元素上的处理程序运行**）的方法是`event.stopImmediatePropagation()`

## 审慎使用阻止事件传播

有时 `event.stopPropagation()` 会产生隐藏的陷阱，以后可能会成为问题，例如：我们创建了一个嵌套菜单，每个子菜单各自处理对自己的元素的点击事件，并调用 `stopPropagation`，以便不会触发外部菜单。之后，我们决定捕获在整个窗口上的点击，以追踪用户的行为（用户点击的位置）。有些分析系统会这样做。通常，代码会使用`document.addEventListener('click'…)` 来捕获所有的点击。我们的分析不适用于被 `stopPropagation` 所阻止点击的区域，因为我们有一个阻止冒泡的“死区”。通常，没有真正的必要去阻止冒泡。一项看似需要阻止冒泡的任务，可以通过其他方法解决。其中之一就是**使用自定义事件**。我们还可以将我们的数据写入一个处理程序中的 `event` 对象，并在另一个处理程序中读取该数据，这样我们就可以向父处理程序传递有关下层处理程序的信息。

## 冒泡顺序

现代浏览器

- `div -> body -> html -> document -> window`

`IE6.0`

- `div -> body -> html -> document`

## 非冒泡事件

少数事件例如`blur`、`focus`、`load`、`unload`、`onmouseenter`、`onmouseleave`事件不会冒泡。

## 事件触发对象 和 事件绑定对象

**引发事件的那个嵌套层级最深的元素被称为目标元素，可以通过 `event.target` 访问。**

- `event.target` —— 是引发事件的“目标”元素，它在冒泡过程中不会发生变化。
- `this === event.currentTarget` —— 是“当前”元素，其中有一个当前正在运行的处理程序。

## 设置捕获阶段触发

```javascript
elem.addEventListener(..., {capture: true})
// 或者，用 {capture: true} 的别名 "true"
elem.addEventListener(..., true)
```

## 处理阶段

针对目标元素来说，虽然形式上有`3`个阶段，但第`2`阶段（“目标阶段”：事件到达元素）没有被单独处理：**捕获阶段和冒泡阶段的处理程序都在该阶段被触发**（按照事件处理函数的注册顺序依次执行）。

针对非目标元素来说，`useCapture`决定了他在什么阶段被触发，例如`useCapture`为`true`时代表仅在捕获阶段触发，那么冒泡阶段就不会触发这个事件处理函数。反之亦然。

## 移除事件注意点

**要移除处理程序，**`removeEventListener` 需要同一阶段，如果我们 `addEventListener(..., true)`，那么我们应该在 `removeEventListener(..., true)` 中提到同一冒泡/捕获阶段，以正确删除处理程序。

# 91.事件委托

## 定义

> 如果我们有许多以**类似方式处理**的元素，那么就不必为每个元素分配一个处理程序 —— 而是将单个处理程序放在它们的**共同祖先**上。在处理程序中，我们通过获取 `event.target` 以查看事件实际发生的元素并进行处理。

## 事件委托：潜在的风险

```html
<table>
    <tr>
        <th colspan="3"><em>Bagua</em> Chart: Direction, Element, Color, Meaning</th>
    </tr>
    <tr>
        <td class="nw"><strong>Northwest</strong><br>Metal<br>Silver<br>Elders</td>
        <td class="n">...</td>
        <td class="ne">...</td>
    </tr>
    <tr>...2 more lines of this kind...</tr>
    <tr>...2 more lines of this kind...</tr>
</table>
```

```javascript
let selectedTd;

table.onclick = function(event) {
    let target = event.target; // 在哪里点击的？
    if (target.tagName != 'TD') return; // 不在 TD 上？那么我们就不会在意
    highlight(target); // 高亮显示它
};

function highlight(td) {
    if (selectedTd) { // 移除现有的高亮显示，如果有的话
        selectedTd.classList.remove('highlight');
    }
    selectedTd = td;
    selectedTd.classList.add('highlight'); // 高亮显示新的 td
}
```

缺陷：点击可能不是发生在 `<td>` 上，而是发生在**其内部**。如果我们看一下`HTML`内部，我们可以看到 `<td>` 内还有**嵌套的标签**，例如 `<strong>`。如果在该 `<strong>` 上点击，那么它将成为 `event.target` 的值（导致事件处理函数内部判断没有“命中”元素）。在处理程序 `table.onclick` 中，我们应该接受这样的 `event.target`，并确定该点击是否在 `<td>` 内。可以使用`event.closet(selector)`进行优化：`Element.closest()` 方法用来获取匹配特定选择器且离当前元素最近的祖先元素（也可以是**当前元素本身**），如果匹配不到，则返回 `null`。

```javascript
table.onclick = function(event) {
    let td = event.target.closest('td'); // (1)
    if (!td) return; // (2)
    if (!table.contains(td)) return; // (3)
    highlight(td); // (4)
};
//(1)elem.closest(selector) 方法返回与 selector 匹配的最近的祖先。在我们的例子中，我们从源元素开始向上寻找 <td>。
//(2)如果 event.target 不在任何 <td> 中，那么调用将立即返回，因为这里没有什么事儿可做。
//(3)对于嵌套的表格，event.target 可能是一个 <td>，但位于当前表格之外。因此我们需要检查它是否是 我们的表格中的 <td>。
//(4)如果是的话，就高亮显示它。

//还有一种方法，就是直接判断触发元素是不是目标元素的后代
```

## 事件委托：标记中的行为

我们想要编写一个有“保存”、“加载”和“搜索”等按钮的菜单。并且，这里有一个具有 `save`、`load` 和 `search` 等方法的对象。如何匹配它们？第一个想法可能是为每个按钮分配一个单独的处理程序。但是有一个更优雅的解决方案。我们可以为整个菜单添加一个处理程序，并为具有方法调用的按钮添加 `data-action` 特性（`attribute`）：

```html
<div id="menu">
    <button data-action="save">Save</button>
    <button data-action="load">Load</button>
    <button data-action="search">Search</button>
</div>

<script>
    class Menu {
        constructor(elem) {
            this._elem = elem;
            elem.onclick = this.onClick.bind(this); // (*)
        }

        save() {
            alert('saving');
        }

        load() {
            alert('loading');
        }

        search() {
            alert('searching');
        }

        onClick(event) {
            let action = event.target.dataset.action;
            if (action) {
                this[action]();
            }
        };
    }

    new Menu(menu);
</script>
```

`this.onClick` 在 `(*)` 行中被绑定到了 `this`。这很重要，因为否则内部的 `this` 将引用 DOM 元素（`elem`），而不是 `Menu` 对象，那样的话，`this[action]` 将不是我们所需要的。

该模式的好处：我们**不需要编写代码来为每个按钮分配一个处理程序**。只需要**创建一个方法并将其放入标记（`markup`）中**即可；`HTML`结构非常灵活，我们可以随时添加/移除按钮。

## “行为”模式（`H5`相关知识）

我们还可以使用事件委托将“行为（`behavior`）”以 **声明方式** 添加到具有**特殊特性（`attribute`）和类的元素中**。

行为模式分为**两个部分**：

1. 我们将自定义特性添加到描述其行为的元素。
2. 用文档范围级的处理程序追踪事件，如果事件发生在具有特定特性的元素上 —— 则执行行为（`action`）。

```html
<!-- 这里的特性 data-counter 给按钮添加了一个“点击增加”的行为。 -->
Counter: <input type="button" value="1" data-counter>
One more counter: <input type="button" value="2" data-counter>
<!-- 我们可以根据需要使用 data-counter 特性，多少都可以。我们可以随时向 HTML 添加新的特性。使用事件委托，我们属于对 HTML 进行了“扩展”，添加了描述新行为的特性。 -->
<script>
    document.addEventListener('click', function(event) {
        if (event.target.dataset.counter != undefined) { // 如果这个特性存在...
            event.target.value++;
        }
    });
</script>
```

```html
<button data-toggle-id="subscribe-mail">
    Show the subscription form
</button>

<form id="subscribe-mail" hidden>
    Your mail: <input type="email">
</form>

<script>
    document.addEventListener('click', function(event) {
        let id = event.target.dataset.toggleId;
        if (!id) return;
        let elem = document.getElementById(id);
        elem.hidden = !elem.hidden;
    });
</script>
<!-- “行为”模式可以替代 JavaScript 的小片段。 -->
```

## 事件委托优缺点总结

### 优点

- 简化初始化并节省内存：**无需添加许多处理程序。**
- 更少的代码：**添加或移除元素时，无需添加/移除处理程序。**
- `DOM`修改 ：我们可以使用 `innerHTML` 等，来**批量添加/移除元素。**

### 缺点

- 首先，**事件必须冒泡**。而有些事件不会冒泡。此外，低级别的处理程序不应该使用 `event.stopPropagation()`。
- 其次，委托可能会**增加`CPU`负载**，因为容器级别的处理程序会对容器中任意位置的事件做出反应，而不管我们是否对该事件感兴趣。但是，通常负载可以忽略不计，所以我们不考虑它。

# 92.浏览器的默认行为有哪些？如何阻止？

## 常见默认事件

- 点击一个**链接** —— 触发导航（`navigation`）到该 `URL`。
- 点击表单的**提交按钮** —— 触发提交到服务器的行为。
- 在文本上**按下鼠标按钮并移动** —— 选中文本。

## 阻止默认行为

有两种方式来告诉浏览器我们不希望它执行默认行为：

- 主流的方式是使用 `event` 对象。有一个 `event.preventDefault()` 方法。
- 如果处理程序是使用 `on<event>`（而不是 `addEventListener`）分配的，那返回 `false` 也同样有效。

事件会相互转化。如果我们阻止了第一个事件，那就没有第二个事件了。例如，在 `<input>` 字段上的 `mousedown` 会导致在其中获得焦点，以及 `focus` 事件。如果我们阻止 `mousedown` 事件，在这就没有焦点了。

## `passive`选项

`addEventListener` 的可选项 `passive: true` 向浏览器发出信号，表明处理程序将不会调用 `preventDefault()`。

为什么需要这样做？[参考链接](https://medium.com/@justjavac/rolling-performance-optimization-passive-event-listeners-4f942eec4dca)

移动设备上会发生一些事件，例如 `touchmove`（当用户在屏幕上移动手指时），**默认情况下会导致滚动**，但是可以使用处理程序的 `preventDefault()` 来阻止滚动。因此，当浏览器检测到此类事件时，**它必须首先处理所有处理程序，然后如果没有任何地方调用 `preventDefault`，则页面可以继续滚动。但这可能会导致 UI 中不必要的延迟和“抖动”。**`passive: true` 选项告诉浏览器，**处理程序不会取消滚动**。然后浏览器**立即滚动页面以提供最大程度的流畅体验，并通过某种方式处理事件。**对于某些浏览器（`Firefox`，`Chrome`），默认情况下，`touchstart` 和 `touchmove` 事件的 `passive` 为 `true`。如果用简单一句话来解释就是：**提升页面滑动的流畅度**。

## 阻止事件传播的替代方案：利用`event.defaultPrevented`标识事件处理情况

> 如果默认行为被阻止，那么 `event.defaultPrevented` 属性为 `true`，否则为 `false`。

以下程序，子元素通过显式地`preventDefault()`改变了`defaultPrevented`的布尔值，父元素可以接收到这个`defaultPrevented`从而获知子类已经处理相应事件，不再继续处理事件。

```html
<p>Right-click for the document menu (added a check for event.defaultPrevented)</p>
<button id="elem">Right-click for the button menu</button>

<script>
    elem.oncontextmenu = function(event) {
        event.preventDefault();
        alert("Button context menu");
    };

    document.oncontextmenu = function(event) {
        if (event.defaultPrevented) return;
        event.preventDefault();
        alert("Document context menu");
    };
</script>
```

# 93.如何创建自定义事件？

> 自定义事件可用于创建“图形组件”。例如，我们自己的基于`JavaScript`的菜单的根元素可能会触发 `open`（打开菜单），`select`（有一项被选中）等事件来告诉菜单发生了什么。另一个代码可能会监听事件，并观察菜单发生了什么。我们不仅可以生成出于自身目的而创建的全新事件，还可以生成例如 `click` 和 `mousedown` 等内建事件。这可能会有助于自动化测试。

## 使用方法

```javascript
let event = new Event(type[, options])
```

- **type** —— 事件类型，可以是像这样 `"click"` 的字符串，或者我们自己的像这样 `"my-event"` 的参数。

- **options** —— 具有两个可选属性的对象：

  - `bubbles: true/false` —— 如果为 `true`，那么事件会冒泡。
  - `cancelable: true/false` —— 如果为 `true`，那么“默认行为”就会被阻止。稍后我们会看到对于自定义事件，它意味着什么。

  默认情况下，以上两者都为`false`：`{bubbles: false, cancelable: false}`。

事件对象被创建后，我们应该使用 `elem.dispatchEvent(event)` 调用在元素上“运行”它。然后，处理程序会对它做出反应，就好像它是一个常规的浏览器事件一样。如果事件是用 `bubbles` 标志创建的，那么它会冒泡。

```html
<button id="elem" onclick="alert('Click!');">Autoclick</button>

<script>
  let event = new Event("click");
  elem.dispatchEvent(event);
</script>
```

## 区分自定义事件

有一种方法可以区分“真实”用户事件和通过脚本生成的事件。对于来自真实用户操作的事件，`event.isTrusted` 属性为 `true`，对于脚本生成的事件，`event.isTrusted` 属性为 `false`。

## 冒泡实例

```html
<h1 id="elem">Hello from the script!</h1>

<script>
  // 在 document 上捕获...
  document.addEventListener("hello", function(event) { // (1)
    alert("Hello from " + event.target.tagName); // Hello from H1
  });

  // ...在 elem 上 dispatch！
  let event = new Event("hello", {bubbles: true}); // (2)
  elem.dispatchEvent(event);

  // 在 document 上的处理程序将被激活，并显示消息。

</script>
```

## 具体的事件类

如果我们想要创建具体存在的事件，我们应该使用具体的事件类生成的自定义事件而不是 `new Event`。例如，`new MouseEvent("click")`。正确的构造器允许为该类型的事件指定**标准属性**。

因为构造器`Event`仅支持设置`冒泡`和`捕获的设置`，无法设置具体的属性来模拟具体的事件。

```javascript
let event = new Event("click", {
  bubbles: true, // 构造器 Event 中只有 bubbles 和 cancelable 可以工作
  cancelable: true,
  clientX: 100,
  clientY: 100
});

alert(event.clientX); // undefined，未知的属性被忽略了！
```

具体的事件类如下

- `UIEvent`
- `FocusEvent`
- `MouseEvent`
- `WheelEvent`
- `KeyboardEvent`
- `...`

## 自定义事件类

对于我们自己的全新事件类型，例如 `"hello"`，我们应该使用 `new CustomEvent`。从技术上讲，`CustomEvent`和 `Event` 一样。除了一点不同：在第二个参数（对象）中，我们可以为我们想要与事件一起传递的任何自定义信息添加一个附加的属性 `detail`。`detail` 属性可以有任何数据。从技术上讲，我们可以不用，因为我们可以在创建后将任何属性分配给常规的 `new Event` 对象中。但是 `CustomEvent` 提供了特殊的 `detail` 字段，以避免与其他事件属性的冲突。

```html
<h1 id="elem">Hello for John!</h1>

<script>
  // 事件附带给处理程序的其他详细信息
  elem.addEventListener("hello", function(event) {
    alert(event.detail.name);
  });

  elem.dispatchEvent(new CustomEvent("hello", {
    detail: { name: "John" }
  }));
</script>
```

## 自定义事件的默认行为与取消

如果自定义事件对应的事件处理函数调用 `event.preventDefault()`，那么事件处理程序可以发出一个信号，指出这些行为应该被取消。在这种情况下，`elem.dispatchEvent(event)` 的调用会返回 `false`。那么分派（`dispatch`）该事件的代码就会知道不应该再继续（事件）。

以下程序：`<button/>`上绑定了自定义事件`'hide'`，但是其处理函数内部却调用了`event.preventDefault`，同时分发的自定义事件设置了`cancelable: true`，所以这个`dispatch(event)`会返回`false`代表绑定的事件处理函数取消了默认事件。

```html
<pre id="rabbit">
  |\   /|
   \|_|/
   /. .\
  =\_Y_/=
   {>o<}
</pre>
<button onclick="hide()">Hide()</button>

<script>
  function hide() {
    let event = new CustomEvent("hide", {
      cancelable: true // 没有这个标志，preventDefault 将不起作用
    });
    if (!rabbit.dispatchEvent(event)) {
      alert('The action was prevented by a handler');
    } else {
      rabbit.hidden = true;
    }
  }

  rabbit.addEventListener('hide', function(event) {
    if (confirm("Call preventDefault?")) {
      event.preventDefault();
    }
  });
</script>
```

## 事件中的事件是同步的

通常事件是在队列中处理的。也就是说：如果浏览器正在处理 `onclick`，这时发生了一个新的事件，例如鼠标移动了，那么它的处理程序会被排入队列，相应的 `mousemove` 处理程序将在 `onclick` 事件处理完成后被调用。但是，如果一个事件是在另一个事件中发起的，那么被发起的事件会被立即处理。

```html
<button id="menu">Menu (click me)</button>

<script>
  menu.onclick = function() {
    alert(1);

    menu.dispatchEvent(new CustomEvent("menu-open", {
      bubbles: true
    }));

    alert(2);
  };

  // 在 1 和 2 之间触发
  document.addEventListener('menu-open', () => alert('nested'));
</script>
<!-- 1 → nested → 2 -->
```

```html
<button id="menu">Menu (click me)</button>

<script>
  menu.onclick = function() {
    alert(1);

    setTimeout(() => menu.dispatchEvent(new CustomEvent("menu-open", {
      bubbles: true
    })));

    alert(2);
  };

  document.addEventListener('menu-open', () => alert('nested'));
</script>

<!-- 1 → 2 → nested -->
```

# 94.event.stopPropagation()和event.preventDefault()，return false的区别

**一、event.stopPropagation()**

阻止事件的冒泡，不让事件向`document`上蔓延，但是默认事件任然会执行，当调用这个方法的时候，如果点击一个连接，这个连接仍然会被打开。

**二、event.preventDefault()**

阻止默认事件的方法，调用此方法时，连接不会被打开，但是会发生冒泡，冒泡会传递到上一层的父元素。

**三、return false；**

```html
// 行内return false可以阻止默认行为、无法阻止冒泡；行内this返回元素本身
<div onclick="click(this); return false"></div>

// 行内调用全局函数 + return false 不可以阻止默认行为、也无法阻止冒泡；行内传入事件对象
<div onclick="click(e)"></div>
<script>
function click(e) {
    console.log(e)
    return false
}
</script>

// 行内调用全局函数 + event.returnValue = false 可以阻止默认行为、无法阻止冒泡；行内传入事件对象
<div onclick="click(e)"></div>
<script>
function click(e) {
    console.log(e)
    e.returnValue = false
}
</script>
```





# 95.`Class`中的原型方法和`Object`中的方法在`this`绑定上的重要差别

> 当调用静态或原型方法时没有指定 *this* 的值，那么方法内的 *this* 值将被置为 **`undefined`**。即使你未设置 "use strict" ，因为 `class` 体内部的代码总是在**严格模式**下执行。通过传统的基于函数的语法来实现，那么依据初始的 *this* 值，在**非严格模式**下方法调用会发生自动装箱。若初始值是 `undefined`，*this* 值会被设为全局对象。严格模式下不会发生自动装箱，*this* 值将保留传入状态。
>

```javascript
class TestClass {
    testThis() {
        console.log(this)
    }
}
let testClass = new TestClass()
let fn1 = test.testThis
fn1() // undefined
function TestFunction() {}
TestFunction.prototype.testThis = function () {
    console.log(this)
}
let testFunction = new TestFunction()
let fn2 = testFunction.testThis
fn2() // window in non-strict mode, undefined in use strict mode
```

## 在Class内绑定this的方法

- 在构造函数constructor内部进行bind绑定显式改变this。

  ```javascript
  class Test {
      constructor() {
          this.fn = this.fn.bind(this)
      }
      fn() {
          console.log(this)
      }
  }
  ```

- 在使用函数时使用bind显式绑定指定实例。

- 使用箭头函数定义实例方法（利用箭头函数的特性）。

- 使用`::`操作符（类似bind），是最新的提案。

# 96.数组空位处理

ES5 对空位的处理，已经很不一致了，**大多数情况下会忽略空位**：

- `forEach()`, `filter()`, `reduce()`, `every()` 和`some()`都会跳过空位。
- `map()`会跳过空位，但会保留这个值
- `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

ES6 则是**明确将空位转为`undefined`**：

`Array.from`方法会将数组的空位，转为`undefined`，也就是说，这个方法不会忽略空位。

```js
Array.from(['a',,'b'])// [ "a", undefined, "b" ]
```

扩展运算符（`...`）也会将空位转为`undefined`。

```js
[...['a',,'b']]// [ "a", undefined, "b" ]
```

`copyWithin()`会连空位一起拷贝。

```js
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]
```

`fill()`会将空位视为正常的数组位置。

```js
new Array(3).fill('a') // ["a","a","a"]
```

`for...of`循环也会遍历空位。

```js
let arr = [, ,];for (let i of arr) {  console.log(1);}// 1// 1
```

上面代码中，数组`arr`有两个空位，`for...of`并没有忽略它们。如果改成`map`方法遍历，空位是会跳过的。

`entries()`、`keys()`、`values()`、`find()`和`findIndex()`会将空位处理成`undefined`。

```js
// entries()[...[,'a'].entries()] // [[0,undefined], [1,"a"]]// keys()[...[,'a'].keys()] // [0,1]// values()[...[,'a'].values()] // [undefined,"a"]// find()[,'a'].find(x => true) // undefined// findIndex()[,'a'].findIndex(x => true) // 0
```

由于空位的处理规则非常不统一，所以建议避免出现空位。

# 区别：`Object.preventExtensions()`, `Object.seal()`, `Object.Freeze()`

| API                          | 可以添加新属性 | 可以配置 | 可以修改、删除属性 | 可以修改原型 |
| ---------------------------- | -------------- | -------- | ------------------ | ------------ |
| `Object.preventExtensions()` | ×              | √        | √                  | √            |
| `Object.seal()`              | ×              | ×        | √                  | √            |
| `Object.Freeze()`            | ×              | ×        | ×                  | ×            |

# 97.每一轮`Event Loop`都会伴随着渲染吗？

