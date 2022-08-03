# 为什么要区分react-dom和react？

React包是React的主干逻辑，是React运行机制的抽象，包括了对组件的实现、更新调度等。react-dom、react-native则是和平台具体的渲染相关的实现。这是一种依赖倒置的典型应用，高层模块不应该依赖底层模块的具体实现。

# 什么是JSX？

JavaScript XML。是由React定义的一种类似于XML的JS扩展语法，本质上是一种语法糖，在React V17之前相当于React.createElement，所以必须显式引入React，在ReactV17之后JSX的实现不依赖于createElement，所以不再需要显式地引入React。

> 因为保留字的原因，class -> className；for -> htmlFor；所有的属性名遵循小写驼峰，除了aria-*属性之外
>
> 内联样式必须以对象的形式去写，并且CSS属性名也是小写驼峰形式
>
> 单个组件必须只有一个根标签（可以使用<Fragment/>或者空标签）

# React中使用表达式和语句？

- 表达式代表会返回一个值
- 语句不会产生值

> React花括号中的JS必须是表达式（会返回值）

```
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

# 什么是纯函数？有什么优点？

定义：

纯函数是这样一种函数，相同的输入永远会得到纯函数相同的输出，而且运算过程中不会产生任何的副作用（不会修改传入引用类型内部的数据 + 不会对外部变量产生影响）。

优点：

- 相同的输入必然得到相同的输出，可以根据输入进行缓存；
- 相同的输入必然得到相同的输出，保证传入纯函数的引用数据的安全性；
- 纯函数依赖明确，便于测试；
- 纯函数没有副作用，不会对外部变量产生污染；
- 纯函数的没有共享得到内存，不会进入竞争态（并行存在依然是纯函数）；
- 纯函数逻辑明确，减少了心智负担。

# 纯函数与React的关系？

## 纯函数组件

首先，React渲染视图的理念就是UI = render(props)，并且规定了所有组件都必须像纯函数一样保护他们的props不被更改。React中的无状态组件就可以很好地践行纯函数的理念，无状态组件保证了相同的props、context注入，一定可以得到相同的视图。React也鼓励尽量将在组件中分离出无状态组件来提高应用的可靠性，所以有可能就尽量使用纯函数组件，React未来可能也会为此类组件进行优化。

纯函数组件的优点：无副作用；类式组件不会被实例化，组件不用进行生命周期的管理和状态管理，因此不需要进行某些特定的检查和内存分配，渲染性能好；可读性好，语法简单，便于测试，容易复用；



# `setState()`是同步的还是异步的？

setState()是同步的，但是他在执行时的状态滞后会给开发者带来是异步操作的错觉，setState()不能作为立即更新状态的语义上的保证，他在React的调度周期内作为一种请求状态更新的方法，实际调用setState()只是调用enqueueUpdate在updateQueue中添加了一次update对象，而React在React调度周期内React会对updateQueue进行更新合并，也就是所谓的**批处理**。所以最安全的读取`setState()`后的最新状态的方法是`setState()`的第二个参数（回调函数）或者是`componentDidUpdate`生命周期函数

# 为什么多次调用`setState()`更新同一个状态只成功一次？

因为批处理合并了状态更新，所以仅有只有最后一次更新是有效的。叠加更新考虑回调形式。

# React生命周期有哪些？

旧生命周期：

![img](https://github.com/NoAlligator/pico/blob/main/img/e12b2e35c8444f19b795b27e38f4c149~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

新生命周期：

![image-20220122130034632](https://github.com/NoAlligator/pico/blob/main/img/image-20220122130034632.png?raw=true)

被标记为`unsafe`的生命周期：

- `unsafe_componentWillMount(nextProps, nextState)`
- `unsafe_componentWillReceiveProps(nextProps)`
- `unsafe_componentWillUpdate(prevProps, prevState)`

新的生命周期：

- `getSnapBeforeUpdate(prevProps, prevState)`
- `getDerivedStateFeomProps(nextProps, nextState)`

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

# 副作用在哪里执行？

副作用只能在commit阶段的layout之后执行，对应的生命周期函数就是`compoenntDidUpdate`和`componentDidMount`以及`componetWillUnmount`。在这些生命周期内组件已经渲染完成，可以安全地执行副作用。

# 何时使用ref？

- 焦点管理，非受控组件内容获取
- 集成第三方DOM库
- 触发强制动画

# React事件处理机制？

React的事件并不是原生事件，而采用了合成事件。首先是利用代理的模式在rootNode上监听注册的所有事件，然后原生事件的触发会被rootNode所捕获，随后React会创建合成事件以及构建事件对象信息，最后在提交给对应fiber上的事件处理函数进行处理。因为它的执行脱离了原生DOM这个环境，所以必须显式地调用`e.preventDefault()`来阻止默认事件。

# 类式组件中this丢失问题？如何绑定？

原因：首先事件处理函数被保存在对应的fiber节点上，它的执行是脱离了类实例这个上下文的，所以this默认会指向全局对象。但是`ES6`的类中的以严格模式执行，不允许其指向`window`，所以`this`就是空的。

- 实例属性 + 箭头函数
- 构造函数显式绑定
- 内联显式绑定
- 箭头函数内联调用

# 受控组件和非受控组件的概念？

非受控组件指的是如`<input>`、 `<textarea>` 和 `<select>`这样可交互元素，他们通常维护着自己的状态，这样的状态游离在React状态控制之外，所以称之为非受控组件。受控组件就是将这些组件的状态控制起来，让组件内部的state成为其唯一的数据源，然后再通过事件回调触发状态的更新。

> 非受控组件通过defaultValue赋予初值；
>
> 受控组件指定了除null、undefined之外的值又没有设置事件回调的话会导致无法输入；

# 受控组件的替代方案

用ref来代替，不过存在局限性。

# 为什么需要key？

key帮助React识别哪些**同级**元素需要进行复用，他是进行Diff时的首要对比条件（type是次要对比条件）。我们应当给列表赋予key，也应当给那些明显可以进行复用的元素附加唯一的key减少组件销毁重建（比如两个重建开销很大而且类型不一致的同级组件，如果他们在更新中可能会产生位移，那么应当添加`key`进行复用）。

# 可以使用索引作为key吗？

不能使用索引作为key，因为使用索引作为key的话会导致错误的复用，在使用索引作为key的时候任何导致元素位移的操作都是有错误复用风险的。

以下情况可以使用索引：静态列表、永远不会进行产生元素位移的操作的列表（只会进行pop、push）

# 能否使用随机值作为key？

没有意义。

# 状态提升

如果两个组件需要共享一个状态，在组件间通信往往是比较麻烦的，而且为了遵循单一数据源这个原则，不能制造多个状态，就可以考虑将该状态提升到共同的父级元素中。缺点是更多的样板代码，优点是单一的数据源逻辑更加清晰，追踪更加方便。

# 组合

组合模式使用props直接传递组件，能够一定程度上解决props透传的问题，但是缺点是可能会导致传入组件和内部组件的交互变的更加复杂。但如果是作为插槽，使用组合模式是非常方便的。

# React哲学

React哲学就是单一职责原则，一个组件原则上只能负责一个功能，按照这个原则对组件进行划分。

# 状态提升有哪些缺点呢？

- 状态提升使得状态脱离了其原宿主本身，作为props游离在其他组件之间，这使得**提升后的状态变得让人难以理解，也会使得代码碎片化**
- 将状态提升后再以props向下流，其存在形式是隐式的，如果该props属性没有被利用到的话（例如删除了组件中利用到该props的相关内容导致props失去存在意义，但是这个是不可感知的）也难以察觉
- 状态升级会导致针对状态对应的props进行诸如重命名等的的修改很棘手，因为提升后的状态作为props向下流向了很多个组件，也有可能涉及到多层透传，这就会导致容易产生遗漏等问题
- 在某些情况下，子组件A本身并不需要指特定的props，而仅仅是因为子组件A**内部的一个组件B**需要用到该特定props，子组件就必须作为中间人的身份传递props，这违反了单一职责原则，使得组件的props耦合在一起
- 最后，也是最重要的一点，使用状态提升会使得被提升的state原本应当所在的组件依赖父组件提供数据源，这导致组件的复用遇到问题（独立功能性降低）。

如何解决？

- Context
- Redux

# 代码如何分割（代码懒加载）？

在你的应用中引入代码分割的最佳方式是通过动态 `import()` 语法。

```jsx
//当 Webpack 解析到该语法时，会自动进行代码分割。
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```



# 组件如何分割（懒加载）？

```jsx
import { lazy, Suspend } from 'react'
//使用之后，此代码将会在组件首次渲染时，自动导入包含 OtherComponent 组件的包。
React.lazy(() => import())
//React.lazy应当配合Suspense组件使用实现加载期间的状态展示。
const Demo = lazy(() => import('./components/demo'))
const App = () => (
    <Suspend fallback={ <h1>Loading...</h1> }>
    	<Demo/>
    </Suspend>
)
```

# 为什么要进行懒加载？

现在前端项目基本都采用打包技术，比如 Webpack，JS逻辑代码打包后会产生一个 bundle.js 文件，而随着我们引用的第三方库越来越多或业务逻辑代码越来越复杂，相应打包好的 bundle.js 文件体积就会越来越大，因为需要先请求加载资源之后，才会渲染页面，这就会严重影响到页面的首屏加载。而为了解决这样的问题，避免大体积的代码包，我们则可以通过技术手段对代码包进行分割，能够创建多个包并在运行时动态地加载。现在像 Webpack、 Browserify等打包器都支持代码分割技术。

什么时候应该考虑进行代码分割？这里举一个平时开发中可能会遇到的场景，比如某个体积相对比较大的第三方库或插件（比如JS版的PDF预览库）只在单页应用（SPA）的某一个不是首页的页面使用了，这种情况就可以考虑代码分割，增加首屏的加载速度。

# 什么是错误边界？

错误边界是一种 React 组件，这种组件**可以捕获发生在其子组件树任何位置的 JavaScript 错误，并打印这些错误，同时展示降级 UI**，而并不会渲染那些发生崩溃的子组件树。错误边界可以捕获发生在**整个子组件树的渲染期间、生命周期方法以及构造函数中的错误。**错误边界**无法**捕获以下场景中产生的错误（因为他们往往不在渲染期间触发）：

- 事件处理（事件处理器不会在渲染期间触发）
- 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调函数）
- 服务端渲染
- 它自身抛出来的错误（并非它的子组件）

如果一个 class 组件中定义了 [`static getDerivedStateFromError()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromerror) 或 [`componentDidCatch()`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidcatch) 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，请使用 `static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。只有 **class 组件才可以成为错误边界组件**。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 `catch {}` 的工作机制。

> 未捕获错误的新行为：
>
> **自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。**增加错误边界能够让你在应用发生异常时提供更好的用户体验。我们也鼓励使用 JS 错误报告服务（或自行构建），这样你能了解关于生产环境中出现的未捕获异常，并将其修复。

# Context

Context 提供了一个**无需为每层组件手动添加 props，就能在组件树间进行数据传递**的方法。Context及其适合那些简单的全局状态状态（地区偏好、主题等），这些状态使用props是极其繁琐的（透传props）。

# 高阶组件HOC

高阶组件HOC是React中用于组件复用的一种方式，他以组件为入参，并返回包装之后的组件。组件是将 props 转换为 UI，而高阶组件是将组件转换为另一个组件。高阶组件往往是在复用原有组件抽象的基础上增加新的功能，HOC 不应该修改传入组件，而应该使用组合的方式，通过将组件包装在容器组件中实现功能，务必记得将不相关的 props 传递给被包裹的组件、复制被包裹组件中的静态方法。

## 优缺点

优点：

- 通过传递props去影响内层组件的状态，不直接改变内层组件的状态、生命周期等，降低了耦合度。
- HOC实现了业务逻辑的复用，比mixin模式更佳，不同于 Mixin 的打平合并（平级结构），HOC 具有天然的层级结构（组件树结构），这又降低了复杂度。

缺点：

- 当有多个HOC一同使用时，无法直接判断子组件的props是哪个HOC负责传递的。
- 组件多层嵌套， **增加复杂度与理解成本**，会出现因为同名上层**props被下层同名props覆盖**的情况，而且上层props的来源如果层级较多的话难以追溯；
- 传递props时存在**ref隔断， 依靠React.forwardRef 来解决**；
- **HOC 无法从外部访问子组件的 State**，因此无法通过`shouldComponentUpdate`滤掉不必要的更新（Mixin可以），React 在支持 ES6 Class 之后提供了`React.PureComponent`来解决这个问题；
- 需要谨慎，不要在 render 方法中使用 HOC，在render中使用HOC会严重影响性能；

# HOC和组合模式有什么区别？

使用HOC可以实现关注点的分离，被包裹组件抽离出来作为视图层被逻辑化，由容器管理订阅和状态，并将 prop 传递给处理 UI 的组件。HOC 使用容器作为其实现的一部分，可以将 HOC 视为**参数化（函数化）容器组件。**

# render props

render props是通过一个**返回组件的回调函数**作为props传递给其他组件，其他组件在接收到该回调之后可以根据回调参数传入props。使用render props能够很好地对组件进行复用。一言蔽之，父组件已经明确自己可以提供的数据源，并且内部存在了一个插槽给相应的回调进行调用、渲染。在这里，父组件数据提供者、操纵者，子组件是数据承载者。render props可以完全代替HOC，并且**render props可以用来很方便地创建HOC。**

> 使用render props时注意：使用**行内回调函数形式**在每一次提供回调props的组件更新的时候回调函数都会被重新创建，引发被传入回调props的组件重新更新。

## 优缺点

优点：

1. 使用回调形式，减少props冲突的可能性；
2. 相较于HOC，不会产生无用的空组件加深层级
3. 更加明确props来源（因为在回调函数中已经**明确指定了需要的props**，父组件作为数据源只要提供相关数据即可）；
4. 耦合度很低，回调函数内返回组件对于调用的父组件只有props上的数据依赖关系（纯函数），所以容易理解。

缺点：

1. 使用繁琐：HOC使用只需要借助装饰器语法通常一行代码就可以进行复用，Render Props无法做到如此简单，需要预定义一个回调函数并对接收组件进行接受性改造；
2. 嵌套过深：Render Props虽然摆脱了组件多层嵌套的问题，但是可能会转化为**函数回调**的嵌套。

# PureComponent和Component

`React.PureComponent` 与 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 很相似。两者的区别在于 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 并未实现 [`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，而 `React.PureComponent` 中以浅层对比 prop 和 state 的方式（`Object.is()`）来实现了该函数。如果赋予 React 组件相同的 props 和 state，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。

