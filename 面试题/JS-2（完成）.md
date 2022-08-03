# 

## 递归与栈溢出

执行栈本身也是有**<u>容量限制</u>**的，当执行栈内部的执行上下文对象积压到一定程度如果继续积压，就会报 “栈溢出（`stack overflow`）” 的错误。栈溢出错误经常会发生在递归中。**<u>递归的使用场景，通常是在运行次数未知的情况下，程序会设定一个限定条件，除非达到该限定条件否则程序将一直调用自身运行下去</u>**。

## 尾调用

当函数的最后一步`return`是**只调用另一个函数**时，才称作尾调用。

```javascript
//尾调用
function f(x){
  return g(x);
}
//不是尾调用
function f(x){
  let y = g(x);
  return y;
}

//不是尾调用
function f(x){
  return g(x) + 1;
}
```

函数调用会在内存形成一个"调用记录"，又称"调用帧"（call frame），**<u>保存调用位置和内部变量等信息。</u>**如果在函数A的内部调用函数B，那么在A的调用记录上方，**<u>还会形成一个B的调用记录</u>**。等到B运行结束，将结果返回到A，B的调用记录才会消失。如果函数B内部还调用函数C，那就还有一个C的调用记录栈，以此类推。所有的调用记录，就形成一个["调用栈"](https://zh.wikipedia.org/wiki/调用栈)（call stack）。尾调用由于是**<u>函数的最后一步操作，所以不需要保留外层函数的调用记录</u>**，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用记录，取代外层函数的调用记录就可以了。（执行上下文栈提前出栈并由尾调用函数执行上下文代替，使得栈溢出不会发生）如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。这就是"尾调用优化"的意义。

## 尾递归优化

**<u>函数调用自身，称为递归。如果尾调用自身，就称为尾递归。</u>**递归非常耗费内存，因为需要同时保存成千上百个调用记录，很容易发生"栈溢出"错误（stack overflow）。但对于尾递归来说，由于只存在一个调用记录，所以永远不会发生"栈溢出"错误。

```javascript
function factorial(n) {
    if (n === 1) return 1;
    return n * factorial(n - 1);
}
factorial(5)

//使用尾递归优化
function factorial(n, total) {
    if (n === 1) return total;
    return factorial(n - 1, n * total);
}
factorial(5, 1)
```

### 尾递归优化的改写方式

> 尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是**把所有用到的内部变量改写成函数的参数。**比如上面的例子，阶乘函数 `factorial`需要用到一个中间变量`total`，那就把这个中间变量改写成函数的参数。 
>

方法一：在尾递归函数之外，再提供一个正常形式的函数。下面代码通过一个正常形式的阶乘函数`factorial`，调用尾递归函数`tailFactorial`。

```javascript
function tailFactorial(n, total) {
    if (n === 1) return total;
    return tailFactorial(n - 1, n * total);
}

function factorial(n) {
    return tailFactorial(n, 1);
}

factorial(5) // 120
```

方法二：采用柯里化的方式

```javascript
function currying(fn, n) {
    return function (m) {
        return fn.call(this, m, n);
    };
}

function tailFactorial(n, total) {
    if (n === 1) return total;
    return tailFactorial(n - 1, n * total);
}

const factorial = currying(tailFactorial, 1);

factorial(5) // 120
```

方法三：`ES6`默认参数

```javascript
function factorial(n, total = 1) {
    if (n === 1) return total;
    return factorial(n - 1, n * total);
}

factorial(5) // 120
```

# 52.说一说`this`指向？

## 情形

### 作为普通函数调用

- 在严格模式下`this`是`undefined`；
- 在非严格模式下`this`是全局对象。

### 作为方法调用

- 默认情况下，`this`指向调用者对象；
- 使用`call`、`apply`的显式绑定可以指定`this`为入参对象；
- 使用`bind`绑定`this`使得方法的`this`被确定下来，不会被调用者所修改，因为`bind`的优先级更高。

### 作为箭头函数

- 定义在**<u>函数作用域</u>**中时，箭头函数的`this`指向其所在函数作用域对应的普通函数的`this`。所继承的普通函数的`this`指向改变，箭头函数的`this`也会随之改变。
- 定义在**<u>全局作用域</u>**时，箭头函数的`this`指向`window`（非严格模式下）。
- 箭头函数的`this`无法通过`call`、`apply`、`bind`所改变，因为它没有自己的`this`。

### 作为构造函数调用

- 构造函数中的`this`指向创建的实例，且优先级高于`bind`；

### 作为事件处理函数回调（普通函数）

- `onclick`和`addEventerListener`事件处理回调（非箭头函数）的`this`是指向绑定事件的元素本身（注意是元素，不是事件）。
- `xhr.onreadystatechange`的`this`指向请求对象本身。
- 一些浏览器，比如`IE6~IE8`下使用`attachEvent`，`this`指向是`window`。

> ##### `this`绑定的优先级:`new` 调用 > `call、apply、bind` 调用 > 对象上的函数调用 > 普通函数调用
>

# 53.说一说`DOM`和`BOM`的区别？

`DOM`指的是**<u>文档对象模型（`Document Object Model`）</u>**，它指的是**<u>把文档当做一个对象</u>**，这个对象主要定义了**<u>处理网页内容的方法和接口</u>**，在`JS`中通过`window.document`来访问（`DOM`也是`BOM`的子对象）。`DOM`是`W3C`标准。

`BOM`指的是**<u>浏览器对象模型</u>**，它指的是**<u>把浏览器当做一个对象来对待</u>**，这个对象主要定义了与**浏览器进行交互的法和接口**。`BOM`的核心是 `window`，而`window`对象具有双重角色，它既是通过 `JS`访问浏览器窗口的一个接口，又是一个 `Global`（全局）对象。这意味着在网页中定义的任何对象，变量和函数，都作为全局对象的一个属性或者方法存在（前提是在非严格模式下使用`var` ）。`window`对象含有`location`对象、`navigator`对象`screen `对象等子对象，并且`DOM`的最根本的对象`document`对象也是`BOM` `windo`对象的子对象。

![img](https://github.com/NoAlligator/pico/blob/main/img/7166236-ac9c88e8fc0cb3c4.png?raw=true)

> `document`代表的是整个文档（对于一个网页来说包括整个网页结构，包括了`DOCTYPE、html`标签），`document.documentElement`是整个文档节点树的根节点，在网页中即整个`<html/>`标签； `document.body`是整个文档`DOM`节点树里的`body`节点，网页中即为`body`标签元素。`document.head`是整个`<head/>`标签。

# 54.`escape`、`encodeURI`、`encodeURIComponent `的区别

`encodeURI`是对整个`URI`进行转义，**将`URI`中的非法字符转换为合法字符**，所以对于一些在`URI`中有特殊意义的字符不会进行转义。`encodeURI`方法不会对下列字符编码 `ASCII字母 数字 ~!@#$&\*()=:/,;?+'`

`encodeURIComponent`是对`URI`的**组成部分进行转义**，所以一些特殊字符也会得到转义。`encodeURIComponent`方法不会对下列字符编码`ASCII字母 数字 ~!\*()'`（转义范围比encodeURI更广，比如encodeURI不会转义?，而encodeURLComponent会转义URL中的一些保留字比如://, ?）

`escape`和`encodeURI`的作用相同，不过它们对于`unicode`编码为`0xff`之外字符的时候会有区别`escape`是直接在字符的`unicode`编码前加上`%u`，而`encodeURI`首先会将字符转换为`UTF-8`的格式，再在每个字节前加上 `%`。`escape`是对字符串进行编码（而另外两种是对`URL`），作用是让它们在所有电脑上可读。**最关键的是，当你需要对`URL`编码时，请忘记这个方法，这个方法是针对字符串使用的，不适用于`URL`。**

如果是编码、解码完整`URI`：`encodeURI`、`decodeURI`；

如果是编码、解码`URL`查询参数：`encodeURIComponent`、`decodeURIComponent`；

编码字符串：`escape`（建议不要使用这个方法，使用`encodeURI`代替）。

# 55.简单说一说`Ajax`?

## 简易`Ajax`请求

```js
let request = new XMLHttpRequest();
//设置请求头
request.open("GET", "/path");
//设置请求行
request.setRequestHeader("token","Bear xxxx");
request.setRequestHeader("Content-type","application/json");
//设置请求体
request.send(JSON.stringify({name: "ycp"}));
//设置请求状态变更时的处理回调
request.onreadystatechange = function() {
    if (request.readyState !== 4) return
    if (this.status === 200) {
        const {responseText} = this;
        console.log(responseText)
    } else {
        const {statusText} = this;
        console.error(statusText)
    }
};
```

## 请求相关

- `request.open`是用来设置`HTTP`请求行的；

- `request.setHeader`是用来设置`HTTP`请求头的，以`key:value`的格式设置；
- `request.send`是用来设置`HTTP`请求体的，是一个字符串的形式,`get`请求也可设置第四部分，不报错，但不起作用。

##### 请求状态：

| 值   | 状态                             | 描述                                                         |
| ---- | -------------------------------- | ------------------------------------------------------------ |
| 0    | `UNSENT`(未打开)                 | `open()`方法还未被调用.                                      |
| 1    | `OPENED ` (未发送)               | `open()`方法已经被调用.                                      |
| 2    | `HEADERS_RECEIVED`(已获取响应头) | `send()`方法已经被调用, 响应头和响应状态已经返回但未开始下载响应体. |
| 3    | `LOADING` (正在下载响应体)       | 响应体下载中; `responseText`中已经获取了部分数据.            |
| 4    | `DONE` (请求完成)                | 整个请求过程已经完毕.                                        |

##### 响应相关：

- `request.status`、`request.statusText`是响应行（状态码、描述）；

- `request.getAllResponseHeaders()`或者`request.getResponseHeader()`是响应头；

- `request.responseText`是响应体。

##### 所有相关属性：

```javascript
const {response, responseText, status, statuText, readyState, responseType, responseURL, responseXML, upload} = xhr

xhr.abort() //中止
xhr.open()	//开启
xhr.send()	//发送

xhr.onabort = handler
xhr.onerror = handler
xhr.onloadend = handler
xhr.onstart = handler
xhr.onprogress = handler
xhr.ontimeout = handler // 超时回调
xhr.onreadystatechange = handler

xhr.timeout = num // 超时时间
xhr.withCredentials = true // 是否携带非同源cookie

xhr.getAllResponseHeaders()
xhr.getResponseHeader(key)
```

# 56.几种常见的`POST`等请求体提交数据方式

## `Ajax`如何指定请求体内容类型？

```javascript
xhr.setHeader("Content-type", "application/json");
```

## 常见的数据请求方式

### `application/x-www-form-urlencoded`

- 最常见的`POST`提交数据的方式，浏览器的原生 `<form>` 表单，如果不设置 `enctype` 属性，那么最终就会以 `application/x-www-form-urlencoded` 方式提交数据；
- 提交的数据按照 `key1=val1&key2=val2`的方式进行编码，`key` 和 `val` 都进行了 `URL` 转码。

### `multipart/form-data`

- 我们使用表单上传文件时，必须让 `<form>` 表单的 `enctype` 等于 `multipart/form-data`。

### `application/json`

- 以`JSON`字符串的格式提交数据，支持比键值对复杂得多的结构化数据，序列化方便。

### `text/xml`

- `XML`数据格式，使用较少。

### `text/plain`

- 纯文本，`ajax`默认属性。

------

# 56.为什么要进行给变量提升？他导致了什么问题？

##### 主要有以下两个原因：

- 提高性能：在JS代码执行之前，会进行**<u>语法检查和预编译（创建执行上下文）</u>**，并且这一操作只进行一次。这么做就是为了**<u>提高性能</u>**，如果没有这一步，那么**<u>每次执行代码前都必须重新解析一遍该函数</u>**，而这是没有必要的，因为函数中的代码并不会改变，解析一遍就够了。在解析的过程中，还会为函数生成**预编译代码**。在预编译时，会**<u>统计声明了哪些变量、创建了哪些函数，并对函数的代码进行压缩，去除注释、不必要的空白等。</u>**这样做的好处就是每次执行函数时都可以直接为该函数分配栈空间，并且因为代码压缩的原因，代码执行也更快了（解析和预编译过程中的声明提升可以提高性能，让函数可以在执行时预先为变量分配栈空间）；
- 容错性更好：变量提升可以在一定程度上**提高`JS`的容错性，让一些不规范的代码也可以正常执行**（可以不必担心在函数定义位置之前调用）；

> 导致问题：忽略了`var`没有作用域，在条件判断中进行声明，实际上会被变量提升到函数执行上下文中，导致变量声明前就可以访问（无视了条件判断）；忽略了`for`循环中的循环变量会被状态提升，最终变成全局对象上的属性；忽略了函数内不使用`var`声明就直接使用的变量会泄漏到全局对象上；所有在定义变量前访问变量导致的异常行为。

# 57.说一说`CommonJS`和`ES6 Module`的模块机制的异同点？

[参考](https://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)

> `commonJS`是`node`的模块化规范。 `ES Module`是`ES6+`的模块化规范。

## `CommonJS`

> 在`node.js`中，每一个模块都有对应的`module`变量，其上存在着`exports`属性，代表对外暴露的内容。每一个模块在还可以访问到`exports`关键字，它仅仅是指向了`module.exports`，最终对外输出的内容最终是由`module.exports`决定。所以，针对`exports`直接赋值会切断`export`对`module.exports`的引用。

```javascript
exports === module.exports
//module.exports 初始值为一个空对象 {}
//exports 是指向的 module.exports 的引用
//require()返回的是module.exports而不是 exports
//结论1：exports只能使用exports.xx = xx的方式赋值，否则就切断了和module.exports的引用关系而导致导出失效。
//结论2：改变module.exports的引用会导致exports的引用切断
```

```javascript
/*
*a.js
*/
module.exports = {
	name: 'ycp'  			//切断引用
}
exports = module.exports  	//重新维系引用
exports.age = 21   			//添加

/*
*b.js
*/
const res = reuqire('./a.js')
console.log(res) //{name: 'ycp', age: 21}
```

> `Node.js` 要求 `ES6` 模块采用`.mjs`后缀文件名。也就是说，只要脚本文件里面使用`import`或者`export`命令，那么就必须采用`.mjs`后缀名。Node.js 遇到`.mjs`文件，就认为它是 `ES6` 模块，默认启用严格模式，不必在每个模块文件顶部指定`"use strict"`。

| 区别       | Common JS        | ES6 Module                                     |
| ---------- | ---------------- | ---------------------------------------------- |
| 拷贝形式   | 输出的是拷贝     | 输出的是静态引用                               |
| 加载方式   | 运行时加载       | 编译时输出接口（代码运行前）                   |
| 同步异步   | 同步加载         | 异步加载                                       |
| 动态导入   | 支持动态导入     | 使用`import()`语法，返回`promise`对象          |
| 使用限制   | 在任何语句中     | 必须在顶层引入                                 |
| 修改源数据 | 针对源值的浅拷贝 | 只读不可直接修改，但是可以修改引用类型内部属性 |
| 异步       | 不支持           | 支持import()异步导入                           |

## 不同点

### 根本区别

- `CommonJS` 模块输出的是一个值的浅拷贝，`ES6` 模块输出的是值的引用。
- `CommonJS` 模块是**运行时加载**，`ES6` 模块是**编译时输出接口**。
- `CommonJS` 模块的`require()`是**同步加载模块**，后面的代码必须等待这个命令执行完，才会执行；`ES6` 模块的的`import`命令是异步加载，有一个独立的模块依赖的解析阶段，但是其中代码的执行是同步的。

### 是否可以修改取值变量

`ES6 Module`引入的变量是**只存只读的，不论是基本数据类型还是引用数据类型，不能直接改变其值**，通过`import`声明的变量类似`const`，但是可以修改引用类型内部属性。

`CommonJS`对于基本数据类型，属于**复制**；对于复杂数据类型，属于**浅拷贝**。可以对通过`require()`取得外部值的变量**重新赋值**。

### 引入后修改的影响

因为`CommonJS`是一个副本，当拷贝副本是**<u>引用类型</u>**时，**<u>第一层属性的修改不会影响模块的导出值</u>**（内部引用除外），基本类型的话根本不会产生影响。最重要的是，由于是拷贝值，**<u>一旦模块输出这个值，模块内部的变化</u>**（仅浅层次的引用类型属性和基本类型）**<u>就影响不到这个值</u>**。

`CommonJS`修改引用类型的内部值会影响模块的导出值（但是因为**<u>只读</u>**）。

### 加载机制

`CommonJS`模块是**运行时加载**，`CommonJS`加载的**<u>是一个对象</u>**，即`module.export`属性的复制/浅拷贝。

`ES6`模块是**编译时输出接口**，该对象**<u>只有在脚本运行结束时才会生成</u>**。而`ES6`模块不是对象，**<u>它的对外接口只是一种静态定义，在代码静态解析阶段就会生成</u>**。

## 动态导入

`CommonJS`支持**<u>动态导入</u>**，就是可以在语句中使用`require()`。

`import`命令具有提升效果，会提升到整个模块的头部，首先执行。这种行为的本质是，`import`命令是**<u>编译阶段执行的，在代码运行之前</u>**。由于`import`是静态执行，所以**<u>不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构</u>**。 `import` **<u>只能声明在该文件的最顶部</u>**，`import`存在**<u>声明提升</u>**。

## 相同点

1. 导出值是引用类型时，引用类型内部属性如果是引用类型，修改之后会产生副作用：`CommonJS`和`ES6 Module`都可以对引入的的对象进行属性变更，其中不同的是`ES6 Module`改变引用对象属性会直接影响模块实例，`Common JS`当且仅当对象的属性是引用类型时会对模块导出值产生影响。
2. 重复导入的特性：`require()`命令**<u>加载同一个模块时，不会再执行该模块，而是取到缓存之中的值</u>**。也就是说，`CommonJS`模块无论加载多少次，**<u>都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果</u>**，除非执行`delete require.cache[require.resolve(relativePath)]`手动清除系统缓存。`ES6 Module`也是相同的机制，多次`import`并不会导致**<u>重新导入</u>**。

# 58.`JS`如何实现动态`import`？

> `ES2020`提案 引入`import()`函数，支持动态加载模块。`import`命令能够接受什么参数，`import()`函数就能接受什么参数，**<u>两者区别主要是后者为动态加载</u>**。`import()`**返回一个 Promise 对象**。`import()`函数可以用在任何地方，**<u>不仅仅是模块，非模块的脚本也可以使用</u>**。它是运行时执行，也就是说，什么时候运行到这一句，就会加载指定的模块。另外，`import()`函数**<u>与所加载的模块没有静态连接关系</u>**，这点也是与`import`语句不相同。`import()`类似于 Node 的`require`方法，区别主要是**<u>前者是异步加载，后者是同步加载。</u>**

```javascript
import(specifier) //import函数的参数specifier，指定所要加载的模块的位置。
```

> ##### 作用：按需加载、条件加载、动态的模块路径、实现`Vue`中的懒加载。
>

# 59.`ES6 Module`的细节

- `export x`的导出形式是错误的，必须以对象形式导出。`export default let x = xxx`的导出形式也是错误的。**`export default`命令其实只是输出一个叫做`default`的变量，所以它后面不能跟变量声明语句**。
- `export`命令可以出现在模块的任何位置，只要处于**模块顶层**就可以。如果**处于块级作用域**内，就会报错，`import`命令也是如此。

```javascript
//单独导出
export let a = 1

//合并导出
let b = 1
let c = 1
export {b, c}

//导出函数
export function d(){}

//导出别名
let e = 1
export {e as E}

//默认导出变量
let f = 1
export default f

//导出函数
export default function g(){}
export default function (){}
//或
g(){}
export default g

//引入非default模块
import {a as aAlias, b} from './target'

//引入所有
import * as whole from './target'

//引入default模块
import {default as alias} from './target'
//或
import alias from './target'

//同时引入default和非default
import defaultAlias, { a as aAlias } from './target' 

//复合import，export
export { foo as xx, bar } from 'my_module';
export * from './target'
export * as alias from './target'

```

# 60.说一说严格模式？

类的方法自动开启严格模式，模块内部自动开启严格模式

> 将"use strict"放在脚本文件的第一行，则整个脚本都将以"严格模式"运行。如果这行语句不在第一行，则无效，整个脚本以"正常模式"运行。如果不同模式的代码文件合并成一个文件，这一点需要特别注意。"use strict"仅针对当前脚本块生效。将"use strict"放在函数体的第一行，则整个函数以"严格模式"运行。

> - **禁止未声明就使用变量并且绑定到全局对象上**
> - **`with`禁用，`eval`不再为包围作用域引入新变量**
> - **禁止`this`指向全局对象**
> - 禁止访问`Function.callee`和`Function.caller`
> - **禁止删除变量、删除函数**
> - **静默失败行为转为抛出错误（不可写属性写入、getter赋值、不可扩展对象赋值、不可删除属性删除）**
> - **变量重名抛出错误**
> - **禁止`0`为前缀表示八进制数字**
> - 禁止`arguments`赋值，`arguments`和实参双向绑定，访问`arguments.callee`
> - 函数不能声明在块级作用域
> - 新增保留字
> - 在作用域 eval() 创建的变量不能被调用

- **变量必须声明后再使用**；
- 函数的参数不能有同名属性，否则报错；
- 不能使用`with`语句；
- 不能对只读属性赋值，否则报错；
- 不能使用前缀 0 表示八进制数，否则报错；
- 不能删除不可删除的属性，否则报错
- 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`
- `eval`不会在它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反映函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`function.caller`
- 禁止`this`指向全局对象
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字（比如`protected`、`static`和`interface`）

# 61.如何判断某一个对象是否属于某个类?

- `instanceof`：判断构造函数的`prototype`属性是否出现在对象的原型链中的任何位置。
- `constructor`：通过对象访问原型对象上的`constructor`属性来判断构造函数（不安全）。
- `Obejct.prototype.toString.call(obj)`：通过这个方式可以判断所有内置引用类型和基本类型。

# 62.`for...in`和`for..of`的区别？

`for…of`是`ES6`新增的遍历方式，**允许遍历一个含有`[Symbol.iterator]`接口的数据结构并且返回各项的值**，和`ES3`中的`for…in`的区别如下：

- **`for…of `用于遍历实现了`iterator`接口的数据解构**，`for…in`用于遍历**<u>对象的键名</u>**。

- `for… in`会遍历对象的**<u>整个原型链（可枚举的+`Symbol`）</u>**，性能非常差不推荐使用（使用`Object.keys()`）。

# 63.解释性语言和编译型语言的区别

##### 解释型语言

使用专门的解释器**对源程序逐行解释成特定平台的机器码并立即执行**。是代码**在执行时才被解释器一行行动态翻译和执行，而不是在执行之前就完成翻译。**解释型语言不需要事先编译，其直接将源代码解释成机器码并立即执行，所以只要某一平台提供了相应的解释器即可运行该程序。其特点总结如下：

- 解释型语言每次运行都需要将源代码解释成机器码并执行，效率较低；
- 只要平台提供相应的解释器，就可以运行源代码，所以可以方便源程序移植；
- `JavaScript`、`Python`等属于解释型语言。

##### 编译型语言 

使用专门的编译器，针对特定的平台，**将高级语言源代码一次性的编译成可被该平台硬件执行的机器码，并包装成该平台所能识别的可执行性程序的格式。**在编译型语言写的程序执行之前，需要一个专门的编译过程，把源代码编译成机器语言的文件，如`exe`格式的文件，以后要再运行时，直接使用编译结果即可，如直接运行`exe`文件。因为只需编译一次，以后运行时不需要编译，所以编译型语言执行效率高。其特点总结如下：

- 一次性的编译成平台相关的机器语言文件，运行时脱离开发环境，运行效率高；
- 与特定平台相关，一般无法移植到其他平台；
- `C`、`C++`等属于编译型语言。

# 64.`xhr`、`axios`、`fetch`的区别

## `XHR`

> 1. `XHR`会自动带上同源的cookie，不会带上不同源的cookie
> 2. 可以通过前端设置`xhr.withCredentials = true`， 后端设置header的方式来让ajax自动带上不同源的cookie，但是这个属性对同源请求没有任何影响， 会被自动忽略。

`XHR`即异步`JavaScript`和 `XML`，是指一种创建交互式网页应用的网页开发技术。它是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。通过在后台与服务器进行少量数据交换，`Ajax`可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。传统的网页（不使用`Ajax`）如果需要更新内容，必须重载整个网页页面。

### 缺点

- 本身是针对`MVC`编程，不符合前端`MVVM`的浪潮。
- 基于原生`XHR`开发，`XHR`本身的架构不清晰。
- 不符合关注分离（`Separation of Concerns`）的原则（请求、响应在同一个`xhr`对象上）。
- 配置和调用方式非常混乱，而且**基于事件的异步模型**不友好。

> 不符合关注分离、基于事件的异步模型

## `Fetch Api` 

> fetch api在默认情况下, 不管是同域还是跨域ajax请求都不会带上cookie, 只有当设置了credentials 时才会带上

`fetch api`号称是`AJAX`的替代品，是在`ES6`出现的，使用了`ES6`中的`promise`对象。`Fetch`是基于`promise`设计的。`Fetch`的代码结构比起`ajax`简单多。`fetch`不是`ajax`的进一步封装，而是原生`JS`，**没有使用`XMLHttpRequest`对象**。

### 优点

- 语法简洁，更加语义化；
- **基于标准`Promise`实现，原生支持`async/await`**；
- **更加底层，提供的`API`丰富（`request`, `response`）**；
- **脱离了`XHR`，是`ES`规范里新的实现方式**；
- `fetch`支持通过`new AbortController()`进行`abort`；
- `fetch`通过**获取响应头的`contentLength`可以计算下载进度**；

`fetch`的缺点：

- `fetch`只对**网络错误报错**，对`400，500`都当做成功的请求，服务器返回`400，500 `错误码时并不会`reject`，只有**网络错误这些导致请求不能完成时**，`fetch`才会被`reject`；
- `fetch`的`credentials`属性在不同浏览器上表现不太一致，一般是`same-origin`，和ajax默认行为一致），最好显式地设置这一属性，需要添加配置项： `fetch(url, {credentials: 'include' | 'same-origin' | 'omit'})`；
- `fetch`无法跟踪上传进度（`XHR`可以支持，上传进度使用xhr.upload.onprogress事件，下载进度使用xhr.onprogress事件.），下载进度可以hack实现，**并且没有很好原生支持超时timeout处理，需要通过AbortController和setTimeout配合Promise.race来进行hack**；
- **不兼容旧浏览器**

## `Axios` 

`Axios`是一种基于`Promise + Ajax`封装的`HTTP`客户端，其特点如下：

- 浏览器端发起`XMLHttpRequests`请求；
- （多端支持）可以在`node`端发起`http`请求；
- （基于promise封装）支持`Promise API`；
- 监听请求和返回；
- 支持拦截请求和相响应；
- 对请求和返回进行转化；
- 可以**取消请求**；
- **自动转换`json`数据**；
- 客户端**支持抵御`XSRF`攻击**；
- 可以**自动检测`data`并设置`Content-Type`**。

# 65.`ajax`检测上传下载的进度？

```javascript
xhr.upload.onprogress = upload => {
    const percent = upload.loaded / upload.total
}
xhr.onprogress = download => {
    const percent = download.loaded / download.total
}
```

# 66.`MVC`，`MVP`和`MVVM`的区别

[参考](https://juejin.cn/post/6844903480126078989#heading-3)

## `MVC`

> 所有通信都是单向的。`MVC`模式的特点在于**实现关注点分离**，即应用程序中的数据模型与业务和展示逻辑解耦。在客户端`WEB`开发中，就是将模型(`M`-数据、操作数据)、视图(`V`-显示数据的`HTML`元素)之间实现代码分离，松散耦合，使之成为一个更容易开发、维护和测试的客户端应用程序。

### 角色

- 视图（`View`）：用户界面。`View`作为视图层，主要负责数据的展示,并且响应用户操作；
- 控制器（`Controller`）：业务逻辑。控制器是模型和视图之间的纽带，接收`View`传来的用户事件并且传递给`Model`，同时利用从`Model`传来的最新模型控制更新`View`；
- 模型（`Model`）：数据保存。`Model`层用于封装和应用程序的业务逻辑相关的数据以及对数据的处理方法。一旦数据发生变化，模型将通知有关的视图。

<img src="https://github.com/NoAlligator/pico/blob/main/img/bg2015020105.png" alt="img" style="zoom: 33%;" />

<img src="https://github.com/NoAlligator/pico/blob/main/img/bg2015020106.png" alt="img" style="zoom:33%;" />

<img src="https://github.com/NoAlligator/pico/blob/main/img/bg2015020107.png" alt="img" style="zoom:33%;" />



### 数据关系

1. `View`接受用户交互请求；
2. `View`将请求转交给`Controller`；
3. `Controller`操作`Model`进行数据更新；
4. 数据更新之后，`Model`通知`View`更新数据变化。`PS：`还有一种是`View`作为`Observer`监听`Model`中的任意更新，一旦有更新事件发出，`View`会自动触发更新以展示最新的`Model`状态.这种方式提升了整体效率，简化了`Controller`的功能，不过也导致了`View`与`Model`之间的紧耦合；
5. `View`更新变化数据。

### 特点

所有方式都是单向通信。

### 缺点

- **`View`层过重:** `View`强依赖于`Model`的，并且可以直接访问`Model`。所以不可避免的`View`还要包括一些**业务逻辑**。导致`view`过重，后期修改比较困难，且复用程度低。
- **`View`层与`Controller`层也是耦合紧密:** `View`与`Controller`虽然看似是相互分离，但却是联系紧密。经常`View`和`Controller`一一对应的，捆绑起来作为一个组件使用。解耦程度不足。

### 实现方式

- `View`：使用组合(`Composite`)模式；
- `View`和`Controller`：使用策略(`Strategy`)模式；
- `Model`和`View`：使用观察者(`Observer`)模式同步信息。

### `MVC`优点

1. 耦合性低，视图层和业务层分离，这样就允许更改视图层代码而不用重新编译模型和控制器代码。
2. 重用性高。
3. 生命周期成本低
4. `MVC`使开发和维护用户接口的技术含量降低
5. 可维护性高，分离视图层和业务逻辑层也使得`WEB`应用更易于维护和修改
   部署快

### `MVC`缺点

1. 不适合小型，中等规模的应用程序，花费大量时间将`MVC`应用到规模并不是很大的应用程序通常会得不偿失。
2. 视图与控制器间过于紧密连接，视图与控制器是相互分离，但却是联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。
3. 视图对模型数据的低效率访问，依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能。

### `MVC`应用

在`web app`流行之初， `MVC` 就应用在了`java（struts2）`和`C#（ASP.NET）`服务端应用中，后来在客户端应用程序中，基于`MVC`模式，`AngularJS`应运而生。

## `MVP`

`MVP`（`Model-View-Presenter`）是`MVC`的改良模式，和`MVC`的相同之处在于：**`Controller/Presenter`负责业务逻辑**，`Model`管理数据，`View`负责显示只不过是将 `Controller` 改名为 `Presenter`，同时改变了通信方向。

<img src="https://github.com/NoAlligator/pico/blob/main/img/bg2015020109.png" alt="img" style="zoom:50%;" />

1. `M、V、P`之间双向通信。
2. `View`与 `Model` 不通信，都通过 `Presenter` 传递。`Presenter`完全把`Model`和`View`进行了分离，主要的程序逻辑在`Presenter`里实现。
3. View 非常薄，不部署任何业务逻辑，称为”被动视图”（`Passive View`），即没有任何主动性，而 `Presenter` 非常厚，所有逻辑都部署在那里。
4. `Presenter`与具体的`View`是没有直接关联的，而是通过定义好的接口进行交互，从而使得在变更`View`时候可以保持`Presenter`的不变，这样就可以重用。不仅如此，还可以编写测试用的`View`，模拟用户的各种操作，从而实现对`Presenter`的测试–从而不需要使用自动化的测试工具。

### 特点

各部分之间都是双向通信。

### 结构实现

- `View` ：使用 组合(`Composite`)模式；
- `View`和`Presenter`：使用 中介者(`Mediator`)模式，`presenter`是中介者；
- `Model`和`Presenter`：使用 命令(`Command`)模式同步信息。

### `MVP`优点

1. `Model`与`View`完全分离，修改互不影响；
2. 更高效地使用，因为所有的逻辑交互都发生在一个地方，即`Presenter`内部；
3. 一个`Preseter`可用于多个`View`，而不需要改变`Presenter`的逻辑（因为`View`的变化总是比`Model`的变化频繁）；
4. 更便于测试。把逻辑放在`Presenter`中，就可以**脱离用户接口来测试逻辑**（单元测试）。

### `MVP`的缺点

- `Presenter`中除了业务逻辑以外，还有大量的`View->Model`，`Model->View`的手动同步逻辑，造成`Presenter`比较笨重，一旦视图需要变更，那么`Presenter`也需要变更，维护起来比较困难。

## MVP和MVC的关系

- `MVP`是`MVC`模式的变种。
- 项目开发中，`UI`是容易变化的，且是多样的，一样的数据会有`N`种显示方式；业务逻辑也是比较容易变化的。为了使得应用具有较大的弹性，我们期望将`UI`、逻辑（`UI`的逻辑和业务逻辑）和数据隔离开来，而`MVP`是一个很好的选择。
- `Presenter`代替了`Controller`，它比`Controller`担当更多的任务，也更加复杂。`Presenter`处理事件，执行相应的逻辑，这些逻辑映射到`Model`操作`Model`。那些处理`UI`如何工作的代码基本上都位于`Presenter`。
- `MVC`中的`Model`和`View`使用`Observer`模式进行沟通；`MVP`中的`Presenter`和`View`则使用`Mediator`模式进行通信；`Presenter`操作`Model`则使用`Command`模式来进行。基本设计和`MVC`相同：`Model`存储数据，`View`对`Model`的表现，`Presenter`协调两者之间的通信。在`MVP`中`View`接收到事件，然后会将它们传递到`Presenter`, 如何具体处理这些事件，将由`Presenter`来完成。

## MVP和MVC的区别

- 在**`MVP`**中，`View`并不直接使用`Model`，它们之间的通信是通过`Presenter `(`MVC`中的`Controller`)来进行的，所有的交互都发生在`Presenter`内部。
- 在**`MVC`**中，`View`会直接从`Model`中读取数据而不是通过 `Controller`。

### `MVP`优点

1. 模型与视图完全分离，我们可以修改视图而不影响模型；
2. 可以更高效地使用模型，因为所有的交互都发生在一个地方——`Presenter`内部；
3. 我们可以将一个`Presenter`用于多个视图，而不需要改变`Presenter`的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁；
4. 如果我们把逻辑放在`Presenter`中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）。

### `MVP`缺点

视图和`Presenter`的交互会过于频繁，使得他们的联系过于紧密。也就是说，一旦视图变更了，`presenter`也要变更。

`MVP`应用：

可应用于`Android`开发。

## `MVVM`

该模式将`Presenter`改名为`ViewModel`，基本上与`MVP`模式完全一致。唯一的区别是，它采用双向绑定（`data-binding`）：`View`的变动，自动反映在`ViewModel`，反之亦然。在 `MVVM` 中，不需要`Presenter`手动地同步`View`和`Model`。`View`是通过数据驱动的，`Model`一旦改变就会相应的刷新对应的 `View`，`View` 如果改变，也会改变对应的`Model`。这种方式就可以在业务处理中只关心数据的流转，而无需直接和页面打交道。`ViewModel` 只关心数据和业务的处理，不关心 `View` 如何处理数据，在这种情况下，`View` 和 `Model` 都可以独立出来，任何一方改变了也不一定需要改变另一方，并且可以将一些可复用的逻辑放在一个 `ViewModel` 中，让多个 `View` 复用这个 `ViewModel`。

`Model` - `Model`层仅仅关注数据本身，不关心任何行为（格式化数据由`View`负责），这里可以把它理解为一个类似`json`的数据对象。

`View` - `MVVM`中的`View`通过使用模板语法来声明式的将数据渲染进`DOM`，当`ViewModel`对`Model`进行更新的时候，会通过数据绑定更新到`View`。

`ViewModel` - 类似于`Presenter`。`ViewModel`会对`View`层的声明进行处理。当 `ViewModel` 中数据变化，`View` 层会进行更新。如果是双向绑定,一旦`View`对绑定的数据进行操作，则`ViewModel`中的数据也会进行自动更新.

<img src="https://github.com/NoAlligator/pico/blob/main/img/bg2015020110.png" alt="img" style="zoom:50%;" />

![img](https://github.com/NoAlligator/pico/blob/main/img/169102df8742a9cf~tplv-t2oaga2asx-watermark.awebp?raw=true)

### 数据关系

- `View`接收用户交互请求；
- `View`将请求转交给`ViewModel`；
- `ViewModel`操作`Model`数据更新；
- `Model`更新完数据，通知`ViewModel`数据发生变化；
- `ViewModel`更新`View`数据。

### 实现方式

双向数据绑定

### 实现数据绑定的方式

- 数据劫持 (`Vue`)
- 发布-订阅模式 (`Knockout、Backbone`)
- 脏值检查 (旧版`Angular`)

### 使用

- 可以兼容你当下使用的 `MVC/MVP`框架。
- 增加你的应用的可测试性。
- 配合一个绑定机制效果最好。

### `MVVM`的优点

`MVVM`模式和`MVC`模式一样，主要目的是分离视图（`View`）和模型（`Model`），有几大优点:

- 低耦合。`View`可以独立于`Model`变化和修改，一个`ViewModel`（即业务逻辑）可以绑定到不同的`View`上，当`View`变化的时候`Model`可以不变，当`Model`变化的时候`View`也可以不变。
- 可重用性。你可以把一些视图逻辑放在一个`ViewModel`里面，让很多`view`重用这段视图逻辑。
- 独立开发。开发人员可以专注于**业务逻辑和数据**的开发（`ViewModel`），设计人员可以专注于页面设计，生成`HTML`代码。
- 可测试。界面素来是比较难于测试的，而现在测试可以**针对`ViewModel`来写。**

### `MVVM`缺点

- 类会增多，`ViewModel`会越加庞大，调用的复杂度增加。

### `React`与`MVVM`的关系

[参考链接](https://blog.yyisyou.tw/1dddc6d7/)

### `MVC`的实现

[参考链接](https://juejin.cn/post/6844903609075777549)

# 67.`JS`异步编程的方案有哪些？

## 回调函数

使用回调函数的方式有一个缺点是，多个回调函数嵌套的时候会造成回调函数地狱，上下两层的回调函数间的代码耦合度太高，不利于代码的可维护。

## `promise`

使用`Promise`的方式可以将**嵌套的回调函数作为链式调用**。但是使用这种方法，有时会造成过多`then`的链式调用，可能会造成代码的语义不够明确。

## `generator`

它可以在函数的执行过程中，将函数的执行权转移出去，在函数外部还可以将执行权转移回来。当遇到异步函数执行的时候，将函数执行权转移出去，当异步函数执行完毕时再将执行权给转移回来。因此在`generator`内部对于异步操作的方式，可以以同步的顺序来书写。使用这种方式需要考虑的问题是何时将函数的控制权转移回来，因此需要有一个自动执行`generator`的机制，比如说`co`模块等方式来实现`generator`的自动执行。

## `async`：`(generator + promise + co)`

`async`函数是`generator`和`promise`实现的一个自动执行的语法糖，它内部自带执行器，当函数内部执行到一个`await`语句的时候，如果语句返回一个`promise`对象，那么函数将会等待`promise`对象的状态变为`resolve`后再继续向下执行。因此可以将异步逻辑，转化为同步的顺序来书写，并且这个函数可以自动执行。

# 68.说一说`Promise`？

```javascript
Promise.all(iterable)			//只要其中的一个promise 失败，就返回那个失败的promise
Promise.allSettled(iterable)	//确保返回所有promise结果
Promise.any(iterable)			//只要其中的一个promise 成功，就返回那个成功的promise
Promise.race(iterable)			//只要其中的一个promise 成功/失败，就返回那个promise

Promise.then(fullfilled, rejected)
Promise.catch(rejected)
Promise.finally(callback)

Promise.resolve(value)
Promise.resolve(reason)
```

# 69.说一说`async`、`await`？

> `async`具有传染性，这是因为调用了`async`函数的方法必须声明一个`await`，而`await`又仅能在`async`函数中声明。`async`用于声明一个函数是异步函数，`await`关键字代表在等待一个`promise`对象的被接受或者拒绝，如果接受被接收，那么返回一个期约结果；**如果期约被拒绝，那么需要通过`try...catch`进行捕获，否则提前中止`async`函数的执行。如果针对非`promise`执行`await`本质上是直接通过`Promise.resolve()`包装。**

## 语法本质

`async/await`其实是`Generator` 的语法糖，他是`generator`生成器、`promise`期约、`co`自动化执行`generator`组成的。它能实现的效果都能用`then`链来实现，它是为优化`then`链而开发出来的。`async`是“异步”的简写，`await`则为异步等待。`async`用于申明一个`function`是异步的，而`await`用于等待一个异步`Promise`执行完成，语法上强制规定`await`只能出现在`asnyc`函数中。

## `async`函数返回值

`async`函数返回的是一个`Promise`对象，如果在函数中 `return` 一个直接量，`async`会把这个直接量通过 `Promise.resolve()` 封装成`Promise`对象。**在没有 `await` 修饰的情况下执行`async`函数，它会立即执行，返回一个`Promise`对象。**

## 处理返回值

`async`函数返回的是一个`Promise`对象，所以在最外层不能用`await`获取其返回值的情况下，应该用原来的方式：`then()` 链来处理这个`Promise`对象。

## `await`在等什么

1. `RHS`不是`Promise`：直接获取该值（运算返回值）并将其`Promise.resolve()`进行包装。
2. `RHS`是`Promise`：等待期约落定后的原`Promise`实例对象的resolve值或者reject抛出错误被外层try...catch捕获。

# 70.`async`、`await`对比`Promise`的优势、劣势？

## 优势

- `async`、`await`代码逻辑更符合同步代码，`Promise`虽然摆脱了回调地狱层层嵌套，但是`then`的链式调用还是会带来**<u>逻辑上的负担</u>**。`Promise`传值只能通过`then`一层层传递（可能会产生很多**<u>不必要的中间值的传递</u>**），**<u>需要定义多个回调，非常麻烦</u>**，而`async`、`await`是同步的写法，可以**<u>把所有的异步业务逻辑写在同一个`async`函数中， 非常优雅</u>**。
- 错误处理非常友好，`async`、`await`可以**<u>使用成熟的`try...catch`进行大范围的错误捕获</u>**。而`Promise`只能依赖`.catch()`**<u>捕获每一个处理阶段的错误或者在最后进行错误处理</u>**（逻辑上也不是很清晰，**<u>非常冗余</u>**）。
- **<u>`async`、`await`调试很友好，`Promise`的调试很差</u>**。如果你在⼀个`.then`代码块中使⽤调试器的步进`(step-over)`功能，调试器并不会进⼊后续的`.then`代码块，因为调试器只能跟踪**<u>同步代码的每⼀步</u>**。`async`、`await`调试时可以轻松**给每个异步调用加断点**

## 劣势

- 太过于**<u>同步的逻辑使得写出异步串行的代码</u>**，降低了代码的运行效率，因为`await`的特性是`promise`实例没有落定的话执行权不会回到`async`函数的，这就导致了函数的暂停执行。所以**<u>如果某些异步任务产生的值并非是执行下一个异步任务所依赖的值，使用`await`就会使得其他无关异步或同步代码被迫等待这个异步任务的完成。</u>**所以，不产生依赖关系的异步任务应当被并行执行（可以考虑使用`Promise.all()`）。（解决方案：发起异步任务和`await`进行分离，也就是先执行异步任务，在的确需要该异步任务返回值的位置对原`promise`进行`await`）。

  ```js
  async optimize() {
      const async_1 = fetchA()
      const async_2 = fetchA()
      const ret1 = await async_1
      const ret2 = await async_2
      return {
          ret1,
          ret2
      }
  }
  ```

# 71.`try...catch`的注意点？

## `try catch`捕获不到的错误

`try catch`捕获不到异步错误（例如`Promise`和`Ajax`请求），但是`async`函数中的`try...catch`可以捕获到异步抛出错误：

```js
// setTimeout中的错误
try {
  setTimeout(function () {
    throw new Error('error in setTimeout'); // 200ms后会把异常抛出到全局
  }, 200);
} catch (err) {
  console.error('catch error', err); // 不会执行
}

// Promise中的错误
try {
  Promise.resolve().then(() => {
    throw new Error('error in Promise.then');
  });
} catch (err) {
  console.error('catch error', err);
}
```

## `try/catch`中的块级作用域

`catch`块指定一个标识符（在上面的示例中为`e`），该标识符保存由`throw`语句指定的值。`catch`块是唯一的，因为当输入`catch`块时，JavaScript 会创建此标识符，并将其添加到当前作用域；标识符仅在`catch`块执行时存在；`catch`块执行完成后，标识符不再可用：

```javascript
try{
    undefined(); //制造一个异常
}
catch(err){
    console.log(err); //正常执行
}
console.log(err);  //ReferenceError:error not found
//err仅存在catch分句内部，当试图从外部引用时，就会抛出错误
```

```java
// message: 错误描述
// fileName: 可选。被创建的Error对象的fileName属性值。默认是调用Error构造器代码所在的文件的名字。
// lineNumber: 可选。被创建的Error对象的lineNumber属性值。默认是调用Error构造器代码所在的文件的行号。
const error = new Error([message [, fileName[, lineNumber]]])
error.message
error.name
Error.prototype.toString()
```

## `finally`

`try`、`catch`无法阻止`finally`块的运行，即使是存在`return`语句。而且，如果`finally`存在`return`语句，会导致`try`、`catch`块中的`return`语句被忽略。

## 错误类型

> 一共有**八种错误类型**，他们都是`Error`的子类型，可以通过`error.name`获取`error`类型，通过`error.message`获取错误描述。`error.message`包含的错误信息常常因为浏览器而异，通过`instanceof`检查错误类型是最通用的方法。

- `InternalError`：该类型的错误会在底层`JavaScript`引擎抛出异常时由浏览器抛出。例如：递归过多导致了栈溢出。这个类型并不是代码中通常要处理的错误，如果真发生了这种错误，很可能代码哪里弄错了或者有危险了。
- `EvalError`：类型的错误会在使用`eval()`函数发生异常时抛出。
- **`RangeError`：该错误会在数值越界时抛出。例如，定义数组时如果设置了并不支持的长度，如`-20`或`Number.MAX_VALUE`，就会报告该错误。（设置非法数组长度）**
- **`ReferenceError`：会在找不到对象时发生。这种错误经常是由访问不存在的变量而导致的。**
- `SyntaxError`：常在给`eval()`传入的字符串包含`JavaScript`语法错误时发生
- **`TypeError`：在`JavaScript`中很常见，主要发生在变量不是预期类型，或者访问不存在的方法时。很多原因可能导致这种错误，尤其是在使用类型特定的操作而变量类型不对时。（对非函数进行函数调用、访问`undefined`上的属性）**
- `URIError`：会在使用`encodeURI()`或`decodeURI()`但传入了格式错误的`URI`时发生。

## `window.onerror`事件

`window.onerror`事件：**在任何错误发生时，无论是否是浏览器生成的，都会触发`error`事件并执行这个事件处理程序。**

```javascript
window.onerror = (message/* 错误消息 */, url/* 发生错误的URL */, line/* 发生错误的行号 */) => {
    console.log(message);
    return false;
    //通过返回false，这个函数实际上就变成了整个文档的try/catch 语句，可以捕获所有未处理的运行时错误。这个事件处理程序应该是处理浏览器报告错误的最后一道防线。理想情况下，最好永远不要用到。
};
```

> 控制台输出调试
>

```javascript
console.log()
console.info()
console.error()
console.warn()
```

# 72.`async`、`await`如何捕获异常？

## 什么是“未捕获的拒绝期约”错误？何时产生？如何在全局捕获？

当`Promise`的状态变为`rejection`时，并且没有被`.then`、`.catch`处理，就会产生一个“未捕获的拒绝期约”错误，它会一直冒泡，直至被进程捕获。这样的错误就被称为`unhandled promise rejection`。可以在`window`对象上设置全局监听`'unhandledrejection'`捕获，但是最好的方式是设置`catch`回调进行合适的异常处理。

```javascript
window.addEventListener('unhandledrejection', event => {/*执行相应的的错误处理*/});
```

## `await`异常捕获机制

`await`可以将一个被拒绝的`promise`转换为**可捕获的错误**（未捕获的话就是`Unhandled Promise Rejection Warning`），通过`try...catch`可以直接捕获这个错误。

## 捕获方式和注意事项

> 注意：`promise`的错误不能被同步捕获是**重点**！

![image-20220227231921487](https://github.com/NoAlligator/pico/blob/main/img/image-20220227231921487.png?raw=true)

- 异步可捕获情形
  - 对于`await`：如果`await`期待的`promise`落定为`reject(reason)`，那么会导致落定这个`promise`为`reject`的`resaon`作为`catch(e)`中的`e`被`try...catch`捕获到；
- 异步未捕获的的情形
  - 如果不设置`try...catch`语句，那么会`await`期待的`promise`落定拒绝之后异常会被抛出到全局（`Unhandled Promise Rejection Warning`）。
  - `try...catch`无法捕获诸如`Promise.reject(new Error())`这样的错误，`Promise`的异常即使是通过同步地创建拒绝期约来往外抛出的，也必须移交异步异常处理，也就是`catch()`。
  - 对于未捕获的错误，我们最好使用自定义的期约拒绝捕获处理程序，即`window.addEventListener('unhandledrejection', event => {/*执行相应的的错误处理*/});`，这是未捕获错误最后的防线。

## 在`async`函数中捕获错误的几种方式

[参考链接](https://segmentfault.com/a/1190000019854513?utm_source=sf-similar-article)

### 混合`await`、`then()`

```javascript
async function run() {
    const err = await throwErrorPromise().then(() => null, reason => reason);
    if(err !== null){
        console.log(`error catched: ${err}`)
    }
}
async function run() {
    const [value, err] = await throwErrorPromise().then(value => [value, null], reason => [null, reason]);
    if(err !== null){
        console.log(`error catched: ${err}`)
    } else {
        console.log(value)
    }
}
```

### 直接使用`try...catch + await`

```javascript
async function run() {
    try{
        let ret = await Promise.reject('reason')
    } catch(err) {
        console.log(`error catched: ${err}`)
    }
}
```

### `async`函数调用后添加`catch`进行错误捕获的收尾工作，防止被全局捕获。

```javascript
async function run() {
    let ret = await Promise.reject('reason')	//这里的错误会被抛出，因为没有catch接收
}
run().catch(err => console.error(err))          //处理抛出错误
```

#### 全局捕获

```javascript
window.addEventListener("unhandledrejection", event => {
    event.preventDefault()//取消默认的报错信息
    console.warn(`UNHANDLED PROMISE REJECTION: ${event.reason}`);//自定义报错提醒
});
```

# 73.并发和并行的区别？

- 并发是**<u>宏观概念</u>**，我分别有任务`A`和任务`B`，**<u>在一段时间内通过任务间的切换完成了这两个任务，这种情况就可以称之为并发</u>**。
- 并行是**<u>微观概念</u>**，假设`CPU`中存在两个核心，那么我就可以同时完成任务 `A、B`。**<u>同时完成多个任务的情况就可以称之为并行</u>**。

# 74.`setTimeout`、`setInterval`、`requestAnimationFrame`各有什么特点？

## 介绍

`setTimeout` 允许我们<u>将函数推迟到一段时间间隔之后再执行</u>。使用`clearTimeout(timerId)`来取消调度。

`setInterval` 允许我们重复运行一个函数，从一段时间间隔之后开始运行，之后<u>以该时间间隔连续重复运行该函数（在一定条件下）</u>。使用`clearInterval(timerId)`来取消调度。

## 语法

```javascript
let timerId1 = setTimeout(func|code, [delay], [arg1], [arg2], ...)
let timerId2 = setInterval(func|code, [delay], [arg1], [arg2], ...)
```

## 执行周期任务时的区别

每个`setTimeout`产生的任务会直接推入到任务队列中等待上一轮宏任务完成后立即执行。（能够确保执行间隔一定）

![image-20220227235643462](https://github.com/NoAlligator/pico/blob/main/img/image-20220227235643462.png?raw=true)

使用`setInterval`会在固定的时间间隔内将回调函数放入任务队列中，但是如果当前队列中由该计时器产生的回调函数排队较多的情况下可能会跳过部分回调的入列。（能够确保入列时间一定）

![image-20220227235719421](https://github.com/NoAlligator/pico/blob/main/img/202203271746577.png?raw=true)

当函数执行花费时间都小于间隔时间的时候，`setInterval`可以确保函数的入列时间保持一定；而当函数执行花费时间都大于间隔时间的时候，`setInterval`无法确保每一个回调都能被推入队列执行。

![理解setTimeout和setInterval](https://github.com/NoAlligator/pico/blob/main/img/setTimeout-3.jpg?raw=true)



## 使用`setTimeout`执行周期性任务的好处

> 使用`setTimeout`模拟调度实现的是函数在每隔一段固定时间后被调用一次，使用`setInterval`模拟调度实现的是函数在每隔一段固定时间后被入列一次，这在本质上是不同的，尤其是当回调函数执行耗时很多的情况下。

- 嵌套的 `setTimeout` 要比 `setInterval` 灵活得多。采用这种方式**<u>可以根据当前执行结果来调度下一次调用，因此下一次调用可以与当前这一次不同。</u>**例如，我们要实现一个服务（`server`），每间隔`5`秒向服务器发送一个数据请求，但如果服务器过载了，那么就要降低请求频率，比如将间隔增加到 `10`、`20`、`40` 秒等。并且，如果我们调度的函数占用大量的`CPU`，那么我们可以测量执行所需要花费的时间，并安排下次调用是应该提前还是推迟。**<u>嵌套的 `setTimeout` 能够精确地设置两次执行之间的延时，而 `setInterval` 却不能。</u>**

## 使用`setInterval`的缺点

- 使用`setInterval`时，**<u>某些间隔可能会被跳过</u>**
- **<u>多个定时器的代码执行之间的间隔可能比预期的小</u>**
- **<u>定时器内部回调运行时间过长的最终结果就是执行累计（无间隔运行）</u>**

```javascript
(function demo() {
    let before = +new Date()
    let i = 1
    setInterval(function() {
        console.log(i++)
        let now = +new Date()
        console.log(`start: ${now - before}`)
        while(+new Date() - now < 4000){}
        let endnow = +new Date()
        console.log(`end: ${endnow - now}`)
        before = endnow
    }, 5)
})()
```

## `setTimeout`回调一定会在到期事件触发之后准时运行吗？

如果主线程并不空闲，推入队列的事件也不会被执行，而是等待主线程运行结束后执行队列任务。所以`setTimeout`并不是一定会在到期之后准时执行的。

## 使用`setTimeout`模拟`setInterval`

```javascript
function _setTimeout(cb, delay, ...args) {
    const that = this
    let timerId
    timerId = setTimeout(function self(...args) {
        cb.call(that, ...args)
        clearTimeout(timerId)
        timerId = setTimeout(self, delay, ...args)
    }, delay) 
    return () => clearTimeout(timerId)
}
```

## `requestAnimationFrame`的使用

[参考](https://segmentfault.com/a/1190000013286862)

如果有**循环定时器**的需求，可以通过 `requestAnimationFrame` 来实现，`requestAnimationFrame` 自带函数节流功能，基本可以保证在`16.6ms`内只执行一次（不掉帧的情况下），并且该函数的延时效果是精确的，没有其他定时器时间不准的问题。

```javascript
function setInterval(callback, interval, ...args) {
    let timer
    let startTime = Date.now()
    let endTime = startTime
    const loop = () => {
        timer = window.requestAnimationFrame(loop)	//开启下一轮调度
        endTime = now()
        if (endTime - startTime >= interval) {
            startTime = endTime = Date.now()
            callback(...args)
        }
    }
    timer = window.requestAnimationFrame(loop)
    return timer
}
```

# 75.说一说`JS`的垃圾回收`GC`？

## 概念

### 	垃圾回收

`JavaScript`代码运行时，需要分配内存空间来储存变量和值。当变量不在参与运行时，就需要系统收回被占用的内存空间，这就是垃圾回收。

### 	回收机制

- `Javascript `具有自动垃圾回收机制，会定期对那些**不再使用的变量、对象**所占用的内存进行释放，原理就是找到不再使用的变量，然后释放掉其占用的内存。
- `JavaScript`中存在两种变量：局部变量和全局变量。全局变量的生命周期会持续到页面卸载；而局部变量声明在函数中，它的生命周期从函数执行开始，直到函数执行结束，在这个过程中，局部变量会在堆或栈中存储它们的值，当函数执行结束后，这些局部变量不再被使用，它们所占有的空间就会被释放（随着执行上下文出栈销毁而被释放）。
- 不过，当局部变量被外部函数使用时，其中一种情况就是**闭包**，在函数执行结束后，函数外部的变量依然指向函数内部的局部变量，此时局部变量依然在被使用，所以不会回收（进入堆内存）。

## 垃圾回收的方式

### 标记清除

目前在 `JavaScript引擎` 里这种算法是最常用的，到目前为止的大多数浏览器的 `JavaScript引擎` 都在采用标记清除算法，只是各大浏览器厂商还对此算法进行了优化加工，且不同浏览器的 `JavaScript引擎` 在运行垃圾回收的频率上有所差异。

### 引用计数

引用计数（`Reference Counting`），这其实是早先的一种垃圾回收算法，它把 `对象是否不再需要` 简化定义为 `对象有没有其他对象引用到它`，如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收，目前很少使用这种算法了，因为它的问题很多（例如循环引用问题会无法被回收）。

## 标记清除

### 原理

`标记清除`分为`标记`和`清除`两个阶段，在标记阶段会从`GC`根对象出发遍历它的所有可达引用并进行标记，在清除阶段中，再次从`GC`根对象出发遍历所有可达引用，并移除标记，如果完成之后仍然携带有标记的变量就会被当作垃圾回收。`GC`根是一组特殊的对象，垃圾收集器将其用作确定哪些对象符合垃圾收集条件的起点。“根”只是一个对象，垃圾回收器假定它在默认情况下是可访问的，然后跟踪它的引用，以便找到所有其他当前可访问的对象。不能通过任何根对象的任何引用链到达的任何对象都被认为是不可到达的，并且最终将被垃圾回收器销毁。在`V8`中，根由当前调用堆栈中的对象(即当前执行函数的局部变量和参数)、活动的`V8`句柄作用域、全局句柄和编译缓存中的对象组成。

### 缺点

- 内存碎片化：空闲内存块是不连续的，容易出现很多空闲内存块，还可能会出现分配所需内存过大的对象时找不到合适的块；
- 分配速度慢：因为即便是使用 `First-fit` 策略，其操作仍是一个 `O(n)` 的操作，最坏情况是每次都要遍历到最后，同时因为碎片化，大对象的分配效率会更慢。

## 引用计数

- 另外一种垃圾回收机制就是引用计数，这个用的相对较少。引用计数就是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型赋值给该变量时，则这个值的引用次数就是`1`。相反，如果包含对这个值引用的变量又取得了另外一个值，则这个值的引用次数就减`1`。当这个引用次数变为`0`时，说明这个变量已经没有价值。因此，在在机回收期下次再运行时，这个变量所占有的内存空间就会被释放出来。

- 这种方法会引起循环引用的问题：例如：` obj1`和`obj2`通过属性进行相互引用，两个对象的引用次数都是`2`。当使用循环计数时，由于函数执行完后，两个对象都离开作用域，函数执行结束，`obj1`和`obj2`还将会继续存在，因此它们的引用次数永远不会是`0`，就会引起循环引用。

  ```javascript
  function fun() {
      let obj1 = {};
      let obj2 = {};
      obj1.a = obj2; // obj1 引用 obj2
      obj2.a = obj1; // obj2 引用 obj1
  }
  ```

  这种情况下，就要手动释放变量占用的内存来触发`GC`

  ```javascript
  obj1.a =  null
  obj2.a =  null
  ```

## V8对GC的优化

[参考](https://juejin.cn/post/6995706341041897486#heading-12)

V8垃圾回收策略主要基于分代式垃圾回收机制。现代的垃圾回收算法中按对象的存活时间将内存的垃圾回收进行不同的分代，然后分别对不同分代的内存施以更高效的算法。在V8中，主要将内存分为新生代和老生代，**新生代内存存储的为存活时间较短的对象**，**老生代内存存储的为存活时间较长或常驻内存的对象**，如下图：

![img](https://github.com/NoAlligator/pico/blob/main/img/16904df8de47c353~tplv-t2oaga2asx-watermark.awebp?raw=true)

V8堆的整体大小就是新生代所用内存空间加上老生代的内存空间。

### V8新生代算法（Scavenge）

### V8老生代算法（Mark-Sweep && Mark-Compact）

### 增量标记（标记阶段切片）

### 惰性清理（清除阶段切片）

### 并发和并行

