# 1.`useCallback()` 使用场景？

[参考](https://segmentfault.com/a/1190000020108840)

# 2.`useCallback()`依赖state/props，但是会导致死循环？

[参考](https://segmentfault.com/a/1190000020108840)

# 3.使用`useCallback()`必然性能更好吗？

[参考](https://segmentfault.com/a/1190000020108840)

[参考 + useCallback()的正确使用姿势](https://zhuanlan.zhihu.com/p/433795516)

# 4.`setState()`的回调形式解决使用props/state但不考虑添加为依赖的问题

[参考](https://zh-hans.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)

# 5.`useReducer()`解决state依赖使用props/state但不考虑添加为依赖的问题

[参考](https://adamrackis.dev/state-and-use-reducer/)

# 6.何时使用`useReducer()`

-  state 逻辑较复杂且包含多个子值；
-  state 之间互相依赖 ；
- 涉及触发深更新需要传递回调。

# 7.Hooks使用规则

## 1.只在最顶层使用 Hook

**不要在循环，条件或嵌套函数中调用 Hook，** 确保总是在你的 React 函数的最顶层以及任何 return 之前调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 `useState` 和 `useEffect` 调用之间保持 hook 状态的正确。

## 2.只在 React 函数中调用 Hook

**不要在普通的 JavaScript 函数中调用 Hook。**你可以：

- ✅ 在 React 的函数组件中调用 Hook
- ✅ 在自定义 Hook 中调用其他 Hook 

# 8.React 怎么知道哪个 state 对应哪个 `useState`？

 React 靠的是 Hook 调用的顺序。因为Hooks**使用规范可以确保Hook顺序的不变性**，所以调用useState的顺序就是state的顺序。

# 9.可以在哪里执行副作用？不可以在哪里执行副作用？

在函数组件主体内（这里指在 React 渲染阶段）改变 DOM、添加订阅、设置定时器、记录日志以及执行其他包含副作用的操作都是不被允许的，因为这可能会产生莫名其妙的 bug 并破坏 UI 的一致性。

使用 `useEffect` 完成副作用操作。赋值给 `useEffect` 的函数会在组件渲染到屏幕之后执行。你可以把 effect 看作从 React 的纯函数式世界通往命令式世界的逃生通道。

# 10.`useEffect()`的执行时机

[官方文档](https://zh-hans.reactjs.org/docs/hooks-reference.html#timing-of-effects)

> 与 `componentDidMount`、`componentDidUpdate` 不同的是，传给 `useEffect` 的函数会在浏览器完成布局与绘制**之后**，在一个延迟事件中被调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因为绝大多数操作不应阻塞浏览器对屏幕的更新。

# 11.优化`useState()`中的初始值，防止重复计算（惰性初始化）

[参考](https://zh-hans.reactjs.org/docs/hooks-reference.html#lazy-initial-state)

# 12.避免向下传递回调的方案？

[Context + useReducer()](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)

# 13.`useReducer()`的第三参数有什么用处？

[参考](https://zh-hans.reactjs.org/docs/hooks-reference.html#lazy-initialization)

# 14.关于state更新未发生变化React仍（可能）会执行渲染的解释以及优化

[参考](https://tipsfordev.com/why-is-my-component-rendering-when-usestate-is-called-with-the-same-state)

# 15.如何在React中实现类似实例变量？

[使用useRef()](https://zh-hans.reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables)

# 16.如何在React中使用回调ref（实现绑定和解绑时触发回调）？

[参考](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)

# 17.如何在函数式中实现像类组件一样使用refs绑定一个组件之后获取其实例属性？

[使用useImperativeHandle()](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)

# 18.有什么是hooks可以做到而class做不到的？

Hook 提供了强大而富有表现力的方式来在**组件间复用（状态以及副作用相关的）功能**。通过 [「自定义 Hook」](https://zh-hans.reactjs.org/docs/hooks-custom.html) 这一节可以了解能用它做些什么。这篇来自一位 React 核心团队的成员的 [文章](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889) 则更加深入地剖析了 Hook 解锁了哪些新的能力。

# 19.hooks暂时还无法覆盖什么场景？

# 20.hooks优缺点

[参考](https://zhuanlan.zhihu.com/p/88593858)



## 优点

- 更容易复用代码；
  - 每一次调用useHook()都会得到一份独立的状态，而且减少了命名冲突；
  - 将 状态 + 副作用的逻辑 和 生命周期 进行分离（自定义hook），更方便地在函数组件之间共享。使用类式组件想要实现共享的话依赖mixin，它的灵活性不如自定义hook，而且存在许多缺点；
  - 代码的可读性也更好，原本分散在生命周期内的逻辑可以通过hooks内聚在一起，遵照了各司其职的单一职责原则。
- 减少代码量；
  - hook通过useEffect()或useReducer()执行副作用，其中useEffect()中的回调可以实现将componentDidMount()和componentDidUpdate()的生命周期逻辑进行合并（当不设置依赖参数的时候），减少了不必要的逻辑重复。而且通过依赖参数设置为`[]`则可以单单实现仅挂载执行单次副作用的生命周期逻辑；
  - 自定义hook在组件间复用使得代码量下降；
  - 不需要像类式组件一样添加很多模板代码。

## 缺点

- **处理状态和依赖时，会带来更多的心智负担**；
  - 写函数组件时，你不得不改变一些写法习惯。你必须清楚代码中`useEffect`和`useCallback`的“依赖项数组”的改变时机。有时候，你的useEffect依赖某个函数的不可变性，这个函数的不可变性又依赖于另一个函数的不可变性，这样便形成了一条依赖链。一旦这条依赖链的某个节点意外地被改变了，你的useEffect就被意外地触发了，如果你的useEffect是幂等的操作，可能带来的是性能层次的问题，如果是非幂等，那就糟糕了。
- hooks不擅长处理异步的代码，会发生状态不同步等；
  - 因为使用异步可能会因为闭包导致取到之前的值，需要使用useRef()对最新（值/函数）引用的动态更新；类式组件中的属性和方法都绑定在实例this上，需要时可以直接获取最新的值，但是在hooks中如若处理不当的话会导致读取固化的闭包值而非最新值.
- hooks有时候严重依赖参数的不可变性，这也导致了在传递回调的时候必须给回调加一层useCallback()；

# 关于依赖不更新的案例

## 原代码

存在问题：点击button更新count之后再resize，读取到一直是初始值。

```jsx
function App() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        // 让resize事件触发handleResize
        window.addEventListener('resize', handleResize)
        return () => window.removeEventListener('resize', handleResize)
    }, [])

    const handleResize = () => {
        // 把count输出
        console.log(`count is ${count}`)
    }

    return (
        <div className="App">
            <button onClick={() => setCount(count + 1)}>+</button>
            <h1>{count}</h1>
        </div>
    );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

## 解法一

使用useRef改造count为ref count，在count更新之后更新ref内含值，但是始终维持了对count的ref的引用使得组件内函数始终能够获取count的最新值，这里的ref就相当于将值作为了实例属性。

```jsx
import React, {useState, useEffect, useRef} from 'react';

export default function HooksTest() {
    const [count, setCount] = useState(0)

    const countRef = useRef(count)

    useEffect(() => {
        countRef.current = count
    }, [count])

    useEffect(() => {
        window.addEventListener('resize', handleResize)
        return () => window.removeEventListener('resize', handleResize)
    }, [])

    const handleResize = () => {
        console.log(`count is ${countRef.current}`)
    }

    return (
        <div className="App">
            <button onClick={() => {
                setCount(count + 1)
            }}>+
            </button>
            <h1>{count}</h1>
        </div>
    )
}

```

## 解法二

使用useCallback对函数进行记忆化，当且仅当count更新就进行函数的更新。

```jsx
import React, {useState, useEffect, useCallback} from 'react';

export default function HooksTest() {
    const [count, setCount] = useState(0)

    const handleCountChange = useCallback(() => {
        console.log(`count is ${count}`)
    }, [count])

    useEffect(() => {
        window.addEventListener('resize', handleCountChange)
        return () => window.removeEventListener('resize', handleCountChange)
    }, [handleCountChange])
    return (
        <div className="App">
            <button onClick={() => {
                setCount(count + 1)
            }}>+
            </button>
            <h1>{count}</h1>
        </div>
    )
}
```

## 解法三

官方推荐的使用方式：直接将函数**移至useEffect内部**，共同依赖count的变更，如果能够这样做的话是最好的。缺点是监听的重复取消和订阅可能是不被希望的。

```jsx
import React, {useState, useEffect, useCallback} from 'react';

export default function HooksTest() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        const handleCountChange = () => {
            console.log(`count is ${count}`)
        }
        window.addEventListener('resize', handleCountChange)
        return () => window.removeEventListener('resize', handleCountChange)
    }, [count])
    return (
        <div className="App">
            <button onClick={() => {
                setCount(count + 1)
            }}>+
            </button>
            <h1>{count}</h1>
        </div>
    )
}
```

# 21.什么时候适合使用useRef()方案来替代useCallback()方案？

[参考](https://zhuanlan.zhihu.com/p/56975681)

# 22.hook生命周期和class的对照

### 生命周期方法要如何对应到 Hook？

- `constructor`：函数组件不需要构造函数。你可以通过调用 [`useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate) 来初始化 state。如果计算的代价比较昂贵，你可以传一个函数给 `useState`。
- `getDerivedStateFromProps`：改为 [在渲染时](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops) 安排一次更新。
- `shouldComponentUpdate`：详见 [下方](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate) `React.memo`.
- `render`：这是函数组件体本身。
- `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`：[`useEffect` Hook](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect) 可以表达所有这些(包括 [不那么](https://zh-hans.reactjs.org/docs/hooks-faq.html#can-i-skip-an-effect-on-updates) [常见](https://zh-hans.reactjs.org/docs/hooks-faq.html#can-i-run-an-effect-only-on-updates) 的场景)的组合。
- `getSnapshotBeforeUpdate`，`componentDidCatch` 以及 `getDerivedStateFromError`：目前还没有这些方法的 Hook 等价写法，但很快会被添加。

# 23.如何使用hook发起数据请求？

[参考](https://www.robinwieruch.de/react-hooks-fetch-data/)

# 24.我该如何实现getDerivedStateFromProps？

[参考](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops)

# 25.hook闭包的解决方案

更新状态

1. 使用setState()的回调形式
2. 使用dispatch来更新状态

获取状态

1. useRef + useEffect

# 26.如何实现shouldComponentUpdate()？

[使用React.memo()](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)

# 27.向下深入传递回调好还是使用useReducer()更好？

[更推荐使用useReducer代替回调深层传递](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)

# 28.使用useReducer()来解除多个(相互)依赖的依赖列表的例子

[参考文章](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E6%AF%8F%E4%B8%80%E6%AC%A1%E6%B8%B2%E6%9F%93%E9%83%BD%E6%9C%89%E5%AE%83%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86%E5%87%BD%E6%95%B0)

[DEMO实例](https://codesandbox.io/s/xzr480k0np?file=/src/index.js)

# 29.useReducer从本质上做了什么？

[参考](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E8%A7%A3%E8%80%A6%E6%9D%A5%E8%87%AAactions%E7%9A%84%E6%9B%B4%E6%96%B0)

使用useReducer()的dispatch从本质上来说是**将effect应当触发什么样的状态更新(dispatch)和如何进行状态更新(action)进行解耦**，也即effect负责**触发**，reducer负责**执行具体更新**。正因为dispatch能够**保证获取到最新鲜的状态**且没有显式引入 依赖，这使得（某些特定条件下，比如reducer维护步数和步长状态，更新步数依靠步数 + 步长，所以需要引入两个依赖，但是使用dispatch的方式更新步数的话就无需引入两个依赖，只需要告诉reducer去更新步数即可）在useEffect内部使用dispatch可以放心地剔除dispatch内部所用到的依赖。

# 30.useReducer使用时如何应对依赖为props？

[把*reducer*函数放到组件内去读取props](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E4%B8%BA%E4%BB%80%E4%B9%88usereducer%E6%98%AFhooks%E7%9A%84%E4%BD%9C%E5%BC%8A%E6%A8%A1%E5%BC%8F)

# 31.如何处理竞态？

[参考 - 使用 布尔值 + 闭包 跟踪取消更新操作](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E8%AF%B4%E8%AF%B4%E7%AB%9E%E6%80%81)