# Mixin为什么存在问题？

1. mixin引入了**隐式依赖**（组件可能依赖mixin上的某个方法，mixin也有可能依赖组件上的某个方法，这导致了隐式依赖的产生，此时如果将mixin进行迁移的话，可能会导致隐式依赖的丢失从而出现为问题。通常，mixin 依赖于其他 mixin，并且删除其中一个会破坏另一个。在这些情况下，很难判断数据如何流入和流出 mixins，以及它们的依赖关系图是什么样的。与组件不同，mixin 不形成层次结构：它们被扁平化并在同一个命名空间中运行）总之，隐式依赖导致依赖关系不透明，维护成本和理解成本迅速攀升 。
2. mixin可能会**导致命名冲突**（不能保证mixin和另一个mixin或者组件可以一起使用而不产生命名上的冲突。此外，要是产生了冲突需要修改，其他直接依赖次mixin的组件都会收到牵连）。
3. mixin导致**复杂性**滚雪球。（随着时间的推移，使用相同 mixin 的组件变得越来越耦合。任何新功能都会被添加到使用该 mixin 的所有组件中。如果不复制代码或在 mixin 之间引入更多的依赖关系和间接性，就无法拆分 mixin 的“更简单”部分。逐渐地，封装边界被侵蚀，并且由于很难更改或删除现有的 mixin，它们变得越来越抽象，直到没有人了解它们是如何工作的）。
4. Mixin 倾向于**增加更多状态**，这降低了应用的可预测性，导致复杂度剧增。
5. 难以快速理解组件行为，需要全盘了解所有依赖 Mixin 的扩展行为，及其之间的相互影响。

