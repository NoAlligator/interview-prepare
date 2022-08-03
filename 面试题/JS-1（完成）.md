# 1. JavaScript有哪些数据类型，它们的区别？

> `JavaScript`共有**八种数据类型**：
>
> `Undefined、Null、Boolean、Number、String、Object、Symbol(ES6)、BigInt(ES6)`

### ES6+新类型

`Symbol` 代表创建后**独一无二**且不可变的数据类型，它主要是为了解决可能出现的**全局变量冲突**的问题。

`BigInt` 是一种**数字类型**的数据，它可以表示**任意精度格式**的整数，使用 `BigInt` 可以安全地存储和操作**大整数**，即使这个数已经超出了`Number`能够表示的**安全整数**范围。

### 类型分类

类型可以分为**原始数据类型**和**引用数据类型**：

- 栈：原始数据类型（`Undefined、Null、Boolean、Number、String`）
- 堆：引用数据类型（`Object`）

### 存储方式

类型的区别在于**存储位置的不同：**

- 原始数据类型直接存储在**栈**（`stack`）中的简单数据段，**占据空间小**、**大小固定**，属于被**频繁使用**数据，所以放入栈中存储；
- 引用数据类型存储在**堆**（`heap`）中的对象，**占据空间大、大小不固定**。如果存储在栈中，将会影响程序运行的性能；引用数据类型**在栈中存储了指针**，该指针**指向堆中该实体的起始地址**。当解释器寻找引用值时，会首先读取其**在栈中的地址**，取得地址后**从堆中获得实体**。

# 2.数据类型检测的方式有哪些？

> 检测数据类型的几种方式：`typeof`、`instanceof`、`constructor`、`Object.prototype.toString.call()`
>

## `typeof`

> `typeof`是一种很常用的判断数据类型的机制，但是在使用的时候需要注意几种特殊情况：`typeof null === 'object'`、`typeof function(){} === 'function' `，除了函数类型，任何引用类型以及`null`返回的都是`object`。但是对于未定义的变量，使用`typeof`返回的也是`undefined`，这是需要注意的。

```javascript
typeof NaN						//number
typeof null 					//object
typeof undefined 				//undefined
typeof 1						//number
typeof ''						//string
typeof true						//boolean
typeof {}						//object
typeof []						//object
typeof Symbol					//function
typeof Symbol(1)				//symbol
typeof BigInt					//function
typeof BigInt(1)				//bigint
typeof function f(){}			//function
typeof new Promise(() => {})    //object
typeof new Map()				//object
//判断数组
Array.isArray(arrayOrNot)
```

## `instanceof`

> `a instanceof b`可以正确判断对象和构造函数之间的关系，**其内部运行机制是判断在实例对象`a`的原型链中能否找到构造函数`b`的原型**。所以，`instanceof`**只能正确判断引用数据类型**，而不能判断**基本数据类型**，基本数据类型需要经过**包装**才能判断。

```javascript
[] instanceof Array 	//true
[] instanceof Object	//true
{} instanceof Object	//true
```

## `constructor`

> 使用`constructor`来进行判断也是仅针对引用类型。原理是访问引用类型原型上的`constructor`字段，他指向了构造函数。使用`constructor`有两个作用，一是判断**数据的类型（但是如果修改了构造函数的原型导致的`constructor`丢失会让这种类型判断失效）**，二是对象实例通过 `constrcutor` 对象可以直接访问它的**构造函数**。
>

## `Object.prototype.toString.call()`

> 使用这种方法是一种最细致的方法，可以准确地判断出所有地基本类型、引用类型的构造函数名，但是必须注意调用方式必须是`Object.prototype.toString.call()`，也就是盗用`Object`上的`toString()`原型方法，其他类型都重写了`toString()`方法，返回的结果不一致。

```javascript
const stringfy = Object.prototype.toString
console.log(stringfy.call(null));						// [object Null]
console.log(stringfy.call(undefined));					// [object Undefined]
console.log(stringfy.call(2));							// [object Number]
console.log(stringfy.call(true));						// [object Boolean]
console.log(stringfy.call('str'));						// [object String]
console.log(stringfy.call(Symbol('description')));		// [object Symbol]
console.log(stringfy.call(BigInt(1)));					// [object BigInt]
console.log(stringfy.call([]));							// [object Array]
console.log(stringfy.call({}));							// [object Object]
console.log(stringfy.call(function(){}));				// [object Function]
console.log(function* () {});							// [object GeneratorFunction]
console.log(new Map());									// [object Map]
console.log(Promise.resolve());							// [object Promise]

//获取
stringfy.call(null).split(' ')[1].slice(0, -1).toLowerCase()

//Symbol.toStringTag实现扩展，默认自己创建的对象都是[object Object]
class Diy{
    get [Symbol.toStringTag]() {
        return "Diy"
    }
}
stringfy.call(new Diy())
```

# 3.判断数组的方式有哪些

```javascript
//api
Array.isArray(obj)
//粗暴法
Object.prototype.toString.call(obj).split(' ')[1].slice(0, -1) === 'Array'
//原型链法
Object.getPrototypeOf(obj) === Array.prototype
Array.prototype.isPrototypeOf(obj)
obj instanceof Array
```

# 4.`null`和`undefined`的区别

> `Undefined` 和 `Null` 都是基本数据类型，这两个基本数据类型分别都只有一个值，就是 `undefined` 和 `null`。

## 共同点

都是`falsy`值。

## 不同点

`null`表示"没有对象"，即该处在定义的时候不应该有值。设置`null` 表示一个值**显式地被定义定义为“空值”**，设置一个值为 `null` 是合理的。典型用法是（`null`是语义上的空值，是一种**主动设置**的空值）：

1. 作为函数的参数，表示该函数的参数**不是对象**（比如，某个函数接受引擎抛出的错误作为参数，如果运行过程中未出错，那么这个参数就会传入`null`，表示未发生错误）；
2. 作为对象原型链的终点。

`undefined`表示"缺少值"，就是**此处应该有一个值，但是还没有定义**。`undefined` 表示**根本不存在定义，所以设置一个值为 `undefined` 是不合理的**（`undefined`是实际上的空值，他是一种默认行为），典型用法是：

1. 变量被声明了，但没有赋值时，就等于`undefined`；
2. 调用函数时，应该提供的参数没有提供，该参数等于`undefined`；
3. 对象没有赋值的属性，该属性的值为`undefined`；
4. 函数没有返回值时，默认返回`undefined`；
5. 以上情况可见`undefined`在`JS`中属于一种默认行为。

```javascript
null == undefined  //true
null === undefined  //false

//检查null
nullOrNot === null //最好

//转换
+null 						//0
null+1						//1
1-null						//1
!null 						//false
if(null) console.log('any') //无输出

```

# 5.`void`关键字

> `void` 是一个一元运算符，它可以出现在任意类型的操作数之前执行操作数，却忽略操作数的返回值，返回一个 `undefined`。`void `在表达式的左边，`void `右边的表达式可以是带括号形式（例如：`void(0)`），也可以是不带括号的形式（例如：`void 0`）。

## 注意点：

**`void`**优先级比较高，**高于加减乘除**，涉及运算应当优先使用括号包裹，防止运算顺序错误。

## 推荐使用`void 0`生成`undefined`的理由

**因为`undefined`不是`JS`保留关键字，可以作为变量存在**，所以加入`undefined`被作为变量定义之后会导致`undefined`成为一个变量而不是存粹的基本类型`undefined`。`void`能保证取到 `undefined` 值，所以是安全的。

## 应用场景

```javascript
// 组织<a>标签的默认事件
<a href="javascript:void(0);">
```

```javascript
// 利用void运算符让js引擎把一个function关键字识别成函数表达式而不是函数声明
void function say(){
   console.log(1) 
}()
```

```javascript
// 在箭头函数（普通函数）中避免返回值泄漏
const safeArrayFunction = (cb) => void cb()
```

# 6.有哪些是`falsy`值？

> `falsy`值指的是在进行`Boolean`类型的隐式转换中会转换成`false`的值，利用排除法，剩余的值进行隐式转换后都是`true`

```js
let falsy = [false, 0, -0, 0n, '', null, undefined, NaN]
falsy.forEach((value) => console.log(Boolean(value)))
```

# 7.常见的显式转换和隐式转换？ 

## [数据的强制和隐式转换](https://segmentfault.com/a/1190000021106485)

## 显式转换

### `toString()`：

`JS`中的内置类基本上都重写了`Object`上的`toString()`方法，表现出不同的返回值（`null`和`undefined`除外，他们不存在对应的内置类）。而`Object`类型原型上的`toString()`方法在被盗用时针对不同类型会有较为一致的结果（即`[object Xxx]`），并且基本数据类型的`null`和`undefined`也支持。

### `String()`： 

- 第一步：先调用对象自身的 `toString()` 方法。如果返回**原始类型**的值，则对该值使用 `String()` 函数，不再进行以下步骤；
- 第二步：如果 `toString()` 方法返回的是**对象**，再调用**原对象**的 `valueOf()` 方法。如果 `valueOf()` 方法返回原始类型的值，则对该值使用 `String()` 函数，不再进行以下步骤；
- 第三步：如果 `valueOf()` 方法返回的是对象，就报错；
- 本质：对基本类型先进行包装，先后调用对应类型上的`toString()`、`valueOf()`，遇到原始类型就进行包装后再使用原始数据类型内置类上的`toString()`进行转换并返回，如果两次都是返回对象，报错。

### `Number()`：

- 第一步：调用对象自身的 `valueOf()` 方法。如果返回原始类型的值，则直接对该值使用 `Number()` 函数，不再进行后续步骤。
- 第二步：如果 `valueOf()` 方法返回的还是对象，则改为调用对象自身的 `toString()` 方法。如果返回原始类型的值，则直接对该值使用 `Number()` 函数，不再进行后续步骤。
- 第三步：如果 `toString()` 方法返回的还是对象，就报错。
- 总结：
  - 纯数字字符串（科学计数、`±Infinity`等字符串数值）转成数字；
  - `""`、`false`、`null`转成`0`，`true`转成`1`；
  - 其他按照上述规则，其余都是`NaN`。

### `Boolean()`：

- `falsy`值都会转成`false`，其余全部都是`true`

### [`parseInt(string[, radix])`：](https://juejin.cn/post/7049161354703273998)

- 如果参数不是**字符串类型**，则使用 `String()`将其转换为字符串；
- `parsetInt`利用匹配机制会尽可能将前段部分转换为数字，自动忽略后续不合法字符；
- 字符串开头和结尾的空白符将会被忽略，字符串开头的 `0` 也会被忽略；
- 如果 `string` 的第一个字符不能被转换成数字，则 `parseInt()` 返回 `NaN`；
- 当`radix`为`undefined`和`0`的时候作为`10`进制处理。

### `parseFloat(string)`：

- 如果参数不是字符串类型，则使用 `String()`将其转换为字符串，开头和结尾的空白符会被忽略，小数点前多余的 `0` 也会被忽略；
- `parsetFloat`利用匹配机制会尽可能将前段部分转换为数字，自动忽略后续不合法字符；
- 第二个小数点的出现也会使解析停止；
- 如果参数字符串的第一个字符不能被解析成为数字，则返回 `NaN`；
- `parseFloat` 也可以解析并返回 `Infinity`。

### `Object()`：

- 当参数为原始类型时，会转换为对应的**包装对象的实例**；
- 参数为空或者 `undefined` 或者 `null` 时，返回一个空对象，和调用`Object()`无区别；
- 参数为引用数据类型时，**总是返回其本身，不做转换**。

## 隐式转换

- `a + b`
  - `String` + `Number`：`String(Number)`
  - `Other ` + `Number`：`Number(Other)`
- `isNaN(params)`（注意区别`Number.isNaN()`）
  - 先调用`Number(params)`，再判断是否是`NaN`
- `a++`
  - 先`Number(a)`，再自增
- `+a、-a`
  - 先`Number(a)`，再`+a`
- `-、*、/、%、**`等运算
  - 先`Number()`，再运算
- `if(xxx)`
  - 隐式转换`Boolean(xxx)`，`falsy`值会自动转换成`false`
- `isNaN（注意区别Number.isNaN）`
  - 先`Number()`，再运算
