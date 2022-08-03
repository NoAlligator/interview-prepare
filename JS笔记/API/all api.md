> Number

```javascript
⭐Number.parseInt(string[, radix]): (number | NaN) //存在toString()隐式转换
⭐Number.parseFloat(string) //存在toString()隐式转换

//返回一个用幂的形式 (科学记数法) 来表示Number 对象的字符串
numObj.toExponential(fractionDigits): string
numObj.toFixed(digits): string //返回保留了digits个小数点后数字（四舍五入，但有精度问题）的string
numObj.toPrecision(precision): string //第一个非零数字开始计算位数，四舍五入到precision参数指定的显示数字位数。
numObj.toString([radix])

Number.isFinite(value): boolean
Number.isInteger(value): boolean
Number.isNaN(value): boolean
Number.isSafeInteger(testValue): boolean
```

> String

```javascript
//获取指定索引值
str.charAt(index)

//获取生成指定索引值对应编码，按照编码生成字符串
str.charCodeAt(index) :number //utf-16
String.fromCharCode(num1[, ...[, numN]]): string
str.codePointAt(index) :number //unicode
String.fromCodePoint(num1[, ...[, numN]]): string

//拼接字符串
str.concat(str2, [, ...strN]): string //可以配合扩展运算符，强烈建议使用赋值操作符（+, +=）代替 concat 方法。

//校验字符串
str.endsWith(searchString[, length]): boolean
str.startsWith(searchString[, position]): boolean
⭐str.includes(searchString[, position]): boolean


//查找指定字符串的索引
//指明fromIndex为负数则视同0，未找到的话 === -1
⭐str.indexOf(searchValue [, fromIndex]): number
//指明fromIndex为负数则视同0，未找到的话 === -1
⭐str.lastIndexOf(searchValue[, fromIndex]): number
⭐str.search(regexp): number // 首次匹配项的索引（隐式regex转换）

//匹配
str.match(regexp) :array | null // 加了g返回匹配数组，不加g返回一个完整匹配及其相关的捕获组
str.matchAll(regexp): iterator // 必须是加g，返回完整匹配及其相关的捕获组的迭代器

//填充和替换
str.padEnd(targetLength [, padString]): string
str.padStart(targetLength [, padString]): string
str.replace(regexp|substr, newSubStr|function)
str.replaceAll(regexp|substr, newSubstr|function)

//重复
str.reapeat(count): string

//裁剪
str.slice(beginIndex[, endIndex]): string //截取范围：[beginIndex, endIndex)，支持负索引
str.substring(indexStart[, indexEnd]): string //截取范围：indexStart[, indexEnd)，不支持负索引

//转数组
str.split([separator[, limit]]): array //加了limit之后忽略后面的

//大小写转换
str.toUpperCase()
str.toLowerCase()

//去除空白
str.trim()
str.trimEnd()
str.trimStart()

strObj.toString()
strObj.valueOf()
```



> RegExp

```javascript
regexObj.exec(str)
regexObj.test(str)
regexObj.toString() // /ab/g -> '/ab/g'
```



> Array