# 为什么不可变性在React中很重要？

记录数据版本；简化数据更新逻辑；帮助确定何时重新渲染；便于践行函数式编程。

1. 不可变性有助于**简化复杂的功能**（例如可以实现历史记录功能）；
2. 不可变性有利于**追踪数据的改变**（如果直接修改数据，那么就很难跟踪到数据的改变。跟踪数据的改变需要**可变对象可以与改变之前的版本进行对比，这样整个对象树都需要被遍历一次**。**跟踪不可变数据的变化相对来说就容易多了。如果发现对象变成了一个新对象，那么我们就可以说对象发生改变了**）；
3. 帮助**确定React中何时重新渲染**（不可变性最主要的又是就是在于它可以帮助我们在React中创建Pure Component，我们可以很轻松的确定不可变数据是否发生了改变，从而确定何时对组件进行重新渲染）。

# 什么是不可变性数据？

1. 不可变数据是一旦创建，就不能被更改的数据；
2. 不可变数据对象的任何修改或添加删除操作都会返回一个新的不可变数据对象；
3. 不可变数据实现的原理是持久化数据结构，也就是是永久数据创建新数据时，要保证旧数据同时可用且不变；
4. 同时为了避免深克隆把所有节点都复制一遍带来的性能损耗，不可变数据使用了结构共享，即如果对象树结点发生变化，只修改这个结点和受它影响的父节点，其他结点进行共享。