- `==`（`==`叫做**宽松相等**，涉及隐式转换，转换规则规则复杂、`===`叫做**严格相等**，判断类型不一致之后不会进一步进行转换，直接返回`false`）
  - 优先规则：**`null`类型**只与自身和**`undefined`**宽松相等，和**自身**相等，与**其他所有值**都不宽松相等。
  - **字符串**类型与**数字类型**相比较时，`字符串类型`会被转换为`数字类型`(`Number()`)。纯数字字符串（科学计数、`Infinity`）转成数字；字符串空，false的时候转成0；其他都是NaN。
  - **布尔类型**与**其他类型**相比较时，该类型就会率先转成`数字类型`。
  - **对象类型**与**其他类型**比较时，如果是`String`类型，就调用`String(object)`转换成字符串类型进行比较；如果是数字类型，就调用`Number`转换成数字类型进行比较；如果是`Symbol`类型会调用`toString`和`valueOf`方法取返回值进行比较，基本原理和`Number()`和`String()`相似。
  - 最后注意两点：**NaN类型**和谁都不宽松/严格相等**（包括自己），**对象类型与对象类型相比较**，如果两个变量指向**同一个对象，相等操作符返回 `true`，否则为`false`。

```javascript
//String <-> Number
"1" == 1					//true
"" == 0						//true
"1.1e+21" == 1.1e+21		//true  ⭐
"Infinity" == Infinity		//true  ⭐
NaN == NaN 					//false ⭐

//Boolean <-> Others
true == 1  		// true
false == 0  	// true
true == 2  		// false
"" == false 	// true
"1" == true 	// true

// Null <-> Others
null == undefined 	//true
null == ''		//false
null == 0		//false
null == 1		//false
null == true	//false
null == false	//false
null == []		//false
null == {}		//false

// 	Undefined <-> Others
undefined == ''		//false
undefined == 0		//false
undefined == 1		//false
undefined == true	//false
undefined == false	//false
undefined == []		//false
undefined == {}		//false

//  Object <-> Others
{} == '[object object]'  // true
[1,2,3] == '1,2,3' // true

//实现a==1&&a==2&&a==3
let a = {
    init: 1,
    [Symbol.toPrimitive]: () => a.init++ //Symbol.toPrimitive 是一个内置的 Symbol 值，它是作为对象的函数值属性存在的，当一个对象转换为对应的原始值时，会调用此函数。
}
a == 1 && a == 2 && a == 3
```

8.运算优先级

- `()`

- `x++, x--`

- `!,~,+x,-x,--x,++x,typeof,void,delete,await`

- `**`

- `*,/,%`

- `+,-`

- `<<,>>,>>>`

- `<,>,<=,>=,in,instanceof`

- `==,!=,===,!==`

- `&`

- `^`

- `|`

- `&&`

- `||`

- `??`

- `... ? ... : ...`

- `...=..., ...*=....,other`

- `yield`

  

- `,`

```js
//逗号操作符  对它的每个操作数求值（从左到右），并返回最后一个操作数的值。
let a = (1 + 2, 2 + 3, 3 + 4) //7
```

# 9.写一个完整的类型判断

> 基于`Object.prototype.toString.call`实现

```javascript
const judge = (tar) => {
    if(typeof tar === 'object'){
        if(tar === null) return 'null'
        if(Array.isArray(tar)) return 'array'
        return Object.prototype.toString.call(tar).split(' ')[1].slice(0, -1).toLowerCase()
    }
    return typeof tar
}
```

# 10.手写`instanceof`

```javascript
const myInstanceof = (obj, constructor) => {
    const proto = constructor.prototype
    let prototype = Object.getPrototypeOf(obj)
    while(prototype !== null){
        if(proto === prototype) return true
        prototype = Object.getPrototypeOf(prototype)
    }
    return false
}
```

# 11.为什么`0.1 + 0.2 !== 0.3`？如何让其相等？



## JS是如何存储数字的？

### 前置知识

> 十进制转二进制的方法是，整数部分**除二取余倒着写**，小数部分是**乘二取整顺着排**（其中小数的转换难免会遇到无限循环小数）。计算机是通过**二进制**的方式存储数据的，所以计算机计算**`0.1+0.2`**的时候，实际上是计算的两个数的**二进制的和**。

双精度实数的`64`位比特（64比特，8字节）分为**三个部分**：

- 符号位`S`：第 `1` 位是正负数**符号位**（`sign`），`0`代表正数，`1`代表负数
- 指数位`E`：中间的 `11` 位存储**指数**（`exponent`），用来表示次方数，`E`是一个无符号整数，因为长度是`11`位，取值范围是 `0~2047`。但是科学计数法中的指数是可以为负数的，所以再减去一个中间数 `1023（双精度数的偏移量）`，`[0,1022]`表示为负，`[1024,2047]` 表示为正。
- 尾数位M：最后的 `52` 位是**尾数**（`mantissa`），超出的部分**自动进一舍零⭐**（遵从“`0舍1入`”的原则）