```javascript
Array.isArray()

//带callback
arrary.forEach(callback(element[, index[, array]])[, thisArg])

arrary.every(callback(element[, index[, array]])[, thisArg])
arrary.some(callback(element[, index[, array]])[, thisArg])

arrary.find(callback(element[, index[, array]])[, thisArg])
arrary.findIndex(callback(element[, index[, array]])[, thisArg])

arrary.map(callback(element[, index[, array]])[, thisArg])
array.filter(callback(element[, index[, array]])[, thisArg])

array.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
array.reduceRight(callback(accumulator, currentValue[, index[, array]])[, initialValue])

//数组操作，针对引用地址
arr.push(element1, ..., elementN): number
array.pop()
arr.unshift(element1, ..., elementN)
array.shift()
//默认是转换成字符串utf-16进行比较，当且仅当compareFunction(a, b)返回值大于0时替换a, b位置
array.sort([compareFunction])  //sort就地排序，不复制
array.reverse() //reverse就地反转，不复制
array.splice(start[, deleteCount[, item1[, item2[, ...]]]])//返回剪切的数组

//数组的剪切（复制）
array.slice([begin[, end]) //[begin, end)，可为负值


//生成数组的遍历器
array.keys()
array.values()
array.entries()

//查找数组元素
array.includes(valueToFind[, fromIndex]): boolean //支持负值
array.indexOf(searchElement[, fromIndex]): number //支持负值
array.lastIndexOf(searchElement[, fromIndex]): number //支持负值

//拼接
array.concat(value1[, value2[, ...[, valueN]]])

//转成字符串
array.join([separator]) :string //形成以separator为分隔的字符串
array.toString(): string //形成以','为分隔的字符串

//展开
array.flat([depth]) //默认1层，返回修改后的数组
arr.flatMap(function callback(currentValue[, index[, array]]) {
    // return element for new_array
}[, thisArg]) //相当于map+flat，返回修改后的数组

//填充
arr.fill(value[, start[, end]]) //[start, end)，并且索引存在才可以fill，返回fill后的数组
arr.copyWithin(target[, start[, end]]) //浅复制数组的一部分到同一数组中的另一个位置，并返回它，不会改变原数组的长度。


//兼容了arrayLike，支持map回调
Array.from(arrayLike[, mapFn[, thisArg]])

// new Array()方式有一个缺陷，不支持生成单个元素数组：new Array(3)，这样只会形成长度为3的数组。
// Array.of(3)则支持生成[3]
Array.of(element0[, element1[, ...[, elementN]]])

//新特性！！！
array.at(index)//支持负索引
```

> 数组遍历速度对比
>
> [参考](https://juejin.cn/post/6844903538175262734)
>
> ⭐`for`>` for-of `>` forEach` > `filter` > `map` > `for-in`



> Symbol

```javascript
const symbol = Symbol(key) //创建一个Symbol（非全局注册的）

symbol.toString() //"Symbol(keyValue)"
symbol.valueOf() //获取原始值
⭐Symbol.for(key) //全局注册的Symbol（先搜索现有的symbol，没有则创建后返回）
⭐Symbol.keyFor(key) //查找某个全局Symbol的key值
```



> Math

```javascript
Math.abs(number)
Math.ceil(number)
Math.floor(number)
Math.max(...numbers)
Math.min(...numbers)
Math.pow(number, e)
Math.round(number)
Math.sign(number)
Math.sqrt(number)
Math.cbrt(number) //立方根
⭐Math.trunc(x) //整数部分
Math.random()

acos
acosh
asin
asinh
atan
atanh
atan2
sin
cos
tan
sinh
cosh
tanh

log
log1p
log2

exp
expm1

clz32
fround
imul
```



> Object

```javascript
//原型相关
Object.getPrototypeOf(object)
prototypeObj.isPrototypeOf(object)
Object.setPrototypeOf(obj, prototype)

//属性定义
Object.defineProperty(obj, prop, descriptor)
Object.defineProperties(obj, props)
Object.getOwnPropertyDescriptor(obj, prop)
Object.getOwnPropertyDescriptors(obj)
obj.propertyIsEnumerable(prop)
obj.hasOwnProperty(prop)

//混入其他对象到目标对象
Object.assign(target, ...sources): target

//以对象作为__proto__创建对象
Object.create(proto，[propertiesObject])

//从entries数组创建对象
Object.fromEntries(iterable)

//对象iterator
Object.keys()
Object.values()
Object.entries()

//获取键值数组
Object.getOwnPropertyNames(obj)
Object.getOwnPropertySymbols(obj)

//“更好”的判断
Object.is(value1, value2)

//限制修改对象
Object.preventExtensions(obj)
Object.isExtensible(obj)

Object.freeze(obj)
Object.isFrozen(obj)

Object.seal(obj)
Object.isSealed(obj)

//转字符串
Object.toString()
Object.valueOf()
```