# Diff算法

## 前提

1. 不同的`type`必将重新渲染；
2. 只对同一层级进行`diff`；
3. 使用key来告诉React那些元素需要被复用； 

Diff对比的双方（数组 `VS`单链表）：

- `current Fiber`。如果该`DOM节点`已在页面中，`current Fiber`代表该`DOM节点`对应的`Fiber节点`。（同级的`Fiber节点`是由`sibling`指针链接形成的单链表）
- `JSX对象`。即`ClassComponent`的`render`方法的返回结果，或`FunctionComponent`的调用结果。`JSX对象`中包含描述`DOM节点`的信息。（以数组形式存在）

单点Diff：

- **单点diff指的是新增的JSX对象是单个节点**；
- 先比较key，`key不一样`的话标记当前old fiber为deletion，根据当前JSX对象在workInProgress树上生成新的fiber节点，继续遍历；
- `key一样，type不同`的话，直接标记old fiber及其兄弟节点为删除，根据当前JSX对象在workInProgress树上生成新的fiber节点，退出遍历；
- `key一样，type一样`的话就地复用old fiber，直接标记兄弟节点为删除，退出遍历。

多点Diff：

- **多点diff指的是新增的JSX对象是多个节点**，React团队发现，执行更新的操作是远比其他操作更频繁的，所以在第一小轮进行的是更新相关操作；
- 第一小轮也是先通过key进行比较，**只比较产生了更新的部分**：
  - key相同，type相同，就地复用；
  - key相同，type不同，标记删除当前old fiber，根据当前JSX对象在workInProgress树上生成新的fiber节点，进入第二轮；
  - key不同，继续遍历；
  - old fiber和当前JSX链表有一方提前比较完毕，进入第二轮；
