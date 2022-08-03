# 1.React前置知识

## React核心库

1. `react.js`：React核心库。
2. `react-dom.js`：提供操作DOM的react扩展库。
3. `babel.min.js`：解析JSX语法代码转为JS代码的库。

### 为什么要区分react-dom和react？

react 包即是抽象逻辑，它包含了 React 的主干逻辑。例如组件实现、更新调度等。react-dom 顾名思义就是一种针对 dom 的平台实现，主要用于在 web 端进行渲染。而声名在外的 react-native 则是原生应用实现，可以通过 react-native 内部的相应机制与操作系统进行通信来调用原生控件进行渲染。这是一个**依赖倒置**原则的典型应用，**高层模块不应依赖底层模块的具体实现**。简单来说，就是我们只需要将组件按一定的形式编写好，而最终 render 函数是以怎样的机制将 JSX 渲染到页面上的，我们不需要关心，只要这个机制同样遵循我编写组件所依赖的规则就好——这个规则就是 react.js。

## JSX

### 全称

JavaScript XML

### 本质

react定义的一种类似于XML的JS扩展语法: JS + XML。本质是`React.createElement(component, props, ...children)`方法的语法糖。

### 作用

用来简化创建虚拟DOM的过程。

### 转义

浏览器不能直接解析JSX代码, 需要babel转译为纯JS的代码才能运行，只要用了JSX，都要加上type="text/babel"，声明需要babel来处理。

### 语法规范

1. 定义虚拟DOM时，**不要写引号**。
2. 标签中**混入JS表达式时要用{}**。
3. 样式的**类名**指定不要用class，**要用className**；for 属性 改为 htmlFor；colspan 属性 改为 colSpan。也即所有的属性名遵循JS使用**小写驼峰命名法**。但是无障碍阅读属性aria-*除外。
4. 内联样式，要用style={ ({key: value}) }的形式去写。
5. 单个组件只有一个根标签。
6. 所有标签必须（自）闭合。
7. 标签首字母。
   1. 若**小写字母**开头，则将该标签转为html中**同名元素**，若html中无该标签对应的同名元素，则报错。
   2. 若**大写字母**开头，react就去渲染对应的**组件**，若组件没有定义，则**报错**。

## 语句和表达式的区别

```javascript
/*
一定注意区分：【js语句(代码)】与【js表达式】
1.表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方
    下面这些都是表达式：
            (1). a
            (2). a+b
            (3). demo(1)
            (4). arr.map()
            (5). function test () {}
2.语句(代码)：
    下面这些都是语句(代码)：
            (1).if(){}
            (2).for(){}
            (3).switch(){case:xxxx}
*/
```



# 2.什么是纯函数？纯函数与React的关系？

> 纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。

它的**重点在于“相同的输入，永远会得到相同的输出”**，后面所说的**副作用**也是为了满足这一点。

## 纯函数的必要条件

- 纯函数接受参数，基于参数计算，返回一个新对象；
- 不会产生副作用，计算过程不会修改输入的参数并且不修改其作用域之外的参数或方法；
- 相同的输入保证相同的输出。

## 纯函数的优点

- 相同的输入必定是相同的输出，所以纯函数可以**根据输入来做缓存**；
- 相同的输入必定是相同的输出，这就保证了**引用的透明性**；
- 纯函数**完全自给自足**，这点的好处就是纯函数的**依赖很明确**，因此更易于观察和理解，而且让我们的测试更加容易；
- 可靠：不用担心有**副作用**，可以更好的工作；
- 代码并行：可以**并行运行任意纯函数**，因为纯函数不需要访问共享的内存，而且也不会因为副作用而进入竞争态。

## 纯函数与React

> React官方原文：所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。