![64 bit allocation](https://github.com/NoAlligator/pico/blob/main/img/687474703a2f2f617461322d696d672e636e2d68616e677a686f752e696d672d7075622e616c6979756e2d696e632e636f6d2f37323637613538623239383932633362373233653364366333663733393035612e706e67?raw=true)



双精度浮点数转十进制**计算公式**：

![31601584-f65ed43e-b21f-11e7-8755-c99b48e5134c](https://github.com/NoAlligator/pico/blob/main/img/31601584-f65ed43e-b21f-11e7-8755-c99b48e5134c.png?raw=true)

十进制反推双精度浮点数计算方式：

1. `0.1`转换为二进制是`0.000110011···`，科学计数法表示为`1.1001100··· * 2^(-4)`；
2. 由上述可知`E`部分的数值减去`1023`才是最终指数，故进行转换：`E - 1023 = -4` ➡️ `E = 1019 = 01111111011`；
3. 由上述可知`M`部分会被自动加上`1`（作为整数部分），所以应当去掉`1.1001100`的1才是原本的M：`1001100···`（总计`52`位）；
4. `S`易知是`0`，因为是正数。

## 为什么`0.1 + 0.2 !== 0.3 `?

答案：一般我们认为数字包括**整数**和**小数**，但是在 `JavaScript` 中**只有一种数字类型**：**`Number`**，它是他是标准的**双精度浮点数（`double`）**，使用**`64`位固定长度**来表示。在计算机是通过**二进制**的方式存储数据的，所以计算`0.1 + 0.2`实际上是在计算两个数的二进制的值。在`0.1`和`0.2`转换成二进制的时候小数部分都是**无限循环数**，而双精度实数的尾数最大只有`52`位，超出的部分**直接舍入**，这就导致了舍入误差。最终相加的就是两个有舍入误差的二进制数，转换为十进制之后就不等于`0.3`了。

补充：所以很多小数**考虑精度**的话其实严格意义上并不和我们**字面量定义的小数本身**是相等的。我们可以通过`Number.toPrecision()`去查看它的后几位会发现并不是全部是`0`。

## 如何解决浮点误差？

因为双精度实数的限制，理论上数字的**精度是有限的**，一个直接的解决方法就是**设置一个极小的误差范围上限**，通常称为“机器精度”。对`JS`来说，这个值通常为`2^(-53)`，在`ES6`中，提供了`Number.EPSILON`属性，而它的值就是`2^(-53)`，只要判断`0.1 + 0.2 - 0.3`是否小于`Number.EPSILON`就可以判断是否相等。

```javascript
const equalWithPrecision = (arg1, arg2) => {
    return Math.abs(arg1 - arg2) < Number.EPSILON
}
console.log(numberepsilon(0.1 + 0.2, 0.3)); // true
```

`Number.toPrecision`：`precision`是从左至右第一个不为`0`的数开始数起，四舍五入到 `precision` 参数指定的显示数字位数。我们也可以使用`Number.toPrecision()`手动选择精度，一般选择`12`就可以解决大部分精度问题，最多选择`16`（因为精度最大就是长`15`位，根据`2^53`）。

```javascript
function strip(num, precision = 12) {
    return +parseFloat(num.toPrecision(precision));
}
```

对于运算类操作，如 `+-*/`，就不能使用 `toPrecision` 了，正确的做法是把**小数转成整数**后再运算。

```javascript
const safeAdd = (figure1, figure2) => {
    const decimal1 = (figure1.toString().split('.')[1] || '').length
    const decimal2 = (figure2.toString().split('.')[1] || '').length
    const shift = Math.pow(10, Math.max(decimal1, decimal2))
    return (figure1 * shift + figure2 * shift) / shift  
}
```

## JS数字的精度是？最大安全数是多少？最大数是多少？

精度是`±2^(53)`，是由双精度浮点数的尾数部分决定的，**转换为十进制数字长度大约是`15`位**。故最大安全数为`2^(52 + 1)-1`。最大数字是`2 ^ 1024 - 1`，实际上并不能计算出这个数据（因为要先算`2 ^ 1024`，他就等于`Infinity`）。

```javascript
2 ** 53 - 1 == Number.MAX_SAFE_INTEGER //true
```

# 12.`isNaN` 和 `Number.isNaN` 函数的区别？

```javascript
Number.isNaN === isNaN //false
```

- 函数 `isNaN`接收参数后，会尝试将这个参数**转换为数值**，任何**不能被转换为数值的的值都会返回 `true`**，因此**非数字值传入也会返回 `true` ，会影响 `NaN`的判断**。
- 函数 `Number.isNaN` 会首先判断传入参数是否为数字，如果是数字再继续判断是否为 `NaN` ，不会进行数据类型的转换，这种方法对于 `NaN` 的判断更为准确。等价于`Object.is`

# 13.`||` 和 `&&` 操作符的返回值？

> `||` 和 `&&` 都首先会**对第一个操作数执行条件判断**，如果其不是布尔值就先**强制转换为布尔类型**，然后再执行条件判断。

- 对于 `||` 来说，如果条件判断结果为 `true`就返回第一个操作数的值，**如果为 `false`就返回第二个操作数的值。**所以`||`具有熔断机制，可以作为一种**数据后备方案（前面的数据都为`falsy`的时候最后一个数据生效）**。
- `&&`则相反，如果条件判断结果为 `true`就返回第二个操作数的值，**如果为 `false` 就返回第一个操作数的值**，可以实现一种验证机制，当前面的条件都是`true`的时候执行最后一步/返回最终值。

# 14.`Object.is()` 与比较操作符 “`===`”、“`==`” 的区别？

- 使用双等号（`==`）进行相等判断时，如果两边的**类型不一致**，则会进行**强制类型转化**后再进行比较。
- 使用三等号（`===`）进行相等判断时，如果两边的**类型不一致**时，不会做强制类型准换，**直接返回 false**。
- 使用 `Object.is() `来进行`相等判断`时，一般情况下和三等号的判断相同，它**优化了一些特殊的情况**，比如 **`-0` 和 `+0` 不再相等**，**两个 `NaN` 是相等的**。

# 15.什么是 `JavaScript` 中的包装类型？

在 `JavaScript` 中，**基本类型是没有属性和方法**的，但是为了便于操作基本类型的值，在**调用基本类型的属性或方法**时 `JavaScript `会在后台**隐式地将基本类型的值转换为对象**。

```javascript
//模拟包装
let a = 'hi'
let _a = String(a)
_a.toUpperCase()
//解包
_a.valueOf()
```

# 16.为什么会有`BigInt`的提案？

`JavaScript`中`Number.MAX_SAFE_INTEGER`表示最⼤安全数字，计算结果是`9007199254740991`，即在这个数范围内不会出现精度丢失（小数除外）。但是⼀旦超过这个范围，`JS`就会出现计算不准确的情况，这在⼤数计算的时候不得不依靠⼀些第三方库进行解决，因此官方提出了新的基本类型`BigInt`来从语言层面来解决此问题。

# 17.`object.assign`和对象扩展运算符是`深拷贝`还是`浅拷贝`，两者区别如何？

## 相同点

- 都是一种浅拷贝，即对对象第一层数据的拷贝；
- 在复制的时候都仅复制对象可枚举属性（包含`Symbol`为`Key`的属性）。

## 区别

- `Object.assign()`方法接收的**第一个参数**作为**目标对象**，后面的**所有参数**作为**源对象**。然后把所有的源对象的所有可枚举值（包含`Symbol`为`Key`的属性）合并到目标对象中。它修改了一个对象，因此会触发**目标对象**的`ES6 setter`。

- 使用扩展操作符`…`，数组或对象中的所有可枚举值（包含`Symbol`为`Key`的属性）都会被**拷贝到一个新的数组或对象**中。它**不复制继承的属性或类的静态属性**。

```javascript
// 完全复制（增加原型属性）
let person = {
    name: 'ycp',
    [Symbol('des')]: "1234"
}
Object.setPrototypeOf(person, {
    age: 21
})
{...person} //{name: 'ycp', Symbol(des): '1234'}
{...person,...Object.getPrototypeOf(person)} //{name: 'ycp', age: 21, Symbol(des): '1234'}
```

# 18.`Number()`的存储空间是多大，如果后台发送了一个超过最大字节的数字怎们办?大数运算该怎么实现？

`Number`的存储空间是`64bits`，只有双精度实数这一种类型，进度只有`2 ** 53 - 1`，大约是`15`位左右，超过这个限制无法使用内置`Number`类型进行处理。

1. 优先考虑控制好`id`大小，也就是必须小于`2^53 - 1`，减少转换成本；
2. 如果**并不需要进行运算**的大数，一律从后端接收字符串，**直接利用字符串进行判断、处理**；
3. 如果是整数，后端使用字符串格式发送数据，前端使用`BigInt()`转换后进行计算或者直接使用字符串进行模拟计算；
4. 如果是带小数的，后端使用字符串格式发送数据，前端将数字拆开成整数和小数两部分并填充`0`之后进行按位相加并进位，最后将其拼接成结果字符串；

# 19.`let` `const` `var`三者之间的区别

|                            | `var`           | `let`           | `const`         |
| -------------------------- | --------------- | --------------- | --------------- |
| 块级**作用域**             | ❌（函数作用域） | ✔️               | ✔️               |
| 作用域内**声明提升**       | ✔️               | ✔️（暂时性死区） | ✔️（暂时性死区） |
| 是否可**重复声明**         | ✔️               | ❌               | ❌               |
| 是否可**重复赋值**         | ✔️               | ✔️               | ❌               |
| 初始化时是否必需赋值       | ❌               | ❌               | ✔️               |
| 是否**添加**到**全局对象** | ✔️               | ❌               | ❌               |
| 能否根据条件声明           | ✔️               | ❌               | ❌               |

## 块级作用域解决了哪些问题？

- 由于存在变量提升，内层变量可能覆盖外层同名变量，如果在定义位置前读取读取到的是变量提升后的`undefined`而不是报错；

  - ```javascript
    var tmp = 'hahaha';
    
    function f() {
      console.log(tmp);
      if (false) {
        var tmp = 'hello world';
      }
    }
    
    f(); // undefined
    ```

- 由于非严格模式的一个**默认绑定**机制，使用`for`计数的循环变量在括号内定义泄露为**绑定在全局对象上变量**，此外，在函数中未经声明就赋值一个变量也会导致其泄露为**绑定在全局对象上变量**。

# 20.`const`一个对象，可以修改他的属性吗？

`const`定义的变量实际上保存的是一个指向堆内存中对应对象内存地址的一个指针，`const`保证了指针不会改变，也就是不能修改成指向另一个对象，但它无法保证对象内部属性的不变。

# 21.`new`操作符的实现原理？实现一下`new`操作符？

> 1. 创建一个新的对象，并将这个对象的隐式原型指向构造函数的显式原型
> 2. 以新对象为调用者盗用构造函数并获取返回值；
> 3. 判断盗用构造函数后的返回值类型：如果是基本类型，返回之前创建的新对象，否则返回构造函数的返回值
>

```typescript
const myNew = (Function, ...args) => {
    if(typeof Function !== 'function') return new TypeError('first argument must be function')
    const obj = Object.create(Function.prototype)
    const ret = Function.call(obj, ...args)
    return (ret && typeof ret === 'obj') || (typeof ret === 'function') ? ret : obj
}
```

# 22.说一说原型和原型链的概念？

> ##### 注意点：**在实例给在原型对象上定义的属性**赋值**时，会在实例上创建一个同名的属性，而不会去赋值原型对象上的属性。**这里不管是引用类型还是基本类型都一样。在实例对在原型对象上定义的**引用类型**属性进行**修改**，会直接对原型对象的该属性进行修改。
>

## 原型

每一个函数都有一个`prototype`属性，但函数作为**构造函数**被调用时，若构造函数没有返回**单独返回一个引用类型**，那么创建的就是由该**构造函数**生成的**实例对象**。这个**实例对象**内部有一个`__proto__`指针指向**构造函数**的原型，而原型上储存了所有实例和子类的实例所共享的**属性**和**方法**。

## 原型的好处

使用原型对象的好处是，在**原型对象**上面定义的**属性**和**方法**可以在**对象实例**间共享。

## 原型相关的API

```javascript
Object.getPrototypeOf(obj) 				 //获取实例对象的原型
Object.setPrototypeOf(obj, value) 		 //设置实例对象的原型
Object.create(obj)						 //以obj作为原型创建一个对象，优化setPrototype的性能问题
Constructor.prototype 	   				 //获取构造函数的原型
Constructor.prototype.isPrototypeOf(obj) //判断目标原型是否存在实例对象的原型链上
obj instanceof Constructor				 //判断目标构造函数的原型是否存在实例对象的原型链上
for(key in obj){}						 //遍历所有可枚举属性，包括原型链上的，但不读取Symbol
key in obj								 //可枚举属性（String/Symbol）是否在对象及其原型链上
Object.hasOwnProperty(prop)				 //判断属性是否在对象自身上（支持不可枚举、Symbol）
```

## 原型关系图

> 注意一点：构造函数都是由`Function`创建的实例，所以内部有`__proto__`指针指向`Function`构造函数，但是由于`Function`的原型也是一个对象，所以`Function`的原型内部又有一个`__proto__`指针指向`Object`的原型。
>

![image-20220217160359361](https://github.com/NoAlligator/pico/blob/main/img/202203271746523.png?raw=true)

## 说一说原型链

原型链是`JS`的主要继承方式，每个构造函数都有一个**原型对象**，而**原型对象内部**又有一个属性（隐式原型）指向**另一个原型**，这样地原型就形成了**链式的结构**，创建的**实例对象**的原型链最终都会指向`Object`的原型对象，而`Object`内部的原型指针指向`null`，也就是原型链的终点。所以，在通过对象访问属性时，先访问实例对象自身的属性，若找不到再去这个实例对应的原型对象上寻找这个属性，再找不到就通过原型链层层向外寻找，直到`null`

```javascript
const getProperty = (o, p) => {
	  if(o.hasOwnProperty(p)) return o[p]
    let proto = Object.getPrototypeOf(o)
    while(proto !== null){
        if(proto.hasOwnProperty(p)) return proto[p]
        proto = Object.getPrototypeOf(proto)
    }
}
```

## 如何确认属性是否在原型链上而不在自身？

```javascript
const isPrototypeProperty = (o, p) => {
	return p in o && !o.hasOwnProperty(p)
}
```

## 对象属性枚举的顺序？

只有`Object.getOwnPropertyNames()`、`Object.getOwnPropertySymbols()`、`Object.assign()`的枚举顺序是固定的：先是**升序**枚举**数值键**，然后以**插入顺序**枚举**字符串**和`Symbol`

## 原型链继承如何实现？原型链继承的缺点？

> 原型继承的本质是替换**子类构造函数的原型**为**父类的实例对象**：这样能够让父类的实例属性成为子类的原型属性，同时子类继承了父类的原型（通过父类实例）
>

### 原型链的问题

存在原型上的引用类型会在所有实例之间共享。

### 原型链继承的问题

1. 父类的实例特性应当有选择地被加在子类实例上，而现在被加在了子类原型上，这使得继承地能力有限；
2. **没有给父类传参的机会**，直接固化了子类的原型为实例，不能再在创建实例的时候传入参数给父级了。

```javascript
function SuperType(superName){
	this.superName = superName
}
function SubType(subName){
	this.subName = subName
}
SubType.prototype = new SuperType('father') //这里就限制了传参，所有的实例拿到的都是同一个原型对象属性superName
const sub = new SubType('son')
```

## 盗用构造函数是怎么实现的？解决了什么问题？还有哪些问题？

> 盗用构造函数是在子类**构造函数**中以子类自己的`this`作为调用者执行了**父类构造函数**，解决了创建子类实例的时候无向父类传参的问题。
>

### 存在问题

必须在构造函数中定义方法（未使用原型进行共享），子类也不能访问父类原型上定义的方法，所以**不能单独使用**。

```javascript
function SuperType(superName){
    this.superName = superName
}
function SubType(subName){
    SuperType.call(this, 'father')
}
```

## 组合式继承是什么？

> 组合式继承就是 原型链继承+盗用构造函数
>

### 缺点

组合式继承调用了两次父类构造函数（原型链继承`new`的时候内部调用一次，盗用构造函数的时候执行一次）。

### 优点

兼具原型继承链和盗用构造函数的优点，保留了`instanceof`和`isPrototypeOf`方法识别合成对象的能力。

## 说一说原型式继承的概念？

原型式继承实现了**不需要显式地定义构造函数来继承`A`对象**，就可创建以`A`对象作为**原型**进行方法属性共享的多个实例对象：其实现思想就是内建一个临时构造函数，将这个构造函数的原型绑定为目标对象，创建实例对象并返回。原型式继承在`ES6`作为`API`出现，就是`Object.create()`

```javascript
const myCreate = (o) => {
    function F(){}
    F.ptototype = o
    return new F()
}
```

## 说一说寄生式继承？

概念：寄生式继承是指创建一个实现继承的函数创建继承后的对象，然后以某种方式增强对象并返回，最常见的就是在修改一个。

## 说一说寄生式组合继承？

> 寄生式组合继承  =  原型式继承+寄生式继承+盗用构造函数
>
> 1. 通过一个指向父类原型对象地空函数创建一个空对象，修饰空对象的`constructor`为子类构造函数；
> 2. 设置这个空对象为子类构造函数的原型；
> 3. 在子类构造函数中盗用父类构造函数（可选）；
> 4. 设置子类构造函数的隐式原型为父类构造函数，实现静态属性的一个继承。

```javascript
const myExtends = (Parent, Child) => {
    const prototype = Object.create(Parent.prototype)  //原型式继承
    prototype.constructor = Child // 寄生式继承
    Child.prototype = prototype
}
function SuperType(superName){
    this.superName = superName
}
function SubType(subName, ...args){
    SuperType.call(this, ...args) //盗用构造函数
    this.subName = subName
}
myExtends(SubType, SuperType)
```

# 23.说一说关于`class`的知识点？

## 类的`constructor`

1. `new`对象实例的时候`constructor`是默认执行的，没有显式定义的话会在类的内部加上空的`constructor(){}`；
2. 如果**继承的子类**创建了构造函数，但是没有调用`super`，那么在出创建实例的时候会报错，所以继承的子类必须调用`super`；
3. 如果**继承的子类**没有显式的调用构造函数，在创建子类实例的时候会默认执行一个构造函数，在这个构造函数内部会执行`super(...args)`，所以需要注意这个默认行为；

## 类的变量提升

存在暂时性死区，这种规定与继承的顺序有关，所以必须保证子类在父类之后定义。

## 类的代码规范

类和模块的内部默认使用**严格模式（导致使用类上面的方法脱离`ReferenceType`之后调用会`this`就是`undefined`）**；

## 类的私有属性与方法提案

私有属性的提案是在变量或者方法前面加上`#`；

## 获取类的名字

访问`class`的名字通过`Constructor.name`；

## 类内部的拦截器

在`class`内部可以定义`getter`和`setter`，对某个实例属性的存取进行拦截（如果存在同名的实例属性会遮蔽同名`getter`与`setter`），它会被设置在属性的`Descriptor`上；

## 静态方法的继承

父类的**静态方法、属性都可以被子类继承**（可以直接通过`子类.(属性/方法)`访问），原理就是将构造函数内部的隐式指针指向父类构造函数，这样通过子类构造函数就可以直接访问到父类构造函数上的属性；

## 继承原理比较+`super()`的限制

> ES6 规定，子类必须在`constructor()`方法中调用`super()`，否则就会报错。这是因为子类自己的`this`对象，**必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，添加子类自己的实例属性和方法。如果不调用`super()`方法，子类就得不到自己的`this`对象**。

### 为什么子类的构造函数，一定要调用`super()`？

原因就在于 ES6 的继承机制，与 ES5 完全不同。ES5 的继承机制，是先创造一个独立的子类的实例对象，然后再将父类的方法添加到这个对象上面，即“实例在前，继承在后”。ES6 的继承机制，则是先将父类的属性和方法，加到一个空的对象上面，然后再将该对象作为子类的实例，即“继承在前，实例在后”。这就是为什么 ES6 的继承必须先调用`super()`方法，因为这一步会生成一个继承父类的`this`对象，没有这一步就无法继承父类；

```java
// 对于ES6
class Child extends Parent {
  constructor(name) {
    super() // 先进行继承，获得实例
    this.name = name // 在实例上进行修饰
  } 
}
// 对于ES5
new Child(...args) // 1.先创建子类

function Child() {
    Parent.call(this, ...args) // 然后继承
		/* ... */
}
```



## `super()`的执行原理

> `super(...args)`作为父类构造函数在**子类构造函数**中被执行的时候，无需指定`this`，内部`this`指向的是子类实例，相当于盗用构造函数；
>
> `super`作为对象被访问出现在**子类原型方法和子类构造函数**中的时候都指向父类的原型对象，可以直接通过`super.x`访问原型上的`x`属性；
>
> `super`作为对象被访问时在**静态方法**中指向父类构造函数，可以访问父类静态方法，也可以通过`super.prototype`间接访问原型；
>
> ⚠ 直接访问`super`关键字会抛出错误；
>
> ⚠ 通过`super.x`进行赋值时，会在**自身创建同名属性并赋值**，但是赋值之后再去访问`super.x`取到的却是父类的值。举例：如果通过`super.x = xxx`进行赋值，等价于`this.x = xxx`，因为`super`绑定了`this`，但赋值后再访问`super.x`得到的是父类原型上的`x`值。可见：`super`设置了一种存取拦截器，设置`super.x`时绑定了`this`，读取`super.x`的时候直接从父类原型上读取。
>
> ⚠ 如果构造函数或实例方法执行`super.x()`，会自动绑定当前函数的`this`，相当于`super.x.call(this)`。

## 通过类判断`class`的继承关系

```javascript
Object.getPrototypeOf(Son.prototype) === Father
// 可以判断Son是否继承了Father（直接继承）

// 通过原型判断继承
// 遍历Son的原型链找到是否有Father.prototype
function isExtends(Son, Father) {
    let prototype = Object.getPrototypeOf(Son.prototype)
    let target = Father.prototype
    while(prototype) {
        if(prototype === target) return true
        prototype = Object.getPrototypeOf(prototype)
    }
    return false
}
```

## `new.target`

使用`new.target`属性可以判断构造函数被调用的方式（用以确保不被当作普通函数调用）以及`new`的调用方是不是自己（用于创建无法实例化的基类），注意：不可直接访问`new`。

```javascript
//防止意外调用，只能被实例化
function Person() {
    if(new.target === undefined) throw new Error('Constructor Person must called with new operator')
}
//防止抽象基类实例化
class Abstract {
    constructor() {
        if(new.target === Abstract) throw new Error('Constructor Abstract can not be instantiated')
    }
}
class Sub extends Abstract {

}
```

## 设置`generator`遍历

```javascript
class Gene{
	* [Symbol.iterator]() {
        //...
    }
}
```



## `this`

> 在类的方法里谨慎使用`this`关键字，因为当`class`的方法被单独提取出来的时候，`this`可能丢失。这是因为类内部方法开启严格模式。

1. 解决方案一是在构造函数中使用`bind`绑定`this`，相当于创建了一个修饰了`this`的实例方法；
2. 解决方案二是使用箭头函数使之成为实例方法，因为箭头函数的`this`依赖外部普通函数的`this`，而构造函数在创建实例的时候`this`指向的是实例，所以箭头函数的`this`也就被指向实例；
3. 解决方案三是`proxy`代理，在获取方法的时候自动绑定`this`。

```javascript
//方案一
class BindClass{
	name = "ycp"
	printName(){
		console.log(this.name)
	}
	constructor(){
		this.printName = this.printName.bind(this)
	}
}
//方案二
class BindClass{
	name = "ycp"
	printName = () => {
		console.log(this.name)
	}
}
//方案三
function Proxied(target){
	const cache = new WeakMap() //减少读取次数，直接缓存
	const handler = {
		get (target, key){
			const value = Reflect.get(target, key)
			if(typeof value !== 'function') return value
			if(!cache.has(value)) {
				cache.set(value, value.bind(target))
			}
			return cache.get(value)
		}
	}
	const proxy = new Proxy(target, handler)
	return proxy
}
```

# 24.`class`的继承机制

## `class`的继承关系

```javascript
class Parent{}
class Child extends Parent{}
Object.getPrototype(Child) === Parent
Object.getPrototype(Child.prototype) === Parent.prototype

//实现方式
class Parent{}
class Child{}
//继承原型方法，作为一个构造函数，Child的原型是父类的实例
Object.setPrototypeOf(Parent.prototype, Child.prototype)
//继承静态方法，作为一个对象，Child的原型是父类A
Object.setPrototypeOf(Parent, Child)
```

## `class`的继承目标

能够被继承的条件：只要是一个有`prototype`属性的函数，就可以被继承

## `class`继承原生构造函数

以往是<u>无法实现</u>继承原生构造函数的，例如：无法通过`Array.apply(this, arguments)`来实现以自己的上下文创建原生对象的过程，因为**原生构造函数会忽略`apply`方法传入的`this`**。因为`ES5`的继承方式是<u>先新建子类的实例对象</u>，再将<u>父类的属性添加到子类身上</u>，而部分父类的内部属性是无法获取的，这就导致无法继承原生的构造函数。

```javascript
function MyArray(){
	Array.apply(this, arguments) //盗用构造函数失效，因为this无法通过apply绑定到原生构造函数上
}
MyArray.prototype = Object.create(Array.prototype,{
	constructor: {
		value: MyArray,
		enumberable: false,
		writable: true,
		configurable: true
	}
})
```

`ES6`的继承方式使得继承原生构造函数成为了现实（因为是先新建父类实例对象`this`，然后再用子类构造函数修饰`this`的）

```javascript
class MyArray extends Array{}
```

## 手写实现`class`的混入

`class`的**混入**指的是将多个类的**静态方法与属性**（需要剔除`prototype`）、原型对象（需要剔除`constructor`）混入混入同一个类中。

```js
function mix(...mixins){
    class Mix {
        constructor() {
            for (let mixin of mixins) {
                copyProperties(this, new mixin()); // 拷贝实例属性
            }
        }
    }
    for (let mixin of mixins) {
        copyProperties(Mix, mixin); // 拷贝静态属性
        copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
    }
    return Mix;
}

function copyProperties(target, source) {
    for(let key of Reflect.ownKeys(source)){
        if(key !== 'constructor' && key !== 'prototype' && key !== 'name') {
            const descriptor = Object.getOwnPropertyDescriptor(source, key);
            Object.defineProperty(target, key, descriptor)
        }
    }
}
```

------

# 25.遍历对象的几种方法和区别

| 遍历方法                         | 原型链上的枚举属性 | 不可枚举属性 | `Symbol` | 描述                |
| :------------------------------- | ------------------ | ------------ | -------- | ------------------- |
| `for(key in obj)`                | ✔️                  |              |          | 所有可枚举          |
| `Object.keys()`                  |                    |              |          | 自身可枚举          |
| `Object.getOwnPropertyNames()`   |                    | ✔️            |          | 自身可枚举+不可枚举 |
| `Object.getOwnPropertySymbols()` |                    |              | ✔️        | 自身所有`Symbol`    |
| `Reflect.ownKeys(obj)`           |                    | ✔️            | ✔️        | 自身所有属性        |

# 25.箭头函数和普通函数有什么区别？

- 箭头函数**没有原型对象**。

- 箭头函数的`this`指向其定义外层位置外层的第一个普通函数的`this`或指向`window`，被继承的普通函数的`this`指向改变，箭头函数的`this`也会随之改变。

  ```javascript
  function A(){
      const say = () => {console.log(this.name)} 
      say()
  }
  A.call({name: 'ycp'})  //'ycp'
  ```

- 箭头函数无法通过`call`、`apply`、`bind`修改`this`指向

  ```javascript
  let a
  let barObj = { msg: 'this指向bar' };
  let fooObj = { msg: 'this指向foo' };
  bar.call(barObj);
  foo.call(fooObj);
  function foo() {
    a(); 
  }
  function bar() {
    a = () => {
      console.log(this, 'this指向定义位置外层第一个普通函数的this或者window');
    };
  }
  ```

- 箭头函数的`this`指向全局对象，会**报`arguments`未声明的错误**，箭头函数的`this`如果指向普通函数,它的`argumens`继承于该普通函数（闭包）。

  ```js
  let obj = {
      say(){
          const inner = () => {console.log(arguments)}
          console.log(arguments)
      }
  }
  obj.say() //[1, 2, 3, callee: ƒ, Symbol(Symbol.iterator): ƒ]
  ```

  

- 箭头函数不支持`new`，如果箭头函数的`this`指向普通函数，**它的`new.target`就是指向该普通函数的引用**。

# 26.可以`new`一个箭头函数吗？为什么不可以？

## 本质原因

`JavaScript`函数有两个内部方法`[[Call]]`和`[[Constrcutor]]`，而箭头函数并没有没有部署`[[Constrcutor]]`内部方法，是无法通过`new`操作符生成实例对象的。

## 特性表现

并且，箭头函数的**特性上**也可以看出他的实现是不应该作为**构造函数**出现的：

1. 首先，它没有`prototype`属性指向自己的原型对象，无法**通过原型链进行原型链继承**；
2. 其次，**箭头函数的`this`继承自普通函数或指向`window`**，所以`this`是无法指向实例对象的；
3. 此外，箭头函数的`this`如果指向普通函数，**它的`argumens`继承于该普通函数**。

# 27.箭头函数有哪些场景不适用的？那些适用的？

- 当心**存储在对象中的箭头函数**，它的`this`指向的并不是对象本身；
- 当心箭头函数**在事件侦听器中使用时和普通函数的区别**（普通函数`this`指向**执行元素**本身、箭头函数**依赖定义位置外部函数的`this`**）；
- 将箭头函数定义在构造函数的原型对象上，实例调用时箭头函数内部`this`并不是实例本身（取决于箭头函数创建的位置）。

# 28.箭头函数好处在哪里？

箭头函数比普通函数简洁：

- 如果没有参数，就直接写一个**空括号**即可；
- 如果只有**一个**参数，可以**省去参数的括号**；
- 如果有**多个参数**，用**逗号**分割；
- 如果函数体的返回值只有一句，可以**省略大括号**；
- 如果函数体不需要返回值，且只有一句话，可以给这个语句前面加一个`void`关键字。

# 29.数组的解构有哪些注意点？

> 针对数组的扩展运算符的本质是调用`[Symbol.iterator]`接口，对象的扩展运算符类似于执行`Object.assign`将枚举属性（包含`Symbol`）扩展到目标对象上。

- 可以以`[,,target]`的方式忽略部分参数。
- `[a, ...tail]`必须在末尾使用扩展运算符`...`，扩展运算符后面不能有用于解构的变量。
- 只要是部署了`iterator`接口的对象都可以进行解构，如果对未部署`iterator`接口的对象进行解构会报错，内部的深层次解构也遵循这一规定。
- 解构允许设置**默认值**，只有一个**数组成员**严格等于`undefined`默认值才会生效（`null`时默认值不生效）。默认值可以引用**其他解构赋值变量**。

# 30.对象的解构有哪些注意点

- 解构时的**父对象如果不存在会报错**。

- 数组和对象的**解构可以混合**。

- 解构赋值给自定义变量别名时，**解构赋值取值和变量别名的声明可以分离**。

- 解构赋值也可以**指定默认值**。

- 扩展运算符支持获取可枚举的对象自身属性（包含`Symbol`作为属性名的属性），这和`Object.assign()`的扩展特征是一样的。

- 解构赋值**支持获取`prototype`上的属性**（扩展运算符则不能）。

- 将一个**已经声明的变量用于解构赋值会导致语法解析问题**，因为 `JavaScript` 引擎会**将`{x}`理解成一个代码块，从而发生语法错误**。只有不将大括号写在行首，避免 `JavaScript` 将其解释为代码块，才能解决这个问题。

  ```javascript
  let x;
  {x} = {x: 1}	//SyntaxError: syntax error
  ({x} = {x: 1})
  ```

- 因为数组本质上就是特殊的对象，也可以**对数组进行解构赋值**。

# 31.一些其他内置对象的解构赋值？

内置对象的解构赋值遵循以下的逻辑：基本类型会被包装成**内置引用类型**再进行解构。

支持解构的基本类型：

- `string`、`number`、`boolean`；
- `null`、`undefined`不支持转换成包装引用类型所以无法解构。

# 32.函数参数的解构赋值？

- 遵循引用类型（数组对象等）、和可以转换的基本类型解构的基本准则

# 33.解构赋值有哪些用处？

- 通过解构数组来**快速交换变量**的值`[x, y] = [y, x]`；
- 针对函数**返回值进行解构**，能够一目了然的获知所需要的返回值内容；
- **函数所需参数通过解构**的形式一目了然；
- 在取值的同时**设置默认值作为后备**更方便；
- **读取`Map`这这一类部署了`iterator`接口的数据结构**非常方便；
- 解构能够优雅地引入依赖库所需的模块。

# 34.说一说函数的`rest`参数？

- `ES6` 引入 `rest` 参数（形式为`...变量名`），用于获取函数的多余参数，这样就不需要使用`arguments`对象了
- `arguments`对象不是数组，而是一个**类似数组的对象**。所以为了使用数组的方法，必须使用`Array.from`先将其转为数组。`rest` 参数就不存在这个问题，**它就是一个真正的数组**，数组特有的方法都可以使用。
- `rest`操作符用于搜集剩余参数，也就是说：`rest`参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。

# 35.说一说扩展运算符的用处？

## 数组

- 替代函数的`apply`方法，可以将数组中的元素用扩展运算符展开，入参到指定函数。例如`Math.max(...args)`、`array.push(...args)`。
- 合并数组更快捷，常用到的到的就是`React`中的更新数组时可以使用扩展运算符更加直观。
- 扩展运算符可以和解构赋值结合，用于**收集剩余元素成为数组**。
- 扩展运算符可以**快速地将字符串转换成数组**（包装类型 + 部署了`Symbol.iterator`）
- 扩展运算符兼容所有部署了`[Symbol.iterator]`接口的引用类型。

## 对象

- 扩展运算符可以和解构赋值结合，用于解构赋值时收集剩余元素成为新对象。
- 扩展运算符可用于取出参数对象的所有**可遍历（包含`Symbol`）属性**，拷贝到当前对象之中（对基本类型执行扩展运算符会默认进行包装`Object(basic)`，这对于`string`类型是有效的）。**对象的扩展运算符类似于使用`Object.assign()`方法。**
- 使用扩展运算符时，后定义的属性会覆盖前面的属性，这在`React`需要更对象的部分属性的时候创建更新的对象副本是非常方便的。

# 36.`JS`操作`Unicode`字符的问题？

[参考链接](https://www.ruanyifeng.com/blog/2014/12/unicode.html)

`JavaScript` 会将**四个字节的 `Unicode`字符，识别为 2 个字符**，究其原因是因为`JS`存储字符串时使用的是`UTF-16`编码，其字符的存储方式为固定的`2/4`个字节，两个字节的文本被视作长度为`1`，要是遇到了占用`4`个字节的字符，就会得到长度为`2`的结果。现在浏览器编码最常用的`UTF-8`这种编码方式通过`1~6`个字节存储字符。

解决方式，如果遇到需要计算字符个数的情形，需要将字符串迭代到数组中再获取数组的长度，因为`String`类型内部部署的`[Symbol.iterator]`是按照**单个字符**进行遍历的：

```javascript
'x\uD83D\uDE80y'.length // 4
[...'x\uD83D\uDE80y'].length // 3
```

# 37.`JS`如何转换进制？

任意进制（字符串） -->  十进制

```js
parseInt(string[, radix]);
```

十进制（数字）  -->   任意进制（字符串）

```javascript
number.toString([radix])
```

# 38.`Map`和`Object`的区别？`Map`如何使用？`Map`有哪些使用场景？

|          | `Map`                                                        | `Object`                                                     |
| :------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 键名冲突 | `Map`默认情况不包含任何键，只包含显式插入的键。是一种纯粹的哈希表的数据结构。 | `Object`有一个原型, 原型链上的键名有可能和自己在对象上的设置的键名产生冲突。（虽然 ES5 开始可以用 `Object.create(null)` 来创建一个没有原型的对象，但是这种用法不太常见）。 |
| 键的类型 | `Map`的键可以是任意值，包括函数、对象或任意基本类型（`null`，`undefined`都可以）。 | `Object`的键必须是`String`或是`Symbol`。其他的类型会调用其上的`toString()`转换。 |
| 键的顺序 | `Map`中的 `key`是**有序的**。因此，当迭代的时候，`Map`对象以**插入的顺序返回键值**。 | `Object`的键在插入了`Symbol`之后是无序的（自`ECMAScript 2015`规范以来，对象确实保留了字符串和`Symbol`键的创建顺序； 因此，在只有字符串键的对象上进行迭代将按插入顺序产生键）。 |
|   大小   | `Map`的键值对个数可以轻易地通过`size`属性获取                | `Object`的键值对个数只能通过获取所有属性数组再计算。         |
|   迭代   | `Map`是 `iterable` 的，所以可以直接被迭代。                  | 迭代`Object`需要以某种方式获取它的键然后才能迭代。           |
|   性能   | 在频繁**增删键值对的场景**下表现更好。                       | 构建对象和读取`key`值对应的`value`更快。                     |

```javascript
let map = new Map([iterable])   
//构造函数，Iterable 可以是一个数组或者其他iterable对象，其元素为键值对(两个元素的数组，例如: [[1, 'one'],[2, 'two']])。 每个键值对都会添加到新的 Map。null 会被当做 undefined。
Map.prototype.size      		//大小
Map.prototype[Symbol.iterator]()//迭代方法
Map.prototype.clear()			//清除
Map.prototype.delete(key)  		//删除
Map.prototype.get(key)			//获取
Map.prototype.has(key)			//查询
Map.prototype.set(key, value)	//设置
Map.prototype.forEach()			//遍历
Map.prototype.entries()
Map.prototype.keys()
Map.prototype.values()
```

## 深入对比性能

### 查找速度

从大型`Object`和`Map`中查找键/值对的性能差异极小，如果只包含少量键/值对，`Object`有时候速度更快。在把`Object`当成数组使用的情况下（比如连续整数作为属性），浏览器引擎进行了优化，在内存中使用了更高效的布局。所以，如果代码涉及**大量查找操作**时，某些情况下选用`Object`性能更好些。

### `JSON`化

`JSON`化直接支持`Object`，但尚未支持`Map`（但可以变通地通过`Map`转化成`entries`数组再被`jsonfy`）。

### 增删速度

`Map`是一个纯哈希结构，而`Object`不是（它拥有自己的内部逻辑）。使用`delete`对`Object`的属性进行删除操作存在很多性能问题。所以，针对于存在大量删除键值对操作的场景，使用`Map`更合适。向`Object`和`Map`中插入新键/值对的消耗大致相当，当涉及到大量的插入操作时，`Map`的性能更佳。

### 存储效率

存储单个键/值对所占用的内存数量都会随着键的数量线性增加。给定固定大小的内存，`Map`大约可以比`Object`多存储`50%`的键/值对。

## 总结

- 希望完美支持`JSON`、考虑创建速度、元素键值确定数据量小且一般只做根据键值获取（更新）元素的操作、只使用`String`和`Symbol`作为键值、不考虑同时包含`String`和`Symbol`两种键值时候的顺序问题，选`Object`；
- 数据量大且频繁增删键值对、键值不确定，希望使用**所有类型**作为键值、希望数据迭代有序、希望获得一个纯粹的数据结构的情况下选择`Map`。

# 39.说一说`WeakMap`的用法和场景？

```javascript
let weakmap = new WeakMap([iterable])
WeakMap.prototype.delete(key)
WeakMap.prototype.get(key)
WeakMap.prototype.has(key)
WeakMap.prototype.set(key, value)
```

## `WeakMap`只接受引用类型作为键名

在 `JavaScript` 里，`map API` 可以通过使其四个 `API `方法共用两个数组(一个存放键,一个存放值)来实现。给这种 `map` 设置值时会同时将键和值添加到这两个数组的末尾。从而使得键和值的索引在两个数组中相对应。当从该 `map`取值的时候，需要遍历所有的键，然后使用索引从存储值的数组中检索出相应的值。但这样的实现会有两个很大的缺点，首先**赋值和搜索操作**都是 `O(n) `的时间复杂度( `n `是键值对的个数)，因为这两个操作都需要遍历全部整个数组来进行匹配。**<u>另外一个缺点是可能会导致内存泄漏，因为数组会一直引用着每个键和值。这种引用使得垃圾回收算法不能回收处理他们，即使没有其他任何引用存在了。</u>**

相比之下，原生的 `WeakMap` 持有的是每个**<u>键对象</u>**的“弱引用”，这意味着在没有其他引用存在时垃圾回收能正确进行。原生 `WeakMap `的结构是特殊且有效的，其用于映射的 `key` 只有在其没有被回收时才是有效的。**正由于这样的弱引用，`WeakMap` 的 `key` 是不可枚举的 (没有方法能给出所有的 `key`)。如果`key` 是可枚举的话，其列表将会受垃圾回收机制的影响，从而得到不确定的结果。**因此，如果你想要这种类型对象的 `key` 值的列表，你应该使用 `Map`

基本上，如果你**<u>要往对象上添加数据</u>**，又**<u>不想干扰垃圾回收机制</u>**，就可以使用 `WeakMap`。

## 总结

`WeakMap`的设计目的在于，有时想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用。一旦不再需要这两个对象，就必须**手动删除这个引用，否则垃圾回收机制就不会释放对象占用的内存。**而`WeakMap`的键名所引用的对象都是弱引用，即**垃圾回收机制不将该引用考虑在内**。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，`WeakMap`里面的键名对象和所对应的键值对会自动消失，不用像`Map`一样需要手动删除引用，但是正因为如此，它是不可遍历的。

# 40.说一说`JSON`？对比`JSON`和`Object`?所有对象都可以转换为`JSON`吗？

## 理解

> `JSON` 是一种基于文本的轻量级的**<u>数据交换格式</u>**。它可以被任何的编程语言读取和作为数据格式来传递。

在项目开发中，使用 `JSON` 作为前后端数据交换的方式。在前端通过将一个符合 `JSON `格式的数据结构序列化为 `JSON` 字符串，然后将它传递到后端，后端通过 `JSON`格式的字符串解析后生成对应的数据结构，以此来实现前后端数据的一个传递。在`JS`，可以通过内置对象`JSON`上的`JSON.parse()`解析`JSON`为对象，通过`JSON.stringfy()`来将对象/值转换成`JSON`字符串。

区别：

| `JS`类型   | `JSON`的不同点                                               |
| ---------- | ------------------------------------------------------------ |
| 对象和数组 | 属性名称必须是**<u>双引号</u>**括起来的字符串；最后一个属性后**<u>不能有逗号</u>**。 |
| 数值       | 禁止出现前**<u>导零</u>**（ `JSON.stringify` 方法自动忽略前导零，而在 `JSON.parse`方法中将会抛出 `SyntaxError`）；如果有小数点, 则**<u>后面至少跟着一位数字</u>**。 |
| 字符串     | 只有有限的一些字符可能会被**<u>转义</u>**；禁止某些**<u>控制字符</u>**； **<u>字符串必须用双引号括起来</u>**。Unicode 行分隔符 （[U+2028](https://link.juejin.cn?target=https%3A%2F%2Funicode-table.com%2Fcn%2F2028%2F)）和段分隔符 （[U+2029](https://link.juejin.cn?target=https%3A%2F%2Funicode-table.com%2Fcn%2F2029%2F)）被允许。 |

## `JSON`支持的值

- 基本数据类型：使用与 `JavaScript` 相同的语法，可以在 `JSON` 中表示字符串、数值（只支持十进制数）、布尔值和 `null`。 **`JSON` 不支持解析 `JavaScript` 中的基本类型 `undefined`（遇到值为`undefined`的对象属性会直接略过，也就是`undefined`的字段在解析后不会出现在`JSON`的数据结构中）；**
- 对象：对象作为一种复杂数据类型，表示的是一组无序的键值对。而每个键值对儿中的值可以是简单值，也可以是复杂数据类型的值；
- 数组：数组也是一种复杂数据类型，表示一组有序的值的列表，可以通过数值索引来访问其中的值。数组的值也可以是任意类型 —— `简单值`、`对象`或`数组`；
- 函数：类似于`undefined`略过该属性，不解析；
- 日期对象：调用`toISOString()`转换为日期字符；
- 其他引用类型：例如`Map`、`Set`、`RegExp`等的内置引用类型，都会统一转化成`{}`，表示为一个引用类型，一些有序的数据结构例如`ArrayBuffer`等的会有不一样的特征。

## 如何存储`Map`、`Set`？

`Map`使用`Object.entries(map)`转换成`[key, value][]`数组，`Set`使用`Object.values(set)`转换成`[value][]`数组

## `JSON`的特殊用例

- 只有有效的值所对应的属性会被解析（非`undefined`）；
- `JSON` 不能存储 `Date` 对象。`JSON.stringify()` 会将所有日期转换为字符串；
- `JSON.stringify(value[, replacer [, space]])` 会被删除对象内部值为`undefined`、`function`的`key`和 `value`；
- `JSON.stringify()` 只会读取对象自身的可枚举属性（不包含`Symbol`属性）；
- `JSON.stringify()`无法处理对象循环引用的情况，会报错。

## `JSON.stringfy`的过滤功能

`replacer` 参数可以是一个函数或者一个数组。作为函数，它有两个参数，键（`key`）和值（`value`），它们都会被序列化。**在开始时，`replacer` 函数会被传入一个空字符串作为 `key` 值，代表着要被 `stringify` 的这个对象，随后每个对象或数组上的属性会被依次传入。** 

函数应当返回`JSON`字符串中的`value`, 如下所示：

- 如果返回一个 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)，转换成相应的字符串作为属性值被添加入 `JSON `字符串。
- 如果返回一个 [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)，该字符串作为属性值被添加入 `JSON`字符串。
- 如果返回一个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)，`true` 或者 `false`作为属性值被添加入 `JSON `字符串。
- 如果返回任何其他对象，该对象递归地序列化成 `JSON` 字符串，对每个属性调用 `replacer` 方法。除非该对象是一个函数，这种情况将不会被序列化成 `JSON` 字符串。
- 如果返回 `undefined`，该属性值不会在 `JSON` 字符串中输出。

注意：不能用 `replacer` 方法，从数组中移除值（`values`），因为数组的长度是固定的，如若针对数组元素返回`undefined `，那么这个元素将会被 `null` 取代。

```javascript
let obj = {name: 'ycp', age: 21, birth: 2000}
JSON.stringify(obj, (k, v) => k !== 'age' ? v : undefined)
//'{"name":"ycp","birth":2000}'
JSON.stringify(obj, ['name', 'birth'])
//'{"name":"ycp","birth":2000}'
```

# 41.说一说类数组对象？

## 怎样构建一个类数组对象

一个对象拥有 `length` 属性而且值是不大于 `2 ** 32 - 1` 的整型，该对象就可以被称为**类数组对象**，类数组对象和数组类似，但是不能调用数组的方法。常见的类数组对象有`arguments`和`nodelist`。函数也可以被看作是类数组对象，因为它也含有 `length` 属性值，代表可接收的参数个数。实现上面这两点，就可以使一个对象与`Array.prototype`中大部分方法兼容。

由于借用 `Array API`，一切以数组为输入，并以数组为输出的 `API` 都可以来做数组转换，例如：

```javascript
Array.from(arrayLike);

Array.prototype.slice.call(arrayLike);
Array.prototype.splice.call(arrayLike, 0);

Array.prototype.concat.apply([], arrayLike); 
//如果arrayLike用作第一个参数，则他会作为一个元素加入新的数组

Array.prototype.map.call(arrayLike, x => x);
Array.prototype.filter.call(arrayLike, x => 1);
//其他Array.prototype上的方法一般也都支持
```

所有的数组方法都可以被**类数组盗用**，因为内部存在一个`Array.from(arrayLike)`的**隐式转换**。

# 42.说一说有哪几种常见的编码方式与字符合集？

## 字符合集

1. `ASCII`：称为美国标准信息交换码，包含了`"A-Z"`(包含大小写)，数据`"0-9"`以及一些常见的符号，是专门为英语而设计的，有`128`个编码，对其他语言无能为力。（计算机内部用一个字节存放一个7位ASCII码值。）
2. `Unicode`：叫做统一码、万国码、单一码。`Unicode` 是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的二进制编码，以满足跨语言、跨平台进行文本转换、处理的要求。`Unicode`的实现方式（也就是编码方式）有很多种，常见的是**`UTF-8`**、**`UTF-16`**、**`UTF-32`**和**`USC-2`**。

## 编码方式

[参考资料：UTF-8](https://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

[参考资料：比较UTF-8和UTF-16](https://github.com/creeperyang/blog/issues/4)

[参考资料：三种编码方式对比](http://c.biancheng.net/cpp/html/3420.html)

## `UTF-8`

> ``UTF-8`是**使用最广泛的`Unicode`编码方式，它是一种可变长的编码方式**，可以是`1—4`个字节不等，它可以完全兼容`ASCII`码的128个字符。

1. 对于**单字节**的符号，字节的第一位为`0`，后面的`7`位为这个字符的`Unicode`编码，因此对于英文字母，它的`Unicode`编码和`ACSII`编码一样。
2. 对于**`n字节`**的符号，第一个字节的前`n`位都是`1`，第`n+1`位设为`0`，后面字节的前两位一律设为`10`，剩下的没有提及的二进制位，全部为这个符号的`Unicode`码 。

| 字符                     | N        | æ                 | ⻬                         |
| ------------------------ | -------- | ----------------- | -------------------------- |
| Unicode 编号（二进制）   | 01001110 | 11100110          | 0010 1110 1110 1100        |
| Unicode 编号（十六进制） | 4E       | E6                | 2E EC                      |
| UTF-8 编码（二进制）     | 01001110 | 11000011 10100110 | 11100010 10111011 10101100 |
| UTF-8 编码（十六进制）   | 4E       | C3 A6             | E2 BB AC                   |

## `UTF-16`

> 介于 `UTF-8` 和 `UTF-32`之间，**使用 `2` 个或者 `4` 个字节来存储，长度既固定又可变**。

1. 平面：`Unicode`编码中有很多很多的字符，它并不是一次性定义的，而是分区进行定义的，每个区存放**`65536`**（`216`）个字符，这称为一个**平面**，目前总共有`17 `个平面。最前面的一个平面称为**基本平面**，它的码点从**`0 — [(2^16)-1]`**，写成`16`进制就是`U+0000 — U+FFFF`，那剩下的`16`个平面就是**辅助平面**，码点范围是 `U+10000—U+10FFFF`。基本平面内，从`U+D800`到`U+DFFF`之间的码位区块是永久保留不映射到`Unicode`字符。`UTF-16`就利用保留下来的`0xD800-0xDFFF`区段的码位来对辅助平面的字符的码位进行编码；
2. 概念：`UTF-16`也是`Unicode`编码集的一种编码形式，把`Unicode`字符集的抽象码位映射为16位长的整数（即码元）的序列，用于数据存储或传递。`Unicode`字符的码位需要1个或者2个16位长的码元来表示，因此`UTF-16`也是用变长字节表示的；
3. 规则：编号在 `U+0000—U+FFFF` 的字符（常用字符集），直接用两个字节表示。编号在 `U+10000—U+10FFFF` 之间的字符，需要用四个字节表示；
4. 举例：以 "**𡠀**" 字为例，它的 `Unicode` 码点为 `0x21800`，该码点超出了基本平面的范围，因此需要用四个字节来表示，步骤如下：
   - 首先计算超出部分的结果：`0x21800 - 0x10000`；
   - 将上面的计算结果转为`20`位的二进制数，不足`20`位就在前面补`0`，结果为：`0001000110 0000000000`；
   - 将得到的两个`10`位二进制数分别对应到两个区间中；
   - `U+D800` 对应的二进制数为 `1101100000000000`， 将`0001000110`填充在它的后10 个二进制位，得到 `1101100001000110`，转成 16 进制数为 `0xD846`。同理，低位为 `0xDC00`，所以这个字的`UTF-16` 编码为 `0xD846 0xDC00`。

| Unicode 编号范围 （十六进制） | 具体的 Unicode 编号 （二进制） | UTF-16 编码                         | 编码后的 字节数 |
| ----------------------------- | ------------------------------ | ----------------------------------- | --------------- |
| 0000 0000 ~ 0000 FFFF         | xxxxxxxx xxxxxxxx              | xxxxxxxx xxxxxxxx                   | 2               |
| 0001 0000---0010 FFFF         | yyyy yyyy yyxx xxxx xxxx       | 110110yy yyyyyyyy 110111xx xxxxxxxx | 4               |

## `UTF-32`

``UTF-32` 就是字符所对应编号的整数二进制形式，每个字符占四个字节，这个是直接进行转换的。该编码方式占用的储存空间较多，所以使用较少。比如“**马**” 字的`Unicode`编号是：`U+9A6C`，整数编号是`39532`，直接转化为二进制：`1001 1010 0110 1100`，这就是它的`UTF-32`编码。

# 43.`Unicode`、`UTF-8`、`UTF-16`、`UTF-32`有什么区别？

> `Unicode` 是编码字符集（字符集），而`UTF-8`、`UTF-16`、`UTF-32`是字符集编码（编码规则）

`UTF-16` 使用变长码元序列的编码方式，相较于定长码元序列的`UTF-32`算法更复杂，甚至比同样是变长码元序列的`UTF-8`也更为复杂，因为其引入了独特的**代理对**这样的代理机制；（占用内存大，逻辑复杂）

`UTF-8`需要判断每个字节中的**<u>开头标志信息</u>**，所以如果某个字节在传送过程中出错了，就会导致**<u>后面的字节也会解析出错</u>**（占用内存小，容错能力差）；而`UTF-16`不会判断开头标志，即使错也只会错一个字符，所以**<u>容错能力较强</u>**；

如果字符内容<u>**全部英文或英文与其他文字混合，但英文占绝大部分（UTF-8中的一字节文本，UTF-16需要固定二字节存储）**</u>，那么用`UTF-8`就比`UTF-16`节省了很多空间；而如果**<u>字符内容全部是中文这样类似的字符或者混合字符中中文占绝大多数（UTF-8用三字节存储二字节的Unicode码位（标志位占用），但在UTF-16中只需要二字节（原样存储））</u>**，那么`UTF-16`就占优势了，可以节省很多空间；

## 总结

在只考虑存储大小的情况下，如果文本中有较多的一字节`Unicode`码位（`ascii`英文字母等），可以使用用`UTF-8`；如果文本中有较多的二字节（及以上）的`Unicode`码位（例如中文字符）等用`UTF-16`。

# 44.说一说`JS`有哪些位运算符？

> 在`JavaScript`内部，数值都是以`64`位浮点数的形式储存，但是做位运算的时候，是以`32`位带符号的整数进行运算的，并且返回值也是一个`32`位带符号的整数。而`Int32`采用**补码**表示，因此取反是在补码的基础上进行的，取反运算（`~n`）需要将整数转为补码，之后再取反，最后转换成原码。使用`Int32`，缺点是仅支持整数，而且整数的最大范围是`2 ** 31 - 1`。

| 运算符 | 名称         | 描述                                                     |
| :----- | :----------- | :------------------------------------------------------- |
| &      | AND          | 如果两位都是 1 则设置每位为 1                            |
| \|     | OR           | 如果两位之一为 1 则设置每位为 1                          |
| ^      | XOR          | 如果两位只有一位为 1 则设置每位为 1                      |
| ~      | NOT          | 反转所有位                                               |
| <<     | 零填充左位移 | 通过从右推入零向左位移，并使最左边的位脱落。             |
| >>     | 有符号右位移 | 通过从左推入最左位的拷贝来向右位移，并使最右边的位脱落。 |
| >>>    | 零填充右位移 | 通过从左推入零来向右位移，并使最右边的位脱落。           |

## 掩码

通过位移的方式，定义一些枚举常量：

```js
const A = 1 << 0; // 0b00000001
const B = 1 << 1; // 0b00000010
const C = 1 << 2; // 0b00000100
```

通过位移定义的一组枚举常量，可以利用位掩码的特性，快速操作这些枚举产量(增加、删除、比较)：

```js
let ABC = A | B | C  // ABC属性合并
let AB = ABC & ~C	 // ABC中剔除C
AB & B === B		 // 验证AB中包含A
AB & C === 0         // 验证AB当中不包含C
A === B				 // A和B相等
lanes & -lanes		 // 分离出所有比特位中最右边的1（分离出最高优先级）
// 原理是：位操作符仅使用补码进行比较，负数补码取反+1，最低位1取反之后是0，再+1通过后面的1进位最低位变成了1，和原先的正数补码进行比较得出结果
```

```ts
//  分离出所有比特位中最左边的1（分离出最高低先级）
function getLowestPriorityLane(lanes: Lanes): Lane {
  // This finds the most significant non-zero bit.
  const index = 31 - clz32(lanes);
  return index < 0 ? NoLanes : 1 << index;
}
```

# 45.`JS`取反是怎么取的？

首先我们先要明白有符号整数，有符号整数的最高位表示的是符号位，0 为正数，1 为负数，例如（为方便理解，使用八位二进制）：

```javascript
// 十进制 => 原码
 3  =>  00000011
-3  =>  10000011
```

然后是**反码**，正整数的反码就是它本身（原码），负整数的反码是在其本身的基础上符号位不变，再把其余各位取反。

```javascript
// 十进制 => 原码 => 反码
 3  =>  00000011  =>  00000011
-3  =>  10000011  =>  11111100
```

最后就是补码了，正整数的补码是它的原码，因此正整数的原码、反码、补码都是一样的；负整数的补码是在反码的基础上加 1 得到的。

```javascript
// 十进制 => 原码 => 反码 => 补码
 3  =>  00000011  =>  00000011  =>  00000011
-3  =>  10000011  =>  11111100  =>  11111101
```

上面这些概念我们理解之后，取反就很好懂了；取反是在补码的基础上进行的，因此取反运算（~n）需要将整数转为补码，之后再取反，最后转换成原码。需要注意的是，正整数的补码取反之后符号位是 1，因此这个取反后的数是一个负整数，我们需要按照负整数计算补码的方式做逆运算得到原码，例如：

```javascript
3  =>  00000011  =>  11111100  =>  11111011  =>  10000100  =>  -4
// 1. 十进制转换成补码（00000011），需要注意正整数反码和补码是它本身
// 2. 对补码进行取反（11111100）
// 3. 把已经取反的补码转换成反码（11111011），补码转换成反码的公式：反码 = 补码 - 1
// 4. 最后把反码逆运算转换成原码（10000100），逆运算的过程是反码的符号位不变其余各位取反
// 5. 此时，结果就是 -4
```

负整数取反：

```javascript
-3  =>  10000011  =>  11111100  =>  11111101  => 00000010 => 2
// 1. 十进制转换成原码（10000011）
// 2. 原码转换成反码（11111100）
// 3. 反码转换成补码（11111101），公式是：补码 = 反码 + 1
// 4. 对反码进行取反（00000010），此时因为取反后的二进制数的符号位为 0，即表明这是一个正整数，上文说过正整数的反码和补码就是它本身，因此最终结果是 2
```

# 45.说一说`JS`位运算符的应用？

> 在使用位运算符之前务必注意使用位运算符存在的`Int32`隐式转换。如果数据超过`2 ** 31 - 1`，不要使用位运算符。

1. 判断整数奇偶

   ```javascript
   let oddOrEven = num => num & 1 === 1 ? 'odd' : 'even'
   ```

2. 返回值`-1`转换为`falsy`

   ```javascript
   //利用： ~-1 = 0
   if(arr.indexof(item) > -1) {
       // TODO
   }
   if(~arr.indexof(item)) {
       // TODO
   }
   ```

3. 整数数字清零

   ```javascript
   let a = 100
   a = a & 0 // a === 0
   ```

4. 整数乘二除二

   ```js
   let a = 2
   console.log(a >> 1) //0010 -> 0001 相当于÷2
   console.log(a << 1) //0010 -> 0100 相当于×2
   ```

# 46.说一说`arguments`？`arguments`的作用？



每个函数都会有一个`Arguments`对象实例`arguments`，`arguments`是一个对象，它的属性是从 `0` 开始依次递增的数字，还有`callee`，`length`和`[Symbol.iterator]`属性。`arguments`引用着函数的实参，可以用数组下标的方式`[]`引用`arguments`的元素。

## 注意点

- 因为`arguments`是类数组，所以他不能**<u>直接调用</u>**数组上的方法，必须通过盗用的方式调用。

- 因为`[Symbol.iterator]`的部署，`arguments`对象是可迭代的（扩展运算符等支持）。



## 关于`callee`和`caller`

`callee`是通过`Arguments`实例对象的`arguments.callee`来访问的，代表被**<u>调用的函数</u>**本身，可以利用它实现方法的递归调用。

`Function`对象的`caller`属性可以指向当前**函数的调用者函数**，如果是当作**顶层的函数**调用的话返回`null`，当调用者函数正在执行时才可访问`Function.caller`，我们可以利用`caller`的特性跟踪函数的调用链。

```javascript
function demo1(){
	console.log(arguments.callee)
}
demo1()  //[Function: demo1]

function demo2(){
	console.log(demo2.caller)
}
function test(){
	demo2()
}
test()  //[Function: test]
```

## 作用

### 方法重载

方法重载是指在一个类中定义多个同名的方法，但要求每个方法具有不同的参数的类型或参数的个数。 `Javascript`并没有重载函数的功能，**但是使用`Arguments`对象根据参数进行逻辑分流能够模拟重载**。

```javascript
function overloadDemo () {
  switch(arguments.length) {
    case 0: 
      console.log(0);
      break;
    case 1: 
      console.log(1);
      break;
    default: 
      console.log(arguments.length);
      break;
  }
}
overloadDemo('name');
```

- **匿名函数**不能通过函数名实现递归调用，必须通过`arguments.callee`实现递归调用（一般情况下不会这么做，可以直接使用命名函数）。
- 用`arguments`修改实参值、限制实参个数、检测参数合法性、获取所有实参、遍历实参。

> 在严格模式下，不允许对`arguments`赋值，`arguments`不再追踪参数的变化（和实参的双向绑定失效），禁止使用`arguments.callee`。

# 47.说一说闭包？

## 定义

> 函数可以**记住并访问**定义时所在的**词法作用域**时，就产生了闭包，**即使函数是在当前（词法）作用域之外执行**。总之，闭包就是一个函数持有对另一个函数的**词法作用域的引用**，也就是**能够记忆化并访问其他函数作用域的一个函数**。

创建闭包的最常见的方式就是**在一个函数内创建另一个函数**，创建的函数对所在函数的词法作用域内的变量进行引用，如果这个函数在创建的**词法作用域之外的地方**被调用那么就产生了闭包。

##### 应用场景：

- 直接返回的函数存在闭包（当且仅当传出的函数对该函数内部词法作用域产生了引用）。

- 函数作为参数 / 回调函数（当且仅当传入的函数对该函数外部某个词法作用域产生了引用）。

- 立即执行函数返回一个函数维持了对执行函数内部作用域的一个引用，形成了闭包（较好的一种封装方法，可以闭包封装来隔离变量）。

- **柯里化**的实现（搜集变量并通过闭包维持引用）。

- 实现**防抖和节流**（通过闭包维持对`timer`或者时间变量的的引用）。

- 实现**模块模式**。

  ```javascript
  const MyModules = (function Manager(){
      const modules = {}
      function define(name, deps, impl) {
          deps.forEach((dep, i, deps) => void (deps[i] = modules[dep]))	//添加依赖
          modules[name] = impl.apply(impl, deps)
      }
      function get(name) {
          return modules[name]
      }
      return {
          define,
          get
      }
  })()
  MyModules.define('bar', [], function(){
      function sayHello(who){
          return (`Hello, ${who}`)
      }
      return {
          sayHello
      }
  })
  MyModules.define('foo', ['bar'], function(bar){
      let fooVar = `foo` 
      function sayHelloUpper(name = fooVar){
          console.log(bar.sayHello(name).toUpperCase())
      }
      return {
          sayHelloUpper
      }
  })
  ```

- 闭包设计**单例模式**。

- 模拟**私有变量**（基本思路与闭包封装隔离变量相似）

  ```javascript
  let sque = (function () {
      let _width = Symbol();
      class Squery {
          constructor(s) {
              this[_width] = s
          }
  
          foo() {
              console.log(this[_width])
          }
      }
      return Squery
  })();
  let ss = new sque(20);
  ss.foo();
  console.log(ss[_width])
  ```

## 闭包变量存在位置

闭包中的变量存储的位置是**堆内存**。假如闭包中的变量存储在栈内存中，那么栈的回收会把处于栈顶的变量自动回收。所以闭包中的变量如果处于栈中那么变量被销毁后（执行上下文），闭包中的变量就没有了。**所以闭包引用的变量是出于堆内存中的。**

## 用途

> 核心用途离不开 **维持引用 + 作用域外的再度访问**

闭包的一个用途是**使已经运行结束的函数上下文中的变量对象继续留在内存**中，因为闭包函数保留了这个变量对象的引用，所以**这个变量对象不会被回收**。

闭包的另一个用途是使我们**在函数外部能够访问到函数内部的变量**。通过使用闭包，可以通过在外部调用闭包函数，从而在外部访问到函数内部的变量，可以使用这种方法来**创建模拟私有变量**。

## 注意点

- 由于闭包会使得**函数中的变量都被保存在内存**中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致**内存泄露**。解决方法是，**在退出函数之前，将不使用的局部变量全部删除**（基本原理就是通过解除对闭包函数的引用，触发闭包函数以及闭包的`GC`）。   
- 闭包会在父函数外部改变父函数内部变量的值。所以，如果你把父函数当作模块使用（模块模式），把闭包当作它的公用方法，把内部变量当作它的私有属性，这时一定要小心，不要随便改变父函数内部变量的值。

# 48.对作用域、作用域链的理解？

> 作用域 & 

[最全面的参考](https://xuoutput.github.io/2019/01/10/js%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE%E5%92%8C%E9%97%AD%E5%8C%85/)

> 所以，**在函数对象被创建的阶段就确定了它的作用域**，作用域保存在函数对象内部的一个`[[Scope]]`属性中。作用域限制了函数内**变量与函数**的**可访问性**：在函数内部申明的属性、函数属于该函数自己的作用域，不会对函数外层的作用域暴露，同时函数内部申明的嵌套函数可以**通过作用域链维持对父级作用域的引用**来获得对父级函数作用域内属性、函数的访问权限。

## 作用域

### 全局作用域

- **任何函数函数**都可以通过作用域链访问**全局作用域**；

- 全局作用域中最重要的是`window`全局对象，它上面的属性全部暴露在全局执行环境中；

- 所以，全局作用域的暴露变量和函数有两种来源：

  - 通过默认绑定机制绑定到全局对象上，由全局对象将自己的所有属性暴露到全局环境；
  - `let`、`const`虽然没有被绑定到全局对象上，但是在创建全局执行环境的时候被保存在全局活动对象上。

- 通过默认绑定行为暴露到全局执行环境的有以下几种情况：

  - `var`声明变量和函数、所有未经声明就直接赋值的变量；
  - 所有未经声明就直接赋值的变量默认绑定到`window`对象上；
  - 全局作用域定义的`for`循环使用`var`声明的循环变量会泄露到全局作用域，原因是括号没有隔离作用域的限制；

  > 过于依赖全局作用域有很大的弊端，因为过多的全局作用域变量会污染全局作用域的命名空间，引起命名冲突。

### 函数作用域

- 定义在函数中的变量、函数就在处在当前函数作用域中。
- 作用域是分层的，内层作用域可以访问外层作用域，外层作用域不可以访问内层作用域。

### 块级作用域

- 使用`ES6`中新增的`let`和`const`指令可以声明块级作用域，块级作用域可以在函数中创建也可以在一个代码块中的创建（由`{ }`包裹的代码片段），使用块级作用域的好出是隔离作用域，限制以块级方式声明的变量仅能在当前域以及子域中被访问，离开了当前域和子域访问都会报错。所以，在循环中比较适合绑定块级作用域，这样就可以把声明的计数器变量限制在循环内部。此外，使用`let`和`const`能够有效减少变量污染全局作用域。

> `let`和`const`声明的变量存在**暂时性死区**，并且**不可以重复声明**。

## 作用域链

> `JavaScript` 中每个函数都都表示为一个**函数对象**，函数对象有一个仅供 `JavaScript` 引擎使用的`[[scope]]` 属性。通过**语法分析和预解析**，将`[[scope]]` 属性**指向函数定义时**作用域中的**所有外部变量对象对象集合**。这个**集合**被称为函数的**作用域链**（`scope chain`），包含函数定义时作用域中所有可访问的数据，它遵循由内而外的一个顺序，确保作用域链的有序访问。
>
> 作用域链的**<u>本质上是一个指向变量对象的指针列表</u>**。**<u>变量对象是一个包含了执行环境中所有变量和函数的对象</u>**。作用域链的前端始终都是**<u>当前执行上下文的变量对象</u>**。全局执行上下文的变量对象始终是**作用域链的最后一个对象**。

函数在被调用时会获取函数实例在创建时就已经确定的内部`[[Scope]]`属性，它保存了函数的作用域链，然后函数会在创建执行上下文构建完成变量对象后，将作用域链的最前端指向这个变量对象。在代码执行阶段访问一个变量，会先在作用域链的顶端也就是当前作用域中查找所需变量，如果该作用域没有这个变量，那这个变量就是**自由变量**。如果在当前作用域找不到该变量就去作用域链中的父级作用域查找，依次向上级作用域查找，直到访问到全局作用域为止，这一层层的关系就是作用域链。作用域链的作用是**<u>保证对执行环境有权访问的所有变量和函数的有序访问</u>**，通过作用域链，可以访问到**<u>父级变量对象的变量和函数</u>**。

# 49.说一说执行上下文？

> [参考](https://juejin.cn/post/6844904158957404167)
>
> [详细参考](https://juejin.cn/post/6844904158957404167)

## 执行上下文

> 注意区分执行上下文 和 上下文，上下文指的是`this`，执行上下文指的是函数执行期间的执行环境，执行环境中包含了上下文`this`。

当 `JS` 引擎解析到可执行代码片段（通常是函数调用阶段）的时候，就会先**做一些执行前的准备工作**，这个 **“准备工作”**，就叫做 **"执行上下文(`execution context` 简称 `EC`)"** 或者也可以叫做**执行环境**。**执行上下文** 为我们的可执行代码块提供了执行前的必要准备工作，例如**变量对象的定义、作用域链的扩展、提供调用者的对象引用**等信息。

当`JavaScript`执行代码时，首先遇到**<u>全局代码</u>**，会创建一个**<u>全局执行上下文</u>**并且压入执行栈中，每当遇到一个函数调用，就会**<u>为该函数创建一个新的执行上下文并压入栈顶</u>**，引擎会**<u>执行位于执行上下文栈顶的函数</u>**，当函数**<u>执行完成</u>**之后，**<u>执行上下文从栈中弹出</u>**，继续执行下一个上下文。当**<u>页面被关闭</u>**时，从栈中**<u>弹出全局执行上下文</u>**。

## 上下文分类

### 全局执行上下文

**这是最外层的执行上下文，一个程序中只会存在一个全局上下文，它在整个  脚本的生命周期内都会存在于执行堆栈的最底部不会被栈弹出销毁。**创建全局执行上下文最重要的是生成一个全局对象（以浏览器环境为例，这个全局对象是 `window`），并且将 `this` 指向这个全局对象上，全局对象的所有属性暴露在全局作用域当中。

### 函数执行上下文

每当一个函数被调用时，都会创建一个新的函数执行上下文压入执行栈中，再调用结束后执行上下文出栈（不管这个函数是不是被重复调用的）。

### `eval`函数执行上下文

执行在`eval`函数中的代码会有属于他自己的执行上下文。因为使用`eval`创建了属于它自己的执行上下文并直接影响到了外部执行上下文的确定，会导致严重的性能问题，不推荐使用。

## 执行上下文包括

1. 变量对象：每个执行环境文都有一个表示变量的对象——变量对象，**<u>全局执行环境的变量对象始终存在</u>**，而**<u>函数这样局部环境的变量，只会在函数执行的过程中存在</u>**，在函数被调用时且在具体的函数代码运行之前，`JS` 引擎会用当前函数的参数列表（`arguments`）初始化一个 “变量对象” 并将当前执行上下文与之关联 ，函数代码块中声明的变量和函数将作为属性添加到这个变量对象上。
2. 活动对象：函数进入**<u>执行阶段</u>**时，原本不能访问的**<u>变量对象被激活成为一个活动对象</u>**，自此，我们可以访问到其中的各种属性。**变量对象和活动对象是一个东西，只不过处于不同的状态和阶段而已**。
3. 作用域链：函数的作用域在函数创建时就已经确定了。当函数创建时，会有一个名为 `[[scope]]` 的内部属性保存所有父变量对象到其中。当函数执行时，会创建一个执行环境，然后通过复制函数的 `[[scope]]`  属性中的对象构建起执行环境的作用域链，然后，变量对象 `VO` 被激活生成 `AO` 并**添加到作用域链的前端**，完成作用域链创建完成。
4. 调用者信息（`this`）：如果当前函数被作为对象方法调用或使用 `bind` `call` `apply` 等 `API` 进行委托调用，则将当前代码块的调用者信息（`this value`）存入当前执行上下文，否则默认为全局对象调用。

## 创建流程

### 函数执行上下文

- 创建阶段

  - 先生成变量对象`VO`：
    - 创建`arguments`
    - 扫描函数声明（同名函数会覆盖）
    - 扫描变量声明（变量声明会覆盖）

  - 获取函数实例上的的作用域链`[[Scope]]`，再将**作用域链指针前端指向当前变量对象**，实现作用域的有序访问
  - 最后确定`this`调用者的指向；

- 执行阶段（`VO`->`AO`）

  - 变量赋值；
  - 执行其他代码；

- 销毁阶段
  - 当前执行上下文出栈、销毁（闭包变量进入堆内存）。

### 全局执行上下文

- 创建阶段
  - 生成变量对象`VO`
    - 初始化内置对象（`Date`、`Math`……）
    - 扫描函数声明（同名函数会覆盖）
    - 扫描变量声明（变量声明会覆盖）
  - 将作用域指向生成的变量对象
  - 确定全局执行上下文中`this`指针的指向，即全局对象
- 代码执行阶段
  - 变量赋值
  - 执行其他代码
- 销毁阶段
  - 全局执行上下文出栈、销毁

## 作用域和上下文的区别

`JS`解释阶段便会确定解释规则  ，因此**作用域在函数被实例化时就已经确定了，而不是在函数被调用时确定**。但是，执行上下文中的上下文`this`是在函数调用阶段确定，`this`的含义就是调用方，是在调用函数的时候指派的。

# 50.ES5中的执行上下文

[参考链接](https://github.com/daydaylee1227/Blog/blob/master/articles/JS/%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%20%E6%89%A7%E8%A1%8C%E6%A0%88.md)

`ES5` 规范又对 `ES3` 中执行上下文的部分概念做了调整，最主要的调整，就是去除了 `ES3` 中变量对象和活动对象，以 **词法环境组件（** **`LexicalEnvironment component`）** 和 **变量环境组件（** **`VariableEnvironment component`）** 替代。所以 `ES5` 的执行上下文概念上表示大概如下：

```js
ExecutionContext = {
  ThisBinding = <this value>,
  LexicalEnvironment = { ... },
  VariableEnvironment = { ... },
}
```

## ES5 中的词法环境

词法环境定义：

> **词法环境**是一种规范类型，基于 ECMAScript 代码的词法嵌套结构来定义**标识符**和具体变量和函数的关联。一个词法环境由环境记录器和一个可能的引用**外部词法环境**的空值组成。

简单来说 **词法环境** 是一种持有 **标识符—变量映射** 的结构。这里的 **标识符** 指的是变量/函数的名字，而 **变量** 是对实际对象（包含函数类型对象）或原始数据的引用。

> 这块看不懂没关系，你可以把它理解为 ES3 中的 变量对象，因为它们本质上做的是类似的事情，这里只是先把官方给出的定义放上来。这块概念比较烦：词法环境还分为两种，然后内部有个环境记录器还分两种，，这些概念在后面会用列表的形式归纳整理出来详细说明。

## ES5 中的变量环境

**变量环境** 它也是一个 **词法环境** ，所以它有着词法环境的所有特性。

之所以在 `ES5` 的规范力要单独分出一个变量环境的概念是为 `ES6` 服务的： 在 `ES6` 中，**词法环境**组件和 **变量环境** 的一个不同就是前者被用来存储函数声明和变量（`let` 和 `const`）绑定，而后者只用来存储 `var` 变量绑定。

> 在上下文创建阶段，引擎检查代码找出变量和函数声明，**变量最初会设置为 undefined（var 情况下），或者未初始化（let 和 const 情况下）**。这就是为什么你可以在声明之前访问 var 定义的变量（虽然是 undefined），但是在声明之前访问 let 和 const 的变量会得到一个引用错误。

## ES5 执行上下文总结

对于 `ES5` 中的执行上下文，我们可以用下面这个列表来概括程序执行的整个过程：

1. 程序启动，全局上下文被创建

   1. 创建全局上下文的词法环境

      1. 创建 **对象环境记录器** ，它用来定义出现在 **全局上下文** 中的变量和函数的关系（负责处理 `let` 和 `const` 定义的变量）
      2. 创建 **外部环境引用**，值为 **`null`**
      
   2. 创建全局上下文的变量环境

      1. 创建 **对象环境记录器**，它持有 **变量声明语句** 在执行上下文中创建的绑定关系（负责处理 `var` 定义的变量，初始值为 `undefined` 造成声明提升）
      2. 创建 **外部环境引用**，值为 **`null`**
      
   3. 确定 `this` 值为全局对象（以浏览器为例，就是 `window` ）
   
2. 函数被调用，函数上下文被创建

   1. 创建函数上下文的 

      词法环境

      1. 创建  **声明式环境记录器** ，存储变量、函数和参数，它包含了一个传递给函数的 **`arguments`** 对象（此对象存储索引和参数的映射）和传递给函数的参数的 **length**。（负责处理 `let` 和 `const` 定义的变量）
      2. 创建 **外部环境引用**，值为全局对象，或者为父级词法环境（作用域）

   2. 创建函数上下文的 

      变量环境

      1. 创建  **声明式环境记录器** ，存储变量、函数和参数，它包含了一个传递给函数的 **`arguments`** 对象（此对象存储索引和参数的映射）和传递给函数的参数的 **length**。（负责处理 `var` 定义的变量，初始值为 `undefined` 造成声明提升）
      2. 创建 **外部环境引用**，值为全局对象，或者为父级词法环境（作用域）

   3. 确定 `this` 值

3. 进入函数执行上下文的执行阶段：

   1. 在上下文中运行/解释函数代码，并在代码逐行执行时分配变量值。
   
      [^]: 
   
      