- 第二轮执行增加、删除、位移的比较，因为是接着上一轮，存在以下几种情况：
  - 同时遍历完，那就可以提前退出；
  - old fiber链提前遍历完，那意味着所有的新增JSX对象都要生成新的fiber节点；
  - 新增JSX对象数组没有遍历完，那意味着剩下的old fiber都要被打上删除标记；
  - 两个都没有遍历完，遍历old fiber链表生成Map[key, fiber]，遍历剩下的JSX对象数组并根据Map进行查找，对old fiber的进行复用。**注意，这里的复用不会产生insertBefore的操作，都是appendChild的操作，这是通过比较元素的相对位置实现的。**

# 什么是编译期严格模式？

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

# 为什么React不“同步”地更新`this.setState()`？

[参考](https://www.cnblogs.com/echolun/p/15510770.html)

- 保证内部一致性；
  - 即便`setState`能做到同步，`react`对于`props`的更新依旧是异步，这是因为对于一个子组件而言，它只有等到**父组件重新渲染**了，它才知道最新的`props`是多少，**所以让`setState`异步的另一个原因是为了让`state、props、refs`更新的行为与表现保持一致。**
- 性能优化；
  - 为了并发模式的异步可中断做铺垫；
- 减少无故重新渲染

# React 虚拟DOM的优缺点

1. 针对单次DOM操作来说，框架隔着一层虚拟DOM（fiber）对真实DOM进行操作，会经历框架的调度、协调、再渲染三个流程，增大了JS线程的压力，这是有很高代价的。
2. 但是在需要操作DOM较多的情况下，框架会对状态更新批处理优化能够使得浏览器重绘次数减少，然后进行虚拟DOM的diff使得其能够以较小的代价进行DOM的部分更新，尽量采取就地复用的形式复用元素，如果原生在没有合理优化的情况下只能进行全量更新，而这种重新渲染的代价相比就地复用是很高的。
3. 从库的依赖来说，使用框架不得不引入框架库，这就会使得初次加载必须引入完整的框架，增加了网络负担，但是现在的浏览器可以进行本地缓存。
4. 使用虚拟DOM脱离了浏览器环境，能够很方便地将其移植到其他平台：React Native、Electron

# React 事件机制

## JSX事件最终会变成什么？

JSX中的绑定事件首先经过Babel解析生成对应的对象，然后再经过ReactDOM.createElement转换成React element。最后在创建fiber的阶段会被作为一个内部props被保存（`fiber`对象上的`memoizedProps` 和 `pendingProps`保存了我们的事件）

![babel.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02eb66989a5444839c4e758b795869e7~tplv-k3u1fbpfcp-watermark.awebp)

![fiber.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2bd1a74076c40d1b5c5e7b53c341f7f~tplv-k3u1fbpfcp-watermark.awebp)

## 原生DOM上有没有绑定事件？

绑定了，但是是一个空函数，最终所有的事件都被绑定到了`rootNode`。

![button_event.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbd5b7c204754983b1eacc7bdcec8f88~tplv-k3u1fbpfcp-watermark.awebp)

我们可以看到 ，`button`上绑定了两个事件，一个是`document`上的事件监听器，另外一个是`button`，但是事件处理函数`handle`，并不是我们的`handerClick`事件，而是`noop`。`noop`就指向一个空函数。

![noop.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b419061b3c114309aeff3a59bd0d2f62~tplv-k3u1fbpfcp-watermark.awebp)

## 是否监听了所有原生事件？

`react`并不是一开始，把所有的事件都绑定在`root Node`上，而是采取了一种按需绑定，比如发现了`onClick`事件,再去绑定`root Node click`事件。

> 我们给`<input>`绑定的`onChange`，并没有直接绑定在`input`上，而是统一绑定在了`document`上，然后我们`onChange`被处理成很多事件监听器，比如`blur` , `change` , `input` , `keydown` , `keyup` 等。

## 为什么使用合成事件？

- 一方面，将事件绑定在`root Node`统一管理，防止很多事件直接绑定在原生的`dom`元素上。造成一些不可控的情况，使用代理方式进行统一管理，也可以减少内存使用（频繁注册事件回调）、提升性能。
- 另一方面， `React` 想实现一个全浏览器的框架， 为了实现这种目标就需要提供全浏览器一致性的事件系统，以此抹平不同浏览器的差异。

## 事件合成机制

JSX读取事件绑定函数到React Element上（后续fiber节点会保存事件处理函数） -> 相关事件注册在rootNode，rootNode开启对相关原生事件的监听 -> 触发原生事件，冒泡至rootNode -> rootNode接收到原生事件 -> 利用原生事件信息创建合成事件对象 -> 模拟原生冒泡逻辑在fiber树上进行冒泡 -> 将命中的需要冒泡的事件push入数组，将命中的需要捕获的事件unshift入数组 -> 以合成事件为入参执行数组中的事件处理函数，遇到中断冒泡就中断继续遍历执行。

> **在 React 17 中，React 将不再向 `document` 附加事件处理器。而会将事件处理器附加到渲染 React 树的根 DOM 容器中：**在 React 16 或更早版本中，React 会对大多数事件执行 `document.addEventListener()`。React 17 将会在底层调用 `rootNode.addEventListener()`。

## 哪些事件是捕获事件？

有些特殊的事件是按照事件捕获处理的：scroll 事件、focus 事件*、*blur 事件。

## 事件池

事件池在V17被删除。

# `this.setState`的执行流程怎样？

`this.setState`的具体更新执行是在`processUpdateQueue`方法，也就是`render`的递阶段：

![image-20220209221547595](https://github.com/NoAlligator/pico/blob/main/img/202203271802560.png?raw=true)

但是`enqueue`，也就是调度更新也就是调用`setState`发生在`enqueueUpdate`，它在`render`阶段之前，合成事件派发之后：

![image-20220209221739036](https://github.com/NoAlligator/pico/blob/main/img/202203271802561.png?raw=true)

setState是异步的吗？**setState 并不是真正的异步函数**，它实际上是通过**队列延迟执行操作实现**的，通过 isBatchingUpdates 来判断 setState 是先存进 state 队列还是直接更新。值为 true 则执行异步操作，false 则直接同步更新。对于isBatchingUpdates 为true的更新并不代表在当前作用域执行之后状态就马上更新，他只是开启react的调度并加入updateQueue，正式的更新会被调度到render的beginWork阶段（具体是updateComponent），setState后面的语句是在合成事件派发之后就执行的，还远没有到达render阶段。

为什么多次设置state得不到累加值？如果同一个周期内调用了多个setState，这些状态更新会被react批处理（**更新对象被合并，并且对相同属性的设置只保留最后一次的设置**），使用回调形式可以有效解决这个问题。此外，setState的第二个回调函数因为是在layout阶段触发的，所以必定可以访问到最新状态。

为什么在微任务、宏任务中是同步的？外部的原生事件中，并没有外层的封装与拦截，无法更新 isBatchingUpdates 的状态为 true。这就造成 isBatchingUpdates 的状态只会为 false，且立即执行。所以在 addEventListener 、setTimeout、setInterval 这些原生事件中都会同步更新。

# 生命周期流程

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

# 组件通信方式

- 父 -> 子：父组件传递props
- 父 -> 子：父组件通过ref获取组件实例并直接调用子组件上的方法（很管用）
- 子 -> 父：父组件传递回调props
- 子 -> 父：事件冒泡
- 兄弟间：状态提升
- 全局属性：Context
- Portals
- 发布订阅
- Redux

# Portals什么作用

Portals只是DOM层面的**外挂**（尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 *React 树*， 且与 *DOM 树* 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 *React 树*的祖先，即便这些元素并不是 *DOM 树* 中的祖先。在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 `<Modal />` 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件）

# React样式方案

- 内联对象
  - 避免了组件间样式冲突；
  - 不能使用伪类；
  - 不能使用媒体查询；
  - 可读性差
- 引入CSS样式表
  - 关注点分离；
  - 全功能CSS；
  - CSS缓存；
  - 会产生冲突；
  - 难以整理；
- CSS Module
  - 避免了样式冲突；
  - 需要额外的构建工具；
- styled-component(css in js)
  - 没有样式冲突；
  - 动态样式；
  - 便于维护，样式和组件对应；
  - 影响性能
- JSS
- Tailwind
  - 原子式CSS

# React Virtual DOM 的工作原理

在React中的虚拟DOM指的是fiber节点，除了某些根节点等的fiber节点没有对应的真实DOM节点，其他fiber节点都对应了真实DOM元素。fiber节点是通过解析JSX生成的React Element再经过React的reconcile阶段生成的，每一个fiber节点都包含了对应的节点信息，比如：alternate、child、firstEffect、props、state、ref、return、sibling、stateNode、key、updateQueue、type。fiber节点是React中的最小工作单元。

![image-20220210222853149](https://github.com/NoAlligator/pico/blob/main/img/202203271802562.png?raw=true)

其他可以详见React源码深入解析。

# React Vs Vue

## 共同点

- 使用 Virtual DOM
- 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
- 将注意力集中保持在**核心库**，而将其他功能如路由和全局状态管理交给相关的库。

## 更新

React：在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 `PureComponent`，或是手动实现 `shouldComponentUpdate` 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。然而，使用 `PureComponent` 和 `shouldComponentUpdate` 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 React 中的组件优化伴随着相当的心智负担。

Vue：在 Vue 应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染（通过Proxy代理实现）。你可以理解为每一个组件都已经自动获得了 `shouldComponentUpdate`，并且没有上述的子树问题限制。Vue 的这个特点使得开发者不再需要考虑此类优化，从而能够更好地专注于应用本身。

## HTML

React：JSX，可以使用完整的编程语言 JavaScript 功能来构建你的视图页面。

Vue：模板，基于标准的HTML，更规范，但是模板是和JS代码分离的，这限制了它直接引入外部数据的能力。

更抽象一点来看，我们可以把组件区分为两类：一类是偏视图表现的 (presentational)，一类则是偏逻辑的 (logical)。在前者中使用模板，在后者中使用 JSX 或渲染函数。

> Vue 的单文件组件，使用 `<template>` 、`<script>` 对代码进行分割，直接导致的问题就是上下文丢失。举个例子，你封装了一些常用的函数，在 Vue 文件中 import 进来。**你这个函数不能在 template 中直接使用。你只能将方法定义在methods中，再引用进来。**模板语法并不知道你有 isNickname 这个函数，简单的操作多了 3 行代码。

## CSS

Vue：原生对CSS的书写支持更友好，可以选择scope和非scope模式。

React：借助CSS in JS，书写CSS更方便，践行了一切皆JS。

## 修改数据方式，以及框架如何发现数据被修改？

Vue：直接修改源数据，通过Proxy或者Object.defineProperty()劫持setter、getter实现数据监听和比较。

React：强调数据的不变性，修改数据的方式是直接生成新的数据去替换，通过Object.is()对比发现数据更改与否。

## 何时开启Diff？

React：触发setState()、forceUpdate()、useState、useReducer自动开启Diff。

Vue：数据更新之后会开启Diff。

## 如何获取数据变化？

Vue：watch监听。

React：componentDidUpdate。

> Vue 的数据对象相比 React 的状态对象在代码膨胀的时候差距就来了。代码少的时候 Vue 的写法更为简洁，但组件状态很多，需要明确数据更新逻辑时，React 简单的 setState({}, callback)，就搞定了，Vue 有点让人摸不到头脑。React 的不可变（immutable）状态在**应用复杂时表现出的透明、可测试性更佳**。

## 模板分割

React的JSX语法使得模板分割更加自然、简单，因此也更容易将无状态组件抽离出来成为单独一部分以提高性能。好的代码组织能将常变与不变的部分进行分割解耦。

## api 和 生态

React：生态更加繁荣，React api相对更少，可以发挥的空间更大，很多场景下没有所谓的最佳实践，需要选择成本。此外；

Vue：生态相对不繁荣，内置了更多的API让开发者更容易上手，这使得使用Vue减少了很多选择成本，因为像状态管理和路由都有和Vue同步更新的库，往往不需要其他库来实现这方面的功能。

相对Vue来说，React只是一个视图框架，正如官网所说的UI = render(props)，React只能充当视图层，而Vue借鉴了MVVM模型，在此基础上实现了View-Model层的完整功能。

## 原生渲染

React：成熟的RN。

Vue：不成熟的Weex。

# Component, Element, Instance 之间有什么区别和联系？

**元素：** 元素是由JSX解析后生成的包含组件有用信息的对象，他并不是最终的fiber节点，他是mount时生成fiber对象以及diff时和old fiber进行比较的依据。

**组件：** 组件对于类式组件就是类本身，对于函数式组件就是函数本身；它存在于fiber节点的type属性上

**实例：** **组件实例是在render阶段实例化组件类后生成的对外暴露的组件属性**，它不是fiber节点，它在render的mount阶段是在constructClassInstance这个方法创建的，**然后它会被被挂载到对应的class component fiber节点的stateNode上面**。

![image-20220211130555815](https://github.com/NoAlligator/pico/blob/main/img/202203271802563.png?raw=true)

![image-20220211130948065](https://github.com/NoAlligator/pico/blob/main/img/202203271802564.png?raw=true)

![image-20220211130933663](https://github.com/NoAlligator/pico/blob/main/img/202203271802565.png?raw=true)

![image-20220211130642403](https://github.com/NoAlligator/pico/blob/main/img/202203271802566.png?raw=true)

# React如何判断什么时候重新渲染组件？

- Son get new Props
- this.setState()
- render()
- useState()'s setState
- useReducer()'s dispatch

# 对React中Fragment的理解，它的使用场景是什么？

在React中，组件返回的元素只能有一个根元素。为了不添加多余的DOM节点，我们可以使用Fragment标签来包裹所有的元素，Fragment标签不会渲染出任何元素。

# React中可以在render阶段访问refs吗？为什么？

不可以，因为无论是在mount还是update阶段，refs的绑定或者更新都是在layout阶段（也就是mutation之后）的commitAttachRef中进行的，只有在这个阶段不管是获取HostComponent还是组件的ref才是安全的。

> 注意：**函数式组件不能获取ref，因为他没有实例**，只能使用forward ref来转发函数式组件内部的ref。

# 类组件与函数组件有什么异同？

相同点：

都是可复用的基本单元，都会生成对应的fiber节点。无论是函数组件还是类组件，在使用方式和最终呈现效果上都是完全一致的。

**不同点：**

- 它们在开发时的心智模型上却存在巨大的差异**。类组件是基于面向对象编程的，它主打的是继承、生命周期等核心概念；而函数组件内核是函数式编程，主打的是 immutable、没有副作用、引用透明等特点。** 
- 之前，在使用场景上，如果存在需要使用生命周期的组件，那么主推类组件；设计模式上，如果需要使用继承，那么主推类组件。但现在**由于 React Hooks 的推出，生命周期概念的淡出，函数组件可以完全取代类组件。其次继承并不是组件最佳的设计模式，官方更推崇“组合优于继承”的设计概念，所以类组件在这方面的优势也在淡出。** 
- 性能优化上，**类组件主要依靠 shouldComponentUpdate 阻断渲染来提升性能，而函数组件依靠 React.memo 缓存渲染结果来提升性能。** 
- 从上手程度而言，**类组件更容易上手，从未来趋势上看，由于React Hooks 的推出，函数组件成了社区未来主推的方案。** 
- **类组件在未来时间切片与并发模式中，由于生命周期带来的复杂度，并不易于优化。而函数组件本身轻量简单，且在 Hooks 的基础上提供了比原先更细粒度的逻辑组织与复用，更能适应 React 的未来发展**

# React.memo

如果你的组件在相同 props 的情况下渲染相同的结果，那么你可以通过将其包装在 `React.memo` 中调用，以此通过记忆组件渲染结果的方式来提高组件的性能表现。这意味着在这种情况下，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果。`React.memo` 仅检查 props 变更。如果函数组件被 `React.memo` 包裹，且其实现中拥有 [`useState`](https://zh-hans.reactjs.org/docs/hooks-state.html)，[`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer) 或 [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext) 的 Hook，当 state 或 context 发生变化时，它仍会重新渲染。

# TS / Prop-types 区别

侧重点不同
 PropTypes是组件接收prop的约束。
 TypeScript类型约束主要是参数传递以及返回值的约束，两个东西侧重点不一样

部分相似功能
 通常我们编写一个 react 组件的时候，我们会去定义一个 prop-types 去校验我们的 class 的参数输入。而 ts 的 interface 的作用当然也是校验 props 的输入。

区别
 TypeScrip 的类型检查是静态的，prop-types 可以在运行时进行检查。你如你传了个offsetTop="abc"，你的编辑器可能会提示你类型有误，但是在浏览器里仍然是可以正常运行的。而如果你使用了 prop-types，在浏览器里就会给出提示。

# React Hooks 解决了哪些问题？

- 在组件之间复用状态逻辑很难
  - 可以使用 Hook 从组件中提取状态逻辑，使得这些逻辑可以单独测试并复用。Hook 使我们在无需修改组件结构的情况下复用状态逻辑。 这使得在组件间或社区内共享 Hook 变得更便捷。以往的HOC、render props、mixin模式往往不能很好的实现这样的功能。
- 复杂组件难以理解
  - Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据），而并非强制按照生命周期划分。以往使用生命周期往往很难将组件拆分为更小的粒度。

# React Hook 的使用限制有哪些

- 不要在循环、条件或嵌套函数中调用 Hook； 
- 只能在 React 的函数组件中调用 Hook。 

为什么不要在循环、条件或嵌套函数中调用 Hook 呢？Hooks是按照**被调用的顺序**保存在fiber节点的memorizedState

# React Hooks在平时开发中需要注意的问题？

- 不要在循环，条件或嵌套函数中调用Hook，必须始终在 React函数的顶层使用Hook；
- 遵循不可变数据原则；
- 善用useCallback优化以减少子组件更新；

# React Hooks 和生命周期的关系

**函数组件** 的本质是函数，没有 state 的概念的，因此**不存在生命周期**一说，仅仅是一个 **render 函数**而已。但是引入 **Hooks** 之后就变得不同了，它能让组件在不使用 class 的情况下拥有 state，所以就有了生命周期的概念，所谓的生命周期其实就是 `useState`、 `useEffect()` 和 `useLayoutEffect()` 。

即：**Hooks 组件（使用了Hooks的函数组件）有生命周期，而函数组件（未使用Hooks的函数组件）是没有生命周期的**。但是这里的生命周期和类式组件不是一套共用的生命周期。

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

# `useImperativeHandle`

这个Hook可以实现类似以往的父组件通过ref调用子组件实例上的方法，案例如下：

```jsx
import './App.css';
import {useImperativeHandle, Fragment, useRef, useState, forwardRef} from 'react'

export default function App() {
    const ref = useRef()
    return (
        <Fragment>
            <ChildWithForwardRef ref={ref}/>
            <button onClick={() => console.log(ref.current)}>Get Ref</button>
            <button onClick={() => {
                ref.current.setCount(ref.current.count + 1)
            }}>Update</button>
        </Fragment>
    )
}

const ChildWithForwardRef = forwardRef(Child)

function Child(props, ref) {
    const inputRef = useRef()
    const [count, setCount] = useState(0)
    useImperativeHandle(ref, () => ({
        inputRef: inputRef,
        count: count,
        setCount: setCount
    }))
    return (
        <Fragment>
            <h1>count: {count}</h1>
            <input type="text" ref={inputRef}/>
        </Fragment>
    )
}

```