[一文带你了解React纯函数组件](https://blog.csdn.net/qq_42033567/article/details/113792471)

如果一个组件没有状态（`state`），那么组件的输出方式，将完全取决于两个参数：`props` 和 `context`，只要有相同的 `props` 和 `context` ，那么他们的输出绝对是相同的。将组件比喻成函数的话，相同的输入(`props` 和 `context`) 永远都会有相同的输出：

```javascript
render(){
    return this.props.sayHi ? <div>Hi</div> : <span>Byebye</span>
}
```

如上面代码所示，props 是输入，只要输入相同，那么输出也一定相同。

### 使用纯函数创建组件

```javascript
// function
function Title (props) {
  return <h1>{ props.title }</h1>
}
// 箭头函数
const Title = ({ props }) => <div>{ props.title }</div>
```

### 使用类组件方式创建的组件

```javascript
// es6 类组件
class Title extends React.Component {
  render() {
    return <h1>{this.props.title}</h1>
  }
}
```

### 对比得出纯函数组件的特点

- 组件不会被实例化，整体渲染性能得到提升；
- 组件不能访问 this 对象；
- 组件无法访问生命周期的方法；
- 无状态组件**只能访问输入的 props，无副作用**。

### 纯函数组件的优点

- 无副作用；
- 占内存更小，首次 render 的性能更好；
- 语法更简洁，可读性好，逻辑简单，测试简单，代码量少，容易复用；
- 更佳的性能表现：因为函数组件中不需要进行生命周期的管理和状态管理，因此 React 并不需要进行某些特定的检查和内存分配，保证了性能。

### 纯函数组件的缺点

无生命周期，且没有 this。

### 使用场景

纯函数组件被鼓励在大型项目中尽可能以简单的写法来分割原本庞大的组件，未来 React 也会像面向无状态组件一样在譬如无意义的检查和内存分配领域进行一系列优化，所以**只要有可能，尽量使用无状态组件**。纯函数不会产生不可预料的行为，建议合理的选择纯函数的方式书写函数。同样，在 React 组件中，如果无需本地 state 去缓存一些数据，也不需要用到生命周期函数，那么就可以把当前组件定义为纯函数组件，可读性好，且性能表现更佳。

# 3.`setState()`的用法？

> 官方文档原文：
>
> 出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。因为 `this.props` 和 `this.state` 可能会异步更新，所以你**不要依赖他们的值来更新下一个状态**。将 `setState()` 视为**请求而不是立即更新组件的命令**。为了更好的**感知性能**，React 会延迟调用它，然后**通过一次传递更新多个组件。**React 并不会保证 state 的变更会立即生效。
>
> `setState()` 并不总是立即更新组件，它会批量推迟更新。这使得在调用 `setState()` 后立即读取 `this.state` 成为了隐患。**为了消除隐患，请使用 `componentDidUpdate` 或者 `setState` 的回调函数（`setState(updater, callback)`）**，这两种方式都可以保证在应用更新后触发。如需基于之前的 state 来设置当前的 state，请阅读下述关于参数 `updater` 的内容。
>
> 除非 `shouldComponentUpdate()` 返回 `false`，否则 `setState()` 将始终执行重新渲染操作。如果可变对象被使用，且无法在 `shouldComponentUpdate()` 中实现条件渲染，那么仅在新旧状态不一时调用 `setState()`可以避免不必要的重新渲染。
>
> 参数一的第一种形式为带有形式参数的 `updater` 函数：
>
> ```javascript
> (state, props) => stateChange
> ```
>
> `state` 是对应用变化时组件状态的引用。**形参state不应直接被修改**，你应该**使用基于 `state` 和 `props` 构建的新对象来表示变化**。例如，假设我们想根据 `props.step` 来增加 state：
>
> ```javascript
> this.setState((state, props) => {
> 	return {counter: state.counter + props.step};
> });
> ```
>
> updater 函数中接收的 `state` 和 `props` 都保证为最新。updater 的返回值会与 `state` 进行浅合并。
>
> `setState()` 的第二个参数为可选的回调函数，它将在 `setState` 完成合并并重新渲染组件后执行。通常，我们**建议使用 `componentDidUpdate()` 来代替此方式**。
>
> `setState()` 的第一个参数除了接受函数外，还可以接受对象类型：
>
> ```javascript
> setState(stateChange[, callback])
> ```
>
> `stateChange` 会将传入的对象浅层合并到新的 state 中，例如，调整购物车商品数：
>
> ```javascript
> this.setState({quantity: 2})
> ```
>
> 这种形式的 `setState()` 也是异步的，并且在同一周期内会对多个 `setState` 进行批处理。例如，如果**在同一周期内多次设置商品数量增加**，则相当于：
>
> ```js
> Object.assign(
>     previousState,
>     {quantity: state.quantity + 1},
>     {quantity: state.quantity + 1},
>     ...
> )
> ```
>
> **后调用的 `setState()` 将覆盖同一周期内先调用 `setState` 的值，因此商品数仅增加一次。**如果**后续状态取决于当前状态**，我们**建议使用 updater 函数的形式代替**：
>
> ```js
> this.setState((state) => {
> 	return {quantity: state.quantity + 1};
> });
> ```
>
> `React`的`setState`如果是被`React`合成事件触发的，那么就`React`就会将所有的更新`Update`放入一个更新队列，并在`render`阶段会针对所有的更新进行批处理，表现出的特征就是：当我们传入参数进行更新时，更新的特性类似于`Object.assign`，所以针对同一个属性的多次扩展拿到的都是一样的值进行更新，而不是我们预期中的依次获取最新值进行更新，但是使用回调形式的更新则不同，因为使用回调形式的更新必须传入当前最新的状态进行一次更新后再混入原来的状态对象，所以可以确保每一次拿到的都是最新的值，也就是前面所说的一次执行。如果想要获取这一轮更新调度完成后的值，我们可以使用`setState`的第二个回调，或者在`ComponentDidUpdate`中获取。

# 4.说一说React生命周期？

## 旧`React`生命周期

![img](https://github.com/NoAlligator/pico/blob/main/img/e12b2e35c8444f19b795b27e38f4c149~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

![2_react生命周期(旧)](https://github.com/NoAlligator/pico/blob/main/img/2_react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F(%E6%97%A7?raw=true).png?raw=true)

### 挂载

- constructor：类式组件构造函数
- componentWillMount：组件即将被挂载到页面的时刻自动执行
- render：组件渲染
- componentDidMount：组件已经被挂载到页面的时刻自动执行

### 更新

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

### 卸载

- componentWillUnmount

#### `constructor()`

constructor中通常只做两件事情：

1. 通过给 this.state 赋值对象来初始化内部的 state ；
2. 为事件绑定实例，也就是 this 。

#### `UNSAFE_componentWillMount()`

在此方法中**同步调用 `setState()` 不会触发额外渲染**。通常，我们建议使用 `constructor()` 来初始化 state。**避免在此方法中引入任何副作用或订阅。**如遇此种情况，请改用 `componentDidMount()`。此方法是服务端渲染唯一会调用的生命周期函数，建议使用`constructor`替代。

#### `UNSAFE_componentWillReceiveProps(nextProps)`

> 使用此生命周期方法通常会出现 bug 和不一致性：
>
> - 如果你需要**执行副作用**（例如，数据提取或动画）以响应 props 中的更改，请改用 [`componentDidUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate) 生命周期。
> - 如果你使用 `componentWillReceiveProps` **仅在 prop 更改时重新计算某些数据**，请[使用 memoization helper 代替](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。
> - 如果你使用 `componentWillReceiveProps` 是为了**在 prop 更改时“重置”某些 state**，请考虑使组件[完全受控](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或[使用 `key` 使组件完全不受控](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) 代替。
>
> 对于其他使用场景，[请遵循此博客文章中有关派生状态的建议](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)。

`UNSAFE_componentWillReceiveProps()` 会在已挂载的组件接收新的 props 之前被调用。如果你需要更新状态以响应 prop 更改（例如，重置它），你可以比较 `this.props` 和 `nextProps` 并在此方法中使用 `this.setState()` 执行 state 转换。请注意，**如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。**如果只想处理更改，请确保进行当前值与变更值的比较。**在[挂载](https://zh-hans.reactjs.org/docs/react-component.html#mounting)过程中，React 不会针对初始 props 调用 `UNSAFE_componentWillReceiveProps()`。**组件只会在组件的 props 更新时调用此方法。调用 `this.setState()` 通常不会触发 `UNSAFE_componentWillReceiveProps()`。

#### `shouldComponentUpdate(nextProps, nextState): boolean`

根据 `shouldComponentUpdate()` 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响。**默认行为是 state 每次发生变化组件都会重新渲染**。**大部分情况下，你应该遵循默认行为**。当 props 或 state 发生变化时，`shouldComponentUpdate()` 会在渲染执行之前被调用。返回值默认为 true。首次渲染或使用 `forceUpdate()` 时不会调用该方法。此方法仅作为**[性能优化的方式](https://zh-hans.reactjs.org/docs/optimizing-performance.html)**而存在。不要企图依靠此方法来“阻止”渲染，因为这可能会产生 bug。你应该**考虑使用内置的 [`PureComponent`](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent) 组件**，而不是手动编写 `shouldComponentUpdate()`。`PureComponent` 会对 props 和 state 进行浅层比较，并减少了跳过必要更新的可能性。如果你一定要手动编写此函数，可以将 `this.props` 与 `nextProps` 以及 `this.state` 与`nextState` 进行比较，并返回 `false` 以告知 React 可以跳过更新。请注意，**返回 `false` 并不会阻止子组件在 state 更改时重新渲染。**我们**不建议在 `shouldComponentUpdate()` 中进行深层比较或使用 `JSON.stringify()`。这样非常影响效率，且会损害性能。**目前，如果 `shouldComponentUpdate()` 返回 `false`，则不会调用 [`UNSAFE_componentWillUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#unsafe_componentwillupdate)，[`render()`](https://zh-hans.reactjs.org/docs/react-component.html#render) 和 [`componentDidUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate)。后续版本，React 可能会将 `shouldComponentUpdate` 视为提示而不是严格的指令，并且，当返回 `false` 时，仍可能导致组件重新渲染。

#### `UNSAFE_componentWillUpdate(nextProps, nextState)`

当组件收到新的 props 或 state 时，会在渲染之前调用 `UNSAFE_componentWillUpdate()`。使用此作为在更新发生之前执行准备更新的机会。初始渲染不会调用此方法。注意，**你不能此方法中调用 `this.setState()`；在 `UNSAFE_componentWillUpdate()` 返回之前，你也不应该执行任何其他操作（例如，dispatch Redux 的 action）触发对 React 组件的更新。**通常，此方法可以替换为 `componentDidUpdate()`。如果你在此方法中读取 DOM 信息（例如，为了保存滚动位置），则可以将此逻辑移至 `getSnapshotBeforeUpdate()` 中。

#### `componentDidUpdate(prevProps, prevState, snapshot)`

这一个生命周期是最常用也是最容易发生死循环的，应当注意该生命周期内会触发更新的条件，**考察其是否会导致死循环。**

`componentDidUpdate()` 会在更新后会被立即调用。首次渲染不会执行此方法。**当组件更新后，可以在此处对 DOM 进行操作（代替`setState()`的回调形式）。**如果你**对更新前后的 props 进行了比较，也可以选择在此处进行网络请求**。（例如，当 props 未发生变化时，则不会执行网络请求）。你也可以在 `componentDidUpdate()` 中**直接调用 `setState()`**，但请注意**它必须被包裹在一个条件语句里**，正如上述的例子那样进行处理，否则会**导致死循环**。它还会导致额外的重新渲染，虽然用户不可见，但会影响组件性能。**不要将 props “镜像”给 state，请考虑直接使用 props。** 欲了解更多有关内容，请参阅[为什么 props 复制给 state 会产生 bug](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)。

#### `componentWillUnmount()`

`componentWillUnmount()` 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 `componentDidMount()` 中创建的订阅等。`componentWillUnmount()` 中**不应调用 `setState()`**，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。

## 新`React`生命周期

![3_react生命周期(新)](https://github.com/NoAlligator/pico/blob/main/img/3_react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F(%E6%96%B0?raw=true).png?raw=true)

![image-20220122130034632](https://github.com/NoAlligator/pico/blob/main/img/image-20220122130034632.png?raw=true)

### 注意点：

三个阶段

- Render阶段
- Pre-commit阶段
- Commit阶段

#### `static getDerivedStateFromProps(nextProps, nextState)`

`getDerivedStateFromProps` 会在调用render方法之前调用，并且在**初始挂载及后续更新时都会被调用**。它应返回一个对象来更新 state，如果返回 `null` 则不更新任何内容。此方法适用于[罕见的用例](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state)，**即 state 的值在任何时候都取决于 props。**例如，实现 `<Transition>` 组件可能很方便，该组件会比较当前组件与下一组件，以决定针对哪些组件进行转场动画。派生状态会导致代码冗余，并使组件难以维护。 [确保你已熟悉这些简单的替代方案：](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

- 如果你需要**执行副作用**（例如，数据提取或动画）以响应 props 中的更改，请改用 [`componentDidUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate)。
- 如果只想在 **prop 更改时重新计算某些数据**，[请使用 memoization helper 代替](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。
- 如果你想**在 prop 更改时“重置”某些 state**，请考虑使组件[完全受控](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或[使用 `key` 使组件完全不受控](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) 代替。

此方法**无权访问组件实例**。如果你需要，可以通过提取组件 props 的纯函数及 class 之外的状态，在`getDerivedStateFromProps()`和其他 class 方法之间重用代码。请注意，不管原因是什么，都会在**每次渲染前触发此方法**。这与 `UNSAFE_componentWillReceiveProps` 形成对比，后者**仅在父组件重新渲染时触发**，而不是在内部调用 `setState` 时。

##### 它可能的使用场景有两个：

- 无条件的根据 prop 来更新内部 state，也就是只要有传入 prop 值， 就更新 state
- 只有 prop 值和 state 值不同时才更新 state 值。

##### `getDerivedStateFromProps` 方法使用的注意点：

- 在使用此生命周期时，**要注意把传入的 prop 值和之前传入的 prop 进行比较。**
- 因为这个生命周期是静态方法，同时**要保持它是纯函数，不要产生副作用**。

上述的情况在大多数情况下都是适用，但是这边还是会有产生 bug 的风险。具体可以官网提供这个[例子](https://link.juejin.cn?target=https%3A%2F%2Fcodesandbox.io%2Fs%2Fmz2lnkjkrx)。在 One 和 Two 的默认账号都相同的情况下，使用同一个输入框组件，在切换到 Two，并不会显示成 Two 的默认账号。

##### 解决方法有四种：

- 第一种是将组件改成完全可控组件（也是状态值和方法全由父类控制）；
- 第二种是改成完全不可控组件（也就是组件不接受在 getDerivedStateFromProps 中通过 prop 值来改变内部状态），然后通过设置在构造函数中把 prop 传给 state 和设置 key 值来处理，因为 key 变化的时候 React 会重新渲染组件，而不是去更新组件。
- 第三种还是保持上述组件模式，然后通过一个唯一 ID 来判断是否更新，而不是通过 color 值来判断。
- 第四种不使用 getDerivedStateFromProps，通过 ref 来把改变邮箱的方法暴露出去。

##### 反模式

常见的反模式有两种，上边也有提到过。

- 无条件地根据 prop 值来更新 state 值；
- 当 prop 值变化并且和 state 不一样时就更新 state （会造成内部变化无效，上述也提到过）。

#### 总结

我们应该谨慎地使用 getDerivedStateFromProps 这个生命周期。我个人使用情况来说，使用时要注意下面几点：

- 因为这个生命周期是静态方法，同时要保持它是纯函数，不要产生副作用。
- 在使用此生命周期时，要注意把传入的 prop 值和之前传入的 prop 进行比较（这个 prop 值最好有唯一性，或者使用一个唯一性的 prop 值来专门比较）。
- 不使用 getDerivedStateFromProps，可以改成组件保持完全不可控模式，通过初始值和 key 值来实现 prop 改变 state 的情景。

[React 中 getDerivedStateFromProps 的用法和反模式](https://juejin.cn/post/6844903760305602568)

#### `getSnapShotBeforeUpdate(prevProps, prevState)`

`getSnapshotBeforeUpdate()` 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期方法的任何返回值将作为参数传递给 `componentDidUpdate()`。此用法并不常见，但它可能出现在 UI 处理中，如需要以特殊方式处理滚动位置的聊天线程等。**应返回 snapshot 的值（或 `null`）。**

#### `componentDidUpdate(prevProps, prevState, snapShot)`



# 5.为什么获取数据要在`componentDidMount()`中进行？

## 区别

componentWillMount获取数据：

1. 执行willMount函数，等待数据返回
2. 执行render函数
3. 执行didMount函数
4. 数据返回， 执行render

didMount获取数据：

1. 执行willMount函数
2. 执行render函数
3. 执行didMount函数， 等待数据返回
4. 数据返回，执行render

## 原因

1. 如果使用**服务端渲染**的话，willMount会在服务端和客户端各自执行一次，这会导致请求两次（接受不了~），而didMount只会在客户端进行。如果你想在服务端渲染时先完成数据的展示再一次性给用户，官方的推荐做法是用constructor代替willMount；
2. 在Fiber之后， 由于任务可中断，**willMount可能会被执行多次**(fiber算法是异步渲染。异步的渲染，很可能因为**高优先级任务的出现而打断现有任务导致componentWillMount就可能执行多次**)；
3. willMount**会被废弃**，目前被标记为不安全；
4. **节省的时间非常少**，跟其他的延迟情况相比，这个优化可以使用九牛一毛的形容（为了这么一点时间而一直不跟进技术的发展，得不偿失），并且render函数是肯定比异步数据到达先执行，白屏时间并不能减少。

# 6.为什么要改变生命周期？

## 原因

1. 从上面的生命周期的图中可以看出，被废弃的三个函数都是在render之前**，因为fiber的出现，很可能因为高优先级任务的出现而打断现有任务导致它们会被执行多次**。
2. 另外的一个原因则是，React想约束使用者，好的框架能够让人不得已写出容易维护和扩展的代码，这一点又是从何谈起，我们可以从新增加以及即将废弃的生命周期分析入手。

#### `componentWillMount（）`

首先这个函数的功能**完全可以使用componentDidMount和constructor来代替**，异步获取的数据的情况上面已经说明了，而如果抛去异步获取数据，其余的即是初始化而已，这些功能都可以在constructor中执行，除此之外，**如果我们在willMount中订阅事件，但在服务端这并不会执行willUnMount事件，也就是说服务端会导致内存泄漏。**所以componentWillMount完全可以不使用，但使用者有时候难免因为各种各样的情况（如作者犯浑）在componentWillMount中做一些操作，那么React为了约束开发者，干脆就抛掉了这个API。

#### `componentWillReceiveProps()`

> 在老版本的 React 中，如果组件自身的**某个 state 跟其 props 密切相关**的话，一直都没有一种很优雅的处理方式去更新 state，而是需要**在 componentWillReceiveProps 中判断前后两个 props 是否相同，如果不同再将新的 props 更新到相应的 state 上去。**这样做一来**会破坏 state 数据的单一数据源，导致组件状态变得不可预测，另一方面也会增加组件的重绘次数。**类似的业务需求也有很多，如一个可以横向滑动的列表，当前高亮的 Tab 显然隶属于列表自身的状态，但很多情况下，业务需求会要求从外部跳转至列表时，根据传入的某个值，直接定位到某个 Tab。

为了解决这些问题，React**引入了第一个新的生命周期**：

```javascript
static getDerivedStateFromProps(nextProps, prevState)
//返回一个对象 和调用setState一样
```

两者在使用上的区别：

**原有的代码**

```java
componentWillReceiveProps(nextProps){
    if (nextProps.tab !== this.props.tab) {
        this.setState({
            tab:nextProps.tab
        });
        this.tabChange();
    }
}
```

**新的代码**

```javascript
static getDerivedStateFromPorps (nextProps,prevState){
    if (nextProps.tab!=prevState.tab) {
        return {
            tab: nextProps.tab
        };
    }
    return null;
}
componentDidUpdate(prevProps,prevState){
    this.tabChange();
}
```

这样看似乎没有什么改变，特别是当我们把`this.tabChange()`也放在`componentDidUpdate()`中执行时（正确做法），完全没有不同，但这也是我们一开始想说的，React通过API来约束开发者写出更好的代码，而新的使用方法有以下的优点：

1. `getDerivedStateFromPorps()`是静态方法，在这里不能使用`this`，也就是一个纯函数，**开发者不能写出副作用的代码**。
2. 开发者**只能通过`prevState`而不是`prevProps`来做对比，保证了`state`和`props`之间的简单关系以及不需要处理第一次渲染时`prevProps`为空的情况**。
3. 基于第一点，将状态变化（`setState`）和昂贵操作（`tabChange`）区分开，更加便于`render`和`commit`阶段操作或者说优化。

#### `componentWillUpdate（）`

> 与 componentWillReceiveProps 类似，许多开发者也会在 componentWillUpdate 中根据 props 的变化去触发一些回调。但不论是 componentWillReceiveProps 还是 componentWillUpdate，都有可能在一次更新中被调用多次，也就是说写在这里的回调函数也有可能会被调用多次，这显然是不可取的。与 componentDidMount 类似，componentDidUpdate 也不存在这样的问题，一次更新中 componentDidUpdate 只会被调用一次，所以将原先写在 componentWillUpdate 中的回调迁移至 componentDidUpdate 就可以解决这个问题。

本段引用自React v16.3 版本新生命周期函数浅析及升级方案

另外一种情况则是我们`需要获取DOM元素状态`，但是由于在fiber中，render可打断，可能在willMount中获取到的元素状态很可能与实际需要的不同，这个通常可以使用第二个新增的生命函数的解决

```reasonml
getSnapshotBeforeUpdate(prevProps, prevState) // 返回的
值作为componentDidUpdate的第三个参数
```

与willMount不同的是，` getSnapshotBeforeUpdate()`会在最终`确定的render执行之后，React更新真实DOMs和refs之前执行`，也就是`能保证其获取到的元素状态与didUpdate中获取到的元素状态相同`。



# 7.什么是完全可控组件？

[参考资料](https://segmentfault.com/a/1190000023441695)

# 8.使用refs注意点

> 1. **你不能在函数组件上使用 `ref` 属性**，因为他们**没有实例**。
> 2. 务必**不要过度使用`ref`。**原因：`react`推崇数据驱动视图，直接通过ref去操作强制修改子组件（DOM）的方法，会导致变化来源变得难以排查，在一些场景会出现问题（例如：setState的异步场景，参考[链接](https://blog.csdn.net/SherrybabyOne/article/details/82254352)），不符合react数据流规范。所以，**应当先考虑状态提升方式能否解决，避免使用 refs 来做任何可以通过声明式实现来完成的事情。**
> 3. 使用**回调形式的ref时**，**尽量不要使用内联函数**，因为当组件更新的时候，内联函数会被当做新的prop处理，让ref属性接受到新函数的时候，react内部会先清空ref，也就是会以null为回调参数先执行⼀次ref这个props，然后在以该组件的实例执行⼀次ref，所以用匿名函数做ref 的时候，有的时候去ref赋值后的属性会取到null。

## 何时使用Refs

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

## Refs的几种形式

- String Refs(Abandoned)
- CallBack Refs(**可以在组件间传递回调形式的refs**)
- React.createRef(**可以在组件间传递对象形式的refs**)
- React.forwardRef(**支持类式组件和函数式组件**)

# 9.React事件处理

## 合成事件

## 阻止默认事件

React中不能通过返回 `false` 的方式阻止默认行为。你必须显式的使用 `preventDefault`。

## 事件处理函数回调this丢失问题

[参考链接](https://juejin.cn/post/6844903605984559118)

如果传入的事件回调是当前组件的**类实例方法**，那么类实力方法中的this会是undefined（丢失context）。原因：类声明和类表达式的主体以 **严格模式** 执行，即构造函数、静态方法和原型方法。Getter 和 setter 函数也在严格模式下执行。

## this绑定的四种方法

- 作为实例属性保存箭头函数（最佳实践）

  - ```jsx
    class App extends React.Component {
        fn = () => {
            console.log(this);
        }
        render() {
            return <div onClick={this.fn}></div>;
        }
    }
    //优点：代码十分简洁，不需要手动写bind、也不需要在constructor中进行额外的操作
    
    //缺点：很多文章都提到这是一种完美写法，但其实每一个实例在初始化的时候都会新建一个新事件回调函数（因为绑定在实例的属性上，每个实例都有一个fn的方法。本质上，这是一种重复浪费），所以其实并不是很完美
    ```

- 作为原型方法，在`constructor`中对该原型方法进行`bind`绑定（可读性和维护性不好，不推荐使用）

  - ```jsx
    class App extends React.Component {
        constructor(props) {
            super(props);
            this.fn = this.fn.bind(this);
        }
        fn() {
            console.log(this);
        }
        render() {
            return <div onClick={this.fn}></div>;
        }
    }
    //优点：fn函数在组件多次实例化过程中只生成一次（因为是用实例的fn属性直接指向了组件的原型，并绑定了this属性）
    
    //缺点：代码写起来比较繁琐，需要在constructor中，手动绑定每一个回调函数
    ```

- **作为原型方法，在render中直接上进行bind绑定**（这种方法很简单，可能是大多数初学开发者在遇到问题后采用的一种方式。然后由于组件每次执行`render`将会重新分配函数这将会影响性能。特别是在你做了一些性能优化之后，它会破坏PureComponent性能。不推荐使用）

  - ```jsx
    class App extends React.Component {
        fn() {
            console.log(this);
        }
        render() {
            return <div onClick={this.fn.bind(this)}></div>;
        }
    }
    //优点：fn函数多次实例化只生成一次，存在类的属性上，类似于4，写法上比4稍微好一点。
    
    //缺点：this.fn.bind(this)会导致每次渲染都是一个全新的函数，在使用了组件依赖属性进行比较、pureComponent、函数组件React.memo的时候会失效。
    ```

- 作为原型方法，使用箭头函数内联调用（存在着相同的性能问题，不推荐使用，但是很多场景下需要灵活传参的时候很管用）

  - ```jsx
    class App extends React.Component {
        fn() {
            console.log(this);
        }
        render() {
            return <div onClick={() => fn()}></div>;
        }
    }
    //优点： 1.写法简洁；2.传参灵活；
    const arr = [1, 2, 3, 4, 5];
    class App extends React.Component {
        fn(val) {
            console.log(val);
        }
        render() {
            return (
                <div>
                    {arr.map(item => (
                        // 采用 6的写法，要打印数组这一项就很方便
                        <button onClick={() => this.fn(item)}>{item}</button>
                    ))}
                </div>
            );
        }
    }
    ```

    ## 向事件处理函数传递参数

    ```jsx
    <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
    <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
    //在这两种情况下，React 的事件对象 e 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 bind 的方式，事件对象以及更多的参数将会被隐式的进行传递。
    //推荐使用第一种
    ```


# 10.受控组件和非受控组件的概念？

[参考](https://github.com/frontend9/fe9-library/issues/195)

[参考](https://juejin.cn/post/6858276396968951822)

> 在 HTML 中，表单元素（如`<input>`、 `<textarea>` 和 `<select>`）通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 [`setState()`](https://zh-hans.reactjs.org/docs/react-component.html#setstate)来更新。我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

1. 受控组件和非受控组件仅就针对**表单元素**而言，因为它可以进行游离于React状态控制之外的用户操作。只要表单元素制定了value属性，那么就可以是是受控组件。如果只指定了defaultValue，那么他就是非受控组件。
2. 在HTML的**表单元素**中，它们通常自己维护一套`state`，并随着用户的输入自己进行`UI`上的更新，这种行为是不被我们程序所管控的。而如果将`React`里的`state`属性和表单元素的值建立依赖关系，再通过`onChange`事件与`setState()`结合更新`state`属性，就能达到控制用户输入过程中表单发生的操作。被`React`以这种方式控制取值的表单输入元素就叫做**受控组件**。
3. 如果对受控组件**指定了固定的value值（非undefined、null）**，会导致用户的输入变更被阻止，但是当value为undefined和null时则不会有上述情况，可以正常输入。

## 受控组件的可替代方案

- 要编写一个非受控组件，而不是为每个状态更新都编写数据处理函数，你可以 [使用 ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 来从 DOM 节点中获取表单数据。

## 表单元素的初始值&非受控组件

在 React 渲染生命周期时，表单元素上的 `value` 将会覆盖 DOM 节点中的值。**在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 `defaultValue` 属性，而不是 `value`。在一个组件已经挂载之后去更新 `defaultValue` 属性的值，不会造成 DOM 上值的任何更新。**

- defaultValue和value不能同时作为表单元素的属性出现，因为他们分别代表了元素的两种模式：非受控组件（defaultValue）和受控组件(value)

> 因为**非受控组件将真实数据储存在 DOM 节点中**，所以在使用非受控组件时，有时候反而更容易**同时集成 React 和非 React 代码**。如果你不介意代码美观性，并且希望快速编写代码，**使用非受控组件往往可以减少你的代码量**。否则，你应该使用受控组件。

# 11.列表渲染与key

> key 帮助 React **识别哪些元素改变了，比如被添加或删除。**因此你应当给数组中的每一个元素赋予一个确定的标识。一个元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串。通常，我们使用数据中的 id 来作为元素的 key。当元素没有确定 id 的时候，**万不得已你可以使用元素索引 index 作为 key**。如果列表项目的顺序可能会变化，我们不建议使用索引来用作 key 值，因为这样做会导致性能变差，还可能引起组件状态的问题。可以看看 Robin Pokorny 的[深度解析使用索引作为 key 的负面影响](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)这一篇文章。如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。要是你有兴趣了解更多的话，这里有一篇文章[深入解析为什么 key 是必须的](https://zh-hans.reactjs.org/docs/reconciliation.html#recursing-on-children)可以参考。

> 当节点绑定唯一key时，是为了告知react以此作为唯一标识，如果key相同并且类型相同，则react会复用组件，而不会对组件进行销毁。

## 首先，为什么需要key？

默认情况下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。在**子元素列表末尾新增元素时，更新开销比较小。如果只是简单的将新增元素插入到表头，那么更新开销会比较大（表尾元素情况下只需要匹配并保留原先的元素，然后增加末尾新元素；表头元素React不会意识到需要进行保留，而是会重建所有的元素），这会带来性能问题**。

为了解决上述问题，React 引入了 `key` 属性。当子元素拥有 key 时，**React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。**

**总结，使用key是React作为优化列表渲染性能的考虑，如果不使用key的话在一些情况下会引发全量更新，而引入了key之后可以根据key更好地复用原有元素来进行增量更新，加入直接使用index作为索引的话，会导致列表渲染错误，因为id并不能和元素进行一一对应，一些改变了元素和id的一一对应关系的数组操作都是危险的，可能导致列表顺序的错误（数组非尾部位置插入元素、删除元素，数组反转等）。**

## 何时可以使用索引作为key？

1. the list and items are **static**–they are not computed and do not change;
2. the items in the list **have no ids**, in which means it **have no need to indentify its order**;
3. the list is ***never* reordered or filtered**.

## 可不可以使用随机值作为key？

如果每一次渲染都会重新生成的随机值作为key的话是不可取的，因为这会导致每一次diff时都会发现全都发生了变化，引发全量更新，这在大量的列表数据渲染中是致命的。

## 其他的注意点

key是不会自动作为prop传递给组件的；

## 结论

1. 列表性能消耗很繁重，需要谨慎使用。 
2. 确保列表中的每个项目都有唯一的key。
3. 除非您确定列表是静态列表（没有添加/重新排序/删除列表），否则优先不使用索引作为key。
4. 切勿使用像Math.random()这样的不稳定方法来生成key。
5. 如果使用不稳定的key，React将遇到性能下降和意外行为。

# 12.状态提升

在 React 应用中，任何可变数据应当只有一个相对应的唯一“数据源”。通常，state 都是首先添加到需要渲染数据的组件中去。然后，如果其他组件也需要这个 state，那么你可以将它提升至这些组件的最近共同父组件中。**你应当依靠[自上而下的数据流](https://zh-hans.reactjs.org/docs/state-and-lifecycle.html#the-data-flows-down)，而不是尝试在不同组件间同步 state。**虽然提升 state 方式比双向绑定方式需要编写更多的“样板”代码，但带来的好处是，排查和隔离 bug 所需的工作量将会变少。由于“存在”于组件中的任何 state，仅有组件自己能够修改它，因此 bug 的排查范围被大大缩减了。此外，你也可以使用自定义逻辑来拒绝或转换用户的输入。

# 13.React组合

## 组合可以解决什么问题？

组合可以解决props层层透传的问题（但是解决的能力有限，因为使用组合之后**传入的组件和目标组件内部的交互会变得更加复杂**）。

## 单个槽位

传入组件内部内容被“打包”成props.children通过props传入组件内部。

```jsx
function UseCase(props) {
    return (
        <Fragment>
            <div>
            	{ props.children }
            </div>
        </Fragment>
    )
}

function App(props) {
    return (
        <Fragment>
        	<UseCase>
                <div>Inner Slot1</div>
                <div>Inner Slot2</div>
            </UseCase>
        </Fragment>
    )
}
```

## 具名槽位

```jsx
function UseCase(props) {
    return (
        <Fragment>
            <div>
                { props.left }
                { props.right }
            </div>
        </Fragment>
    )
}

function App() {
    return (
        <Fragment>
            <UseCase 
                left = {
                    <div>Inner Slot1</div>
                }
                right = {
                    <div>Inner Slot2</div>   
                }/>
        </Fragment>
    )
}
```

# 14.React哲学

如何确定应该将哪些部分划分到一个组件中呢？你可以将组件当作一种函数或者是对象来考虑，根据[单一功能原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)来判定组件的范围。也就是说，一个组件原则上只能负责一个功能。如果它需要负责更多的功能，这时候就应该考虑将它拆分成更小的组件。

## state vs. props

[参考](https://lucybain.com/blog/2016/react-state-vs-pros/)

# 15.给类式组件的prop设置默认值

[参考](https://zh-hans.reactjs.org/docs/react-component.html#defaultprops)

# 16.状态提升有什么潜在的缺点？

[参考](https://liz-hard.medium.com/react-lifting-state-up-de2a5ffaed5)

[参考](https://medium.com/swlh/react-state-doing-the-least-bad-thing-6f2cb20986e4)

## 存在哪些问题？

- Harder to maintain code, your code can become quite fragmented if you have props being passed through multiple components, if you add a new prop it can be easier to miss adding it in all the places.（状态提升使得状态脱离了其原宿主本身，作为props游离在其他组件之间，这使得**提升后的状态变得让人难以理解，也会使得代码碎片化**）
- Removing components means some props might be left in unnecessarily.（将状态提升后再以props向下流，其存在形式是隐式的，如果该props属性没有被利用到的话（例如删除了组件中利用到该props的相关内容导致props失去存在意义，但是这个是不可感知的）也难以察觉）
- Renaming props can be an issue, are you sure you remember all the places it was originally and even if you do, does the next developer who comes along to edit the code know. This is an easy way to introduce bugs.（状态升级会导致针对状态对应的props进行诸如重命名等的的修改很棘手，因为提升后的状态作为props向下流向了很多个组件，也有可能涉及到多层透传，这就会导致容易产生遗漏等问题）
- once the codebase becomes big enough another unfortunate problem rears its ugly head: *God components.* These components grow like a cancer, absorbing state until it becomes this eldritch horror with an unholy amalgamation of 15+ separate state systems and a severe case of callback purgatory.（过多的状态提升会导致公共父组件膨胀，使得公共父组件充斥着业务逻辑和状态定义，容易混淆本应当属于子类的状态和属于公共父类本身的状态）
- Since all this state now lives within a single top-level source, data must be *drilled* down through child props until it reaches its intended target. This clutters child components by introducing props that in no way relate to them, but can’t be removed since their children still need the data.（在某些情况下，子组件A本身并不需要指特定的props，而仅仅是因为子组件A**内部的一个组件B**需要用到该特定props，子组件就必须作为中间人的身份传递props，这违反了单一职责原则，使得组件的props耦合在一起）
- 最后，也是最重要的一点，使用状态提升会使得被提升的state原本应当所在的组件依赖父组件提供数据源，这导致组件的复用遇到问题。

## 如何解决呢？

- Context
- Redux
- Other centralized state management tools

# 17.如何引入代码分割？

> 对你的应用进行代码分割能够帮助你“懒加载”当前用户所需要的内容，能够显著地提高你的应用性能。尽管并没有减少应用整体的代码体积，但你可以避免加载用户永远不需要的代码，并在初始加载的时候减少所需加载的代码量。
>
> 在你的应用中引入代码分割的最佳方式是通过动态 `import()` 语法。

## 动态`import()`（针对代码的懒加载分割）

```jsx
//使用之前
import { add } from './math';
console.log(add(16, 26));

//使用之后
import("./math").then(math => {
    console.log(math.add(16, 26));
});
```

## `React.lazy`（针对组件的懒加载api）

> 注意:
>
> `React.lazy` 和 Suspense 技术还不支持服务端渲染。如果你想要在使用服务端渲染的应用中使用，我们推荐 [Loadable Components](https://github.com/gregberge/loadable-components) 这个库。它有一个很棒的[服务端渲染打包指南](https://loadable-components.com/docs/server-side-rendering/)。

```jsx
//使用之前
import OtherComponent from './OtherComponent';

//使用之后，此代码将会在组件首次渲染时，自动导入包含 OtherComponent 组件的包。
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

`React.lazy` 函数能让你像渲染常规组件一样处理动态引入（的组件）。`React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `default export` 的 **React 组件**。

## React.lazy应当配合Suspense组件使用实现code spliting

```jsx
import { lazy, Suspend } from 'react'
const Demo = lazy(() => import('./components/demo'))
const App = () => (
    <Suspend fallback={ <h1>Loading...</h1> }>
    	<Demo/>
    </Suspend>
)
```

`Suspend`组件的作用:

- 配合 React.lazy 在等待组件时 suspend（暂停）渲染，并显示加载标识（fallback）；
- Suspend可以用来实现一种**最佳的异步取数方式**，但是目前依然处于试验阶段：[参考1](https://zhuanlan.zhihu.com/p/113463166) [参考2](https://juejin.cn/post/6844903789959315470) [参考：官方文档](https://zh-hans.reactjs.org/docs/concurrent-mode-suspense.html)

## （实验功能）如何在 Suspense 中使用 Data Fetching

当前 `Suspense` 的使用分为三个部分：

第一步: 用 `Suspens` 组件包裹子组件

```jsx
import { Suspense } from 'react'

<Suspense fallback={<Loading />}>
  <ChildComponent>
</Suspense>
```

第二步: 在子组件中使用 `unstable_createResource`:

```jsx
import { unstable_createResource } from 'react-cache'

const resource = unstable_createResource((id) => {
  return fetch(`/demo/${id}`)
})
```

第三步: 在 `Component` 中使用第一步创建的 `resource`:

```jsx
const data = resource.read('demo')
```

实例

```jsx
import { Suspense } from 'react-cache'
import { unstable_createResource } from 'react-cache'
//unstable_createResource必须返回一个promise对象代表期约落定状态，可以渲染
const resource = unstable_createResource((id) => {
    return fetchAPI(`/api/demo`)
})

function Demo {
    //必须使用resource.read的方式读取，使得在Suspend组件在渲染内部时检测到异步数据从而使用fallback暂时代替。
    const data = resource.read(this.props.id)
    const { name } = data;
    return (
        <div>{name}</div>
    );
}
```

## suspend和路由配合实现按需加载、懒加载

> 大多数网络用户习惯于页面之间能有个加载切换过程。你也可以选择重新渲染整个页面，这样您的用户就不必在渲染的同时再和页面上的其他元素进行交互。

```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  </Router>
);
```

## React.lazy只支持默认导出

> `React.lazy` 目前只支持默认导出（default exports）。如果你想被引入的模块使用命名导出（named exports），你可以**创建一个中间模块，来重新导出为默认模块。这能保证 tree shaking 不会出错，并且不必引入不需要的组件。**



# 18.错误边界

> 错误边界是一种 React 组件，这种组件**可以捕获发生在其子组件树任何位置的 JavaScript 错误，并打印这些错误，同时展示降级 UI**，而并不会渲染那些发生崩溃的子组件树。错误边界可以捕获发生在**整个子组件树的渲染期间、生命周期方法以及构造函数中的错误。**
>
> 错误边界**无法**捕获以下场景中产生的错误：
>
> - 事件处理（[了解更多](https://zh-hans.reactjs.org/docs/error-boundaries.html#how-about-event-handlers)）
> - 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调函数）
> - 服务端渲染
> - 它自身抛出来的错误（并非它的子组件）

## 定义错误边界组件

如果一个 class 组件中定义了 [`static getDerivedStateFromError()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromerror) 或 [`componentDidCatch()`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidcatch) 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，请使用 `static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。

```jsx
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false };
    }

    static getDerivedStateFromError(error) {
        // 更新 state 使下一次渲染能够显示降级后的 UI
        return { hasError: true };
    }

    componentDidCatch(error, errorInfo) {
        // 你同样可以将错误日志上报给服务器
        logErrorToMyService(error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            // 你可以自定义降级后的 UI 并渲染
            return <h1>Something went wrong.</h1>;
        }

        return this.props.children; 
    }
}
```

> 只有 **class 组件才可以成为错误边界组件**。大多数情况下, 你只需要声明一次错误边界组件, 并在整个应用中使用它。**错误边界仅可以捕获其子组件的错误**，它无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 `catch {}` 的工作机制。

## 未捕获错误的新行为

> **自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。**增加错误边界能够让你在应用发生异常时提供更好的用户体验。我们也鼓励使用 JS 错误报告服务（或自行构建），这样你能了解关于生产环境中出现的未捕获异常，并将其修复。

## 关于事件处理函数内的错误捕获

> React 不需要错误边界来捕获事件处理器中的错误。与 render 方法和生命周期方法不同，事件处理器不会在渲染期间触发。**因此，如果它们抛出异常，React 仍然能够知道需要在屏幕上显示什么。如果你需要在事件处理器内部捕获错误，使用普通的 JavaScript `try` / `catch` 语句**

## 版本更改

> React 15 中有一个支持有限的错误边界方法 unstable_handleError。此方法不再起作用，同时自 React 16 beta 发布起你需要在代码中将其修改为 componentDidCatch。

# 19.Context

> Context 提供了一个**无需为每层组件手动添加 props，就能在组件树间进行数据传递**的方法。

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法**对于某些类型的属性而言是极其繁琐的**（例如：地区偏好，UI 主题），这些属性是应用程序中**许多组件都需要的**。Context 提供了一种**在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。**

总结：使用props传递state（也就是状态提升）在状态被很多组件所依赖的情况下，需要多次手动传入组件（产生了较多的片段代码），而且必须逐层进行传递直至到达所需的组件（哪怕传递的组件并不需要这个props，它也必须担负其传递下去的职责），这是及其繁琐的。这使得我们有必要将一些理应成为全局状态的状态使用Context进行全局传递。

## 何时使用Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言。

**如果你只是想避免层层传递一些属性，[组件组合（component composition）](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html)有时候是一个比 context 更好的解决方案。**这种对组件的**控制反转**减少了在你的应用中要传递的 props 数量，这在很多场景下会使得你的代码更加干净，使你对根组件有更多的把控。但是，这并不适用于每一个场景：**这种将逻辑提升到组件树的更高层次来处理，会使得这些高层组件变得更复杂，并且会强行将低层组件适应这样的形式（传入组件），这可能不会是你想要的**。而且你的组件并不限制于接收单个子组件。你可能会传递多个子组件，甚至会为这些子组件（children）封装多个单独的“接口（slots）”。

如果子组件需要在渲染前和父组件进行一些交流，你可以进一步使用 [render props](https://zh-hans.reactjs.org/docs/render-props.html)。

## 使用Context

```jsx
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
//只有当组件所处的树中没有匹配到 Provider 时，其 defaultValue 参数才会生效。
//此默认值有助于在不使用 Provider 包装组件的情况下（不设置value属性）对组件进行测试。注意：将 undefined 传递给 Provider 的 value 时，消费组件的 defaultValue 不会生效。
const ThemeContext = React.createContext('light');
class App extends React.Component {
    render() {
        // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
        // 无论多深，任何组件都能读取这个值。
        // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
        return (
            <ThemeContext.Provider value="dark">
                <Toolbar />
            </ThemeContext.Provider>
        );
    }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
    return (
        <div>
            <ThemedButton />
        </div>
    );
}

class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context。
    // React 会往上找到最近的 theme Provider，然后使用它的值。
    // 在这个例子中，当前的 theme 值为 “dark”。
    static contextType = ThemeContext;
     render() {
        return <Button theme={this.context} />;
    }
}
```

## Context的值发生变化与生命周期的关系

> 当Provider 的 value 值发生*变化时*，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 shouldComponentUpdate 函数。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。**从 Provider 到其内部 consumer 组件（包括 [.contextType](https://zh-hans.reactjs.org/docs/context.html#classcontexttype) 和 [useContext](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)）的传播不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件跳过更新的情况下也能更新。**通过新旧值检测来确定变化，使用了与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 相同的算法。

## 使用`<Context.Consumer>`

此组件可以让你在[函数式组件](https://zh-hans.reactjs.org/docs/components-and-props.html#function-and-class-components)中可以订阅 context。

```jsx
<Context.Consumer>
    {
        contextValue => <h1> { contextValue } </h1>
    }
</Context.Consumer>
```

这种方法需要一个[函数作为子元素（function as a child）](https://zh-hans.reactjs.org/docs/render-props.html#using-props-other-than-render)。这个函数接收当前的 context 值，并返回一个 React 节点。传递给函数的 `value` 值等价于组件树上方离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

## 将Context和State结合使用

将Context和State结合使用可以使得Context可以动态更新。

## 使用Context的重复渲染陷阱

因为 context 会根据引用标识来决定何时进行渲染（本质上是 `value` 属性值的浅比较），所以这里可能存在一些陷阱，当 provider 的父组件进行重渲染时，可能会在 consumers 组件中触发意外的渲染。举个例子，当每一次 Provider 重渲染时，由于 `value` 属性总是被赋值为新的对象，以下的代码会重新渲染下面所有的 consumers 组件：

```javascript
class App extends React.Component {
  render() {
    return (
      <MyContext.Provider value={{something: 'something'}}>
        <Toolbar />
      </MyContext.Provider>
    );
  }
}

//优化
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      <MyContext.Provider value={this.state.value}>
        <Toolbar />
      </MyContext.Provider>
    );
  }
}
```

# 20.高阶组件HOC

> 高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。
>
> 具体而言，**高阶组件是参数为组件，返回值为新组件的函数。**
>
> 组件是将 props 转换为 UI，而高阶组件是将组件转换为另一个组件。

> 我们之前建议使用 mixins 用于解决横切关注点相关的问题。但我们已经意识到 mixins 会产生更多麻烦。[阅读更多](https://zh-hans.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html) 以了解我们为什么要抛弃 mixins 以及如何转换现有组件。

使用HOC的特征

1. HOC 不需要关心数据的使用方式或原因，而被包装组件也不需要关心数据是怎么来的。
2. 与普通组件一样，`withSubscription` 和包装组件之间的契约完全基于之间传递的 props。这种依赖方式使得替换 HOC 变得容易，只要它们为包装的组件提供相同的 prop 即可。例如你需要改用其他库来获取数据的时候，这一点就很有用。

## HOC优缺点

优点：通过传递props去影响内层组件的状态，不直接改变内层组件的状态、生命周期等，降低了耦合度。HOC实现了业务逻辑的复用，比mixin模式更佳。不同于 Mixin 的打平合并（**平级结构**），HOC 具有天然的**层级结构（组件树结构）**，这又降低了复杂度

缺点：

1. 组件多层嵌套， **增加复杂度与理解成本**；
2. 会出现因为同名上层**props被下层同名props覆盖**的情况，而且上层props的来源如果层级较多的话难以追溯；
3. HOC中传递props时存在**ref隔断， 依靠React.forwardRef 来解决**；
4. **HOC 无法从外部访问子组件的 State**，因此无法通过`shouldComponentUpdate`滤掉不必要的更新，React 在支持 ES6 Class 之后提供了`React.PureComponent`来解决这个问题；
6. 在render中使用HOC会严重影响性能；

## 不要改变原始组件，而应该使用组合

不要试图在 HOC 中修改组件原型（或以其他方式改变它）。这样做会产生一些不良后果。其一是**输入组件再也无法像 HOC 增强之前那样使用了**。更严重的是，如果你**再用另一个同样会修改 `componentDidUpdate` 的 HOC 增强它，那么前面的 HOC 就会失效！**同时，**这个 HOC 也无法应用于没有生命周期的函数组件。**修改传入组件的 HOC 是一种糟糕的抽象方式。调用者必须知道他们是如何实现的，以避免与其他 HOC 发生冲突。

⭐HOC 不应该修改传入组件，而应该**使用组合的方式，通过将组件包装在容器组件中实现功能**。

```jsx
function logProps(WrappedComponent) {
    return class extends React.Component {
        componentDidUpdate(prevProps) {
            console.log('Current props: ', this.props);
            console.log('Previous props: ', prevProps);
        }
        render() {
            // 将 input 组件包装在容器中，而不对其进行修改。Good!
            return <WrappedComponent {...this.props} />;
        }
    }
}
```

## HOC和容器组件模式的关系

容器组件担任将高级和低级关注点分离的责任，由容器管理订阅和状态，并将 prop 传递给处理 UI 的组件。HOC 使用容器作为其实现的一部分，你可以将 HOC 视为**参数化（函数化）容器组件。**

## 使用HOC的约定：将不相关的 props 传递给被包裹的组件

HOC 为组件添加特性。自身不应该大幅改变约定。HOC 返回的组件与原组件应保持类似的接口（HOC化之后的组件接口只能是原组件接口的增量更新）。这样的约定保证了 HOC 的灵活性以及可复用性

```jsx
render() {
  // 过滤掉非此 HOC 额外的 props，且不要进行透传
  const { extraProp, ...passThroughProps } = this.props;

  // 将 props 注入到被包装的组件中。
  // 通常为 state 的值或者实例方法。
  const injectedProp = someStateOrInstanceMethod;

  // 将 props 传递给被包装组件
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

## HOC命名

```jsx
function withSubscription(WrappedComponent) {
    class WithSubscription extends React.Component {/* ... */}
    WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
    return WithSubscription;
}

function getDisplayName(WrappedComponent) {
    return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

## HOC注意事项

### 不要在 render 方法中使用 HOC（动态HOC）

> React 的 diff 算法（称为[协调](https://zh-hans.reactjs.org/docs/reconciliation.html)）使用组件标识来确定它是应该更新现有子树还是将其丢弃并挂载新子树。 如果从 `render` 返回的组件与前一个渲染中的组件相同（`===`），则 React 通过将子树与新子树进行区分来递归更新子树。 如果它们不相等，则完全卸载前一个子树。

通常，你不需要考虑这点。但对 HOC 来说这一点很重要，因为这代表着你不应在组件的 render 方法中对一个组件应用 HOC：

```jsx
render() {
  // 每次调用 render 函数都会创建一个新的 EnhancedComponent
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  return <EnhancedComponent />;
}
```

这不仅仅是性能问题 - **重新挂载组件会导致该组件及其所有子组件的状态丢失。**如果在组件之外创建 HOC，这样一来**组件只会创建一次（原有引用）**。因此，每次 render 时都会是同一个组件。一般来说，这跟你的预期表现是一致的。在极少数情况下，你需要动态调用 HOC。你可以在组件的**生命周期方法或其构造函数中进行调用**。

### 务必复制静态方法

当你将 HOC 应用于组件时，原始组件**将使用容器组件进行包装**。这意味着**新组件没有原始组件的任何静态方法**。

```jsx
// 定义静态函数
WrappedComponent.staticMethod = function() {/*...*/}
// 现在使用 HOC
const EnhancedComponent = enhance(WrappedComponent);

// 增强组件没有 staticMethod
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

为了解决这个问题，你可以在返回之前把这些方法拷贝到容器组件上：

```jsx
function enhance(WrappedComponent) {
    class Enhance extends React.Component {/*...*/}
    // 必须准确知道应该拷贝哪些方法 :(
    Enhance.staticMethod = WrappedComponent.staticMethod;
    return Enhance;
}
```

### ⭐refs不会被传递

虽然高阶组件的约定是将所有 props 传递给被包装组件，但这对于 refs 并不适用。那是因为 `ref` 实际上并不是一个 prop - 就像 `key` 一样，它是由 React 专门处理的。如果将 ref 添加到 HOC 的返回组件中，则 **ref 引用指向容器组件，而不是被包装组件**。这个问题的解决方案是通过使用 `React.forwardRef` API（React 16.3 中引入）。[前往 ref 转发章节了解更多](https://zh-hans.reactjs.org/docs/forwarding-refs.html)。

## 高阶组件转发refs

高阶组件理应透传所有props到其包裹的组件，有一点需要注意：refs 将不会透传下去。这是因为 `ref` 不是 prop 属性。就像 `key` 一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件。我们可以使用 `React.forwardRef` API 明确地将 refs 转发到内部的 `FancyButton` 组件。`React.forwardRef` 接受一个渲染函数，其接收 `props` 和 `ref` 参数并返回一个 React 节点。

```jsx
function logProps(Component) {
    class LogProps extends React.Component {
        componentDidUpdate(prevProps) {
            console.log('old props:', prevProps);
            console.log('new props:', this.props);
        }

        render() {
            const {forwardedRef, ...rest} = this.props;

            // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
            return <Component ref={forwardedRef} {...rest} />;
        }
    }

    // 注意 React.forwardRef 回调的第二个参数 “ref”。
    // 我们可以将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”
    // 然后它就可以被挂载到被 LogProps 包裹的子组件上。
    return React.forwardRef((props, ref) => {
        return <LogProps {...props} forwardedRef={ref} />;
    });
}
```

# 21.render props

## render props优缺点

优点：

1. props命名可修改，不存在相互覆盖（回调形式，形参名称的修改无所谓）；
2. 清楚props来源（因为在回调函数中已经**明确指定了需要的props**，父组件作为数据源只要提供相关数据即可）；
3. 耦合度很低，回调函数内返回组件对于调用的父组件只有props上的数据依赖关系（纯函数），所以容易理解；

缺点：

1. 使用繁琐：HOC使用只需要借助装饰器语法通常一行代码就可以进行复用，Render Props无法做到如此简单，需要预定义一个回调函数并对接收组件进行接受性改造；
2. 嵌套过深：Render Props虽然拜托了组件多层嵌套的问题，但是可能会转化为函数回调的嵌套。

> a render prop is a function prop that a component uses to know what to render.

> 术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个**值为函数的 prop 共享代码的简单技术。**

具有 render prop 的组件接受一个返回 React 元素的函数，并在组件内部通过调用此函数来实现自己的渲染逻辑。

```jsx
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

## 使用 Render Props 来解决横切关注点（Cross-Cutting Concerns）

组件是 React 代码复用的主要单元，但如何将一个组件封装的状态或行为共享给其他需要相同状态的组件并不总是显而易见。

## render props和React.PureComponent一起使用时的注意点

如果在render方法里创建函数，那么使用render props会抵消使用React.PureComponent带来的优势（每一次调用都会生成新的render回调函数传入，和上一次比较是不同的引用），因为浅比较 props 的时候总会得到 false，并且在这种情况下每一个 `render` 对于 render prop 将会生成一个新的值。

为了绕过这一问题，有时可以定义一个 prop **作为实例方法（维持引用）**。要是不可以作为实例方法的话，就继承React.Component，不要继承React.PureComponent。

## 总结

render props是通过一个**返回组件的回调函数**作为props传递给其他组件，其他组件在接收到该回调之后可以根据回调参数传入props。使用render props能够很好地对组件进行复用。一言蔽之，父组件已经明确自己可以提供的数据源，并且内部存在了一个插槽给相应的回调进行调用、渲染。在这里，父组件数据提供者、操纵者，子组件是数据承载者。

# 22.PureComponent和Component

`React.PureComponent` 与 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 很相似。两者的区别在于 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 并未实现 [`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，而 `React.PureComponent` 中以浅层对比 prop 和 state 的方式来实现了该函数。

如果赋予 React 组件相同的 props 和 state，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。

> 注意
>
> `React.PureComponent` 中的 `shouldComponentUpdate()` 仅作对象的浅层比较。如果对象中包含复杂的数据结构，则有可能因为无法检查深层的差别，产生错误的比对结果。**仅在你的 props 和 state 较为简单时，才使用 `React.PureComponent`，或者在深层数据结构发生变化时调用 [`forceUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#forceupdate) 来确保组件被正确地更新。你也可以考虑使用 [immutable 对象](https://facebook.github.io/immutable-js/)加速嵌套数据的比较。**
>
> 此外，`React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新。因此，请确保所有子组件也都是“纯”的组件。

## 为什么保证子组件也是PureComponent？

性能方面考虑：倘若父组件使用PureComponent，子组件使用Component，那么父组件的更新**势必**引起Component子组件的更新（如果父组件的更新在比较后被取消，那么子组件势必不更新，无论是哪种类型的子组件，这还是按照生命周期的规律来进行的）；使用PureComponent的话能够保证PureComponent的在性能的优化上做到一致性，只有**父组件传递给子组件的状态有相应发生了变化**的时候，子组件**才相应地产生更新**，因此PureComponent的子组件最好也是继承自PureComponent。

## 总结

PureComponent作为性能优化的手段，在组件内部帮助我们实现了shouldComponentUpdate()这个生命周期的判断逻辑的部署，PureComponent在这个生命周期内进行了state和props的浅层比较，也就是Object.is()判别新旧数据变化，当且仅当数据发生变化的时候引发进一步更新，否则拒绝更新。深层次的数据如果想要进行更新，就只能使用替换引用的方式（浅层次的使用 对象字面量 + rest操作符浅拷贝 + 目标属性覆盖， 层次太深的话做个深拷贝）使得触发更新或者使用forceUpdate()强制更新。使用PureComponent的时候注意其子组件也都应该一致使用PureComponent。

# 23.Mixin为什么存在问题？

1. mixin引入了**隐式依赖**（组件可能依赖mixin上的某个方法，mixin也有可能依赖组件上的某个方法，这导致了隐式依赖的产生，此时如果将mixin进行迁移的话，可能会导致隐式依赖的丢失从而出现为问题。通常，mixin 依赖于其他 mixin，并且删除其中一个会破坏另一个。在这些情况下，很难判断数据如何流入和流出 mixins，以及它们的依赖关系图是什么样的。与组件不同，mixin 不形成层次结构：它们被扁平化并在同一个命名空间中运行）总之，隐式依赖导致依赖关系不透明，维护成本和理解成本迅速攀升 。
2. mixin可能会**导致命名冲突**（不能保证mixin和另一个mixin或者组件可以一起使用而不产生命名上的冲突。此外，要是产生了冲突需要修改，其他直接依赖次mixin的组件都会收到牵连）。
3. mixin导致**复杂性**滚雪球。（随着时间的推移，使用相同 mixin 的组件变得越来越耦合。任何新功能都会被添加到使用该 mixin 的所有组件中。如果不复制代码或在 mixin 之间引入更多的依赖关系和间接性，就无法拆分 mixin 的“更简单”部分。逐渐地，封装边界被侵蚀，并且由于很难更改或删除现有的 mixin，它们变得越来越抽象，直到没有人了解它们是如何工作的）。
4. Mixin 倾向于**增加更多状态**，这降低了应用的可预测性，导致复杂度剧增。
5. 难以快速理解组件行为，需要全盘了解所有依赖 Mixin 的扩展行为，及其之间的相互影响。

## mixin现在有哪些实用的用途

- 在组件之间共享实用功能（方法）

## mixin和HOC的区别

# 24.React与第三方库协同

# 25.React移除虚拟DOM绑定

```jsx
ReactDOM.unmountComponentAtNode(container)
// 从 DOM 中卸载组件，会将其事件处理器（event handlers）和 state 一并清除。如果指定容器上没有对应已挂载的组件，这个函数什么也不会做。
```

# 26.为什么必须引入React？

由于 JSX 会编译为 `React.createElement` 调用形式，所以 `React` 库也必须包含在 JSX 代码作用域内。如果你不使用 JavaScript 打包工具而是直接通过 `<script>` 标签加载 React，则必须将 `React` 挂载到全局变量中。

# 27.JSX注意点

## 转义

当你将字符串字面量赋值给 prop 时，它的值是未转义的。所以，以下两个 JSX 表达式是等价的：

```jsx
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

### Props 默认值为 “True”

如果你没给 prop 赋值，它的默认值是 `true`。以下两个 JSX 表达式是等价的：

```jsx
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

通常，我们**不建议不传递 value 给 prop**，因为这可能与 [ES6 对象简写](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)混淆，`{foo}` 是 `{foo: foo}` 的简写，而不是 `{foo: true}`。这样实现只是为了保持和 HTML 中标签属性的行为一致。

### 布尔类型、Null 以及 Undefined 将会忽略

`false`, `null`, `undefined`, and `true` 是合法的子元素。但它们并不会被渲染。以下的 JSX 表达式渲染结果相同：

```jsx
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

这有助于依据特定条件来渲染其他的 React 元素。例如，在以下 JSX 中，仅当 `showHeader` 为 `true` 时，才会渲染 `<Header />` 组件：

```jsx
<div>
    {showHeader && <Header />}  <Content />
</div>
```

值得注意的是有一些 [“falsy” 值](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)，如数字 `0`，仍然会被 React 渲染。例如，以下代码并不会像你预期那样工作，因为当 `props.messages` 是空数组时，`0` 仍然会被渲染：

```jsx
<div>
    {props.messages.length &&    <MessageList messages={props.messages} />
    }
</div>
```

要解决这个问题，确保 `&&` 之前的表达式总是布尔值：

```jsx
<div>
    {props.messages.length > 0 &&    <MessageList messages={props.messages} />
    }
</div>
```

反之，如果你想渲染 `false`、`true`、`null`、`undefined` 等值，你需要先将它们[转换为字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion)：

```jsx
<div>
    My JavaScript variable is {String(myVariable)}.
</div>
```

## JSX中的合法falsy

- null
- undefined
- false

# 28.React性能优化

[参考](https://juejin.cn/post/6844903924302888973#heading-8)

[一直以来useCallback的使用姿势都不对](https://github.com/yaofly2012/note/issues/144)

[参考](https://juejin.cn/post/6908895801116721160#heading-55)

[重要参考](https://juejin.cn/post/6935584878071119885#heading-12)

1. 使用`PureComponent`，`React.memo`，`shouldComponentUpdate`优化，`React.memo`是函数式组件`props`更新过滤；
2. 使用`useMemo`缓存计算值，使用`useMemo`还可以用来缓存函数式组件；
3. 时刻关注父组件传递的`props`会在什么时候被改变，尤其是传入一个字面量对象的时候，是非常危险的，会使得子组件的优化失效（字面量处在`render`中，每次重新渲染都会被重新构建）；没有传参要求的时候避免使用内联函数，原理如上；
4. 使用`React.lazy`按需加载；
5. 避免重复组件的频繁卸载和加载，这一类情况可以选择使用`CSS`隐藏后进行操作；
6. 使用`Fragment`，避免在`DOM`上添加过多无效节点；
7. 防止层层`props`透传给被动重渲染带来的危害，使用`Context`绕过中间组件进行更新；但也要合理地使用`Context`，因为`Context`会引发消费组件的强制更新；
8. 函数式组件中，如果子组件很重又不得不用到从父组件中传递下来的函数，考虑使用`useMemo` + `useCallback`优化（注意，这不会节省构建函数的开销），如果函数内部有外部依赖，那么可以考虑使用`useRef` + `useEffect`将状态“实例属性化”并在`useCallback`内部函数中引用`ref`值。最好的方式是使用`useReducer`，因为`useReducer`总是能确保`dispatch`引用稳定的同时调用`dispatch`对状态最新值进行操作；
9. 状态尽量下放到合适地组件，非必要不使用状态提升；

# 29.为什么不可变性在React中很重要?

1. 不可变性有助于**简化复杂的功能**（例如可以实现历史记录功能）；
2. 不可变性有利于**追踪数据的改变**（如果直接修改数据，那么就很难跟踪到数据的改变。跟踪数据的改变需要**可变对象可以与改变之前的版本进行对比，这样整个对象树都需要被遍历一次**。**跟踪不可变数据的变化相对来说就容易多了。如果发现对象变成了一个新对象，那么我们就可以说对象发生改变了**）；
3. 帮助**确定React中何时重新渲染**（不可变性最主要的又是就是在于它可以帮助我们在React中创建Pure Component，我们可以很轻松的确定不可变数据是否发生了改变，从而确定何时对组件进行重新渲染）。

## 什么是immutable？

[参考](https://juejin.cn/post/6949184783473704974)

1. Immutable是一旦创建，就不能被更改的数据。
2. 对Immutable对象的任何修改或添加删除操作都会返回一个新的Immutable对象。
3. Immutable实现的原理是Persistent Data Structure（持久化数据结构），也就是是永久数据创建新数据时，要保证旧数据同时可用且不变。
4. 同时为了避免deepCopy把所有节点都复制一遍带来的性能损耗，Immutable使用了Structural Sharing（结构共享），即如果对象树结点发生变化，只修改这个结点和受它影响的父节点，其他结点进行共享。



# 30.diffing算法

## 基本规则

- **当元素类型变化时，会销毁重建**
- **当元素类型不变时，对比属性**
- **当组件元素类型不变时，通过props递归判断子节点**
- **递归对比子节点，当子节点是列表时，通过key和props来判断。若key一致，则进行更新，若key不一致，就销毁重建**

## 性能损耗

1. 两种不同类型的组件之间切换必然引起卸载和重渲染，可以的话应当尽量使用同一种组件，这样可以避免卸载后重新渲染；
2. key应该是稳定可靠且唯一的，每一次都被赋予随机值的key必然会引发全量更新消耗性能。

# 31.开启编译期严格模式

```jsx
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

`StrictMode` 目前有助于：

- [识别不安全的生命周期](https://zh-hans.reactjs.org/docs/strict-mode.html#identifying-unsafe-lifecycles)
- [关于使用过时字符串 ref API 的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-legacy-string-ref-api-usage)
- [关于使用废弃的 findDOMNode 方法的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)
- [检测意外的副作用](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects)
- [检测过时的 context API](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-legacy-context-api)

# 32.为什么React不同步地更新this.setState() ？

主要有两个原因：

- 这样会破坏掉 `props` 和 `state` 之间的一致性，造成一些难以 debug 的问题。
- 这样会让一些我们正在实现的新功能变得无法实现。

# 33.什么时候setState是同步的？

只要你**进入了 `react` 的调度流程，那就是异步的**。只要你**没有进入 `react` 的调度流程，那就是同步的。**什么东西不会进入 `react` 的调度流程？ `setTimeout` `setInterval` ，直接在 `DOM` 上绑定原生事件等。这些都不会走 `React` 的调度流程，你在这种情况下调用 `setState` ，那这次 `setState` 就是同步的。 否则就是异步的。而 `setState` 同步执行的情况下， `DOM` 也会被同步更新，也就意味着如果你多次 `setState` ，会导致多次更新，这是毫无意义并且浪费性能的。

# 34.为什么要避免使用props生成defaultState？有什么替代的方案吗？

[参考](https://segmentfault.com/a/1190000023441695)

## 为什么使用props生成defaultState会存在问题？

1. 使用父组件传入的props生成子组件的defaultState的时候，因为在props变更之后子组件不会走constructor（因为之后走更新流程），导致子组件内部state无法被重置为新的defaultState。
2. props相同的情况下，组件更新也不会触发默认值的重置；

## 可控组件方案

1. 状态提升，由父组件全权管理子组件的defaultValue、defaultValue重置函数，子组件通过props接收状态以及defaultValue重置函数。

## 不可控组件方案

1. 使用key属性强制触发组件走construct的流程；
2. 在传递props的时候给数据添加一个ID，子组件验证ID之后发现确实发生了变化，就进行重置；
3. 子组件维持自己的状态和状态重置函数，父组件使用ref来调用子组件上的重置函数。

# 35.React 虚拟DOM的优缺点

1. 针对单次DOM操作来说，框架隔着一层虚拟DOM（fiber）对真实DOM进行操作，会经历框架的调度、协调、再渲染三个流程，增大了JS线程的压力，这是有很高代价的。
2. 但是在需要操作DOM较多的情况下，框架会对状态更新批处理优化能够使得浏览器重绘次数减少，然后进行虚拟DOM的diff使得其能够以较小的代价进行DOM的部分更新，尽量采取就地复用的形式复用元素，如果原生在没有合理优化的情况下只能进行全量更新，而这种重新渲染的代价相比就地复用是很高的。
3. 从库的依赖来说，使用框架不得不引入框架库，这就会使得初次加载必须引入完整的框架，增加了网络负担，但是现在的浏览器可以进行本地缓存。
4. 使用虚拟DOM脱离了浏览器环境，能够很方便地将其移植到其他平台：React Native、Electron

# 36.React 事件机制

## JSX事件最终会变成什么？

首先经过Babel转换、然后再经过ReactDOM.createElement转换成React element

![babel.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02eb66989a5444839c4e758b795869e7~tplv-k3u1fbpfcp-watermark.awebp)

然后在创建fiber的阶段会被作为一个props被保存（`fiber`对象上的`memoizedProps` 和 `pendingProps`保存了我们的事件）：

![fiber.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2bd1a74076c40d1b5c5e7b53c341f7f~tplv-k3u1fbpfcp-watermark.awebp)

## 原生DOM上有没有绑定事件？

绑定了，但是是一个空函数，最终所有的事件都被绑定到了rootNode

![button_event.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbd5b7c204754983b1eacc7bdcec8f88~tplv-k3u1fbpfcp-watermark.awebp)

我们可以看到 ，`button`上绑定了两个事件，一个是`document`上的事件监听器，另外一个是`button`，但是事件处理函数`handle`，并不是我们的`handerClick`事件，而是`noop`。`noop`就指向一个空函数。

![noop.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b419061b3c114309aeff3a59bd0d2f62~tplv-k3u1fbpfcp-watermark.awebp)



> 我们给`<input>`绑定的`onChange`，并没有直接绑定在`input`上，而是统一绑定在了`document`上，然后我们`onChange`被处理成很多事件监听器，比如`blur` , `change` , `input` , `keydown` , `keyup` 等。

结论：

- **我们在 `jsx` 中绑定的事件(demo中的`handerClick`，`handerChange`),根本就没有注册到真实的`dom`上。是绑定在`document`上统一管理的。**
- **真实的`dom`上的`click`事件被单独处理,已经被`react`底层替换成空函数。**
- **我们在`react`绑定的事件,比如`onChange`，在`document`上，可能有多个事件与之对应。**
- **`react`并不是一开始，把所有的事件都绑定在`document`上，而是采取了一种按需绑定，比如发现了`onClick`事件,再去绑定`document click`事件。**

## 为什么采取合成事件机制？

- 一方面，将事件绑定在`document`统一管理，防止很多事件直接绑定在原生的`dom`元素上。造成一些不可控的情况，使用代理方式进行统一管理，也可以减少内存使用（频繁注册事件回调）、提升性能。
- 另一方面， `React` 想实现一个全浏览器的框架， 为了实现这种目标就需要提供全浏览器一致性的事件系统，以此抹平不同浏览器的差异。

## 事件合成机制流程？

[参考](https://juejin.cn/post/6955636911214067720)

> 有些特殊的事件是按照事件捕获处理的：scroll 事件、focus 事件*、*blur 事件

## React 17变更绑定位置

> **在 React 17 中，React 将不再向 `document` 附加事件处理器。而会将事件处理器附加到渲染 React 树的根 DOM 容器中：**在 React 16 或更早版本中，React 会对大多数事件执行 `document.addEventListener()`。React 17 将会在底层调用 `rootNode.addEventListener()`。
>
> ```js
> const rootNode = document.getElementById('root');
> ReactDOM.render(<App />, rootNode);
> ```

在 React 16 或更早版本中，React 会对大多数事件执行 `document.addEventListener()`。React 17 将会在底层调用 `rootNode.addEventListener()`。由于此更改，**现在可以更加安全地进行新旧版本 React 树的嵌套**。此更改还使得**将 React 嵌入使用其他技术构建的应用程序变得更加容易**。

[![A diagram showing how React 17 attaches events to the roots rather than to the document](https://zh-hans.reactjs.org/static/bb4b10114882a50090b8ff61b3c4d0fd/1e088/react_17_delegation.png)](https://zh-hans.reactjs.org/static/bb4b10114882a50090b8ff61b3c4d0fd/31868/react_17_delegation.png)

[完整参考](https://juejin.cn/post/6844903790198571021) [事件池](https://segmentfault.com/a/1190000038251163#:~:text=%E4%B8%80%E3%80%81%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%BB%8D,%E5%8E%9F%E7%94%9F%E4%BA%8B%E4%BB%B6%E7%9B%B8%E5%90%8C%E7%9A%84%E6%8E%A5%E5%8F%A3%E3%80%82) [简要概述](https://segmentfault.com/a/1190000038251163#:~:text=%E4%B8%80%E3%80%81%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%BB%8D,%E5%8E%9F%E7%94%9F%E4%BA%8B%E4%BB%B6%E7%9B%B8%E5%90%8C%E7%9A%84%E6%8E%A5%E5%8F%A3%E3%80%82) [源码](https://segmentfault.com/a/1190000039108951)

> React中捕获模式使用`onClickCapture`

> React使用了一种代理的形式在rootNode上**监听所有的事件**，在监听到事件之后会创建合成事件，然后通过模拟冒泡和捕获的流程遍历fiber树，针对遍历到目标触发元素派发合成事件。

> **不要同时使用原生事件和React合成事件**，因为在React中定义的事件回调针对的是React创建的合成事件，在这些回调中取消冒泡和默认事件都是针对合成事件，不会阻止原生事件的冒泡和默认事件。

# 37.React this.setState是同步的还是异步的

> this.setState的具体更新执行是在processUpdateQueue方法，也就是render的递阶段：
>
> ![image-20220209221547595](https://github.com/NoAlligator/pico/blob/main/img/202203271805403.png)
>
> 但是enqueue，也就是调度更新也就是调用setState发生在enqueueUpdate，它在render阶段之前，合成事件派发之后：
>
> ![image-20220209221739036](https://github.com/NoAlligator/pico/blob/main/img/202203271805404.png)
>
> 为什么在setState后面调用状态获取不到最新状态值？setState 并不是真正的异步函数，它实际上是通过队列延迟执行操作实现的，通过 isBatchingUpdates 来判断 setState 是先存进 state 队列还是直接更新。值为 true 则执行异步操作，false 则直接同步更新。对于isBatchingUpdates 为true的更新并不代表在当前作用域执行之后状态就马上更新，他只是开启react的调度并加入updateQueue，正式的更新会被调度到render的beginWork阶段（具体是updateComponent），setState后面的语句是在合成事件派发之后就执行的，还远没有到达render阶段。
>
> 为什么多次设置state得不到累加值？如果同一个周期内调用了多个setState，这些状态更新会被react批处理（**更新对象被合并，并且对相同属性的设置只保留最后一次的设置**），使用回调形式可以有效解决这个问题。此外，setState的第二个回调函数因为是在layout阶段触发的，所以必定可以访问到最新状态。
>
> 为什么在微任务、宏任务中是同步的？外部的原生事件中，并没有外层的封装与拦截，无法更新 isBatchingUpdates 的状态为 true。这就造成 isBatchingUpdates 的状态只会为 false，且立即执行。所以在 addEventListener 、setTimeout、setInterval 这些原生事件中都会同步更新。

> 注：在微任务、宏任务中执行的setState不在React批处理逻辑调度的范畴，所以可以不走批处理更新。其次，注意不引入闭包导致的无法获取最新状态： 
>
> ```jsx
> //在setTimeout中更新 +2
> this.setState({ count: this.state.count + 1 })
> this.setState({ count: this.state.count + 1 })
> 
> //在setTimeout中更新 +1
> this.setState({ count: count + 1 }) //相当于 this.setState({ count: 1 })
> this.setState({ count: count + 1 })
> ```

[参考](https://zhuanlan.zhihu.com/p/362653407)

# 38.React 使用纯函数的好处

所谓纯函数，它是这样一种函数：**即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用**。

从纯函数的定义，可以提取出纯函数的必要条件：

- 纯函数接受参数，基于参数计算，返回一个新对象；
- 不会产生副作用，计算过程不会修改输入的参数并且不修改其作用域之外的参数或方法；
- 相同的输入保证相同的输出。

纯函数的优点：

- 相同的输入必定是相同的输出，所以纯函数**可以根据输入来做缓存**；
- 相同的输入必定是相同的输出，这就保证了**引用的透明性**；
- 纯函数完全自给自足，这点的好处就是纯函数的**依赖很明确**，因此更易于观察和理解，而且让我们的测试更加容易；
- 可靠：**不用担心有副作用**，可以更好的工作；
- 代码并行：**可以并行运行任意纯函数，因为纯函数不需要访问共享的内存，而且也不会因为副作用而进入竞争态**。

> 相同的输入必定有着相同的输出 + 没有副作用

## 组件使用纯函数

React 组件应该是纯的，这意味着它的结果 `render`方法应仅依赖于 `props`和 `state` ，并且对于相同的属性和状态 `render`应该给出相同的结果。如果 render 不是纯的，则意味着它可以为相同的输入返回不同的结果，因此 React 无法根据组件的更改来判断 DOM 的哪些部分需要更新。这很关键，因为 React 的性能取决于此。

**纯函数组件的特点**:

- 组件不会被实例化，整体渲染性能得到提升；
- 组件不能访问 this 对象；
- 组件无法访问生命周期的方法；
- 无状态组件只能访问输入的 props，无副作用。

**纯函数组件的优点**：

- 无副作用；
- 占内存更小，首次 render 的性能更好；
- 语法更简洁，可读性好，逻辑简单，测试简单，代码量少，容易复用；
- 更佳的性能表现：因为函数组件中不需要进行生命周期的管理和状态管理，因此 React 并不需要进行某些特定的检查和内存分配，保证了性能。

**纯函数组件的短板：**

- 组件往往依赖数据源，不能自力更生
- 纯函数组件没有生命周期和`this`

纯函数组件被鼓励在大型项目中尽可能以简单的写法来分割原本庞大的组件，未来 React 也会像面向无状态组件一样在譬如无意义的检查和内存分配领域进行一系列优化，所以只要有可能，尽量使用无状态组件。纯函数不会产生不可预料的行为，建议合理的选择纯函数的方式书写函数。同样，在 React 组件中，如果无需本地 state 去缓存一些数据，也不需要用到生命周期函数，那么就可以把当前组件定义为纯函数组件，可读性好，且性能表现更佳。

> 纯函数组件是无状态组件。纯函数组件不会被实例化，占用内存更小、整体渲染性能更好（纯函数组件没有实例以及对应的生命周期方法，不需要进行生命周期的管理和状态管理，因此 React 并不需要进行某些特定的检查和内存分配，保证了性能）；纯函数组件的逻辑明确，不会产生不可预料地行为，还不会产生不可预料的副作用

> 在进行实践的时候，应当遵从能够纯函数化尽量纯函数化、不能够纯函数化的话考虑抽离可以被纯函数化地部分，这样使得组件化地能力达到最高。

## 纯函数编程

- 纯函数编程降低了认知负荷，因为不会对外部产生副作用，可以使得我们地注意力集中在纯函数逻辑本身而非复杂的外部依赖；
- 另一个使用纯函数的原因是测试以及重构，纯函数使得其输入输出都是可以预测的： 如果传入相同的参数，它们将始终产生相同的结果。
- 正确地使用纯函数可以产生更加高质量的代码。并且也是一种更加干净的编码方式。
- 纯函数还使得维护和重构代码变得更加容易。你可以放心地重构一个纯函数，不必操心没注意到的副作用搞乱了整个应用而导致终调试地狱。
- 如果纯函数调用纯函数，则不产生副作用依旧是纯函数。

> 有哪些场景是副作用？
>
> - 改变引用入参的函式
> - 时间性质的函式（setTimeout、setInterval、Date.now()、....）
> - 发起Ajax请求
> - 输出数据到屏幕或者控制台
> - DOM 操作
> - 产生不稳定值（Math.random()等）
>
> 在ReactJS中则是：有状态组件一般都是存在副作用的，因为状态会改变，状态的改变往往会引起视图的改变。

# 39.React生命周期

> 为啥getDerivedStateFromProps生命周期被设置成静态方法？因为React官方认为getDerivedStateFromProps必须是纯净而没有副作用的纯函数，所以将他设置为静态的方法，无法直接访问实例对象。实际上我们在使用该方法时也**不应当做副作用相关操作**，只能进行派生状态相关的计算和返回。
>
> getDerivedStateFromProps和componentWillReceiveProps有区别吗？区别在于触发的时机，getDerivedStateFromProps在父组件提供新的props、setState、forceUpdate的情况下都会被调用，而componentWillReceiveProps仅针对父组件传递的props更新的情况下才会被调用，

- constructor
  - 可以初始化内部state
  - 可以为事件处理函数绑定实例
  - 不可以副作用
- static getDerivedStateFromProps(nextProps, nextState)
  - 返回一个对象来更新state，如果返回null则不更新state，可以利用nextProps和nextState来加一些限制条件防止无用的state更新
  - 不可以进行副作用
- render()
  - class组件必须实现的方法，用于提供JSX原始文本
  - 不可以进行副作用
- componentDidMount()
  - 在layout阶段被调用，标志着DOM正式被渲染到页面上
  - 可以执行副作用：setState、网络请求、开启事件监听
- shouldComponentUpdate(nextProps, nextState)
  - 利用返回值决定使是否进行更新
  - 不可以执行副作用
  - PureComponent对于shouldComponentUpdate存在默认的实现
- getSnapshotBeforeUpdate(prevProps, prevState)
  - 在beforeMutation阶段被调用，也就是在DOM真正更新之前被调用并返回一个数据传入componentDidUpdate的第三个位置
  - 不可以执行副作用
- componentDidUpdate(prevProps, prevState, snapShot)
  - componentDidUpdate在layout阶段被调用
  - 可以执行副作用，但是要注意，使用setState必须加以限制，否则就会导致死循环。
- componentWillUnmount
  - 在组件即将被卸载的时候调用，一般执行取消网络请求、移除事件监听、清理DOM元素、清理定时器等操作防止内存泄漏。

## 父子组件mount顺序

Parent constructor()

Parent getDerivedStateFromProps()

Parent render()

Child constructor()

Child getDerivedStateFromProps()

Child render()

-------------------------------------------------------------------------------------------------------

------------ Child Update DOM & Refs ------------ 

Child componentDidMount()

------------ Parent Update DOM & Refs ------------ 

Parent componentDidMount()

## 子组件setState顺序

Child getDerivedStateFromProps()

Child shouldComponentUpdate()

Child render()

Child getSnapShotBeforeUpdate()

----------------------------------------------------------------------

------------ Child Update DOM & Refs ------------ 

Child componentDidUpdate()

## 父组件props更新

Parent： getDerivedStateFromProps()

Parent： shouldComponentUpdate()

Parent： render()

Child： getDerivedStateFromProps()

Child： shouldComponentUpdate()

Child： render()

Child： getSnapshotBeforeUpdate()

Parent： getSnapshotBeforeUpdate()

---------------------------------------------------------------------------

------------ Child Update DOM & Refs ------------ 

Child： componentDidUpdate()

------------ Parent Update DOM & Refs ------------ 

Parent： componentDidUpdate()

## 卸载子组件

Parent ： getDerivedStateFromProps()

Parent： shouldComponentUpdate()

Parent： render()

Parent： getSnapshotBeforeUpdate()

-------------------------------------------------------------------------------

**Child： componentWillUnmount()**

**Parent： componentDidUpdate()**

## 挂载子组件

Parent 组件： getDerivedStateFromProps()

Parent 组件： shouldComponentUpdate()

Parent 组件： render()

Child 组件： constructor()

Child 组件： getDerivedStateFromProps()

Child 组件： render()

**Parent 组件： getSnapshotBeforeUpdate()**

------------------------------------------------------------------------------------------------

------------ Child Remove DOM & Refs ------------ 

**Child 组件： componentDidMount()**

------------ Parent Update DOM & Refs ------------

**Parent 组件： componentDidUpdate()**

# 40.React 事件和原生事件的区别

命名不同：原生全小写，React小驼峰

阻止默认行为：原生内联模式是通过return false，React必须在事件回调中明确调用e.preventDefault()

事件本质：原生事件的触发源和事件处理函数相互绑定。React事件捕获依赖原生事件冒泡后在rootNode上触发，并创建合成对象派发给对应的fiber节点执行回调

# 41.React废弃了哪些生命周期？

componentWillUpdate、componentWillMount、componentWillReceiveProps这三个生命周期

为什么废弃？他们现在没有被废弃，在V17中仍然处于过渡的状态而被标记了unsafe，但是在未来的版本有可能会被弃用。他们的废弃主要是为了适应新出现的fiber架构中的并发模式，由于并发模式的异步可中断特性，低优先级的任务在render阶段可能被高优先级的任务打断，这样的话会导致这些生命周期被多次调用，如果在以上生命周期中的代码实现是预期一次更新仅引发一次调用，那么使用新的并发模式在运行时就不一定会按照预期只更新一次。此外，因为以往这些生命周期的存在没有很严格的约束，导致在开发中会产生和很多的反模式（**在这些render阶段的生命周期内产生副作用**），可能会**导致很多不可预料的问题**，所以打算废弃这些生命周期方法（在实际开发过程中这些生命周期方法的用途其实也不大）。

**为什么抛弃componentWillMount？**这个生命周期完全可以使用constructor（初始化）+ componentDidMount（副作用如数据获取等）代替。（除此之外，如果在 willMount 中订阅事件，但在服务端这并不会执行 willUnMount事件，也就是说服务端会导致内存泄漏所以componentWilIMount完全可以不使用，但使用者有时候难免因为各 种各样的情况在 componentWilMount中做一些操作，那么React为了约束开发者，干脆就抛掉了这个API）

**为什么抛弃componentWillReceiveProps？**在老版本的 React 中，如果组件自身的某个 state 跟其 props 密切相关的话，一直都没有一种很优雅的处理方式去更新 state，而是需要在 componentWilReceiveProps 中判断前后两个 props 是否相同，如果不同再将新的 props更新到相应的 state 上去。这样做一来会破坏 state 数据的单一数据源，导致组件状态变得不可预测。为了解决这些问题，React引入了第一个新的生命周期：getDerivedStateFromProps，他有以下优点：

- getDSFP是静态方法，在这里不能使用this，也就是一个纯函数，开发者不能写出副作用的代码
- 开发者只能通过prevState而不是prevProps来做对比，保证了state和props之间的简单关系以及不需要处理第一次渲染时prevProps为空的情况
- 基于第一点，将状态变化（setState）和昂贵操作（tabChange）区分开，更加便于 render 和 commit 阶段操作或者说优化。

为什么抛弃componentWillUpdate？与 componentWillReceiveProps 类似，许多开发者也会在 componentWillUpdate 中根据 props 的变化去触发一些回调 。 但不论是 componentWilReceiveProps 还 是 componentWilUpdate，都有可能在一次更新中被调用多次，也就是说**写在这里的回调函数也有可能会被调用多次**，这显然是不可取的。componentDidUpdate 不存在这样的问题，一次更新中 componentDidUpdate 只会被调用一次，所以将原先写在 componentWillUpdate 中 的 回 调 迁 移 至 componentDidUpdate 就可以解决这个问题。另外一种情况则是需要获取之前的状态，但是由于在fiber中，render可打断，可能在componentWillUpdate中获取到的状态很可能与实际需要的不同（高优先级插队导致），这个通常可以使用第二个新增的生命函数的解决 getSnapshotBeforeUpdate(prevProps, prevState)，getSnapshotBeforeUpdate进入了commit阶段，所以不会被打断。

# 42.组件通信方式

## 父子

父传子：父组件传递props

子 传父：父组件传递带状态更新的回调props，子组件调用回调

## 深层次关系

1. props透传
2. context

## 非嵌套关系

1. 简单的订阅发布
2. redux状态管理
3. 如果是兄弟组件间通信，可以考虑使用状态提升

## props层级过深怎么解决？

- 使用context
- 使用redux

## 总结

- **⽗组件向⼦组件通讯**: ⽗组件可以向⼦组件通过传 props 的⽅式，向⼦组件进⾏通讯
- **⼦组件向⽗组件通讯**: props+回调的⽅式，⽗组件向⼦组件传递props进⾏通讯，此props为作⽤域为⽗组件⾃身的函 数，⼦组件调⽤该函数，将⼦组件想要传递的信息，作为参数，传递到⽗组件的作⽤域中
- **兄弟组件通信**: 找到这两个兄弟节点共同的⽗节点,结合上⾯两种⽅式由⽗节点转发信息进⾏通信
- **跨层级通信**: Context 设计⽬的是为了共享那些对于⼀个组件树⽽⾔是“全局”的数据，例如当前认证的⽤户、主题或⾸选语⾔，对于跨越多层的全局数据通过 Context 通信再适合不过
- **发布订阅模式**: 发布者发布事件，订阅者监听事件并做出反应,我们可以通过引⼊event模块进⾏通信
- **全局状态管理⼯具**: 借助Redux或者Mobx等全局状态管理⼯具进⾏通信,这种⼯具会维护⼀个全局状态中⼼Store,并根据不同的事件产⽣新的状态

## [所有通信方法](https://segmentfault.com/a/1190000023585646)

- 父 -> 子：父组件传递props
- 父 -> 子：父组件通过ref获取组件实例并直接调用子组件上的方法（很管用）
- 子 -> 父：父组件传递回调props
- 子 -> 父：事件冒泡
- 兄弟间：状态提升
- 全局属性：Context
- Portals
- 发布订阅
- Redux

# 43.Portals什么作用

Portals只是DOM层面的**外挂**（尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 *React 树*， 且与 *DOM 树* 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 *React 树*的祖先，即便这些元素并不是 *DOM 树* 中的祖先。在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 `<Modal />` 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件）

```jsx
render() {
  // React 并*没有*创建一个新的 div。它只是把子元素渲染到 `domNode` 中。
  // `domNode` 是一个可以在任何位置的有效 DOM 节点。
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```



# 44.React样式方案

## 对象内联样式 

> 内联样式就是在JSX元素中，直接定义行内的样式；
>
> 避免了组件间冲突；不能使用伪类；不能使用媒体查询

内联样式的优点：

- **使用简单：** 使用内联样式的好处就是简单的以组件为中心来实现样式的添加；
- **扩展方便：** 通过使用对象进行样式设置，可以方便的扩展对象来扩展样式；
- **避免冲突：** 样式通过对象的形式定义在组件中，避免了和其他样式的冲突。



在大型项目中，内联样式可能并不是一个很好的选择，因为内联样式还是有局限性的：

- **不能使用伪类：** 这意味着 :hover、:focus、:actived、:visited等都将无法使用；
- **不能使用媒体查询：** 媒体查询相关的属性不能使用。
- **减低代码可读性：** 如果使用很多的样式，代码的可读性将大大降低。
- **没有代码提示**：当使用对象来定义样式时，是没有代码提示的，所以如果拼错样式属性，也很难检查出来。

## 引入CSS样式表

> 这也是我们最常用的样式策略，使用单独的样式表，使用CSS或者SCSS等来为元素设置样式；
>
> 实现了JS和CSS的分离；容易发生冲突；

CSS样式表的优点：

- **关注分离：** 实现了样式和JavaScript的分离，像往常一样编写CSS语法；
- **使用CSS所有功能：** 此方法允许我们使用CSS的任何语法，包括伪类、媒体查询等；
- **缓存和性能：** 标准的CSS文件有利于浏览器优化，在本地缓存文件从而允许重复的方法，以提高性能；
- **易编写**：CSS样式表在书写时会有代码提示；

当然，CSS样式表也是有缺点的：

- **产生冲突：** CSS选择器都具有相同的全局作用域，如果选择器使用错误，有可能造成样式的冲突；
- **可读性：** 如果结构不正确，随着应用程序变得复杂，CSS或者SASS样式表就的很长，并且难以阅读；
- **难以整理：** 随着样式表变得越来越复杂，一些旧的或者未使用的样式属性就很难清理；
- **没有真正的动态样式：** 在CSS表中难以实现动态设置样式。

## **CSS Module**

> CSS模块是一个文件，默认情况下所有类名和动画名都在本地范围；

优点：

- **模块化：** 实现样式的模块化，避免样式冲突；
- **SSR时无重复：** 在使用服务端渲染（SSR）时没有代码的重复；

缺点：

- **额外的构建工具：** 需要额外的构建工具（webpack）；
- **书写麻烦：** 本地模块和全局模块书写起来很麻烦；
- **驼峰命名：** 只允许使用驼峰命名。

## **styled-components**

> 这是一个用于React和React Native的样式组件库，它允许我们在应用中使用组件级样式，这些样式就是使用CSS-in-JS的技术来编写的；

style-components的优点：

- **开箱即用的Sass语法**：在style-components支持开箱即用的Sass语法，而不需要进行额外的安装或设置；
- **支持使用主题：** 在styled-components 提供了一个ThemeContext，可以直接传递主题对象的方法，方便使用全局主题；
- **动态样式：** 可以使用props来动态的设置和改变样式；
- **没有类型冲突：** 样式组件会我们生成唯一的类名，不会与其他组件的类型产生冲突；
- **便于维护：** 我们不需要在各种样式文件中查看样式，只需要在样式组件中查看其样式即可，便于维护；

style-components的缺点：

- **影响性能：** style-components在构建时会将React 组件中所有的样式定义转化为纯CSS，并将内容注入到index.html文件的`<style>`标签中。这样不仅增加了HTML文件的大小，并且无法对输出的CSS进行分块，影响了应用的性能；

## **JSS**

> JSS是一个CSS创作工具，它允许我们使用JavaScript以声明式、无冲突和可重复的方式来描述样式。

[参考](https://juejin.cn/post/7041745627323056142)

# 45.React有什么特点

1. 它使用`虚拟DOM`而不是真正的DOM。
2. 它可以用服务器端渲染。
3. 它遵循单向数据流或数据绑定。

# 46.React的主要优点

1. 它提高了应用的性能
2. 可以方便地在客户端和服务器端使用（SSR）
3. 由于 JSX，代码的可读性很好
4. React 很容易与 Meteor，Angular 等其他框架集成
5. 使用React，编写UI测试用例变得非常容易

# 47.React有哪些限制

1. **React 只是一个库，而不是一个完整的框架**
2. 它的库非常庞大，需要时间来理解
3. 新手程序员可能很难理解
4. 编码变得复杂，因为它使用内联模板和 JSX

# 48.Virtual DOM 的工作原理

在React中的虚拟DOM指的是fiber节点，除了某些根节点等的fiber节点没有对应的真实DOM节点，其他fiber节点都对应了真实DOM元素。fiber节点是通过解析JSX生成的React Element再经过React的reconcile阶段生成的，每一个fiber节点都包含了对应的节点信息，比如：alternate、child、firstEffect、props、state、ref、return、sibling、stateNode、key、updateQueue、type。

![image-20220210222853149](https://github.com/NoAlligator/pico/blob/main/img/202203271805406.png)

Virtual DOM 是一个轻量级的 JavaScript 对象，它最初只是 real DOM 的副本。它是一个节点树，它将元素、它们的属性和内容作为对象及其属性。 React 的渲染函数从 React 组件中创建一个节点树。然后它响应数据模型中的变化来更新该树，该变化是由用户或系统完成的各种动作引起的。

Virtual DOM 工作过程有三个简单的步骤。

1. 每当底层数据发生改变时，整个 UI 都将在 Virtual DOM 描述中重新渲染。
2. 然后计算之前 DOM 表示与新表示的之间的差异。
3. 完成计算后，将只用实际更改的内容更新 real DOM。

# 49.react diff 原理

详见React源码文档

# 50.React Vs Vue

## 共同点

- 使用 Virtual DOM
- 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
- 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。

## 优化

在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 `PureComponent`，或是手动实现 `shouldComponentUpdate` 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。然而，使用 `PureComponent` 和 `shouldComponentUpdate` 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 React 中的组件优化伴随着相当的心智负担。

在 Vue 应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。你可以理解为每一个组件都已经自动获得了 `shouldComponentUpdate`，并且没有上述的子树问题限制。Vue 的这个特点使得开发者不再需要考虑此类优化，从而能够更好地专注于应用本身。

## HTML

使用 JSX 的渲染函数有下面这些优势：

- 你可以使用完整的编程语言 JavaScript 功能来构建你的视图页面。比如你可以使用临时变量、JS 自带的流程控制、以及直接引用当前 JS 作用域中的值等等。
- 开发工具对 JSX 的支持相比于现有可用的其他 Vue 模板还是比较先进的 (比如，linting、类型检查、编辑器的自动完成)。

事实上 Vue 也提供了[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)，甚至[支持 JSX](https://cn.vuejs.org/v2/guide/render-function.html#JSX)。然而，我们默认推荐的还是模板。任何合乎规范的 HTML 都是合法的 Vue 模板，这也带来了一些特有的优势：

- 对于很多习惯了 HTML 的开发者来说，模板比起 JSX 读写起来更自然。这里当然有主观偏好的成分，但如果这种区别会导致开发效率的提升，那么它就有客观的价值存在。
- 基于 HTML 的模板使得将已有的应用逐步迁移到 Vue 更为容易。
- 这也使得设计师和新人开发者更容易理解和参与到项目中。
- 你甚至可以使用其他模板预处理器，比如 Pug 来书写 Vue 的模板。

更抽象一点来看，我们可以把组件区分为两类：一类是偏视图表现的 (presentational)，一类则是偏逻辑的 (logical)。我们推荐在前者中使用模板，在后者中使用 JSX 或渲染函数。这两类组件的比例会根据应用类型的不同有所变化，但整体来说我们发现表现类的组件远远多于逻辑类组件。

## CSS

除非你把组件分布在多个文件上 (例如 [CSS Modules](https://github.com/gajus/react-css-modules))，CSS 作用域在 React 中是通过 CSS-in-JS 的方案实现的 (比如 [styled-components](https://github.com/styled-components/styled-components) 和 [emotion](https://github.com/emotion-js/emotion))。这引入了一个新的面向组件的样式范例，它和普通的 CSS 撰写过程是有区别的。另外，虽然在构建时将 CSS 提取到一个单独的样式表是支持的，但 bundle 里通常还是需要一个运行时程序来让这些样式生效。当你能够利用 JavaScript 灵活处理样式的同时，也需要权衡 bundle 的尺寸和运行时的开销。

如果你是一个 CSS-in-JS 的爱好者，许多主流的 CSS-in-JS 库也都支持 Vue (比如 [styled-components-vue](https://github.com/styled-components/vue-styled-components) 和 [vue-emotion](https://github.com/egoist/vue-emotion))。这里 React 和 Vue 主要的区别是，Vue 设置样式的默认方法是[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)里类似 `style` 的标签。

[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)让你可以在同一个文件里完全控制 CSS，将其作为组件代码的一部分。

```vue
<style scoped>
  @media (min-width: 250px) {
    .list-container:hover {
      background: orange;
    }
  }
</style>
```

这个可选 `scoped` attribute 会自动添加一个唯一的 attribute (比如 `data-v-21e5b78`) 为组件内 CSS 指定作用域，编译的时候 `.list-container:hover` 会被编译成类似 `.list-container[data-v-21e5b78]:hover`。

最后，Vue 的单文件组件里的样式设置是非常灵活的。通过 [vue-loader](https://github.com/vuejs/vue-loader)，你可以使用任意预处理器、后处理器，甚至深度集成 [CSS Modules](https://vue-loader.vuejs.org/en/features/css-modules.html)——全部都在 `<style>` 标签内。

## 我如何修改数据？

- Vue：使用响应式数据（利用了代理Proxy/Object.defineObject），修改数据是在原数据上直接修改被代理捕获；
- React：使用 setState 方法主动触发数据更新，并且强调数据的不变性。

## 框架如何发现数据被修改？

- Vue：使用 [Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 或者`Proxy`，劫持 `setter、getter` 实现数据监听和比较；
- React：使用`Object.is()`进行对比。

## 我如何发现数据被修改？

- Vue：使用 watcher 侦听变化；
- React：可以在 commit 阶段对比状态改变前后区别。

## 框架何时渲染修改的数据？我如何知道已经渲染好了？

- Vue：在适当的时候渲染，你通过使用 watcher，或者 computed 属性发现
- React：setState 调用后在适当的时候重新渲染，并调用相关生命周期钩子

Vue 的数据对象相比 React 的状态对象在代码膨胀的时候差距就来了。代码少的时候 Vue 的写法更为简洁，但组件状态很多，需要明确数据更新逻辑时，React 简单的 setState({}, callback)，就搞定了，Vue 有点让人摸不到头脑。React 的不可变（immutable）状态在**应用复杂时表现出的透明、可测试性更佳**。

## 模板语法对比JSX

Vue 的单文件组件，使用 `<template>` 、`<script>` 对代码进行分割，直接导致的问题就是上下文丢失。

举个例子，你封装了一些常用的函数，在 Vue 文件中 import 进来。你这个函数不能在 template 中直接使用。你只能将方法定义在methods中，再引用进来。模板语法并不知道你有 isNickname 这个函数，简单的操作多了 3 行代码。

## 模板分割

React的JSX语法使得模板分割更加自然、简单，因此也更容易将无状态组件抽离出来成为单独一部分以提高性能。

> 好的代码组织能将常变与不变的部分进行分割解耦

## api & 生态

React生态更加繁荣，React api相对更少，可以发挥的空间更大，很多场景下没有所谓的最佳实践，需要选择成本。此外；

Vue生态相对不繁荣，内置了更多的API让开发者更容易上手，这使得使用Vue减少了很多选择成本，因为像状态管理和路由都有和Vue同步更新的库，往往不需要其他库来实现这方面的功能。

相对Vue来说，React只是一个视图框架，正如官网所说的UI = render(props)，React只能充当视图层，而Vue借鉴了MVVM模型，在此基础上实现了View-Model层的完整功能。

## 向上扩展

Vue 和 React 都提供了强大的路由来应对大型应用。React 社区在状态管理方面非常有创新精神 (比如 Flux、Redux)，而这些状态管理模式甚至 [Redux 本身](https://yarnpkg.com/en/packages?q=redux vue&p=1)也可以非常容易的集成在 Vue 应用中。实际上，Vue 更进一步地采用了这种模式 ([Vuex](https://github.com/vuejs/vuex))，更加深入集成 Vue 的状态管理解决方案 Vuex 相信能为你带来更好的开发体验。

两者另一个重要差异是，Vue 的路由库和状态管理库都是由官方维护支持且与核心库同步更新的。React 则是选择把这些问题交给社区维护，因此创建了一个更分散的生态系统。但相对的，React 的生态系统相比 Vue 更加繁荣。

最后，Vue 提供了 [CLI 脚手架](https://github.com/vuejs/vue-cli)，能让你通过交互式的脚手架引导非常容易地构建项目。你甚至可以使用它[快速开发组件的原型](https://cli.vuejs.org/zh/guide/prototyping.html#快速原型开发)。React 在这方面也提供了 [create-react-app](https://github.com/facebookincubator/create-react-app)，但是现在还存在一些局限性：

- 它不允许在项目生成时进行任何配置，而 Vue CLI 运行于可升级的运行时依赖之上，该运行时可以通过[插件](https://cli.vuejs.org/zh/guide/plugins-and-presets.html#插件)进行扩展。
- 它只提供一个构建单页面应用的默认选项，而 Vue 提供了各种用途的模板。
- 它不能用用户自建的[预设配置](https://cli.vuejs.org/zh/guide/plugins-and-presets.html#预设配置)构建项目，这对企业环境下预先建立约定是特别有用的。

而要注意的是这些限制是故意设计的，这有它的优势。例如，如果你的项目需求非常简单，你就不需要自定义生成过程。你能把它作为一个依赖来更新。如果阅读更多关于[不同的设计理念](https://github.com/facebookincubator/create-react-app#philosophy)。

## 向下扩展

React 学习曲线陡峭，在你开始学 React 前，你需要知道 JSX 和 ES2015，因为许多示例用的是这些语法。你需要学习构建系统，虽然你在技术上可以用 Babel 来实时编译代码，但是这并不推荐用于生产环境。

就像 Vue 向上扩展好比 React 一样，Vue 向下扩展后就类似于 jQuery。你只要把如下标签放到页面就可以运行：

```vue
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
```

然后你就可以编写 Vue 代码并应用到生产中，你只要用 min 版 Vue 文件替换掉就不用担心其他的性能问题。由于起步阶段不需学 JSX，ES2015 以及构建系统，所以开发者只需不到一天的时间阅读[指南](https://cn.vuejs.org/v2/guide/)就可以建立简单的应用程序。

## 原生渲染

React Native 能使你用相同的组件模型编写有本地渲染能力的 APP (iOS 和 Android)。能同时跨多平台开发，对开发者是非常棒的。相应地，Vue 和 [Weex](https://weex.apache.org/) 会进行官方合作，Weex 是阿里巴巴发起的跨平台用户界面开发框架，同时也正在 Apache 基金会进行项目孵化，Weex 允许你使用 Vue 语法开发不仅仅可以运行在浏览器端，还能被用于开发 iOS 和 Android 上的原生应用的组件。在现在，Weex 还在积极发展，成熟度也不能和 React Native 相抗衡。但是，Weex 的发展是由世界上最大的电子商务企业的需求在驱动，Vue 团队也会和 Weex 团队积极合作确保为开发者带来良好的开发体验。

[详细区别](https://juejin.cn/post/6847009771355127822)

# 51.谈一谈React

React 是一个用于构建用户界面的JavaScript库。React 主要用于构建 UI，很多人认为 React 是 MVC 中的 V（视图）

React 特点有：

1. 声明式设计 −React 采用声明范式，可以轻松描述应用。
2. 高效 −React 通过对 DOM 的模拟，最大限度地减少与 DOM 的交互。灵活 −React 可以与已知的库或框架很好地配合。
3. JSX − JSX 是 JavaScript 语法的扩展。React 开发不一定使用 JSX ，但我们建议使用它。
4. 组件 − 通过 React 构建组件，使得代码更加容易得到复用，能够很好的应用在大项目的开发中。
5. 单向响应的数据流 − React 实现了单向响应的数据流，从而减少了重复代码， 这也是它为什么比传统数据绑定更简单。

# 52.React 单向数据流

在 React 中，数据是单向流动的，是从上向下的方向，即从父组件到子组件的方向。state 和 props 是其中重要的概念，如果顶层组件初始化 props，那么 React 会向下遍历整颗组件树，重新渲染相关的子组件。其中 state 表示的是每个组件中内部的的状态，这些状态只在组件内部改变。把组件看成是一个函数，那么他接受 props 作为参数，内部由 state 作为函数的内部参数，返回一个虚拟 dom 的实现。

# 53.怎么获取真正的 dom

- ReactDOM.findDOMNode()
- this.refs

# 54.React 高阶组件、Render props、hooks 有什么区别，为什么要不断迭代

这三者是目前react解决代码复用的主要方式。

高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。具体而言，高阶组件是参数为组件，返回值为新组件的函数。HOC是一种组件的设计模式，HOC接受一个组件和额外的参数（如果需要），返回一个新的组件。HOC 是纯函数，没有副作用。

hoc的优缺点∶

- 优点∶ 逻辑复用、不影响被包裹组件的内部逻辑。
- 缺点∶ hoc传递给被包裹组件的props容易和被包裹后的组件重名，进而被覆盖

render props是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术，更具体的说，render prop 是一个用于告知组件需要渲染什么内容的函数 prop。

render props的优缺点∶

- 优点：数据共享、代码复用，将组件内的state作为props传递给调用者，将渲染逻辑交给调用者。
- 缺点：无法在 return 语句外访问数据、嵌套写法不够优雅

通常，render props 和高阶组件只渲染一个子节点。让 Hook 来服务这个使用场景更加简单。这两种模式仍有用武之地，（例如，一个虚拟滚动条组件或许会有一个 renderltem 属性，或是一个可见的容器组件或许会有它自己的 DOM 结构）。但在大部分场景下，Hook 足够了，并且能够帮助减少嵌套。

hook的优点∶ 

- 使用直观 
- 解决hoc的prop 重名问题 
- 解决render props 因共享数据 而出现嵌套地狱的问题 
- 能在return之外使用数据的问题 

需要注意的是∶hook只能在组件顶层使用，不可在分支语句中使用。

# 55.React.Component 和 React.PureComponent 的区别



PureComponent表示一个纯组件，可以用来优化React程序，减少render函数执行的次数，从而提高组件的性能。

在React中，当prop或者state发生变化时，可以通过在shouldComponentUpdate生命周期函数中执行return false来阻止页面的更新，从而减少不必要的render执行。React.PureComponent会自动执行 shouldComponentUpdate。

不过，pureComponent中的 shouldComponentUpdate() 进行的是**浅比较**，也就是说如果是引用数据类型的数据，只会比较不是同一个地址，而不会比较这个地址里面的数据是否一致。浅比较会忽略属性和或状态突变情况，其实也就是数据引用指针没有变化，而数据发生改变的时候render是不会执行的。如果需要重新渲染那么就需要重新开辟空间引用数据。PureComponent一般会用在一些纯展示组件上。

使用pureComponent的**好处**：当组件更新时，如果组件的props或者state都没有改变，render函数就不会触发。省去虚拟DOM的生成和对比过程，达到提升性能的目的。这是因为react自动做了一层浅比较。

# 56.Component, Element, Instance 之间有什么区别和联系？

**元素：** 一个元素`element`是一个普通对象(plain object)，描述了对于一个DOM节点或者其他组件`component`，你想让它在屏幕上呈现成什么样子。元素`element`可以在它的属性`props`中包含其他元素(译注：用于形成元素树)。创建一个React元素`element`成本很低。元素`element`创建之后是不可变的。

**组件：** 一个组件`component`可以通过多种方式声明。可以是带有一个`render()`方法的类，简单点也可以定义为一个函数。这两种情况下，它都把属性`props`作为输入，把返回的一棵元素树作为输出。

**实例：** 一个实例`instance`是你在所写的组件类`component class`中使用关键字`this`所指向的东西(译注：组件实例)。它用来存储本地状态和响应生命周期事件很有用。组件实例在mount阶段是在constructClassInstance这个方法创建的，然后它会被被挂载到对应的class component fiber节点的stateNode上面。

![image-20220211130555815](https://github.com/NoAlligator/pico/blob/main/img/202203271805407.png)

![image-20220211130948065](https://github.com/NoAlligator/pico/blob/main/img/202203271805408.png)

![image-20220211130933663](https://github.com/NoAlligator/pico/blob/main/img/202203271805409.png)

![image-20220211130642403](https://github.com/NoAlligator/pico/blob/main/img/202203271805410.png)



函数式组件(`Functional component`)根本没有实例`instance`。类组件(`Class component`)有实例`instance`，但是永远也不需要直接创建一个组件的实例，因为React帮我们做了这些。

# 57.React.createClass和extends Component的区别有哪些？


React.createClass和extends Component的区别主要在于：
**（1）语法区别**

- createClass本质上是一个工厂函数，extends的方式更加接近最新的ES6规范的class写法。两种方式在语法上的差别主要体现在方法的定义和静态属性的声明上。 
- createClass方式的方法定义使用逗号，隔开，因为creatClass本质上是一个函数，传递给它的是一个Object；而class的方式定义方法时务必谨记不要使用逗号隔开，这是ES6 class的语法规范。 

**（2）propType 和 getDefaultProps**

- React.createClass：通过proTypes对象和getDefaultProps()方法来设置和获取props. 
- React.Component：**通过设置两个属性propTypes和defaultProps** 

**（3）状态的区别**

- React.createClass：通过getInitialState()方法返回一个包含初始值的对象 
- React.Component：**通过constructor设置初始状态** 

**（4）this区别**

- React.createClass：会正确绑定this 
- React.Component：**由于使用了 ES6，这里会有些微不同，属性并不会自动绑定到 React 类的实例上。** 

**（5）Mixins**

- React.createClass 的话，可以在创建组件时添加一个叫做 mixins 的属性，并将可供混合的类的集合以数组的形式赋给 mixins。 
- **如果使用 ES6 的方式来创建组件，那么 `React mixins` 的特性将不能被使用了。**

# 58.React如何判断什么时候重新渲染组件？

- 父组件更新；
- this.setState()；
- render()；
- useState()'s setState；
- useReducer()'s dispatch；

组件状态的改变可以因为`props`的改变，或者直接通过`setState`方法改变。组件获得新的状态然后React决定是否应该重新渲染组件。只要组件的state发生变化，React就会对组件进行重新渲染。这是因为React中的`shouldComponentUpdate`方法默认返回`true`，这就是导致每次更新都重新渲染的原因。当React将要渲染组件时会执行`shouldComponentUpdate`方法来看它是否返回`true`（组件应该更新，也就是重新渲染）。所以需要重写`shouldComponentUpdate`方法让它根据情况返回`true`或者`false`来告诉React什么时候重新渲染什么时候跳过重新渲染。

# 59.对React中Fragment的理解，它的使用场景是什么？

在React中，组件返回的元素只能有一个根元素。为了不添加多余的DOM节点，我们可以使用Fragment标签来包裹所有的元素，Fragment标签不会渲染出任何元素。React官方对Fragment的解释：

> React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点。

# 60.React中可以在render访问refs吗？为什么？

不可以，因为无论是在mount还是update阶段，refs的绑定或者更新都是在layout阶段（也就是mutation之后）的commitAttachRef中进行的，只有在这个阶段不管是获取HostComponent还是组件的ref才是安全的。

> 注意：**函数式组件不能获取ref，因为他没有实例**，只能使用forward ref来转发函数式组件内部的ref。

# 61.对 React context 的理解

在React中，数据传递一般使用props传递数据，维持单向数据流，这样可以让组件之间的关系变得简单且可预测，但是单项数据流在某些场景中并不适用。单纯一对的父子组件传递并无问题，但要是组件之间层层依赖深入，props就需要层层传递显然，这样做太繁琐了。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。可以把context当做是特定一个组件树内共享的store，用来做数据传递。**简单说就是，当你不想在组件树中通过逐层传递props或者state的方式来传递数据时，可以使用Context来实现跨层级的组件数据传递。**

JS的代码块在执行期间，会创建一个相应的作用域链，这个作用域链记录着运行时JS代码块执行期间所能访问的活动对象，包括变量和函数，JS程序通过作用域链访问到代码块内部或者外部的变量和函数。假如以JS的作用域链作为类比，React组件提供的Context对象其实就好比一个提供给子组件访问的作用域，而 Context对象的属性可以看成作用域上的活动对象。由于组件 的 Context 由其父节点链上所有组件通 过 getChildContext（）返回的Context对象组合而成，所以，组件通过Context是可以访问到其父组件链上所有节点组件提供的Context的属性。

> Context能够很好的提供哪些需要跨越很多不同层级被使用并且透传很麻烦的状态，Context是这样使用的，首先调用createContext在内存中生成一个上下文并设置一个默认值，然后使用Context.Provider注入Context并设置value，这个value值由组件状态提供，然后对于类式组件，如果需要指定Context的话通过添加静态属性contextType并赋值为对应的Context来使用，然后就可以调用组件实例上的Context字段获取Context值了，其次可以通过Consumer直接消费多个Context。value**不应当设置为引用类型**，会导致每一次更新value都被重新构建而引发diff更新。

# 62.受控组件和非受控组件

**（1）受控组件**
在使用表单来收集用户输入时，例如input、select、textearea等元素都要绑定一个change事件，当表单的状态发生变化，就会触发onChange事件，更新组件的state。这种组件在React中被称为**受控组件**，在受控组件中，组件渲染出的状态与它的value或checked属性相对应，react通过这种方式消除了组件的局部状态，使整个状态可控。react官方同样推荐使用受控表单组件。 

受控组件更新state的流程：

- 可以通过初始state中设置表单的默认值 
- 每当表单的值发生变化时，调用onChange事件处理器 
- 事件处理器通过事件对象e拿到改变后的状态，并更新组件的state 
- 一旦通过setState方法更新state，就会触发视图的重新渲染，完成表单组件的更新 

**受控组件缺陷：**
表单元素的值都是由React组件进行管理，当有多个输入框，或者多个这种组件时，如果想同时获取到全部的值就必须每个都要编写事件处理函数，这会让代码看着很臃肿，所以为了解决这种情况，出现了非受控组件。

**（2）非受控组件**
如果一个表单组件没有value props（单选和复选按钮对应的是checked props）时，就可以称为非受控组件。在非受控组件中，可以使用一个ref来从DOM获得表单值。而不是为每个状态更新编写一个事件处理程序。

React官方的解释：

> 在 React 渲染生命周期时，表单元素上的 `value` 将会覆盖 DOM 节点中的值。在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 `defaultValue` 属性，而不是 `value`。在一个组件已经挂载之后去更新 `defaultValue` 属性的值，不会造成 DOM 上值的任何更新。

> 要编写一个非受控组件，而不是为每个状态更新都编写数据处理函数，你可以使用 ref来从 DOM 节点中获取表单数据。

> 因为非受控组件将真实数据储存在 DOM 节点中，所以在使用非受控组件时，有时候反而更容易同时集成 React 和非 React 代码。如果你不介意代码美观性，并且希望快速编写代码，使用非受控组件往往可以减少你的代码量。否则，你应该使用受控组件。

![img](https://s2.51cto.com/oss/202107/09/d436796c87305f30133267aa3be7bec0.jpg)

#  63.React中refs的作用是什么？有哪些应用场景？

Refs 提供了一种方式，用于访问在 render 方法中创建的 React 元素或 DOM 节点。Refs 应该谨慎使用，如下场景使用 Refs 比较适合：

- **处理焦点、获取文本或者媒体的控制**
- 触发必要的动画
- **集成第三方 DOM 库**

`ref` 的返回值取决于节点的类型： 

- 当 `ref` 属性被用于一个普通的 HTML 元素时，`React.createRef()` 将接收底层 DOM 元素作为他的 `current` 属性以创建 `ref`。 
- 当 `ref` 属性被用于一个自定义的类组件时，`ref` 对象将接收该组件已挂载的实例作为他的 `current`。 

当在父组件中需要访问子组件中的 `ref` 时可使用传递 Refs 或回调 Refs。

# 64.类组件与函数组件有什么异同？

**相同点：**
组件是 React 可复用的最小代码片段，它们会返回要在页面中渲染的 React 元素。也正因为组件是 React 的最小编码单位，所以无论是函数组件还是类组件，在使用方式和最终呈现效果上都是完全一致的。我们甚至可以将一个类组件改写成函数组件，或者把函数组件改写成一个类组件（虽然并不推荐这种重构行为）。从使用者的角度而言，很难从使用体验上区分两者，而且在现代浏览器中，闭包和类的性能只在极端场景下才会有明显的差别。所以，基本可认为两者作为组件是完全一致的。

**不同点：**

- 它们在开发时的心智模型上却存在巨大的差异**。类组件是基于面向对象编程的，它主打的是继承、生命周期等核心概念；而函数组件内核是函数式编程，主打的是 immutable、没有副作用、引用透明等特点。** 
- 之前，在使用场景上，如果存在需要使用生命周期的组件，那么主推类组件；设计模式上，如果需要使用继承，那么主推类组件。但现在**由于 React Hooks 的推出，生命周期概念的淡出，函数组件可以完全取代类组件。其次继承并不是组件最佳的设计模式，官方更推崇“组合优于继承”的设计概念，所以类组件在这方面的优势也在淡出。** 
- 性能优化上，**类组件主要依靠 shouldComponentUpdate 阻断渲染来提升性能，而函数组件依靠 React.memo 缓存渲染结果来提升性能。** 
- 从上手程度而言，**类组件更容易上手，从未来趋势上看，由于React Hooks 的推出，函数组件成了社区未来主推的方案。** 
- **类组件在未来时间切片与并发模式中，由于生命周期带来的复杂度，并不易于优化。而函数组件本身轻量简单，且在 Hooks 的基础上提供了比原先更细粒度的逻辑组织与复用，更能适应 React 的未来发展**

# 65.React组件的state和props有什么区别？

**（1）props**
props是一个从外部传进组件的参数，主要作为就是从父组件向子组件传递数据，它具有可读性和不变性，只能通过外部组件主动传入新的props来重新渲染子组件，否则子组件的props以及展现形式不会改变。
**（2）state**
state的主要作用是用于组件保存、控制以及修改自己的状态，它只能在constructor中初始化，它算是组件的私有属性，不可通过外部访问和修改，只能通过组件内部的this.setState来修改，修改state属性会导致组件的重新渲染。
**（3）区别**

- props 是传递给组件的（类似于函数的形参），而state 是在组件内被组件自己管理的（类似于在一个函数内声明的变量）。 
- props 是不可修改的，所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。 
- state 是在组件中创建的，一般在 constructor中初始化 state。state 是多变的、可以修改，每次setState都异步更新的。

# 66.React中的props为什么是只读的？

- `this.props`是组件之间沟通的一个接口，原则上来讲，它只能从父组件流向子组件。React具有浓重的函数式编程的思想。 

提到函数式编程就要提一个概念：纯函数。它有几个特点：

- 给定相同的输入，总是返回相同的输出。 
- 过程没有副作用。 
- 不依赖外部状态。 

`this.props`就是汲取了纯函数的思想。props的不可以变性就保证的相同的输入，页面显示的内容是一样的，并且不会产生副作用

# 67.React中怎么检验props？验证props的目的是什么？

**React**为我们提供了**PropTypes**以供验证使用。当我们向**Props**传入的数据无效（向Props传入的数据类型和验证的数据类型不符）就会在控制台发出警告信息。它可以避免随着应用越来越复杂从而出现的问题。并且，它还可以让程序变得更易读。当然，如果项目汇中使用了TypeScript，那么就可以不用PropTypes来校验，而使用TypeScript定义接口来校验props。

TS / Prop-types 区别

侧重点不同
 PropTypes是组件接收prop的约束。
 TypeScript类型约束主要是参数传递以及返回值的约束，两个东西侧重点不一样

部分相似功能
 通常我们编写一个 react 组件的时候，我们会去定义一个 prop-types 去校验我们的 class 的参数输入。而 ts 的 interface 的作用当然也是校验 props 的输入。

区别
TypeScrip 的类型检查是静态的，prop-types 可以在运行时进行检查。你如你传了个offsetTop="abc"，你的编辑器可能会提示你类型有误，但是在浏览器里仍然是可以正常运行的。而如果你使用了 prop-types，在浏览器里就会给出提示。

# 68.对 React Hook 的理解，它的实现原理是什么？

React 组件本身的定位就是函数，一个输入数据、输出 UI 的函数。作为开发者，我们编写的是声明式的代码，而 React 框架的主要工作，就是及时地把声明式的代码转换为命令式的 DOM 操作，把数据层面的描述映射到用户可见的 UI 变化中去。这就意味着从原则上来讲，React 的数据应该总是紧紧地和渲染绑定在一起的，而类组件做不到这一点。**函数组件就真正地将数据和渲染绑定到了一起。函数组件是一个更加匹配其设计理念、也更有利于逻辑拆分与重用的组件表达形式。**函数组件比起类组件少了很多东西，比如生命周期、对 state 的管理等。这就给函数组件的使用带来了非常多的局限性，导致我们并不能使用函数这种形式，写出一个真正的全功能的组件。而React-Hooks 的出现，就是为了帮助函数组件补齐这些（相对于类组件来说）缺失的能力。

# 69.React Hooks 解决了哪些问题？

**（1）在组件之间复用状态逻辑很难**
React 没有提供将可复用性行为“附加”到组件的途径（例如，把组件连接到 store）解决此类问题可以使用 render props ，HOC 和 mixins。但是这类方案需要重新组织组件结构，这可能会很麻烦，并且会使代码难以理解。由 providers，consumers，高阶组件，render props 等其他抽象层组成的组件会形成“嵌套地狱”。尽管可以在 DevTools 过滤掉它们，但这说明了一个更深层次的问题：React 需要为共享状态逻辑提供更好的原生途径。

可以使用 Hook 从组件中提取状态逻辑，使得这些逻辑可以单独测试并复用。Hook 使我们在无需修改组件结构的情况下复用状态逻辑。 这使得在组件间或社区内共享 Hook 变得更便捷。
**（2）复杂组件变得难以理解**
在组件中，每个生命周期常常包含一些不相关的逻辑。例如，组件常常在 componentDidMount 和 componentDidUpdate 中获取数据。但是，同一个 componentDidMount 中可能也包含很多其它的逻辑，如设置事件监听，而之后需在 componentWillUnmount 中清除。相互关联且需要对照修改的代码被进行了拆分，而完全不相关的代码却在同一个方法中组合在一起。如此很容易产生 bug，并且导致逻辑不一致。
在多数情况下，不可能将组件拆分为更小的粒度，因为状态逻辑无处不在。这也给测试带来了一定挑战。同时，这也是很多人将 React 与状态管理库结合使用的原因之一。但是，这往往会引入了很多抽象概念，需要你在不同的文件之间来回切换，使得复用变得更加困难。

为了解决这个问题，Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据），而并非强制按照生命周期划分。你还可以使用 reducer 来管理组件的内部状态，使其更加可预测。
我们将在使用 Effect Hook 中对此展开更多讨论。

**（3）难以理解的 class**
除了代码复用和代码管理会遇到困难外，class 是学习 React 的一大屏障。我们必须去理解 JavaScript 中 this 的工作方式，这与其他语言存在巨大差异。还不能忘记绑定事件处理器。没有稳定的语法提案，这些代码非常冗余。大家可以很好地理解 props，state 和自顶向下的数据流，但对 class 却一筹莫展。即便在有经验的 React 开发者之间，对于函数组件与 class 组件的差异也存在分歧，甚至还要区分两种组件的使用场景。为了解决这些问题，Hook 使你在非 class 的情况下可以使用更多的 React 特性。 从概念上讲，React 组件一直更像是函数。而 Hook 则拥抱了函数，同时也没有牺牲 React 的精神原则。Hook 提供了问题的解决方案，无需学习复杂的函数式或响应式编程技术。

# 70.React Hook 的使用限制有哪些？

React Hooks 的限制主要有两条：

- 不要在循环、条件或嵌套函数中调用 Hook； 
- 在 React 的函数组件中调用 Hook。 

那为什么会有这样的限制呢？Hooks 的设计初衷是为了改进 React 组件的开发模式。在旧有的开发模式下遇到了三个问题。

- 组件之间难以复用状态逻辑。过去常见的解决方案是高阶组件、render props 及状态管理框架。 
- 复杂的组件变得难以理解。生命周期函数与业务逻辑耦合太深，导致关联部分难以拆分。 
- 人和机器都很容易混淆类。常见的有 this 的问题，但在 React 团队中还有类难以优化的问题，希望在编译优化层面做出一些改进。 

这三个问题在一定程度上阻碍了 React 的后续发展，所以为了解决这三个问题，Hooks **基于函数组件**开始设计。然而第三个问题决定了 Hooks 只支持函数组件。

那为什么不要在循环、条件或嵌套函数中调用 Hook 呢？因为 Hooks 的设计是基于数组实现。在调用时按顺序加入数组中，如果使用循环、条件或嵌套函数很有可能导致数组取值错位，执行错误的 Hook。当然，实质上 React 的源码里不是数组，是[链表]()。

这些限制会在编码上造成一定程度的**心智负担**，新手可能会写错，为了避免这样的情况，可以引入 ESLint 的 Hooks 检查插件进行预防。

# 71.useEffect 与 useLayoutEffect 的区别

useEffect在before Mutaion阶段被调度，在layout阶段完成之后，先按照hooks链中的顺序执行所有的destory函数，然后依次执行create回调函数

useLayoutEffect函数在mutation阶段执行destory函数，在layout阶段执行create回调函数

# 72.React Hooks在平时开发中需要注意的问题和原因

**（1）不要在循环，条件或嵌套函数中调用Hook，必须始终在 React函数的顶层使用Hook**

这是因为React需要利用调用顺序来正确更新相应的状态，以及调用相应的钩子函数。一旦在循环或条件分支语句中调用Hook，就容易导致调用顺序的不一致性，从而产生难以预料到的后果。
**（2）使用useState时候，使用push，pop，splice等直接更改数组对象的坑**
使用push直接更改数组无法获取到新值，应该采用新构建的方式，但是在class里面不会有这个问题。

**（3）useState设置状态的时候，只有第一次生效，后期需要更新状态，必须通过useEffect**
TableDeail是一个公共组件，在调用它的父组件里面，我们通过set改变columns的值，以为传递给TableDeail 的 columns是最新的值，所以tabColumn每次也是最新的值，但是实际tabColumn是最开始的值，不会随着columns的更新而更新。

**（4）善用useCallback**
父组件传递给子组件事件句柄时，如果我们没有任何参数变动可能会选用useMemo。但是每一次父组件渲染子组件即使没变化也会跟着渲染一次。
**（5）不要滥用useContext**
可以使用基于 useContext 封装的状态管理工具。

# 73.React Hooks 和生命周期的关系？

**函数组件** 的本质是函数，没有 state 的概念的，因此**不存在生命周期**一说，仅仅是一个 **render 函数**而已。但是引入 **Hooks** 之后就变得不同了，它能让组件在不使用 class 的情况下拥有 state，所以就有了生命周期的概念，所谓的生命周期其实就是 `useState`、 `useEffect()` 和 `useLayoutEffect()` 。

即：**Hooks 组件（使用了Hooks的函数组件）有生命周期，而函数组件（未使用Hooks的函数组件）是没有生命周期的**。

| **class 组件**           | **Hooks 组件**            |
| ------------------------ | ------------------------- |
| constructor              | useState                  |
| getDerivedStateFromProps | useState 里面 update 函数 |
| shouldComponentUpdate    | useMemo                   |
| render                   | 函数本身                  |
| componentDidMount        | useEffect                 |
| componentDidUpdate       | useEffect                 |
| componentWillUnmount     | useEffect 里面返回的函数  |
| componentDidCatch        | 无                        |
| getDerivedStateFromError | 无                        |

# 74.React的状态提升是什么？使用场景有哪些？

React的状态提升就是用户对子组件操作，子组件不改变自己的状态，通过自己的props把这个操作改变的数据传递给父组件，改变父组件的状态，从而改变受父组件控制的所有子组件的状态，这也是React单项数据流的特性决定的。官方的原话是：共享 state(状态) 是通过将其移动到需要它的组件的最接近的共同祖先组件来实现的。 这被称为“状态提升(Lifting State Up)”。概括来说就是**将多个组件需要共享的状态提升到它们最近的父组件上**，**在父组件上改变这个状态然后通过props分发给子组件。**

# 75.React的严格模式如何使用，有什么用处？

`StrictMode` 是一个用来突出显示应用程序中潜在问题的工具。与 `Fragment` 一样，`StrictMode` 不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告。
可以为应用程序的任何部分启用严格模式。

`StrictMode` 目前有助于：

- 识别不安全的生命周期 
- 关于使用过时字符串 ref API 的警告 
- 关于使用废弃的 findDOMNode 方法的警告 
- 检测意外的副作用 
- 检测过时的 context API

# 76.为什么使用jsx的组件中没有看到使用react却需要引入react？

本质上来说JSX是`React.createElement(component, props, ...children)`方法的语法糖。在React 17之前，如果使用了JSX，其实就是在使用React， `babel` 会把组件转换为 `CreateElement` 形式。在React 17之后，就不再需要引入，因为 `babel` 已经可以帮我们自动引入react。
