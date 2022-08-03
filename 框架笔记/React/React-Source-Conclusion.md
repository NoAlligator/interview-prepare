# React理念

React的理念是**快速响应**，制约快速响应的**两个**因素是：CPU瓶颈、IO瓶颈。[官网](https://zh-hans.reactjs.org/docs/thinking-in-react.html)

其中，CPU瓶颈指的是执行密集运算时JS线程抢占了GUI渲染的线程，导致页面掉帧、卡顿（主流浏览器刷新率为60Hz，即每16.6ms刷新一次，在每一帧时间内浏览器需要完成JS脚本执行、样式布局、样式绘制）。新的React架构通过时间切片（5ms[源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/forks/SchedulerHostConfig.default.js#L119)）的方式将长任务分散到每一帧中，React利用此时间进行组件更新，剩余的时间交给浏览器执行**样式布局和样式绘制**。但是这样有可能会牺牲低优先级任务的更新速度，而提高高优先级任务的响应速度（例如输入相关的更新）

IO瓶颈一般指的是网络延迟，这是前端开发者无法解决的，React通过在此类更新中使用**用户不可感知的短时间内进行延迟**，超过一定时间进入加载状态（fallback）的方式优化用户体验，这个功能需要配合[Suspense](https://zh-hans.reactjs.org/docs/concurrent-mode-suspense.html)功能及配套的hook——[useDeferredValue](https://zh-hans.reactjs.org/docs/concurrent-mode-reference.html#usedeferredvalue)进行使用。而在源码内部，为了支持这些特性，同样需要将**同步的更新**变为**可中断的异步更新**。

# 老Fiber架构(V15-)

老的Fiber架构(V15)分为两层：

- Reconciler：协调器，寻找变化的组件[（解释）](https://zh-hans.reactjs.org/docs/codebase-overview.html#reconcilers)
- Renderer：渲染器，将变化的组件渲染到页面上[（解释）](https://zh-hans.reactjs.org/docs/codebase-overview.html#renderers)

## Reconciler协调器

React通过this.setState()、this.forceUpdate()、ReactDOM.render()触发更新（老的的架构中并没有hook的概念），每当有更新发生的时候，Reconciler会做如下的工作：

- 调用/读取函数组件/类式组件的render方法/返回值，将返回的JSX转化为虚拟DOM；
- 将虚拟DOM和上次的更新后的虚拟DOM进行对比；
- 通过对比找出本次更新中变化的虚拟DOM；
- 通知Renderer将变化的的虚拟DOM渲染到页面上。

## Renderer渲染器

React支持跨平台，在不同的运行环境拥有不同的Renderer。在浏览器中的Renderer就是ReactDOM，除此之外还有ReactNative、ReactTest、ReactArt。当Render收到Renconciler的通知，会见变化的组件渲染在当前的宿主环境。

## 缺点

在**Reconciler**中，mount的组件会调用[mountComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，update的组件会调用[updateComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会**递归更新子组件**。递归更新子组件的去欸但是**更新一旦开始就无法中断**，如果更新是一个代价很高的操作，对应的JS线程会长时间阻塞GUI线程，造成用户交互的卡顿、假死。

源码：

更新组件会调用[updateComponent()](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)，该方法内部继续调用[_updateDOMChildren()](https://github.com/facebook/react/blob/571a9208d5133e8737f565fe60b762d201f0d37c/src/renderers/dom/shared/ReactDOMComponent.js#L1079)来更新子组件，[_updateDOMChildren()](https://github.com/facebook/react/blob/571a9208d5133e8737f565fe60b762d201f0d37c/src/renderers/dom/shared/ReactDOMComponent.js#L1079)存在递归调用。

## 老架构不支持异步更新

递归更新一旦开始，Reconciler和Renderer就会交替工作，如果此时强行中断更新，就会导致”不完全更新“，按照这样的设计，老的React架构是**无法实现异步更新的**。

# 新React架构(V16+)

新的React架构分为三层：

- Scheduler：调度器，调度任务的优先级，高优任务优先进入Reconciler
- Reconciler：协调器，负责找出变化的组件
- Renderer：渲染器，负责将变化的组件渲染到页面上

## Scheduler调度器

新的React进行任务中断是以浏览器是否有剩余时间作为标准的（Concurrent模式下），原生的API——[requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)因为存在缺陷（兼容性、稳定性），React实现了自己的的Polyfill（MessageChannel、setTimeout），除了在空闲时触发回调的功能外，**Scheduler**还提供了多种调度优先级供任务设置。[Scheduler](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/README.md)是独立于`React`的库。

## Reconciler协调器

- 递归 --> 异步可中断
- 协调器渲染器交替工作 --> 协调器打上标记完成所有工作后移交渲染器（完全内存中进行）

在老架构中，Reconciler递归处理虚拟DOM，但是在新的架构中，更新工作从**递归**变成了可**以中断的循环过程**，每次循环都会通过调用`shouldYield`判断当前是否有剩余时间：[源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1673)

```javascript
function workLoopConcurrent() {
    // Perform work until Scheduler asks us to yield
    while (workInProgress !== null && !shouldYield()) {
        performUnitOfWork(workInProgress);
    }
}
```

注：前提是开启了Concurrent模式，默认使用的ReactDOM.render依然是同步更新，但是已经摒弃了老架构的**严格”递归“更新**的模式：[源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1597)

```js
// The work loop is an extremely hot path. Tell Closure not to inline it.
/** @noinline */
function workLoopSync() {
    // Already timed out, so perform work without checking if we need to yield.
    while (workInProgress !== null) {
        performUnitOfWork(workInProgress);
    }
}
```

此外，Reconciler与Renderer的工作方式**不再是交替工作**，当**Scheduler**将任务交给**Reconciler**后，**Reconciler**会为变化的虚拟DOM打上代表增/删/更新的标记，类似这样：[标记源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js)

```js
export const Placement = /*             */ 0b0000000000010;
export const Update = /*                */ 0b0000000000100;
export const PlacementAndUpdate = /*    */ 0b0000000000110;
export const Deletion = /*              */ 0b0000000001000;
```

整个**Scheduler**与**Reconciler**的工作都在内存中进行。只有当所有组件都完成**Reconciler**的工作，才会统一交给**Renderer**。

## Renderer渲染器

**Renderer**根据**Reconciler**为虚拟DOM打的标记，同步执行对应的DOM操作。注：在协调器中被打断的更新因为都是在内存中进行的，所以**打断无法被用户感知**。

# Fiber心智模型

## React践行代数效应

React中做的就是践行**代数效应**，`代数效应`能够将`副作用`从函数逻辑中分离，使函数关注点保持纯粹。

## 具体应用

1. 对于类似`useState`、`useReducer`、`useRef`这样的`Hook`，我们不需要关注`FunctionComponent`的`state`在`Hook`中是如何保存的，`React`会为我们处理。我们只需要假设`useState`返回的是我们想要的`state`，并编写业务逻辑就行。
2. 以官方的[Suspense Demo](https://codesandbox.io/s/frosty-hermann-bztrp?file=/src/index.js:152-160)为例，在`Demo`中`ProfileDetails`用于展示`用户名称`。而`用户名称`是`异步请求`的，但是`Demo`中完全是`同步`的写法。因为使用`React Hook`使其能够解耦 数据请求 与 `DOM`更新 的逻辑（形式上异步转同步函数逻辑）

## 代数效应与Generator

异步可中断意味着在执行过程中（低优）更新可能会被打断（浏览器分片时间用尽、高优插队），当可以继续执行时恢复之前执行的中间状态。浏览器原生支持的可中断执行逻辑是Generator，但是其：

- 具有传染性，使用了`Generator`则上下文的其他函数也需要作出改变。
- 中间状态上下文关联且无法维持中间状态（只能依靠全局变量），更详细的解释可以参考[这个issue](https://github.com/facebook/react/issues/7942#issuecomment-254987818)

## 代数效应与Fiber

我们可以将`纤程`(Fiber)、`协程`(Generator)理解为`代数效应`思想在`JS`中的体现。`React`内部实现的一套状态更新机制。

`React Fiber`可以理解为：

`React`内部实现的一套状态更新机制。支持任务不同`优先级`，可中断与恢复，并且恢复后可以复用之前的`中间状态`。其中**每个任务更新单元为`React Element`对应的`Fiber节点`**。

# Fiber实现原理

## 三层含义

Fiber包含了三层含义：

1. 架构：作为架构来说，之前`React15`的`Reconciler`采用递归的方式执行，数据保存在递归调用栈中，所以被称为`stack Reconciler`。**`React16`的`Reconciler`基于`Fiber节点`实现，被称为`Fiber Reconciler`。**
2. 数据结构：作为静态的数据结构来说，每个`Fiber节点`对应一个`React element`，保存了该组件的类型（函数组件/类组件/原生组件...）、对应的DOM节点等信息。
3. 工作单元：作为动态的工作单元来说，每个`Fiber节点`保存了本次更新中该组件改变的状态、要执行的工作（需要被删除/被插入页面中/被更新...）。

## Fiber节点的结构

你可以从这里看到[Fiber节点的属性定义 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiber.new.js#L117)。虽然属性很多，但我们可以按三层含义将他们分类来看：

1. 架构层面
2. 静态数据结构层面
3. 动态工作单元层面

```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // （静态数据结构层面）作为静态数据结构的属性
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // （架构层面）用于连接其他Fiber节点形成Fiber树
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  // （动态工作单元层面）作为动态的工作单元的属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // （动态工作单元）调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
}
```

## Fiber作为 [架构]

Fiber节点依靠以下三个属性与其他Fiber节点形成Fiber树：

```js
// 指向父级Fiber节点
this.return = null;
// 指向子Fiber节点
this.child = null;
// 指向右边第一个兄弟Fiber节点
this.sibling = null;
```

```js
function App() {
  return (
    <div>
      i am
      <span>KaSong</span>
    </div>
  )
}
```

![Fiber架构](https://react.iamkasong.com/img/fiber.png)

**为什么父级指针叫做return？**因为Fiber节点作为工作单元，在执行完当前节点的completeWork且没有更多sibling节点的时候，会返回其父节点执行completeWork操作，所以直接使用return属性来指代父级节点。

## Fiber作为 [静态数据结构]

作为一种静态的数据结构，Fiber节点保存了组件相关的信息：

```js
// Fiber对应组件的类型 Function/Class/Host...
this.tag = tag;
// key属性
this.key = key;
// 大部分情况同type，某些情况不同，比如FunctionComponent使用React.memo包裹
this.elementType = null;
// 对于 FunctionComponent，指函数本身，对于ClassComponent，指class，对于HostComponent，指DOM节点tagName
this.type = null;
// Fiber对应的真实DOM节点
this.stateNode = null;
```

## Fiber [作为动态工作单元]

作为动态的工作单元，`Fiber`中如下参数保存了本次更新相关的信息：

```js
// 保存本次更新造成的状态改变相关信息
this.pendingProps = pendingProps;
this.memoizedProps = null;
this.updateQueue = null;
this.memoizedState = null;
this.dependencies = null;

this.mode = mode;

// 保存本次更新会造成的DOM操作
this.effectTag = NoEffect;
this.nextEffect = null;

this.firstEffect = null;
this.lastEffect = null;
```

```js
// 调度优先级相关
this.lanes = NoLanes;
this.childLanes = NoLanes;
```

# Fiber工作原理

## 双缓存技术

使用“双缓存技术”，在内存中绘制当前帧动画，绘制完毕后直接用当前帧替换上一帧画面，由于省去了两帧替换间的计算时间，不会出现从白屏到出现画面的闪烁情况。这种**在内存中构建并直接替换**的技术叫做[双缓存](https://baike.baidu.com/item/双缓冲)。`React`使用“双缓存”来完成`Fiber树`的构建与替换——对应着`DOM树`的创建与更新。

## 双缓存Fiber树

在`React`中最多会同时存在两棵`Fiber树`：

- 当前屏幕上显示内容对应的`Fiber树`称为`current Fiber树`，`current Fiber树`中的`Fiber节点`被称为`current fiber`。
- 正在内存中构建的`Fiber树`称为`workInProgress Fiber树`，`workInProgress Fiber树`中的`Fiber节点`被称为`workInProgress fiber`。

其中，`current fiber`和`workInProgress fiber`通过`alternate`连接（双向）：

```js
currentFiber.alternate === workInProgressFiber;
workInProgressFiber.alternate === currentFiber;
```

`React`应用的根节点`FiberRootNode`通过使`current`指针在不同`Fiber树`的`rootFiber`间切换来完成`current Fiber`树指向的切换。当`workInProgress Fiber树`构建完成交给`Renderer`渲染在页面上后，应用根节点的`current`指针指向`workInProgress Fiber树`，此时`workInProgress Fiber树`就变为`current Fiber树`。每次状态更新都会产生新的`workInProgress Fiber树`，通过`current`与`workInProgress`的替换，完成`DOM`更新。



## Fiber树挂载/更新工作流

如下组件为例：

```js
function App() {
  const [num, add] = useState(0);
  return (
    <p onClick={() => add(num + 1)}>{num}</p>
  )
}

ReactDOM.render(<App/>, document.getElementById('root'));
```

### mount

首次执行`ReactDOM.render`会创建`fiberRootNode`（源码中叫`fiberRoot`）和`rootFiber`。其中`fiberRootNode`是整个应用的根节点，`rootFiber`是`<App/>`所在组件树的根节点。

之所以要区分`fiberRootNode`与`rootFiber`，是因为在应用中我们**可以多次调用`ReactDOM.render`渲染不同的组件树**，他们会拥有不同的`rootFiber`，并通过`current`在不同的`rootFiber`之间切换完成更新。但是，整个应用的根节点只有一个，那就是`fiberRootNode` 

`fiberRootNode`的`current`会指向当前页面上已渲染内容对应`Fiber树`，即`current Fiber树`。`rootFiber`通过`stateNode`指向`fiberRootNode`

![Untitled.png](https://github.com/NoAlligator/pico/blob/main/img/7f6502636689492da29c8cd783c3b37a~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

```js
fiberRootNode.current = rootFiber
rootFiber.stateNode = fiberRootNode
```

接下来进入`render阶段`，根据组件返回的`JSX`在内存中依次创建`Fiber节点`并连接在一起构建`Fiber树`，被称为`workInProgress Fiber树`。在构建`workInProgress Fiber树`时会尝试复用`current Fiber树`中已有的`Fiber节点`内的属性，在`首屏渲染`时只有`rootFiber`存在对应的`current fiber`（即`rootFiber.alternate`）。

![workInProgressFiber](https://react.iamkasong.com/img/workInProgressFiber.png)

图中右侧已构建完的`workInProgress Fiber树`在`commit阶段`渲染到页面。此时`DOM`更新为右侧树对应的样子。`fiberRootNode`的`current`指针指向`workInProgress Fiber树`使其变为`current Fiber 树`。

![workInProgressFiberFinish](https://react.iamkasong.com/img/wipTreeFinish.png)

### update

接下来我们点击`p节点`触发状态改变，这会开启一次新的`render阶段`并构建一棵新的`workInProgress Fiber 树`。![wipTreeUpdate](https://react.iamkasong.com/img/wipTreeUpdate.png)

和`mount`时一样，`workInProgress fiber`的创建可以复用`current Fiber树`对应的节点数据，这个决定是否复用的过程就是Diff算法。

`workInProgress Fiber 树`在`render阶段`完成构建后进入`commit阶段`渲染到页面上。渲染完毕后，`workInProgress Fiber 树`变为`current Fiber 树`。

![currentTreeUpdate](https://react.iamkasong.com/img/currentTreeUpdate.png)

# 源码文件结构

除去配置文件和隐藏文件夹，根目录的文件夹包括三个：

```shell
根目录
├── fixtures        # 包含一些给贡献者准备的小型 React 测试项目
├── packages        # 包含元数据（比如 package.json）和 React 仓库中所有 package 的源码（子目录 src）
├── scripts         # 各种工具链的脚本，比如git、jest、eslint等
```

需要重点关注的是**packages**目录中的代码，他包含了React的核心代码

## packages folder

### [react](https://github.com/facebook/react/tree/master/packages/react)文件夹

React的核心，包含所有全局 React API，如：

- React.createElement
- React.Component
- React.Children

这些 API 是全平台通用的，它不包含`ReactDOM`、`ReactNative`等平台特定的代码。在 NPM 上作为[单独的一个包](https://www.npmjs.com/package/react)发布。

### [scheduler](https://github.com/facebook/react/tree/master/packages/scheduler)文件夹

Scheduler的实现。

### [shared](https://github.com/facebook/react/tree/master/packages/shared)文件夹

源码中其他模块公用的**方法**和**全局变量**。

### Renderer相关

如下几个文件夹为对应的**Renderer**

```text
- react-art
- react-dom                 # 注意这同时是DOM和SSR（服务端渲染）的入口
- react-native-renderer
- react-noop-renderer       # 用于debug fiber（后面会介绍fiber）
- react-test-renderer
```

### 试验性包的文件夹

`React`将自己流程中的一部分抽离出来，形成可以独立使用的包，由于他们是试验性质的，所以不被建议在生产环境使用。包括如下文件夹：

```text
- react-server        # 创建自定义SSR流
- react-client        # 创建自定义的流
- react-fetch         # 用于数据请求
- react-interactions  # 用于测试交互相关的内部特性，比如React的事件模型
- react-reconciler    # Reconciler的实现，你可以用他构建自己的Renderer
```

### 辅助包的文件夹

`React`将一些辅助功能形成单独的包。包括如下文件夹：

```text
- react-is       # 用于测试组件是否是某类型
- react-client   # 创建自定义的流
- react-fetch    # 用于数据请求
- react-refresh  # “热重载”的React官方实现
```

### [react-reconciler](https://github.com/facebook/react/tree/master/packages/react-reconciler)文件夹

需要重点关注**react-reconciler**，虽然他是一个实验性的包，内部的很多功能在正式版本中还未开放。但是他一边对接**Scheduler**，一边对接不同平台的**Renderer**，**构成了整个 React16 的架构体系**。

# JSX

## 旧的JSX转换

在ReactV17之前使用地JSX转换**必须显式地引入React**：`import React from 'react'`，否则在运行时该模块内就会报未定义变量 React的错误。

## 新的JSX转换

虽然 React 17 [并未包含新特性](https://zh-hans.reactjs.org/blog/2020/08/10/react-v17-rc.html)，但它将提供一个全新版本的 JSX 转换。在浏览器中无法直接使用 JSX，所以大多数 React 开发者需依靠 Babel 或 TypeScript 来**将 JSX 代码转换为 JavaScript**。在React17中，已经***不需要显式导入React了***，全新的转换，你可以**单独使用 JSX 而无需引入 React**。**旧的 JSX 转换**会把 JSX **直接转换为 React.createElement(...)调用**。新的 JSX 转换**不会将 JSX 转换为 React.createElement**，而是自动从 React 的 package 中引入新的入口函数并调用。现在Babel源代码**无需引入 React** 即可使用 JSX（但仍需引入 React，以便使用 React 提供的 Hook 或其他导出），**此次升级不会改变 JSX 语法，也并非必须**。旧的 JSX 转换将继续工作，没有计划取消对它的支持。[新的JSX官网介绍](https://zh-hans.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)

[简单聊聊React17-RC.2 新的 JSX 转换逻辑](https://juejin.cn/post/6877068521184952328)

## React Element

### React.createElement

该方法是React**用于生成React Element**的方法

```js
export function createElement(type, config, children) {
  let propName;

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    // 将 config 处理后赋值给 props
    // ...省略
  }

  const childrenLength = arguments.length - 2;
  // 处理 children，会被赋值给props.children
  // ...省略

  // 处理 defaultProps
  // ...省略

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}

const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记这是个 React Element
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  };

  return element;
};
```

`React.createElement`最终会调用`ReactElement`方法返回一个包含组件数据的对象，该对象有个参数`$$typeof: REACT_ELEMENT_TYPE`标记了该对象是个`React Element`。

如何确定该方法返回的就是React Element？

`React`提供了验证合法`React Element`的全局API [React.isValidElement](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react/src/ReactElement.js#L547)。可以看到，`$$typeof === REACT_ELEMENT_TYPE`的非`null`对象就是一个合法的`React Element`。换言之，在`React`中，所有`JSX`在运行时的返回结果（即`React.createElement()`的返回值）都是`React Element`。

```js
export function isValidElement(object) {
  return (
    typeof object === 'object' &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```

## React Component

在`React`中，我们常使用`ClassComponent`与`FunctionComponent`构建组件。他们对应的`Element`上的`type`属性是不同的：

- `ClassComponent`对应的`Element`的`type`字段为`AppClass`自身。
- `FunctionComponent`对应的`Element`的`type`字段为`AppFunc`自身。

```jsx
class AppClass extends React.Component {
  render() {
    return <p>KaSong</p>
  }
}
console.log('这是ClassComponent：', AppClass);
console.log('这是Element：', <AppClass/>);


function AppFunc() {
  return <p>KaSong</p>;
}
console.log('这是FunctionComponent：', AppFunc);
console.log('这是Element：', <AppFunc/>);
```

```js
// class component对应的React Element 
{
    type: ƒ AppClass() {}
    key: null
    ref: null
    props: Object
    _owner: null
    _store: Object
}

// function component对应的React Element
{
    type: ƒ AppFunc() {}
    key: null
    ref: null
    props: Object
    _owner: null
    _store: Object
}
```

注：无法通过引用类型区分`ClassComponent`和`FunctionComponent`。因为两者都是由`Function`类型实现的实例，`React`通过`ClassComponent`实例原型上的`isReactComponent`变量判断是否是`ClassComponent`。

```js
ClassComponent.prototype.isReactComponent = {};
FunctionComponent.prototype.isReactComponent = undefined;
```

## React Component 和 React Element

`React.createElement()`的第一个参数`type`指的是`React Component`（如果是`function component`，`type`对应的是`function`本身；如果是`class component`，`type`对应的就是该`class`），调用`React.createElement()`返回`React Element`。React Element对象是JSX初步解析后形成的对象，通过后续`render`阶段中的`createFiberFromElement(element, returnFiber.mode, lanes)`操作后形成`Fiber`节点。

# Render阶段

## Render入口函数`workLoopSync` / `workLoopConcurrent`

`render阶段`开始于`performSyncWorkOnRoot`或`performConcurrentWorkOnRoot`方法的调用。这取决于本次更新是**同步更新（Legacy）**还是**异步更新（Concurrent）**。他们唯一的区别是是否调用`shouldYield`。如果当前浏览器帧没有剩余时间，`shouldYield`会中止循环，直到浏览器有空闲时间后再继续遍历。[源码](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1599)

```js
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
    //workInProgress代表当前已创建的workInProgress fiber
  while (workInProgress !== null) {
      //performUnitOfWork方法会创建下一个Fiber节点并赋值给workInProgress，并将workInProgress与已创建的Fiber节点连接起来构成Fiber树（“递归”）。
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

## Render工作流 “递” → “归”

`Fiber Reconciler`是从`Stack Reconciler`而来，**通过遍历的方式实现可中断的递归，所以`performUnitOfWork`的工作可以分为两部分：“递”和“归”。**

### “递”阶段

> 首先从`rootFiber`开始向下**深度优先遍历DFS**。为遍历到的每个`Fiber节点`调用[beginWork方法 ](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3058)。该方法会根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来。

### “归”阶段

> 在“归”阶段会调用[completeWork](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L652)处理`Fiber节点`。当某个`Fiber节点`执行完`completeWork`，如果其存在`兄弟Fiber节点`（即`fiber.sibling !== null`），会进入其`兄弟Fiber`的“递”阶段。如果不存在`兄弟Fiber`，会进入`父级Fiber`的“归”阶段。“递”和“归”阶段会交错执行直到“归”到`rootFiber`。至此，`render阶段`的工作就结束了。（该阶段主要执行的操作是**在mount阶段生成Fiber对应的真实DOM节点并将子孙DOM节点插入刚生成的DOM节点**、处理DOM节点的props）

### 概览

以<App/>组件为例：

```js
function App() {
  return (
    <div>
      i am
      <span>KaSong</span>
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById("root"));
```

对应的Fiber Tree结构：

![Fiber架构](https://react.iamkasong.com/img/fiber.png)

Render阶段依次执行：

```sh
1. rootFiber beginWork
2. App Fiber beginWork
3. div Fiber beginWork
4. "i am" Fiber beginWork
5. "i am" Fiber completeWork
6. span Fiber beginWork
7. span Fiber completeWork
8. div Fiber completeWork
9. App Fiber completeWork
10. rootFiber completeWork
```

> 之所以没有 “KaSong” Fiber 的 beginWork/completeWork，是因为作为一种性能优化手段，针对**只有单一文本子节点的`Fiber`，`React`会特殊处理。**

## Render之“递”阶段

[源码](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3075)。`beginWork`的工作是传入`当前Fiber节点 → current`，创建/复用`子Fiber节点 → workInProgress.child`。

### 函数参数

- current：当前组件对应的`Fiber节点`在上一次更新时的`Fiber节点`，即`workInProgress.alternate`
- workInProgress：当前组件对应的`Fiber节点`
- renderLanes：优先级相关

```js
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  // ...省略函数体
}
```

从[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html)我们知道，除[`rootFiber`](https://react.iamkasong.com/process/doubleBuffer.html#mount时)以外， 组件`mount`时，由于是首次渲染，是不存在当前组件对应的`Fiber节点`在上一次更新时的`Fiber节点`，即`mount`时`current === null`。组件`update`时，由于之前已经`mount`过，所以`current !== null`。所以我们可以通过`current === null ?`来区分组件是处于`mount`还是`update`。

基于此原因，`beginWork`的工作可以分为两部分：

- `update`时：如果`current`存在，在满足一定条件时可以复用`current`节点，这样就能克隆`current.child`作为`workInProgress.child`，而不需要新建`workInProgress.child`。
- `mount`时：除`fiberRootNode`以外，`current === null`。会根据`fiber.tag`不同，创建不同类型的`子Fiber节点`

```js
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
): Fiber | null {

  // update时：如果current存在可能存在优化路径，可以复用current（即上一次更新的Fiber节点）
  if (current !== null) {
    // ...省略

    // 复用current
    return bailoutOnAlreadyFinishedWork(
      current,
      workInProgress,
      renderLanes,
    );
  } else {
    didReceiveUpdate = false;
  }

  // mount时：根据tag不同，创建不同的子Fiber节点
  switch (workInProgress.tag) {
    case IndeterminateComponent: 
      // ...省略
    case LazyComponent: 
      // ...省略
    case FunctionComponent: 
      // ...省略
    case ClassComponent: 
      // ...省略
    case HostRoot:
      // ...省略
    case HostComponent:
      // ...省略
    case HostText:
      // ...省略
    // ...省略其他类型
  }
}
```

### “递”阶段mount时

当不满足优化路径时，beginWork就进入第二部分，新建`子Fiber`。beginWork根据`fiber.tag`不同，进入不同类型`Fiber`的创建逻辑。

> 可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactWorkTags.js)看到`tag`对应的组件类型，其中rootFiber对应的tag是3，fiberRootNode对应的是0。

```js
// mount时：根据tag不同，创建不同的Fiber节点
switch (workInProgress.tag) {
  case IndeterminateComponent: 
    // ...省略
  case LazyComponent: 
    // ...省略
  case FunctionComponent: 
    // ...省略
  case ClassComponent: 
    // ...省略
  case HostRoot:
    // ...省略
  case HostComponent:
    // ...省略
  case HostText:
    // ...省略
  // ...省略其他类型
}
```

对于我们常见的组件类型，如（`FunctionComponent` / `ClassComponent` /  `HostComponent`），最终会进入[reconcileChildren](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L233)方法。

### “递”阶段update时

我们可以看到，满足如下情况时`didReceiveUpdate === false`，**即可以直接复用前一次更新的`子Fiber`，不需要新建`子Fiber`**：

1. `oldProps === newProps && workInProgress.type === current.type`，即`props`与`fiber.type`不变（type + props不变）；
2. `!includesSomeLane(renderLanes, updateLanes)`，即当前`Fiber节点`优先级不够。

```js
if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;

    if (
      oldProps !== newProps ||
      hasLegacyContextChanged() ||
      (__DEV__ ? workInProgress.type !== current.type : false)
    ) {
      didReceiveUpdate = true;
    } else if (!includesSomeLane(renderLanes, updateLanes)) {
      didReceiveUpdate = false;
      switch (workInProgress.tag) {
        // 省略处理
      }
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderLanes,
      );
    } else {
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }
```

### reconcileChildren

从该函数名就能看出这是`Reconciler`模块的核心部分。那么他究竟做了什么呢？

- 对于`mount`的组件，他会创建新的`子Fiber节点`；
- 对于`update`的组件，他会将当前组件与该组件在上次更新时对应的`Fiber节点`比较（也就是俗称的`Diff`算法），将比较的结果生成新`Fiber节点`。

rootFiber进入reconcileChildren时的调用栈

![image-20220207202529291](https://github.com/NoAlligator/pico/blob/main/img/202203271805116.png)

```js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes
) {
  if (current === null) {
    // 对于mount的组件
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes,
    );
  } else {
    // 对于update的组件
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```

从代码可以看出，和`beginWork`一样，他也是通过`current === null ?`区分`mount`与`update`。

不论走哪个逻辑，最终他会生成新的子`Fiber节点`并赋值给`workInProgress.child`，作为本次`beginWork`[返回值](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L1158)，并作为下次`performUnitOfWork`执行时`workInProgress`的[传参](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1702)。

> 注：值得一提的是，`mountChildFibers`与`reconcileChildFibers`这两个方法的逻辑基本一致，都是进入ChildReconciler(shouldTrackSideEffects)。唯一的区别是：`reconcileChildFibers`会为生成的`Fiber节点`带上`effectTag`属性（表示产生了更新、删除或其他副作用），而`mountChildFibers`不会附带`effectTag`。`rootFiber`要进入`reconcileChildFibers`来获得`Placement effect tag`从而得以在`mount`阶段将整个应用的`fiber`树挂载到`fiberRootNode`上。
>
> ```js
> var reconcileChildFibers = ChildReconciler(true);
> var mountChildFibers = ChildReconciler(false);
> ```

### effectTag

`render阶段`的工作是在内存中进行，**当工作结束后会通知`Renderer`需要执行的`DOM`操作**。要执行`DOM`操作的具体类型就保存在`fiber.effectTag`中。

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js)看到`effectTag`对应的`DOM`操作

```js
// DOM需要插入到页面中
export const Placement = /*                */ 0b00000000000010;
// DOM需要更新
export const Update = /*                   */ 0b00000000000100;
// DOM需要插入到页面中并更新
export const PlacementAndUpdate = /*       */ 0b00000000000110;
// DOM需要删除
export const Deletion = /*                 */ 0b00000000001000;
```

为什么使用二进制来表示`effectTag`？

通过二进制表示`effectTag`，可以很方便的使用位操作为`fiber.effectTag`赋值多个`effect`。也可以很方便的通过位运算确定是否属于某个状态。

### 问题

- 为什么rootFiber会进入reconcileChildren？

假设`mountChildFibers`也会赋值`effectTag`，那么可以预见`mount`时整棵`Fiber树`所有节点都会有`Placement effectTag`。那么`commit阶段`在执行`DOM`操作时每个节点都会执行一次插入操作，这样大量的`DOM`操作是极低效的。为了解决这个问题，在`mount`时只有`rootFiber`会赋值`Placement effectTag`，在`commit阶段`只会执行一次插入操作。`rootFiber`是为了给子元素附上`Placement`的`effect tag`，使得的其在`commit`的`mutation`阶段能够通过`placement effectTag`被正确地渲染。

- 其他子元素没有被附加上`Placement effect tag`，那么首屏渲染如何完成呢？

所有子元素的fiber都会被挂载在父元素fiber的children属性上，最终必定有一个fiber元素指向rootFiber，通过绑定在rootFiber上的stateNode属性便可以获取到fiberRootNode。而正因为rootFiber的子节点都被附加了placement effectTag，所以它以及它的子组件最终都会被渲染到页面上。

### 流程图

![image-20220207204759415](https://github.com/NoAlligator/pico/blob/main/img/202203271805118.png)

## Render之“归阶段”

### 流程概览

类似`beginWork`，`completeWork`也是针对不同`fiber.tag`调用不同的处理逻辑：

```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;

  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      return null;
    case ClassComponent: {
      // ...省略
      return null;
    }
    case HostRoot: {
      // ...省略
      updateHostContainer(workInProgress);
      return null;
    }
    case HostComponent: {
      // ...省略
      return null;
    }
  // ...省略
```

### 处理HostComponent

和`beginWork`一样，我们根据`current === null ?`判断是`mount`还是`update`。同时针对`HostComponent`，判断`update`时我们还需要考虑`workInProgress.stateNode != null ?`（即该`Fiber节点`是否存在对应的`DOM节点`）

```js
case HostComponent: {
  popHostContext(workInProgress);
  const rootContainerInstance = getRootHostContainer();
  const type = workInProgress.type;

  if (current !== null && workInProgress.stateNode != null) {
    // update的情况
    // ...省略
  } else {
    // mount的情况
    // ...省略
  }
  return null;
}
```

### “归”阶段mount时

`mount`时的主要逻辑包括三个：

- 为`Fiber节点`生成对应的`DOM节点`；
- 将子孙`DOM节点`插入刚生成的`DOM节点`中；
- 与`update`逻辑中的`updateHostComponent`类似的处理`props`的过程。

```js
// mount的情况

// ...省略服务端渲染相关逻辑

const currentHostContext = getHostContext();
// 为fiber创建对应DOM节点
const instance = createInstance(
    type,
    newProps,
    rootContainerInstance,
    currentHostContext,
    workInProgress,
  );
// 将子孙DOM节点插入刚生成的DOM节点中
appendAllChildren(instance, workInProgress, false, false);
// DOM节点赋值给fiber.stateNode
workInProgress.stateNode = instance;

// 与update逻辑中的updateHostComponent类似的处理props的过程
if (
  finalizeInitialChildren(
    instance,
    type,
    newProps,
    rootContainerInstance,
    currentHostContext,
  )
) {
  markUpdate(workInProgress);
}
```

`mount`时只会在`rootFiber`存在`Placement effectTag`。那么`commit阶段`是如何通过一次插入`DOM`操作（对应一个`Placement effectTag`）将整棵`DOM树`插入页面的呢？

原因就在于`completeWork`中的`appendAllChildren`方法。由于`completeWork`属于“归”阶段调用的函数，每次调用`appendAllChildren`时都会将已生成的子孙`DOM节点`插入当前生成的`DOM节点`下。那么当“归”到`rootFiber`时，我们已经有一个构建好的离屏`DOM树`。



### “归”阶段update时

当`update`时，`Fiber节点`已经存在对应`DOM节点`，所以不需要生成`DOM节点`。**需要做的主要是处理`props`**，比如：

- `onClick`、`onChange`等回调函数的注册
- 处理`style prop`
- 处理`DANGEROUSLY_SET_INNER_HTML prop`
- 处理`children prop`

`update`中最主要的逻辑是调用`updateHostComponent`方法：

```js
if (current !== null && workInProgress.stateNode != null) {
  // update的情况
  updateHostComponent(
    current,
    workInProgress,
    type,
    newProps,
    rootContainerInstance,
  );
}
```

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L225)看到`updateHostComponent`方法定义。

在`updateHostComponent`内部，被处理完的`props`会被赋值给`workInProgress.updateQueue`，并最终会在`commit阶段`被渲染在页面上。

```ts
workInProgress.updateQueue = (updatePayload: any);
```

其中`updatePayload`为数组形式，他的偶数索引的值为变化的`prop key`，奇数索引的值为变化的`prop value`。

> 具体渲染过程见[mutation阶段一节](https://react.iamkasong.com/renderer/mutation.html#hostcomponent-mutation)

## effectList 实现部分更新的关键

完成`effectList`的构建意味着`render阶段`的绝大部分工作已经完成。作为`DOM`操作的依据，`commit阶段`需要找到所有有`effectTag`的`Fiber节点`并依次执行`effectTag`对应操作。在`commit阶段`再遍历一次`Fiber树`寻找`effectTag !== null`的`Fiber节点`是很低效的。为了解决这个问题，在`completeWork`的上层函数`completeUnitOfWork`中，每个执行完`completeWork`且存在`effectTag`的`Fiber节点`会被保存在一条被称为`effectList`的**单向链表**中。

`effectList`中第一个`Fiber节点`保存在`fiber.firstEffect`，最后一个元素保存在`fiber.lastEffect`。类似`appendAllChildren`，在“归”阶段，所有有`effectTag`的`Fiber节点`都会被追加在`effectList`中，最终形成一条以`rootFiber.firstEffect`为起点的**单向链表**。

```js
                       nextEffect         nextEffect
rootFiber.firstEffect -----------> fiber -----------> fiber
```

这样，在`commit阶段`只需要遍历`effectList`就能执行所有`effect`进行部分更新了。

#### effectList链表演示代码：

```jsx
import  { useState } from 'react'
function App() {
    const [num, setNum] = useState(0)
    return (
        <div>
            <header>
                <p onClick={() => setNum(num => num + 1)}>
                    <code title={num}>{num}</code>
                </p>
                <a href="https://reactjs.org" target="_blank">
                    Learn React
                </a>
            </header>
        </div>
    )	
}
```

生成的链表如下：

![image-20220202104657599](https://github.com/NoAlligator/pico/blob/main/img/image-20220202104657599.png)

## completeWork流程图

![img](https://github.com/NoAlligator/pico/blob/main/img/completeWork.png)

# Commit阶段

> `commitRoot`方法是`commit阶段`工作的起点。`fiberRootNode`会作为传参。
>
> ```js
> commitRoot(root);
> ```
>
> ![image-20220208131833508](https://github.com/NoAlligator/pico/blob/main/img/202203271805119.png)

## 阶段概述

1. 执行effectList：在`rootFiber.firstEffect`上保存了一条需要执行`副作用`的`Fiber节点`的单向链表`effectList`，这些`Fiber节点`的`updateQueue`中保存了变化的`props`。这些`副作用`对应的`DOM操作`在`commit`阶段执行。
2. 执行生命周期函数/钩子函数：生命周期钩子（比如`componentDidXXX`）、`hook`（比如`useEffect`）需要在`commit`阶段执行。

## 阶段划分

`commit`阶段的主要工作（即`Renderer`的工作流程）分为三部分：

- before mutation阶段（执行`DOM`操作前）
- mutation阶段（**执行`DOM`操作**）
- layout阶段（执行`DOM`操作后，可以确保当前最新`DOM`的获取）

## ⭐before mutation之前 / layout之后执行的操作

### before mutation之前

`commitRootImpl`方法中直到第一句`if (firstEffect !== null)`之前属于`before mutation`之前。可以看到，`before mutation`之前主要做一些变量赋值，状态重置的工作。只需要关注最后赋值的`firstEffect`，在`commit`的三个子阶段都会用到他。

```js
do {
    // 触发useEffect回调与其他同步任务。由于这些任务可能触发新的渲染，所以这里要一直遍历执行直到没有任务
    flushPassiveEffects();
  } while (rootWithPendingPassiveEffects !== null);

  // root指 fiberRootNode
  // root.finishedWork指当前应用的rootFiber
  const finishedWork = root.finishedWork;

  // 凡是变量名带lane的都是优先级相关
  const lanes = root.finishedLanes;
  if (finishedWork === null) {
    return null;
  }
  root.finishedWork = null;
  root.finishedLanes = NoLanes;

  // 重置Scheduler绑定的回调函数
  root.callbackNode = null;
  root.callbackId = NoLanes;

  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  // 重置优先级相关变量
  markRootFinished(root, remainingLanes);

  // 清除已完成的discrete updates，例如：用户鼠标点击触发的更新。
  if (rootsWithPendingDiscreteUpdates !== null) {
    if (
      !hasDiscreteLanes(remainingLanes) &&
      rootsWithPendingDiscreteUpdates.has(root)
    ) {
      rootsWithPendingDiscreteUpdates.delete(root);
    }
  }

  // 重置全局变量
  if (root === workInProgressRoot) {
    workInProgressRoot = null;
    workInProgress = null;
    workInProgressRootRenderLanes = NoLanes;
  } else {
  }

  // 将effectList赋值给firstEffect
  // 由于每个fiber的effectList只包含他的子孙节点
  // 所以根节点如果有effectTag则不会被包含进来
  // 所以这里将有effectTag的根节点插入到effectList尾部
  // 这样才能保证有effect的fiber都在effectList中
  let firstEffect;
  if (finishedWork.effectTag > PerformedWork) {
    if (finishedWork.lastEffect !== null) {
      finishedWork.lastEffect.nextEffect = finishedWork;
      firstEffect = finishedWork.firstEffect;
    } else {
      firstEffect = finishedWork;
    }
  } else {
    // 根节点没有effectTag
    firstEffect = finishedWork.firstEffect;
  }
```

### layout之后

```js
const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;

// useEffect相关
if (rootDoesHavePassiveEffects) {
  rootDoesHavePassiveEffects = false;
  rootWithPendingPassiveEffects = root;
  pendingPassiveEffectsLanes = lanes;
  pendingPassiveEffectsRenderPriority = renderPriorityLevel;
} else {
    //...
}

// 性能优化相关
if (remainingLanes !== NoLanes) {
  if (enableSchedulerTracing) {
    // ...
  }
} else {
  // ...
}

// 性能优化相关
if (enableSchedulerTracing) {
  if (!rootDidHavePassiveEffects) {
    // ...
  }
}

// ...检测无限循环的同步任务
if (remainingLanes === SyncLane) {
  // ...
} 

// 在离开commitRoot函数前调用，触发一次新的调度，确保任何附加的任务被调度
ensureRootIsScheduled(root, now());

// ...处理未捕获错误及老版本遗留的边界问题


// 执行同步任务，这样同步任务不需要等到下次事件循环再执行
// 比如在 componentDidMount 中执行 setState 创建的更新会在这里被同步执行
// 或useLayoutEffect
flushSyncCallbackQueue();

return null;
```

> 可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2195)看到这段代码

主要包括三点内容：

1. **执行`useEffect`相关的处理。**
2. **执行性能追踪相关操作。**
   - 源码里有很多和`interaction`相关的变量。他们都和追踪`React`渲染时间、性能相关，在[Profiler API](https://zh-hans.reactjs.org/docs/profiler.html)和[DevTools ](https://github.com/facebook/react-devtools/pull/1069)中使用。你可以在这里看到[interaction的定义](https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16#overview)
3. **执行commit阶段触发的同步任务。**这样同步任务不需要等到下次事件循环再执行。比如在 `componentDidMount` 中执行 `setState` 创建的更新会**在这里被同步执行**，或者是`useLayoutEffect`传入回调触发的**同步更新**。
   - 而对于“异步”的回调方法中触发新的更新会开启**新的`render-commit`流程**。

## ⭐commit阶段一：before mutation

### 概览

`before mutation阶段`的代码很短，整个过程就是遍历`effectList`并调用`commitBeforeMutationEffects`函数处理。

> 这部分[源码在这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2104-L2127)。

```js
// 保存之前的优先级，以同步优先级执行，执行完毕后恢复之前优先级
const previousLanePriority = getCurrentUpdateLanePriority();
setCurrentUpdateLanePriority(SyncLanePriority);

// 将当前上下文标记为CommitContext，作为commit阶段的标志
const prevExecutionContext = executionContext;
executionContext |= CommitContext;

// 处理focus状态
focusedInstanceHandle = prepareForCommit(root.containerInfo);
shouldFireAfterActiveInstanceBlur = false;

// beforeMutation阶段的主函数
commitBeforeMutationEffects(finishedWork);

focusedInstanceHandle = null;
```

### 阶段核心函数commitBeforeMutationEffects

整体可以分为三部分：

1. 处理`DOM节点`渲染/删除后的 `autoFocus`、`blur` 逻辑。
2. 调用`getSnapshotBeforeUpdate`生命周期钩子。
3. 调度`useEffect`。

```js
function commitBeforeMutationEffects() {
  while (nextEffect !== null) {
    const current = nextEffect.alternate;

    if (!shouldFireAfterActiveInstanceBlur && focusedInstanceHandle !== null) {
      // ...focus blur相关
    }

    const effectTag = nextEffect.effectTag;

    // 调用getSnapshotBeforeUpdate
    if ((effectTag & Snapshot) !== NoEffect) {
      commitBeforeMutationEffectOnFiber(current, nextEffect);
    }

    // 调度useEffect
    if ((effectTag & Passive) !== NoEffect) {
      if (!rootDoesHavePassiveEffects) {
        rootDoesHavePassiveEffects = true;
        scheduleCallback(NormalSchedulerPriority, () => {
          flushPassiveEffects();
          return null;
        });
      }
    }
    nextEffect = nextEffect.nextEffect;
  }
}
```

#### 在beforeMuation调用`getSnapshotBeforeUpdate`：commitBeforeMutationEffectOnFiber

`commitBeforeMutationEffectOnFiber`是`commitBeforeMutationLifeCycles`的别名。在该方法内会调用`getSnapshotBeforeUpdate`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCommitWork.old.js#L222)看到这段逻辑

从`React`v16开始，`componentWillXXX`钩子前增加了`UNSAFE_`前缀。

究其原因，是因为`Stack Reconciler`重构为`Fiber Reconciler`后，`render阶段`的任务可能中断/重新开始，对应的组件在`render阶段`的生命周期钩子（即`componentWillXXX`）可能触发多次。

这种行为和`React`v15不一致，所以标记为`UNSAFE_`。

> 更详细的解释参照[这里(opens new window)](https://juejin.im/post/6847902224287285255#comment)

为此，`React`提供了替代的生命周期钩子`getSnapshotBeforeUpdate`。

我们可以看见，`getSnapshotBeforeUpdate`是在`commit阶段`内的`before mutation阶段`调用的，由于`commit阶段`是同步的，所以不会遇到多次调用的问题。

#### 在beforeMutation调度`useEffect`回调：scheduleCallback + flushPassiveEffects

在这几行代码内，`scheduleCallback`方法由`Scheduler`模块提供，用于以某个优先级异步调度一个回调函数。在此处，被异步调度的回调函数就是触发`useEffect`的方法`flushPassiveEffects`。

```js
// 调度useEffect
if ((effectTag & Passive) !== NoEffect) {
  if (!rootDoesHavePassiveEffects) {
    rootDoesHavePassiveEffects = true;
    scheduleCallback(NormalSchedulerPriority, () => {
      // 触发useEffect
      flushPassiveEffects();
      return null;
    });
  }
}
```

##### 如何异步调度

在`flushPassiveEffects`方法内部会从全局变量`rootWithPendingPassiveEffects`获取`effectList`。关于`flushPassiveEffects`的具体讲解参照[useEffect与useLayoutEffect一节](https://react.iamkasong.com/hooks/useeffect.html)。`effectList`中保存了需要执行副作用的`Fiber节点`。其中副作用包括：

- 插入`DOM节点`（Placement）
- 更新`DOM节点`（Update）
- 删除`DOM节点`（Deletion）

除此外，当一个`FunctionComponent`含有`useEffect`或`useLayoutEffect`，他对应的`Fiber节点`也会被赋值`effectTag`（`Hook Effect Tag`）。

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactHookEffectTags.js)看到`hook`相关的`effectTag`

在`flushPassiveEffects`方法内部会遍历`rootWithPendingPassiveEffects`（即`effectList`）执行`effect`回调函数。

`rootWithPendingPassiveEffects`会在何时赋值呢？

`layout之后`根据`rootDoesHavePassiveEffects === true?`决定是否赋值`rootWithPendingPassiveEffects`。

```js
const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;
if (rootDoesHavePassiveEffects) {
  rootDoesHavePassiveEffects = false;
  rootWithPendingPassiveEffects = root;
  pendingPassiveEffectsLanes = lanes;
  pendingPassiveEffectsRenderPriority = renderPriorityLevel;
}
```

所以整个`useEffect`异步调用分为三步：

1. `before mutation阶段`在`scheduleCallback`中调度`flushPassiveEffects`
2. `layout阶段`之后将`effectList`赋值给`rootWithPendingPassiveEffects`
3. `scheduleCallback`触发`flushPassiveEffects`，`flushPassiveEffects`内部遍历`rootWithPendingPassiveEffects`

##### 为什么需要异步调用

摘录自`React`文档[effect 的执行时机](https://zh-hans.reactjs.org/docs/hooks-reference.html#timing-of-effects)：

> 与 `componentDidMount`、`componentDidUpdate` 不同的是，在浏览器完成布局与绘制之后，传给 `useEffect` 的函数会延迟调用（延迟到`layout`完成之后）。这使得它适用于许多常见的副作用场景，比如**设置订阅和事件处理**等情况，因此不应在函数中执行阻塞浏览器更新屏幕的操作。

由此可知：`useEffect`异步执行的原因主要是**防止同步执行时阻塞浏览器渲染**。

## ⭐commit阶段二：mutation

### 概览

类似`before mutation阶段`，`mutation阶段`也是**遍历`effectList`，执行函数**。这里执行的是`commitMutationEffects`。

```js
nextEffect = firstEffect;
do {
  try {
      commitMutationEffects(root, renderPriorityLevel);
    } catch (error) {
      invariant(nextEffect !== null, 'Should be working on an effect.');
      captureCommitPhaseError(nextEffect, error);
      nextEffect = nextEffect.nextEffect;
    }
} while (nextEffect !== null);
```

### 执行fiber节点effectList更新的关键方法：commitMutationEffects

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.old.js#L2091)看到`commitMutationEffects`源码

`commitMutationEffects`会遍历`effectList`，对每个`Fiber节点`执行如下三个操作：

1. 根据`ContentReset effectTag`重置文字节点；
2. 更新`ref`；
3. 根据`effectTag`分别处理，其中`effectTag`包括(`Placement` | `Update` | `Deletion` | `Hydrating`)。

```js
function commitMutationEffects(root: FiberRoot, renderPriorityLevel) {
  // 遍历effectList
  while (nextEffect !== null) {

    const effectTag = nextEffect.effectTag;

    // 根据 ContentReset effectTag重置文字节点
    if (effectTag & ContentReset) {
      commitResetTextContent(nextEffect);
    }

    // 更新ref
    if (effectTag & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
    }

    // 根据 effectTag 分别处理
    const primaryEffectTag =
      effectTag & (Placement | Update | Deletion | Hydrating);
    switch (primaryEffectTag) {
      // 插入DOM
      case Placement: {
        commitPlacement(nextEffect);
        nextEffect.effectTag &= ~Placement;
        break;
      }
      // 插入DOM 并 更新DOM
      case PlacementAndUpdate: {
        // 插入
        commitPlacement(nextEffect);

        nextEffect.effectTag &= ~Placement;

        // 更新
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // SSR
      case Hydrating: {
        nextEffect.effectTag &= ~Hydrating;
        break;
      }
      // SSR
      case HydratingAndUpdate: {
        nextEffect.effectTag &= ~Hydrating;

        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // 更新DOM
      case Update: {
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // 删除DOM
      case Deletion: {
        commitDeletion(root, nextEffect, renderPriorityLevel);
        break;
      }
    }

    nextEffect = nextEffect.nextEffect;
  }
}
```

#### 新增：Placement effect

当`Fiber节点`含有`Placement effectTag`，意味着该`Fiber节点`对应的`DOM节点`需要插入到页面中。调用的方法为`commitPlacement`。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1156)看到`commitPlacement`源码

该方法所做的工作分为三步：

- 获取父级`DOM节点`。其中`finishedWork`为传入的`Fiber节点`。

```js
const parentFiber = getHostParentFiber(finishedWork);
// 父级DOM节点
const parentStateNode = parentFiber.stateNode;
```

- 获取`Fiber节点`的`DOM`兄弟节点

```js
const before = getHostSibling(finishedWork);
```

- 根据`DOM`兄弟节点是否存在决定调用`parentNode.insertBefore`或`parentNode.appendChild`执行`DOM`插入操作。

```js
// parentStateNode是否是rootFiber
if (isContainer) {
  insertOrAppendPlacementNodeIntoContainer(finishedWork, before, parent);
} else {
  insertOrAppendPlacementNode(finishedWork, before, parent);
}
```

值得注意的是，`getHostSibling`（获取兄弟`DOM节点`）的执行很耗时，当在同一个父`Fiber节点`下依次执行多个插入操作，`getHostSibling`算法的复杂度为指数级。这是由于`Fiber节点`不只包括`HostComponent`，所以`Fiber树`和渲染的`DOM树`节点并不是一一对应的。要从`Fiber节点`找到`DOM节点`很可能跨层级遍历。

考虑如下例子：

```jsx
function Item() {
  return <li><li>;
}

function App() {
  return (
    <div>
      <Item/>
    </div>
  )
}

ReactDOM.render(<App/>, document.getElementById('root'));
```

对应的`Fiber树`和`DOM树`结构为：

```js
// Fiber树
          child      child      child       child
rootFiber -----> App() -----> div -----> Item -----> li

// DOM树
#root ---> div ---> li
```

当在`div`的子节点`Item`前插入一个新节点`p`，即`App`变为：

```jsx
function App() {
  return (
    <div>
      <p></p>
      <Item/>
    </div>
  )
}
```

对应的`Fiber树`和`DOM树`结构为：

```js
// Fiber树
          child      child      child
rootFiber -----> App -----> div -----> p 
                                       | sibling       child
                                       | -------> Item -----> li 
// DOM树
#root ---> div ---> p
             |
               ---> li
```

此时`DOM节点` `p`的兄弟节点为`li`，而`Fiber节点` `p`对应的兄弟`DOM节点`为：

```js
In Dom: DOM(p) <-sibling-> DOM(li)
In Fiber: fiber(p).sibling.child = fiber(li)
```

即`fiber p`的`兄弟fiber` `Item`的`子fiber` `li`。在真实`DOM`中为相邻的两个元素，在实际的`fiber`树中其实是跨越层级的。所以要尽量减少此类操作（在`App/ Class Component`前插入`HostComponent`导致触发`getHostSibling()`）以优化性能。

#### 更新：Update effect

当`Fiber节点`含有`Update effectTag`，意味着该`Fiber节点`需要更新。调用的方法为`commitWork`，他会根据`Fiber.tag`分别处理。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1441)看到`commitWork`源码

这里我们主要关注`FunctionComponent`和`HostComponent`。

##### 调用useLayoutEffect的销毁函数：FunctionComponent mutation

当`fiber.tag`为`FunctionComponent`，会调用`commitHookEffectListUnmount`。该方法会遍历`effectList`，执行所有`useLayoutEffect hook`的销毁函数。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L314)看到`commitHookEffectListUnmount`源码

所谓“销毁函数”，见如下例子：

```js
useLayoutEffect(() => {
  // ...一些副作用逻辑

  return () => {
    // ...这就是销毁函数
  }
})
```

##### 渲染HostComponent上对应的更新内容：HostComponent mutation

当`fiber.tag`为`HostComponent`，会调用`commitUpdate`。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-dom/src/client/ReactDOMHostConfig.js#L423)看到`commitUpdate`源码

最终会在[`updateDOMProperties`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-dom/src/client/ReactDOMComponent.js#L378)中将[`render阶段 completeWork`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L229)中为`Fiber节点`赋值的`updateQueue`对应的内容渲染在页面上。

```js
function commitUpdate(domElement, updatePayload, type, oldProps, newProps, internalInstanceHandle) {
    // Update the props handle so that we know which props are the ones with
    // with current event handlers.
    updateFiberProps(domElement, newProps); // Apply the diff to the DOM node.

    updateProperties(domElement, updatePayload, type, oldProps, newProps);
}
```

```js
for (let i = 0; i < updatePayload.length; i += 2) {
  const propKey = updatePayload[i];
  const propValue = updatePayload[i + 1];

  // 处理 style
  if (propKey === STYLE) {
    setValueForStyles(domElement, propValue);
  // 处理 DANGEROUSLY_SET_INNER_HTML
  } else if (propKey === DANGEROUSLY_SET_INNER_HTML) {
    setInnerHTML(domElement, propValue);
  // 处理 children
  } else if (propKey === CHILDREN) {
    setTextContent(domElement, propValue);
  } else {
  // 处理剩余 props
    setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
  }
}
```

#### 删除：Deletion effect

当`Fiber节点`含有`Deletion effectTag`，意味着该`Fiber节点`对应的`DOM节点`需要从页面中删除。调用的方法为`commitDeletion`。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L1421)看到`commitDeletion`源码

该方法会执行如下操作：

1. 递归调用`Fiber节点`及其子孙`Fiber节点`中`fiber.tag`为`ClassComponent`的[`componentWillUnmount`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L920)生命周期钩子，从页面移除`Fiber节点`对应`DOM节点`；
2. 解绑`ref`；
3. 调度被解绑的组件上的`useEffect`的**销毁函数**。

## ⭐commit阶段三：layout

> 该阶段之所以称为`layout`，因为**该阶段的代码都是在`DOM`渲染完成（`mutation阶段`完成）后执行的。**该阶段触发的生命周期钩子和`hook`**可以直接访问到已经改变后的`DOM`，即该阶段是可以参与`DOM layout`的阶段。**（注意，正式的`DOM`变更发生在`mutation`阶段，而标志着正式进入`layout`阶段的标识是`root.current`切换到`workInProgress`上）

与前两个阶段类似，`layout阶段`也是遍历`effectList`，执行函数。具体执行的函数是`commitLayoutEffects`。

```js
root.current = finishedWork;

nextEffect = firstEffect;
do {
  try {
    commitLayoutEffects(root, lanes);
  } catch (error) {
    invariant(nextEffect !== null, "Should be working on an effect.");
    captureCommitPhaseError(nextEffect, error);
    nextEffect = nextEffect.nextEffect;
  }
} while (nextEffect !== null);

nextEffect = null;
```

### 执行layout阶段副作用的关键：commitLayoutEffects

> 你可以在[这里 ](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2302)看到`commitLayoutEffects`源码

```js
function commitLayoutEffects(root: FiberRoot, committedLanes: Lanes) {
  while (nextEffect !== null) {
    const effectTag = nextEffect.effectTag;

    // 调用生命周期钩子和hook
    if (effectTag & (Update | Callback)) {
      const current = nextEffect.alternate;
      commitLayoutEffectOnFiber(root, current, nextEffect, committedLanes);
    }

    // 赋值ref
    if (effectTag & Ref) {
      commitAttachRef(nextEffect);
    }

    nextEffect = nextEffect.nextEffect;
  }
}
```

`commitLayoutEffects`一共做了两件事：

1. commitLayoutEffectOnFiber（调用`生命周期钩子`和`hook`相关操作）
2. commitAttachRef（赋值 ref）

#### 执行layout相关的生命周期或钩子函数的关键：commitLayoutEffectOnFiber

`commitLayoutEffectOnFiber`方法会根据`fiber.tag`对不同类型的节点分别处理。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L459)看到`commitLayoutEffectOnFiber`源码（`commitLayoutEffectOnFiber`为别名，方法原名为`commitLifeCycles`）

- 对于`ClassComponent`，他会通过`current === null?`区分是`mount`还是`update`，调用[`componentDidMount`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L538)或[`componentDidUpdate`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L592)。触发`状态更新`的`this.setState`如果赋值了第二个参数`回调函数`，也会在此时调用。

```js
this.setState({ xxx: 1 }, () => {
  console.log("i am update~"); //会在commitLayoutEffectOnFiber这个阶段被调用
});
```

- 对于`FunctionComponent`及相关类型，他会调用`useLayoutEffect hook`的`回调函数`，调度`useEffect`的`销毁`与`回调`函数

> 相关类型指特殊处理后的`FunctionComponent`，比如`ForwardRef`、`React.memo`包裹的`FunctionComponent`

```js
switch (finishedWork.tag) {
        // 以下都是FunctionComponent及相关类型
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent:
    case Block: {
        // 执行useLayoutEffect的回调函数
        commitHookEffectListMount(HookLayout | HookHasEffect, finishedWork);
        // 调度useEffect的销毁函数与回调函数（并非执行！！！执行要等到commit完成之后）
        schedulePassiveEffects(finishedWork);
        return;
    }
```

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCommitWork.old.js#L465-L491)看到这段代码

在上一节介绍[Update effect](https://react.iamkasong.com/renderer/mutation.html#update-effect)时介绍过，`mutation阶段`会执行`useLayoutEffect hook`的`销毁函数`。结合这里我们可以发现，`useLayoutEffect hook`从上一次更新的`销毁函数`调用到本次更新的`回调函数`调用是同步执行的。而`useEffect`则需要先调度，**在`Layout阶段`完成后再异步执行。**这就是`useLayoutEffect`与`useEffect`的区别。

- 对于`HostRoot`，即`rootFiber`，如果赋值了第三个参数`回调函数`，也会在此时调用。

```js
ReactDOM.render(<App />, document.querySelector("#root"), function() {
  console.log("i am mount~");
});
```

#### commitAttachRef

`commitLayoutEffects`会做的第二件事是`commitAttachRef`。

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L823)看到`commitAttachRef`源码

```js
function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    const instance = finishedWork.stateNode;

    // 获取DOM实例
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent:
        instanceToUse = getPublicInstance(instance);
        break;
      default:
        instanceToUse = instance;
    }

    if (typeof ref === "function") {
      // 如果ref是函数形式，调用回调函数
      ref(instanceToUse);
    } else {
      // 如果ref是ref实例形式，赋值ref.current
      ref.current = instanceToUse;
    }
  }
}
```

代码逻辑很简单：获取`DOM`实例，更新`ref`。至此，

#### 从mutation → layour的转折点：current Fiber树切换

我们关注下这行代码：

```js
root.current = finishedWork;
```

> 你可以在[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L2022)看到这行代码

在[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html#什么是-双缓存)我们介绍过，`workInProgress Fiber树`在`commit阶段`完成渲染后会变为`current Fiber树`。**这行代码的作用就是切换`fiberRootNode`指向的`current Fiber树`。**

那么这行代码为什么在在`mutation阶段`结束后，`layout阶段`开始前呢？

我们知道`componentWillUnmount`会在`mutation阶段`执行。此时`current Fiber树`还指向前一次更新的`Fiber树`，在生命周期钩子内获取的`DOM`还是更新前的。`componentDidMount`和`componentDidUpdate`会在`layout阶段`执行。此时`current Fiber树`已经指向更新后的`Fiber树`，在生命周期钩子内获取的`DOM`就是更新后的。

## 钩子函数和commit阶段的关系

![image-20220202112809451](https://github.com/NoAlligator/pico/blob/main/img/202203271805120.png)

# Diff算法

> 一个`DOM节点`在某一时刻最多会有4个节点和他相关。
>
> 1. `current Fiber`。如果该`DOM节点`已在页面中，`current Fiber`代表该`DOM节点`对应的`Fiber节点`。
> 2. `workInProgress Fiber`。如果该`DOM节点`将在本次更新中渲染到页面中，`workInProgress Fiber`代表该`DOM节点`对应的`Fiber节点`。
> 3. `DOM节点`本身。
> 4. `JSX对象`。即`ClassComponent`的`render`方法的返回结果，或`FunctionComponent`的调用结果。`JSX对象`中包含描述`DOM节点`的信息。
>
> `Diff算法`的本质是对比1和4，生成2。

## Diff的瓶颈

即使在最前沿的算法中，将前后两棵树完全比对的算法的复杂程度为 [O(n^3 )](https://www.zhihu.com/question/66851503)，其中`n`是树中元素的数量。如果在`React`中使用了该算法，那么展示1000个元素所需要执行的计算量将在十亿的量级范围。这个开销实在是太过高昂。为了降低算法复杂度，`React`的`diff`会预设三个限制：

1. 只对同级元素进行`Diff`。如果一个`DOM节点`在前后两次更新中跨越了层级，那么`React`不会尝试复用他。
2. 两个不同类型的(`type`) 元素会产生出不同的树。如果元素由`div`变为`p`，React会销毁`div`及其子孙节点，并新建`p`及其子孙节点。
3. 开发者可以通过 `key prop`来暗示哪些子元素在不同的渲染下能保持稳定。考虑如下案例：

```jsx
// 更新前
<div>
  <p key="ka">ka</p>
  <h3 key="song">song</h3>
</div>

// 更新后
<div>
  <h3 key="song">song</h3>
  <p key="ka">ka</p>
</div>
```

如果没有`key`，`React`会认为`div`的第一个子节点由`p`变为`h3`，第二个子节点由`h3`变为`p`。这符合限制2的设定，会销毁并新建。但是当我们用`key`指明了节点前后对应关系后，`React`知道`key === "ka"`的`p`在更新后还存在，所以`DOM节点`可以复用，只是需要交换下顺序。这就是`React`为了应对算法性能瓶颈做出的三条限制。

## Diff是如何实现的

Diff的核心：`reconcileChildFibers`

我们从`Diff`的入口函数`reconcileChildFibers`出发，该函数会根据`newChild`（即`JSX对象`）类型调用不同的处理函数。

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L1280)看到`reconcileChildFibers`的源码。

```js
// 根据newChild类型选择不同diff函数处理
function reconcileChildFibers(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  newChild: any,
): Fiber | null {

  const isObject = typeof newChild === 'object' && newChild !== null;

  if (isObject) {
    // object类型，可能是 REACT_ELEMENT_TYPE 或 REACT_PORTAL_TYPE
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE:
        // 调用 reconcileSingleElement 处理
      // // ...省略其他case
    }
  }

  if (typeof newChild === 'string' || typeof newChild === 'number') {
    // 调用 reconcileSingleTextNode 处理
    // ...省略
  }

  if (isArray(newChild)) {
    // 调用 reconcileChildrenArray 处理
    // ...省略
  }

  // 一些其他情况调用处理函数
  // ...省略

  // 以上都没有命中，删除节点
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```

我们可以从同级的节点数量将Diff分为两类：

1. 当`newChild`类型为`object`、`number`、`string`，代表同级只有一个节点
2. 当`newChild`类型为`Array`，同级有多个节点

# 单点Diff

单点Diff的核心：`reconcileSingleElement`

对于单个节点，我们以类型`object`为例，会进入`reconcileSingleElement`

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L1141)看到`reconcileSingleElement`源码

```javascript
const isObject = typeof newChild === 'object' && newChild !== null;

if (isObject) {
    // 对象类型，可能是 REACT_ELEMENT_TYPE 或 REACT_PORTAL_TYPE
    switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
            // 调用 reconcileSingleElement 处理
            // ...其他case
    }
}
```

这个函数会做如下事情：

![diff](https://react.iamkasong.com/img/diff.png)

让我们看看第二步**判断DOM节点是否可以复用**是如何实现的。

```javascript
function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
  
  // 首先判断是否存在对应DOM节点
  while (child !== null) {
    // 上一次更新存在DOM节点，接下来判断是否可复用

    // 首先比较key是否相同
    if (child.key === key) {

      // key相同，接下来比较type是否相同

      switch (child.tag) {
        // ...省略case
        
        default: {
          if (child.elementType === element.type) {
            // type相同则表示可以复用
            // 返回复用的fiber
            return existing;
          }
          
          // type不同则跳出switch
          break;
        }
      }
      // 代码执行到这里代表：key相同但是type不同
      // 将该fiber及其兄弟fiber标记为删除
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // key不同，将该fiber标记为删除
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }

  // 创建新Fiber，并返回 ...省略
}
```

从代码可以看出，React通过先判断`key`是否相同，如果`key`相同则判断`type`是否相同，只有都相同时一个`DOM节点`才能复用。

这里有个细节需要关注下：

- 当`child !== null`且`key相同`且`type不同`时执行`deleteRemainingChildren`将`child`及其兄弟`fiber`都标记删除。
- 当`child !== null`且`key不同`时仅将`child`标记删除。

考虑如下例子：

当前页面有3个`li`，我们要全部删除，再插入一个`p`。

```js
// 当前页面显示的
ul > li * 3

// 这次需要更新的
ul > p
```

由于本次更新时只有一个`p`，属于单一节点的`Diff`，会走上面介绍的代码逻辑。

在`reconcileSingleElement`中遍历之前的3个`fiber`（对应的`DOM`为3个`li`），寻找本次更新的`p`是否可以复用之前的3个`fiber`中某个的`DOM`。

当`key相同`且`type不同`时，代表我们已经找到本次更新的`p`对应的上次的`fiber`，但是`p`与`li` `type`不同，不能复用。既然唯一的可能性已经不能复用，则剩下的`fiber`都没有机会了，所以都需要标记删除。

当`key不同`时只代表遍历到的该`fiber`不能被`p`复用，后面还有兄弟`fiber`还没有遍历到。所以仅仅标记该`fiber`删除。

# 多点Diff

```jsx
function List () {
  return (
    <ul>
      <li key="0">0</li>
      <li key="1">1</li>
      <li key="2">2</li>
      <li key="3">3</li>
    </ul>
  )
}
```

他的返回值`JSX对象`的`children`属性不是单一节点，而是包含四个对象的数组

```js
{
  $$typeof: Symbol(react.element),
  key: null,
  props: {
    children: [
      {$$typeof: Symbol(react.element), type: "li", key: "0", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "1", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "2", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "3", ref: null, props: {…}, …}
    ]
  },
  ref: null,
  type: "ul"
}
```

这种情况下，`reconcileChildFibers`的`newChild`参数类型为`Array`，在`reconcileChildFibers`函数内部对应如下情况：

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L1352)看到这段源码逻辑

```js
  if (isArray(newChild)) {
    // 调用 reconcileChildrenArray 处理
    // ...省略
  }
```

我们以**之前**代表更新前的`JSX对象`，**之后**代表更新后的`JSX对象`

## [#](https://react.iamkasong.com/diff/multi.html#情况1-节点更新)情况1：节点更新

```jsx
// 之前
<ul>
  <li key="0" className="before">0<li>
  <li key="1">1<li>
</ul>

// 之后 情况1 —— 节点属性变化
<ul>
  <li key="0" className="after">0<li>
  <li key="1">1<li>
</ul>

// 之后 情况2 —— 节点类型更新
<ul>
  <div key="0">0</div>
  <li key="1">1<li>
</ul>
```

## [#](https://react.iamkasong.com/diff/multi.html#情况2-节点新增或减少)情况2：节点新增或减少

```jsx
// 之前
<ul>
  <li key="0">0<li>
  <li key="1">1<li>
</ul>

// 之后 情况1 —— 新增节点
<ul>
  <li key="0">0<li>
  <li key="1">1<li>
  <li key="2">2<li>
</ul>

// 之后 情况2 —— 删除节点
<ul>
  <li key="1">1<li>
</ul>
```

## [#](https://react.iamkasong.com/diff/multi.html#情况3-节点位置变化)情况3：节点位置变化

```jsx
// 之前
<ul>
  <li key="0">0<li>
  <li key="1">1<li>
</ul>

// 之后
<ul>
  <li key="1">1<li>
  <li key="0">0<li>
</ul>
```

同级多个节点的`Diff`，一定属于以上三种情况中的一种或多种。

## 多点Diff思路

`React团队`发现，在日常开发中，相较于`新增`和`删除`，`更新`组件发生的频率更高。所以`Diff`会优先判断当前节点是否属于`更新`。

> 注：在我们做数组相关的算法题时，经常使用**双指针**从数组头和尾同时遍历以提高效率，但是这里却不行。虽然本次更新的`JSX对象` `newChildren`为数组形式，但是和`newChildren`中每个组件进行比较的是`current fiber`，同级的`Fiber节点`是由`sibling`指针链接形成的单链表，即不支持双指针遍历。即 `newChildren[0]`与`fiber`比较，`newChildren[1]`与`fiber.sibling`比较。
>
> 所以无法使用**双指针**优化。

基于以上原因，`Diff算法`的整体逻辑会经历两轮遍历：

**第一轮遍历：处理`更新`的节点。**

**第二轮遍历：处理剩下的不属于`更新`的节点（新增、减少、位移）。**

## 多点Diff的第一轮遍历

第一轮遍历步骤如下：

1. `let i = 0`，遍历`newChildren`，将`newChildren[i]`与`oldFiber`比较，判断`DOM节点`是否可复用。
2. 如果可复用，`i++`，继续比较`newChildren[i]`与`oldFiber.sibling`，可以复用则继续遍历。
3. 如果不可复用，分两种情况：
   1. `key`不同导致不可复用，立即跳出整个遍历，**第一轮遍历结束。**
   2. `key`相同`type`不同导致不可复用，会将`oldFiber`标记为`DELETION`，并继续遍历。
4. 如果`newChildren`遍历完（即`i === newChildren.length - 1`）或者`oldFiber`遍历完（即`oldFiber.sibling === null`），跳出遍历，**第一轮遍历结束。**

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L818)看到这轮遍历的源码

当遍历结束后，会有两种结果：

### [#](https://react.iamkasong.com/diff/multi.html#步骤3跳出的遍历)步骤3跳出的遍历

此时`newChildren`没有遍历完，`oldFiber`也没有遍历完。

举个例子，考虑如下代码：

```jsx
// 之前
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
            
// 之后
<li key="0">0</li>
<li key="2">1</li>
<li key="1">2</li>
```

第一个节点可复用，遍历到`key === 2`的节点发现`key`改变，不可复用，跳出遍历，等待第二轮遍历处理。此时`oldFiber`剩下`key === 1`、`key === 2`未遍历，`newChildren`剩下`key === 2`、`key === 1`未遍历。

### [#](https://react.iamkasong.com/diff/multi.html#步骤4跳出的遍历)步骤4跳出的遍历

可能`newChildren`遍历完，或`oldFiber`遍历完，或他们同时遍历完。

举个例子，考虑如下代码：

```jsx
// 之前
<li key="0" className="a">0</li>
<li key="1" className="b">1</li>
            
// 之后 情况1 —— newChildren与oldFiber都遍历完
<li key="0" className="aa">0</li>
<li key="1" className="bb">1</li>
            
// 之后 情况2 —— newChildren没遍历完，oldFiber遍历完
// newChildren剩下 key==="2" 未遍历
<li key="0" className="aa">0</li>
<li key="1" className="bb">1</li>
<li key="2" className="cc">2</li>
            
// 之后 情况3 —— newChildren遍历完，oldFiber没遍历完
// oldFiber剩下 key==="1" 未遍历
<li key="0" className="aa">0</li>
```

带着第一轮遍历的结果，开始第二轮遍历。

## 第二轮遍历

对于第一轮遍历的结果，我们分别讨论：

### `newChildren`与`oldFiber`同时遍历完

那就是最理想的情况：只需在第一轮遍历进行组件[`更新`](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L825)。此时`Diff`结束。

### `newChildren`没遍历完，`oldFiber`遍历完

已有的`DOM节点`都复用了，这时还有**新加入的节点**，意味着本次更新有新节点插入，我们只需要遍历剩下的`newChildren`为生成的`workInProgress fiber`依次标记`Placement`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L869)看到这段源码逻辑

### `newChildren`遍历完，`oldFiber`没遍历完

意味着本次更新比之前的节点数量少，**有节点被删除**了。所以需要遍历剩下的`oldFiber`，依次标记`Deletion`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L863)看到这段源码逻辑

### `newChildren`与`oldFiber`都没遍历完

这意味着有节点在这次更新中改变了位置。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L893)看到这段源码逻辑

### 处理移动的节点

由于有节点改变了位置，所以不能再用位置索引`i`对比前后的节点，那么如何才能将同一个节点在两次更新中对应上呢？我们需要使用`key`。为了快速的找到`key`对应的`oldFiber`，我们将所有还未处理的`oldFiber`存入以`key`为`key`，`oldFiber`为`value`的`Map`中。

```javascript
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
```

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L890)看到这段源码逻辑

接下来遍历剩余的`newChildren`，通过`newChildren[i].key`就能在`existingChildren`中找到`key`相同的`oldFiber`。

### 标记节点是否移动

标记节点是否移动的方式是通过`Map` + 相对位置，遍历新的节点位置，如果查询到新的节点位置在原先节点位置中靠前了，那么需要后移，否则不移动节点。所以整个过程中只有针对旧的节点的`appendChild`操作，没有`insertBefore`操作。

# 状态更新全流程

## 概览

### 关键点

#### render

`render阶段`开始于`performSyncWorkOnRoot`或`performConcurrentWorkOnRoot`方法的调用。这取决于本次更新是同步更新还是异步更新。

#### commit

`commit阶段`开始于`commitRoot`方法的调用。其中`rootFiber`会作为传参。`render阶段`完成后会进入`commit阶段`。

### 状态更新机制：Update对象

在`React`中，有如下方法可以触发状态更新（排除`SSR`相关）：

- ReactDOM.render
- this.setState
- this.forceUpdate
- useState
- useReducer

调用的场景各不相同，但是都接入了同一套状态更新机制：每次`状态更新`都会创建一个保存**更新状态相关内容**的对象，我们叫他`Update`。在`render阶段`的`beginWork`中会根据`Update`计算新的`state`。

### 节点更新的传递

触发状态更新的fiber节点上已经包含了update对象，但是render阶段是从rootFiber向下开始遍历的，在源码中是通过markUpdateLaneFromFiberToRoot方法来从触发状态更新的fiber节点一直向上遍历到rootFiber并返回，再从rootFiber开始进入render阶段。由于不同更新优先级不尽相同，所以过程中还会更新遍历到的`fiber`的优先级。

### 调度更新

现在我们拥有一个`rootFiber`，该`rootFiber`对应的`Fiber树`中某个`Fiber节点`包含一个`Update`。接下来通知`Scheduler`根据**更新**的优先级，决定以**同步**还是**异步**的方式调度本次更新。这里调用的方法是`ensureRootIsScheduled`。以下是`ensureRootIsScheduled`最核心的一段代码：

```js
if (newCallbackPriority === SyncLanePriority) {
  // 任务已经过期，需要同步执行render阶段
  newCallbackNode = scheduleSyncCallback(
    performSyncWorkOnRoot.bind(null, root)
  );
} else {
  // 根据任务优先级异步执行render阶段
  var schedulerPriorityLevel = lanePriorityToSchedulerPriority(
    newCallbackPriority
  );
  newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root)
  );
}
```

> 你可以从[这里](https://github.com/facebook/react/blob/b6df4417c79c11cfb44f965fab55b573882b1d54/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L602)看到`ensureRootIsScheduled`的源码

其中，`scheduleCallback`和`scheduleSyncCallback`会调用`Scheduler`提供的调度方法根据`优先级`调度回调函数执行。

可以看到，这里调度的回调函数为：

```js
performSyncWorkOnRoot.bind(null, root);
performConcurrentWorkOnRoot.bind(null, root);
```

即`render阶段`的入口函数。

至此，`状态更新`就和我们所熟知的`render阶段`连接上了。

### 总结

```sh
触发状态更新（根据场景调用不同方法）
    |
    |
    v

创建Update对象（接下来三节详解）
    |
    |
    v

从fiber到root（`markUpdateLaneFromFiberToRoot`）
    |
    |
    v

调度更新（`ensureRootIsScheduled`）
    |
    |
    v

render阶段（`performSyncWorkOnRoot` 或 `performConcurrentWorkOnRoot`）
    |
    |
    v

commit阶段（`commitRoot`）
```

## 更新心智模型

### 同步更新

在`React`中，所有通过`ReactDOM.render`创建的应用（其他创建应用的方式参考[ReactDOM.render一节](https://react.iamkasong.com/state/reactdom.html#react的其他入口函数)）都是通过类似的方式`更新状态`。即没有`优先级`概念，`高优更新`（红色节点）需要排在其他`更新`后面执行。

![流程1](https://react.iamkasong.com/img/git1.png)

### 并发更新

在`React`中，通过`ReactDOM.createBlockingRoot`和`ReactDOM.createRoot`创建的应用会采用`并发`的方式`更新状态`。`高优更新`（红色节点）中断正在进行中的`低优更新`（蓝色节点），先完成`render - commit流程`。待`高优更新`完成后，`低优更新`基于`高优更新`的结果`重新更新`。

![流程3](https://react.iamkasong.com/img/git3.png)

![优先级如何决定更新的顺序](https://react.iamkasong.com/img/update-process.png)

## Update对象

### update分类

我们将可以触发更新的方法所隶属的组件分类：



- ReactDOM.render —— HostRoot
- this.setState —— ClassComponent
- this.forceUpdate —— ClassComponent



- useState —— FunctionComponent
- useReducer —— FunctionComponent

可以看到，一共三种组件（`HostRoot` | `ClassComponent` | `FunctionComponent`）可以触发更新。由于不同类型组件工作方式不同，所以**存在两种不同结构的`Update`，其中`ClassComponent`与`HostRoot`共用一套`Update`结构，`FunctionComponent`单独使用一种`Update`结构。**虽然他们的结构不同，但是他们工作机制与工作流程大体相同。

### update对象结构

`ClassComponent`与`HostRoot`（即`rootFiber.tag`对应类型）共用同一种`Update结构`。

对应的结构如下：

```js
const update: Update<*> = {
    eventTime,
    lane,
    suspenseConfig,
    tag: UpdateState,
    payload: null,
    callback: null,
    next: null,
};
```

> `Update`由`createUpdate`方法返回，你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.old.js#L189)看到`createUpdate`的源码。

```js
function createUpdate(eventTime, lane) {
    var update = {
        eventTime: eventTime,
        lane: lane,
        tag: UpdateState,
        payload: null,
        callback: null,
        next: null
    };
    return update;
}
```

字段意义：

- eventTime：任务时间，通过`performance.now()`获取的毫秒数。由于该字段在未来会重构，当前我们不需要理解他。
- lane：优先级相关字段。当前还不需要掌握他，只需要知道不同`Update`优先级可能是不同的。你可以将`lane`类比`心智模型`中`需求的紧急程度`。

- suspenseConfig：`Suspense`相关。
- tag：**更新的类型，包括`UpdateState` | `ReplaceState` | `ForceUpdate` | `CaptureUpdate`。**
- payload：**更新挂载的数据，不同类型组件挂载的数据不同。对于`ClassComponent`，`payload`为`this.setState`的第一个传参。对于`HostRoot`，`payload`为`ReactDOM.render`的第一个传参。**
- callback：**更新的回调函数。即在[commit 阶段的 layout 子阶段一节](https://react.iamkasong.com/renderer/layout.html#commitlayouteffectonfiber)中提到的`回调函数`。**
- next：**与其他`Update`连接形成链表。**

### Update链与Fiber链

`Update`存在一个连接其他`Update`形成链表的字段`next`。联系`React`中另一种以链表形式组成的结构`Fiber`，他们之间有什么关联么？

`Fiber节点`组成`Fiber树`，页面中最多同时存在两棵`Fiber树`：

- 代表当前页面状态的`current Fiber树`
- 代表正在`render阶段`的`workInProgress Fiber树`

类似`Fiber节点`组成`Fiber树`，`Fiber节点`上的多个`Update`会组成链表并被包含在`fiber.updateQueue`中。

> 什么情况下一个Fiber节点会存在多个Update？
>
> 你可能疑惑为什么一个`Fiber节点`会存在多个`Update`。这其实是很常见的情况。在这里介绍一种最简单的情况：
>
> ```js
> onClick() {
>   this.setState({
>     a: 1
>   })
> 
>   this.setState({
>     b: 2
>   })
> }
> ```
>
> 在一个`ClassComponent`中触发`this.onClick`方法，方法内部调用了两次`this.setState`。**这会在该`fiber`中产生两个`Update`。**

`Fiber节点`最多同时存在两个`updateQueue`：

- `current fiber`保存的`updateQueue`即`current updateQueue`
- `workInProgress fiber`保存的`updateQueue`即`workInProgress updateQueue`

在`commit阶段`完成页面渲染后，`workInProgress Fiber树`变为`current Fiber树`，`workInProgress Fiber树`内`Fiber节点`的`updateQueue`就变成`current updateQueue`。

### updateQueue

`updateQueue`有三种类型，其中针对`HostComponent`的类型我们在[completeWork一节](https://react.iamkasong.com/process/completeWork.html#update时)介绍过。剩下两种类型和`Update`的两种类型对应。

`ClassComponent`与`HostRoot`使用的`UpdateQueue`结构如下：

```js
const queue: UpdateQueue<State> = {
    baseState: fiber.memoizedState,
    firstBaseUpdate: null,
    lastBaseUpdate: null,
    shared: {
      pending: null,
    },
    effects: null,
  };
```

> `UpdateQueue`由`initializeUpdateQueue`方法返回，你可以从[这里 ](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.new.js#L157)看到`initializeUpdateQueue`的源码

字段说明如下：

- baseState：本次更新前该`Fiber节点`的`state`，`Update`基于该`state`计算更新后的`state`。

- `firstBaseUpdate`与`lastBaseUpdate`：本次更新前该`Fiber节点`已保存的`Update`。以链表形式存在，链表头为`firstBaseUpdate`，链表尾为`lastBaseUpdate`。之所以在更新产生前该`Fiber节点`内就存在`Update`，是由于某些`Update`优先级较低所以在上次`render阶段`由`Update`计算`state`时被跳过。。

- `shared.pending`：触发更新时，产生的`Update`会保存在`shared.pending`中形成单向环状链表。当由`Update`计算`state`时这个环会被剪开并连接在`lastBaseUpdate`后面。

- effects：数组。保存`update.callback !== null`的`Update`。

### 例子

`updateQueue`相关代码逻辑涉及到大量链表操作，比较难懂。在此我们举例对`updateQueue`的工作流程讲解下。

假设有一个`fiber`刚经历`commit阶段`完成渲染。

该`fiber`上有两个由于优先级过低所以在上次的`render阶段`并没有处理的`Update`。他们会成为下次更新的`baseUpdate`。

我们称其为`u1`和`u2`，其中`u1.next === u2`。

```js
fiber.updateQueue.firstBaseUpdate === u1;
fiber.updateQueue.lastBaseUpdate === u2;
u1.next === u2;
```

我们用`-->`表示链表的指向：

```js
fiber.updateQueue.baseUpdate: u1 --> u2
```

现在我们在`fiber`上触发两次状态更新，这会先后产生两个新的`Update`，我们称为`u3`和`u4`。

每个 `update` 都会通过 `enqueueUpdate` 方法插入到 `updateQueue` 队列上

当插入`u3`后：

```js
fiber.updateQueue.shared.pending === u3;
u3.next === u3;
```

`shared.pending`的环状链表，用图表示为：

```js
fiber.updateQueue.shared.pending:   u3 ─────┐ 
                                     ^      |                                    
                                     └──────┘
```

接着插入`u4`之后：

```js
fiber.updateQueue.shared.pending === u4;
u4.next === u3;
u3.next === u4;
```

`shared.pending`是环状链表，用图表示为：

```js
fiber.updateQueue.shared.pending:   u4 ──> u3
                                     ^      |                                    
                                     └──────┘
```

`shared.pending` 会保证始终指向最后一个插入的`update`，你可以在[这里 ](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.new.js#L208)看到`enqueueUpdate`的源码

更新调度完成后进入`render阶段`。

此时`shared.pending`的环被剪开并连接在`updateQueue.lastBaseUpdate`后面：

```js
fiber.updateQueue.baseUpdate: u1 --> u2 --> u3 --> u4
```

接下来遍历`updateQueue.baseUpdate`链表，以`fiber.updateQueue.baseState`为`初始state`，依次与遍历到的每个`Update`计算并产生新的`state`（该操作类比`Array.prototype.reduce`）。

在遍历时如果有优先级低的`Update`会被跳过。

当遍历完成后获得的`state`，就是该`Fiber节点`在本次更新的`state`（源码中叫做`memoizedState`）。

> `render阶段`的`Update操作`由`processUpdateQueue`完成，你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.new.js#L405)看到`processUpdateQueue`的源码。

`state`的变化在`render阶段`产生与上次更新不同的`JSX`对象，通过`Diff算法`产生`effectTag`，在`commit阶段`渲染在页面上。

渲染完成后`workInProgress Fiber树`变为`current Fiber树`，整个更新流程结束。

![image-20220208205709411](https://github.com/NoAlligator/pico/blob/main/img/202203271805121.png)

![image-20220208205653396](https://github.com/NoAlligator/pico/blob/main/img/202203271805122.png)

### 优先级

在[新的React结构一节](https://react.iamkasong.com/preparation/newConstructure.html)讲到，`React`通过`Scheduler`调度任务。具体到代码，每当需要调度任务时，`React`会调用`Scheduler`提供的方法`runWithPriority`。该方法接收一个`优先级`常量与一个`回调函数`作为参数。`回调函数`会以`优先级`高低为顺序排列在一个`定时器`中并在合适的时间触发。对于更新来讲，传递的`回调函数`一般为[状态更新流程概览一节](https://react.iamkasong.com/state/prepare.html#render阶段的开始)讲到的`render阶段的入口函数`。

> 你可以在[==unstable_runWithPriority== 这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/scheduler/src/Scheduler.js#L217)看到`runWithPriority`方法的定义。在[这里 ](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/scheduler/src/SchedulerPriorities.js)看到`Scheduler`对优先级常量的定义。

### 例子

优先级最终会反映到`update.lane`变量上。当前我们只需要知道这个变量能够区分`Update`的优先级。接下来我们通过一个例子结合上一节介绍的`Update`相关字段讲解优先级如何决定更新的顺序。

> 该例子来自[React Core Team Andrew向网友讲解Update工作流程的推文](https://twitter.com/acdlite/status/978412930973687808)

![优先级如何决定更新的顺序](https://react.iamkasong.com/img/update-process.png)

在这个例子中，有两个`Update`。我们将“关闭黑夜模式”产生的`Update`称为`u1`，输入字母“I”产生的`Update`称为`u2`。

其中`u1`先触发并进入`render阶段`。其优先级较低，执行时间较长。此时：

```js
fiber.updateQueue = {
  baseState: {
    blackTheme: true,
    text: 'H'
  },
  firstBaseUpdate: null,
  lastBaseUpdate: null
  shared: {
    pending: u1
  },
  effects: null
};
```

在`u1`完成`render阶段`前用户通过键盘输入字母“I”，产生了`u2`。`u2`属于**受控的用户输入**，优先级高于`u1`，于是中断`u1`产生的`render阶段`。

此时：

```js
fiber.updateQueue.shared.pending === u2 ----> u1
                                     ^        |
                                     |________|
// 即
u2.next === u1;
u1.next === u2;
```

其中`u2`优先级高于`u1`。

接下来进入`u2`产生的`render阶段`。

在`processUpdateQueue`方法中，`shared.pending`环状链表会被剪开并拼接在`baseUpdate`后面。

需要明确一点，`shared.pending`指向最后一个`pending`的`update`，所以实际执行时`update`的顺序为：

```js
u1 -- u2
```

接下来遍历`baseUpdate`，处理优先级合适的`Update`（这一次处理的是更高优的`u2`）。

由于`u2`不是`baseUpdate`中的第一个`update`，在其之前的`u1`由于优先级不够被跳过。

`update`之间可能有依赖关系，所以被跳过的`update`及其后面所有`update`会成为下次更新的`baseUpdate`。（即`u1 -- u2`）。

最终`u2`完成`render - commit阶段`。

此时：

```js
fiber.updateQueue = {
  baseState: {
    blackTheme: true,
    text: 'HI'
  },
  firstBaseUpdate: u1,
  lastBaseUpdate: u2
  shared: {
    pending: null
  },
  effects: null
};
```

在`commit`阶段结尾会再调度一次更新。在该次更新中会基于`baseState`中`firstBaseUpdate`保存的`u1`，开启一次新的`render阶段`。

最终两次`Update`都完成后的结果如下：

```js
fiber.updateQueue = {
  baseState: {
    blackTheme: false,
    text: 'HI'
  },
  firstBaseUpdate: null,
  lastBaseUpdate: null
  shared: {
    pending: null
  },
  effects: null
};
```

我们可以看见，`u2`对应的更新执行了两次，相应的`render阶段`的生命周期勾子`componentWillXXX`也会触发两次。这也是为什么这些勾子会被标记为`unsafe_`。

#### 如何保证状态正确

现在我们基本掌握了`updateQueue`的工作流程。还有两个疑问：

- `render阶段`可能被中断。如何保证`updateQueue`中保存的`Update`不丢失？
- 有时候当前`状态`需要依赖前一个`状态`。如何在支持跳过`低优先级状态`的同时保证**状态依赖的连续性**？

我们分别讲解下。

#### 如何保证`Update`不丢失

在[上一节例子](https://react.iamkasong.com/state/update.html#例子)中我们讲到，在`render阶段`，`shared.pending`的环被剪开并连接在`updateQueue.lastBaseUpdate`后面。实际上`shared.pending`会被同时连接在`workInProgress updateQueue.lastBaseUpdate`与`current updateQueue.lastBaseUpdate`后面。

> 具体代码见[这里](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactUpdateQueue.new.js#L424)

当`render阶段`被中断后重新开始时，会基于`current updateQueue`克隆出`workInProgress updateQueue`。由于`current updateQueue.lastBaseUpdate`已经保存了上一次的`Update`，所以不会丢失。

当`commit阶段`完成渲染，由于`workInProgress updateQueue.lastBaseUpdate`中保存了上一次的`Update`，所以 `workInProgress Fiber树`变成`current Fiber树`后也不会造成`Update`丢失。

#### 如何保证状态依赖的连续性

当某个`Update`由于优先级低而被跳过时，保存在`baseUpdate`中的不仅是该`Update`，还包括链表中该`Update`之后的所有`Update`。

考虑如下例子：

```js
baseState: ''
shared.pending: A1 --> B2 --> C1 --> D2
```

其中`字母`代表该`Update`要在页面插入的字母，`数字`代表`优先级`，值越低`优先级`越高。

第一次`render`，`优先级`为1。

```js
baseState: ''
baseUpdate: null
render阶段使用的Update: [A1, C1]
memoizedState: 'AC'
```

其中`B2`由于优先级为2，低于当前优先级，所以他及其后面的所有`Update`会被保存在`baseUpdate`中作为下次更新的`Update`（即`B2 C1 D2`）。

这么做是为了保持`状态`的前后依赖顺序。

第二次`render`，`优先级`为2。

```js
baseState: 'A'
baseUpdate: B2 --> C1 --> D2
render阶段使用的Update: [B2, C1, D2]
memoizedState: 'ABCD'
```

注意这里`baseState`并不是上一次更新的`memoizedState`。这是由于`B2`被跳过了。

即当有`Update`被跳过时，`下次更新的baseState !== 上次更新的memoizedState`。

> 跳过`B2`的逻辑见[这里(opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactUpdateQueue.new.js#L479)

通过以上例子我们可以发现，`React`保证最终的状态一定和用户触发的`交互`一致，但是中间过程`状态`可能由于设备不同而不同。

## ReactDOM.render

### 创建fiber

从[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html#mount时)我们知道，首次执行`ReactDOM.render`会创建`fiberRootNode`和`rootFiber`。其中`fiberRootNode`是整个应用的根节点，`rootFiber`是要渲染组件所在组件树的`根节点`。这一步发生在调用`ReactDOM.render`后进入的`legacyRenderSubtreeIntoContainer`方法中。

```js
// container指ReactDOM.render的第二个参数（即应用挂载的DOM节点）
root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
  container,
  forceHydrate,
);
fiberRoot = root._internalRoot;
```

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-dom/src/client/ReactDOMLegacy.js#L193)看到这一步的代码

`legacyCreateRootFromDOMContainer`方法内部会调用`createFiberRoot`方法完成`fiberRootNode`和`rootFiber`的创建以及关联。并初始化`updateQueue`。

```js
export function createFiberRoot(
  containerInfo: any,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
): FiberRoot {
  // 创建fiberRootNode
  const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);
  
  // 创建rootFiber
  const uninitializedFiber = createHostRootFiber(tag);

  // 连接rootFiber与fiberRootNode
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  // 初始化updateQueue
  initializeUpdateQueue(uninitializedFiber);

  return root;
}
```

根据以上代码，现在我们可以在[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html#mount时)基础上补充上`rootFiber`到`fiberRootNode`的引用。

![fiberRoot](https://react.iamkasong.com/img/fiberroot.png)

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberRoot.new.js#L97)看到这一步的代码

### 创建update

我们已经做好了组件的初始化工作，接下来就等待创建`Update`来开启一次更新。这一步发生在`updateContainer`方法中（只会在mount时调用一次）。

```js
export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): Lane {
  // ...省略与逻辑不相关代码

  // 创建update
  const update = createUpdate(eventTime, lane, suspenseConfig);
  
  // update.payload为需要挂载在根节点的组件
  update.payload = {element};

  // callback为ReactDOM.render的第三个参数 —— 回调函数
  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }

  // 将生成的update加入updateQueue
  enqueueUpdate(current, update);
  // 调度更新
  scheduleUpdateOnFiber(current, lane, eventTime);

  // ...省略与逻辑不相关代码
}
```

> 你可以从[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberReconciler.new.js#L255)看到`updateContainer`的代码

值得注意的是其中`update.payload = {element};`这就是我们在[Update一节](https://react.iamkasong.com/state/update.html#update的结构)介绍的，对于`HostRoot`，`payload`为`ReactDOM.render`的第一个传参。

### 流程概览

至此，`ReactDOM.render`的流程就和我们已知的流程连接上了。整个流程如下：

```sh
创建fiberRootNode、rootFiber、updateQueue（`legacyCreateRootFromDOMContainer`）

    |
    |
    v

创建Update对象（`updateContainer`）

    |
    |
    v

从fiber到root（`markUpdateLaneFromFiberToRoot`）

    |
    |
    v

调度更新（`ensureRootIsScheduled`）

    |
    |
    v

render阶段（`performSyncWorkOnRoot` 或 `performConcurrentWorkOnRoot`）

    |
    |
    v

commit阶段（`commitRoot`）
```

### React的其他入口函数

当前`React`共有三种模式：

- `legacy`，这是当前`React`使用的方式。当前没有计划删除本模式，但是这个模式可能不支持一些新功能。
- `blocking`，开启部分`concurrent`模式特性的中间模式。目前正在实验中。作为迁移到`concurrent`模式的第一个步骤。
- `concurrent`，面向未来的开发模式。我们之前讲的`任务中断/任务优先级`都是针对`concurrent`模式。

你可以从下表看出各种模式对特性的支持：

|                                                              | legacy 模式 | blocking 模式 | concurrent 模式 |
| ------------------------------------------------------------ | ----------- | ------------- | --------------- |
| [String Refs(opens new window)](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#legacy-api-string-refs) | ✅           | 🚫**           | 🚫**             |
| [Legacy Context(opens new window)](https://zh-hans.reactjs.org/docs/legacy-context.html) | ✅           | 🚫**           | 🚫**             |
| [findDOMNode(opens new window)](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage) | ✅           | 🚫**           | 🚫**             |
| [Suspense(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-suspense.html#what-is-suspense-exactly) | ✅           | ✅             | ✅               |
| [SuspenseList(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#suspenselist) | 🚫           | ✅             | ✅               |
| Suspense SSR + Hydration                                     | 🚫           | ✅             | ✅               |
| Progressive Hydration                                        | 🚫           | ✅             | ✅               |
| Selective Hydration                                          | 🚫           | 🚫             | ✅               |
| Cooperative Multitasking                                     | 🚫           | 🚫             | ✅               |
| Automatic batching of multiple setStates                     | 🚫*          | ✅             | ✅               |
| [Priority-based Rendering(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#splitting-high-and-low-priority-state) | 🚫           | 🚫             | ✅               |
| [Interruptible Prerendering(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-intro.html#interruptible-rendering) | 🚫           | 🚫             | ✅               |
| [useTransition(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#transitions) | 🚫           | 🚫             | ✅               |
| [useDeferredValue(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#deferring-a-value) | 🚫           | 🚫             | ✅               |
| [Suspense Reveal "Train"(opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#suspense-reveal-train) | 🚫           | 🚫             | ✅               |

*：`legacy`模式在合成事件中有自动批处理的功能，但仅限于一个浏览器任务。非`React`事件想使用这个功能必须使用 `unstable_batchedUpdates`。在`blocking`模式和`concurrent`模式下，所有的`setState`在默认情况下都是批处理的。

**：会在开发中发出警告。

模式的变化影响整个应用的工作方式，所以无法只针对某个组件开启不同模式。

基于此原因，可以通过不同的`入口函数`开启不同模式：

- `legacy` -- `ReactDOM.render(<App />, rootNode)`
- `blocking` -- `ReactDOM.createBlockingRoot(rootNode).render(<App />)`
- `concurrent` -- `ReactDOM.createRoot(rootNode).render(<App />)`

> 你可以在[这里 (opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#why-so-many-modes)看到`React`团队解释为什么会有这么多模式

虽然不同模式的`入口函数`不同，但是他们仅对`fiber.mode`变量产生影响，对我们在[流程概览](https://react.iamkasong.com/state/reactdom.html#流程概览)中描述的流程并无影响。

## this.setState

### 流程概览

可以看到，`this.setState`内会调用`this.updater.enqueueSetState`方法：

```js
Component.prototype.setState = function (partialState, callback) {
  if (!(typeof partialState === 'object' || typeof partialState === 'function' || partialState == null)) {
    {
      throw Error( "setState(...): takes an object of state variables to update or a function which returns an object of state variables." );
    }
  }
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react/src/ReactBaseClasses.js#L57)看到这段代码

在`enqueueSetState`方法中就是我们熟悉的从`创建update`到`调度update`的流程了。

```js
enqueueSetState(inst, payload, callback) {
  // 通过组件实例获取对应fiber
  const fiber = getInstance(inst);

  const eventTime = requestEventTime();
  const suspenseConfig = requestCurrentSuspenseConfig();

  // 获取优先级
  const lane = requestUpdateLane(fiber, suspenseConfig);

  // 创建update
  const update = createUpdate(eventTime, lane, suspenseConfig);

  update.payload = payload;

  // 赋值回调函数
  if (callback !== undefined && callback !== null) {
    update.callback = callback;
  }

  // 将update插入updateQueue
  enqueueUpdate(fiber, update);
  // 调度update
  scheduleUpdateOnFiber(fiber, lane, eventTime);
}
```

> 你可以在[这里看到`enqueueSetState`代码

这里值得注意的是对于`ClassComponent`，`update.payload`为`this.setState`的第一个传参（即要改变的`state`）。

### this.forceUpdate

在`this.updater`上，除了`enqueueSetState`外，还存在`enqueueForceUpdate`，当我们调用`this.forceUpdate`时会调用他。

可以看到，除了赋值`update.tag = ForceUpdate;`以及没有`payload`外，其他逻辑与`this.setState`一致。

```js
enqueueForceUpdate(inst, callback) {
    const fiber = getInstance(inst);
    const eventTime = requestEventTime();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const lane = requestUpdateLane(fiber, suspenseConfig);

    const update = createUpdate(eventTime, lane, suspenseConfig);

    // 赋值tag为ForceUpdate
    update.tag = ForceUpdate;

    if (callback !== undefined && callback !== null) {
      update.callback = callback;
    }

    enqueueUpdate(fiber, update);
    scheduleUpdateOnFiber(fiber, lane, eventTime);
  },
};
```

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberClassComponent.old.js#L260)看到`enqueueForceUpdate`代码

那么赋值`update.tag = ForceUpdate;`有何作用呢？

在判断`ClassComponent`是否需要更新时有两个条件需要满足：

```js
const shouldUpdate =
      checkHasForceUpdateAfterProcessing() ||
      checkShouldComponentUpdate(
          workInProgress,
          ctor,
          oldProps,
          newProps,
          oldState,
          newState,
          nextContext,
      );
```

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberClassComponent.old.js#L1137)看到这段代码

- checkHasForceUpdateAfterProcessing：内部会判断本次更新的`Update`是否为`ForceUpdate`。即如果本次更新的`Update`中存在`tag`为`ForceUpdate`，则返回`true`。
- checkShouldComponentUpdate：内部会调用`shouldComponentUpdate`方法。以及当该`ClassComponent`为`PureComponent`时会浅比较`state`与`props`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberClassComponent.old.js#L294)看到`checkShouldComponentUpdate`代码

所以，当某次更新含有`tag`为`ForceUpdate`的`Update`，那么当前`ClassComponent`不会受其他`性能优化手段`（`shouldComponentUpdate`|`PureComponent`）影响，一定会更新。

# Hooks 理念

你可以从[这里 (opens new window)](https://zh-hans.reactjs.org/docs/hooks-intro.html#motivation)了解`Hooks`的设计动机。作为一名`框架使用者`，了解`设计动机`对于我们日常开发就足够了。

但是，为了更好的理解`Hooks`的`源码架构`，我们需要转换身份，以`框架开发者`的角度来看待`Hooks`的`设计理念`。

## [#](https://react.iamkasong.com/hooks/prepare.html#从logo聊起)从LOGO聊起

![LOGO](https://react.iamkasong.com/img/logo.png)

`React` `LOGO`的图案是代表`原子`（`atom`）的符号。世间万物由`原子`组成，`原子`的`类型`与`属性`决定了事物的外观与表现。

同样，在`React`中，我们可以将`UI`拆分为很多独立的单元，每个单元被称为`Component`。这些`Component`的`属性`与`类型`决定了`UI`的外观与表现。

讽刺的是，`原子`在希腊语中的意思为`不可分割的`（`indivisible`），但随后科学家在原子中发现了更小的粒子 —— 电子（`electron`）。电子可以很好的解释`原子`是如何工作的。

在`React`中，我们可以说`ClassComponent`是一类`原子`。

但对于`Hooks`来说，与其说是一类`原子`，不如说他是更贴近事物`运行规律`的`电子`。

我们知道，`React`的架构遵循`schedule - render - commit`的运行流程，这个流程是`React`世界最底层的`运行规律`。

`ClassComponent`作为`React`世界的`原子`，他的`生命周期`（`componentWillXXX`/`componentDidXXX`）是为了介入`React`的运行流程而实现的更上层抽象，这么做是为了方便`框架使用者`更容易上手。

相比于`ClassComponent`的更上层抽象，`Hooks`则更贴近`React`内部运行的各种概念（`state` | `context` | `life-cycle`）。

作为使用`React`技术栈的开发者，当我们初次学习`Hooks`时，不管是官方文档还是身边有经验的同事，总会拿`ClassComponent`的生命周期来类比`Hooks API`的执行时机。

这固然是很好的上手方式，但是当我们熟练运用`Hooks`时，就会发现，这两者的概念有很多割裂感，并不是同一抽象层次可以互相替代的概念。

比如：替代`componentWillReceiveProps`的`Hooks`是什么呢？

可能有些同学会回答，是`useEffect`：

```js
  useEffect( () => {
    console.log('something updated');
  }, [props.something])
```

但是`componentWillReceiveProps`是在`render阶段`执行，而`useEffect`是在`commit阶段`完成渲染后异步执行。

> 这篇文章可以帮你更好理解`componentWillReceiveProps`：[深入源码剖析componentWillXXX为什么UNSAFE(opens new window)](https://juejin.im/post/5f05a3e25188252e5c576cdb)

所以，从源码运行规律的角度看待`Hooks`，可能是更好的角度。这也是为什么上文说`Hooks`是`React`世界的`电子`而不是`原子`的原因。

> 以上见解参考自[React Core Team Dan在 React Conf2018的演讲(opens new window)](https://www.youtube.com/watch?v=dpw9EHDh2bM&feature=youtu.be)

## [#](https://react.iamkasong.com/hooks/prepare.html#总结)总结

`Concurrent Mode`是`React`未来的发展方向，而`Hooks`是能够最大限度发挥`Concurrent Mode`潜力的`Component`构建方式。

正如Dan在`React Conf 2018`演讲结尾所说：你可以从`React`的`LOGO`中看到这些围绕着`核心`的`电子飞行轨道`，`Hooks`可能一直就在其中。

# 极简Hooks实现

为了更好理解`Hooks`原理，这一节我们遵循`React`的运行流程，实现一个不到100行代码的极简`useState Hook`。建议对照着代码来看本节内容。

## [#](https://react.iamkasong.com/hooks/create.html#工作原理)工作原理

对于`useState Hook`，考虑如下例子：

```js
function App() {
  const [num, updateNum] = useState(0);

  return <p onClick={() => updateNum(num => num + 1)}>{num}</p>;
}
```

可以将工作分为两部分：

1. 通过一些途径产生`更新`，`更新`会造成组件`render`。
2. 组件`render`时`useState`返回的`num`为更新后的结果。

其中`步骤1`的`更新`可以分为`mount`和`update`：

1. 调用`ReactDOM.render`会产生`mount`的`更新`，`更新`内容为`useState`的`initialValue`（即`0`）。
2. 点击`p`标签触发`updateNum`会产生一次`update`的`更新`，`更新`内容为`num => num + 1`。

接下来讲解这两个步骤如何实现。

## [#](https://react.iamkasong.com/hooks/create.html#更新是什么)更新是什么

> 1. 通过一些途径产生`更新`，`更新`会造成组件`render`。

首先我们要明确`更新`是什么。

在我们的极简例子中，`更新`就是如下数据结构：

```js
const update = {
  // 更新执行的函数
  action,
  // 与同一个Hook的其他更新形成链表
  next: null
}
```

对于`App`来说，点击`p`标签产生的`update`的`action`为`num => num + 1`。

如果我们改写下`App`的`onClick`：

```js
// 之前
return <p onClick={() => updateNum(num => num + 1)}>{num}</p>;

// 之后
return <p onClick={() => {
  updateNum(num => num + 1);
  updateNum(num => num + 1);
  updateNum(num => num + 1);
}}>{num}</p>;
```

那么点击`p`标签会产生三个`update`。

## [#](https://react.iamkasong.com/hooks/create.html#update数据结构)update数据结构

这些`update`是如何组合在一起呢？

答案是：他们会形成`环状单向链表`。

调用`updateNum`实际调用的是`dispatchAction.bind(null, hook.queue)`，我们先来了解下这个函数：

```js
function dispatchAction(queue, action) {
  // 创建update
  const update = {
    action,
    next: null
  }

  // 环状单向链表操作
  if (queue.pending === null) {
    update.next = update;
  } else {
    update.next = queue.pending.next;
    queue.pending.next = update;
  }
  queue.pending = update;

  // 模拟React开始调度更新
  schedule();
}
```

环状链表操作不太容易理解，这里我们详细讲解下。

当产生第一个`update`（我们叫他`u0`），此时`queue.pending === null`。

`update.next = update;`即`u0.next = u0`，他会和自己首尾相连形成`单向环状链表`。

然后`queue.pending = update;`即`queue.pending = u0`

```js
queue.pending = u0 ---> u0
                ^       |
                |       |
                ---------
```

当产生第二个`update`（我们叫他`u1`），`update.next = queue.pending.next;`，此时`queue.pending.next === u0`， 即`u1.next = u0`。

`queue.pending.next = update;`，即`u0.next = u1`。

然后`queue.pending = update;`即`queue.pending = u1`

```js
queue.pending = u1 ---> u0   
                ^       |
                |       |
                ---------
```

你可以照着这个例子模拟插入多个`update`的情况，会发现`queue.pending`始终指向最后一个插入的`update`。

这样做的好处是，当我们要遍历`update`时，`queue.pending.next`指向第一个插入的`update`。

## [#](https://react.iamkasong.com/hooks/create.html#状态如何保存)状态如何保存

现在我们知道，`更新`产生的`update`对象会保存在`queue`中。

不同于`ClassComponent`的实例可以存储数据，对于`FunctionComponent`，`queue`存储在哪里呢？

答案是：`FunctionComponent`对应的`fiber`中。

我们使用如下精简的`fiber`结构：

```js
// App组件对应的fiber对象
const fiber = {
  // 保存该FunctionComponent对应的Hooks链表
  memoizedState: null,
  // 指向App函数
  stateNode: App
};
```

## [#](https://react.iamkasong.com/hooks/create.html#hook数据结构)Hook数据结构

接下来我们关注`fiber.memoizedState`中保存的`Hook`的数据结构。

可以看到，`Hook`与`update`类似，都通过`链表`连接。不过`Hook`是`无环`的`单向链表`。

```js
hook = {
  // 保存update的queue，即上文介绍的queue
  queue: {
    pending: null
  },
  // 保存hook对应的state
  memoizedState: initialState,
  // 与下一个Hook连接形成单向无环链表
  next: null
}
```

注意

注意区分`update`与`hook`的所属关系：

每个`useState`对应一个`hook`对象。

调用`const [num, updateNum] = useState(0);`时`updateNum`（即上文介绍的`dispatchAction`）产生的`update`保存在`useState`对应的`hook.queue`中。

## [#](https://react.iamkasong.com/hooks/create.html#模拟react调度更新流程)模拟React调度更新流程

在上文`dispatchAction`末尾我们通过`schedule`方法模拟`React`调度更新流程。

```js
function dispatchAction(queue, action) {
  // ...创建update
  
  // ...环状单向链表操作

  // 模拟React开始调度更新
  schedule();
}
```

现在我们来实现他。

我们用`isMount`变量指代是`mount`还是`update`。

```js
// 首次render时是mount
isMount = true;

function schedule() {
  // 更新前将workInProgressHook重置为fiber保存的第一个Hook
  workInProgressHook = fiber.memoizedState;
  // 触发组件render
  fiber.stateNode();
  // 组件首次render为mount，以后再触发的更新为update
  isMount = false;
}
```

通过`workInProgressHook`变量指向当前正在工作的`hook`。

```js
workInProgressHook = fiber.memoizedState;
```

在组件`render`时，每当遇到下一个`useState`，我们移动`workInProgressHook`的指针。

```js
workInProgressHook = workInProgressHook.next;
```

这样，只要每次组件`render`时`useState`的调用顺序及数量保持一致，那么始终可以通过`workInProgressHook`找到当前`useState`对应的`hook`对象。

到此为止，我们已经完成第一步。

> 1. 通过一些途径产生`更新`，`更新`会造成组件`render`。

接下来实现第二步。

> 1. 组件`render`时`useState`返回的`num`为更新后的结果。

## [#](https://react.iamkasong.com/hooks/create.html#计算state)计算state

组件`render`时会调用`useState`，他的大体逻辑如下：

```js
function useState(initialState) {
  // 当前useState使用的hook会被赋值该该变量
  let hook;

  if (isMount) {
    // ...mount时需要生成hook对象
  } else {
    // ...update时从workInProgressHook中取出该useState对应的hook
  }

  let baseState = hook.memoizedState;
  if (hook.queue.pending) {
    // ...根据queue.pending中保存的update更新state
  }
  hook.memoizedState = baseState;

  return [baseState, dispatchAction.bind(null, hook.queue)];
}
```

我们首先关注如何获取`hook`对象：

```js
if (isMount) {
  // mount时为该useState生成hook
  hook = {
    queue: {
      pending: null
    },
    memoizedState: initialState,
    next: null
  }

  // 将hook插入fiber.memoizedState链表末尾
  if (!fiber.memoizedState) {
    fiber.memoizedState = hook;
  } else {
    workInProgressHook.next = hook;
  }
  // 移动workInProgressHook指针
  workInProgressHook = hook;
} else {
  // update时找到对应hook
  hook = workInProgressHook;
  // 移动workInProgressHook指针
  workInProgressHook = workInProgressHook.next;
}
```

当找到该`useState`对应的`hook`后，如果该`hook.queue.pending`不为空（即存在`update`），则更新其`state`。

```js
// update执行前的初始state
let baseState = hook.memoizedState;

if (hook.queue.pending) {
  // 获取update环状单向链表中第一个update
  let firstUpdate = hook.queue.pending.next;

  do {
    // 执行update action
    const action = firstUpdate.action;
    baseState = action(baseState);
    firstUpdate = firstUpdate.next;

    // 最后一个update执行完后跳出循环
  } while (firstUpdate !== hook.queue.pending.next)

  // 清空queue.pending
  hook.queue.pending = null;
}

// 将update action执行完后的state作为memoizedState
hook.memoizedState = baseState;
```

完整代码如下：

```js
function useState(initialState) {
  let hook;

  if (isMount) {
    hook = {
      queue: {
        pending: null
      },
      memoizedState: initialState,
      next: null
    }
    if (!fiber.memoizedState) {
      fiber.memoizedState = hook;
    } else {
      workInProgressHook.next = hook;
    }
    workInProgressHook = hook;
  } else {
    hook = workInProgressHook;
    workInProgressHook = workInProgressHook.next;
  }

  let baseState = hook.memoizedState;
  if (hook.queue.pending) {
    let firstUpdate = hook.queue.pending.next;

    do {
      const action = firstUpdate.action;
      baseState = action(baseState);
      firstUpdate = firstUpdate.next;
    } while (firstUpdate !== hook.queue.pending.next)

    hook.queue.pending = null;
  }
  hook.memoizedState = baseState;

  return [baseState, dispatchAction.bind(null, hook.queue)];
}
```

## [#](https://react.iamkasong.com/hooks/create.html#对触发事件进行抽象)对触发事件进行抽象

最后，让我们抽象一下`React`的事件触发方式。

通过调用`App`返回的`click`方法模拟组件`click`的行为。

```js
function App() {
  const [num, updateNum] = useState(0);

  console.log(`${isMount ? 'mount' : 'update'} num: `, num);

  return {
    click() {
      updateNum(num => num + 1);
    }
  }
}
```

## [#](https://react.iamkasong.com/hooks/create.html#在线demo)在线Demo

至此，我们完成了一个不到100行代码的`Hooks`。重要的是，他与`React`的运行逻辑相同。

<details class="custom-block details" style="display: block; position: relative; border-radius: 2px; margin: 1.6em 0px; padding: 1.6em; background-color: rgb(238, 238, 238); color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Ubuntu, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><summary style="outline: none; cursor: pointer;">精简Hooks的在线Demo</summary><p style="line-height: 1.7;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code></p><p style="line-height: 1.7;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code></p><div class="language-js extra-class" style="position: relative; background-color: rgb(40, 44, 52); border-radius: 6px;"><pre class="language-js" style="color: rgb(204, 204, 204); background: transparent; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; line-height: 1.4; tab-size: 4; hyphens: none; padding: 1.25rem 1.5rem; margin: 0.85rem 0px; overflow: auto; border-radius: 6px; position: relative; z-index: 1;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(255, 255, 255); padding: 0px; margin: 0px; font-size: 0.85em; background-color: transparent; border-radius: 0px;"><span class="token keyword" style="color: rgb(204, 153, 205);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token keyword" style="color: rgb(204, 153, 205);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token number" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token keyword" style="color: rgb(204, 153, 205);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token number" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token template-string"><span class="token template-punctuation string" style="color: rgb(126, 198, 153);"></span><span class="token interpolation"><span class="token interpolation-punctuation punctuation" style="color: rgb(204, 204, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token interpolation-punctuation punctuation" style="color: rgb(204, 204, 204);"></span></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token template-punctuation string" style="color: rgb(126, 198, 153);"></span></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token template-string"><span class="token template-punctuation string" style="color: rgb(126, 198, 153);"></span><span class="token interpolation"><span class="token interpolation-punctuation punctuation" style="color: rgb(204, 204, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token interpolation-punctuation punctuation" style="color: rgb(204, 204, 204);"></span></span><span class="token string" style="color: rgb(126, 198, 153);"></span><span class="token template-punctuation string" style="color: rgb(126, 198, 153);"></span></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token keyword" style="color: rgb(204, 153, 205);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token parameter"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token number" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token function" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token parameter"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token operator" style="color: rgb(103, 205, 204);"></span><span class="token number" style="color: rgb(240, 141, 73);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span><span class="token punctuation" style="color: rgb(204, 204, 204);"></span></code></pre></div><p style="line-height: 1.7; margin-bottom: 0px; padding-bottom: 0px;"><a href="https://react.iamkasong.com/me.html" class="" style="font-weight: 500; text-decoration: none; color: rgb(62, 175, 124);"></a><strong style="font-weight: 600;"></strong></p></details>

## [#](https://react.iamkasong.com/hooks/create.html#与react的区别)与React的区别

我们用尽可能少的代码模拟了`Hooks`的运行，但是相比`React Hooks`，他还有很多不足。以下是他与`React Hooks`的区别：

1. `React Hooks`没有使用`isMount`变量，而是在不同时机使用不同的`dispatcher`。换言之，`mount`时的`useState`与`update`时的`useState`不是同一个函数。
2. `React Hooks`有中途跳过`更新`的优化手段。
3. `React Hooks`有`batchedUpdates`，当在`click`中触发三次`updateNum`，`精简React`会触发三次更新，而`React`只会触发一次。
4. `React Hooks`的`update`有`优先级`概念，可以跳过不高优先的`update`。

更多的细节，我们会在本章后续小节讲解。

# Hooks 数据结构

在上一节我们实现了一个极简的`useState`，了解了`Hooks`的运行原理。

本节我们讲解`Hooks`的数据结构，为后面介绍具体的`hook`打下基础。

## [#](https://react.iamkasong.com/hooks/structure.html#dispatcher)dispatcher

在上一节的极简`useState`实现中，使用`isMount`变量区分`mount`与`update`。

在真实的`Hooks`中，组件`mount`时的`hook`与`update`时的`hook`来源于不同的对象，这类对象在源码中被称为`dispatcher`。

```js
// mount时的Dispatcher
const HooksDispatcherOnMount: Dispatcher = {
  useCallback: mountCallback,
  useContext: readContext,
  useEffect: mountEffect,
  useImperativeHandle: mountImperativeHandle,
  useLayoutEffect: mountLayoutEffect,
  useMemo: mountMemo,
  useReducer: mountReducer,
  useRef: mountRef,
  useState: mountState,
  // ...省略
};

// update时的Dispatcher
const HooksDispatcherOnUpdate: Dispatcher = {
  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  useImperativeHandle: updateImperativeHandle,
  useLayoutEffect: updateLayoutEffect,
  useMemo: updateMemo,
  useReducer: updateReducer,
  useRef: updateRef,
  useState: updateState,
  // ...省略
};
```

可见，`mount`时调用的`hook`和`update`时调用的`hook`其实是两个不同的函数。

在`FunctionComponent` `render`前，会根据`FunctionComponent`对应`fiber`的以下条件区分`mount`与`update`。

```js
current === null || current.memoizedState === null
```

并将不同情况对应的`dispatcher`赋值给全局变量`ReactCurrentDispatcher`的`current`属性。

```js
ReactCurrentDispatcher.current =
      current === null || current.memoizedState === null
        ? HooksDispatcherOnMount
        : HooksDispatcherOnUpdate;  
```

> 你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L409)看到这行代码

在`FunctionComponent` `render`时，会从`ReactCurrentDispatcher.current`（即当前`dispatcher`）中寻找需要的`hook`。

换言之，不同的调用栈上下文为`ReactCurrentDispatcher.current`赋值不同的`dispatcher`，则`FunctionComponent` `render`时调用的`hook`也是不同的函数。

> 除了这两个`dispatcher`，你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1775)看到其他`dispatcher`定义

## [#](https://react.iamkasong.com/hooks/structure.html#一个dispatcher使用场景)一个dispatcher使用场景

当错误的书写了嵌套形式的`hook`，如：

```js
useEffect(() => {
  useState(0);
})
```

此时`ReactCurrentDispatcher.current`已经指向`ContextOnlyDispatcher`，所以调用`useState`实际会调用`throwInvalidHookError`，直接抛出异常。

```js
export const ContextOnlyDispatcher: Dispatcher = {
  useCallback: throwInvalidHookError,
  useContext: throwInvalidHookError,
  useEffect: throwInvalidHookError,
  useImperativeHandle: throwInvalidHookError,
  useLayoutEffect: throwInvalidHookError,
  // ...省略
```

> 你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L458)看到这段逻辑

## [#](https://react.iamkasong.com/hooks/structure.html#hook的数据结构)Hook的数据结构

接下来我们学习`hook`的数据结构。

```js
const hook: Hook = {
  memoizedState: null,

  baseState: null,
  baseQueue: null,
  queue: null,

  next: null,
};
```

> 你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L546)看到创建`hook`的逻辑

其中除`memoizedState`以外字段的意义与上一章介绍的[updateQueue](https://react.iamkasong.com/state/update.html#updatequeue)类似。

## [#](https://react.iamkasong.com/hooks/structure.html#memoizedstate)memoizedState

注意

`hook`与`FunctionComponent fiber`都存在`memoizedState`属性，不要混淆他们的概念。

- `fiber.memoizedState`：`FunctionComponent`对应`fiber`保存的`Hooks`链表。
- `hook.memoizedState`：`Hooks`链表中保存的单一`hook`对应的数据。

不同类型`hook`的`memoizedState`保存不同类型数据，具体如下：

- useState：对于`const [state, updateState] = useState(initialState)`，`memoizedState`保存`state`的值
- useReducer：对于`const [state, dispatch] = useReducer(reducer, {});`，`memoizedState`保存`state`的值
- useEffect：`memoizedState`保存包含`useEffect回调函数`、`依赖项`等的链表数据结构`effect`，你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1181)看到`effect`的创建过程。`effect`链表同时会保存在`fiber.updateQueue`中
- useRef：对于`useRef(1)`，`memoizedState`保存`{current: 1}`
- useMemo：对于`useMemo(callback, [depA])`，`memoizedState`保存`[callback(), depA]`
- useCallback：对于`useCallback(callback, [depA])`，`memoizedState`保存`[callback, depA]`。与`useMemo`的区别是，`useCallback`保存的是`callback`函数本身，而`useMemo`保存的是`callback`函数的执行结果

有些`hook`是没有`memoizedState`的，比如：

- useContext

# useState与useReducer

`Redux`的作者`Dan`加入`React`核心团队后的一大贡献就是“将`Redux`的理念带入`React`”。

这里面最显而易见的影响莫过于`useState`与`useReducer`这两个`Hook`。本质来说，`useState`只是预置了`reducer`的`useReducer`。

本节我们来学习`useState`与`useReducer`的实现。

## [#](https://react.iamkasong.com/hooks/usestate.html#流程概览)流程概览

我们将这两个`Hook`的工作流程分为`声明阶段`和`调用阶段`，对于：

```js
function App() {
  const [state, dispatch] = useReducer(reducer, {a: 1});

  const [num, updateNum] = useState(0);
  
  return (
    <div>
      <button onClick={() => dispatch({type: 'a'})}>{state.a}</button>  
      <button onClick={() => updateNum(num => num + 1)}>{num}</button>  
    </div>
  )
}
```

`声明阶段`即`App`调用时，会依次执行`useReducer`与`useState`方法。

`调用阶段`即点击按钮后，`dispatch`或`updateNum`被调用时。

## [#](https://react.iamkasong.com/hooks/usestate.html#声明阶段)声明阶段

当`FunctionComponent`进入`render阶段`的`beginWork`时，会调用[renderWithHooks (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L1419)方法。

该方法内部会执行`FunctionComponent`对应函数（即`fiber.type`）。

> 你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L415)看到这段逻辑

对于这两个`Hook`，他们的源码如下：

```js
function useState(initialState) {
  var dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
function useReducer(reducer, initialArg, init) {
  var dispatcher = resolveDispatcher();
  return dispatcher.useReducer(reducer, initialArg, init);
}
```

正如上一节[dispatcher](https://react.iamkasong.com/hooks/structure.html#dispatcher)所说，在不同场景下，同一个`Hook`会调用不同处理函数。

我们分别讲解`mount`与`update`两个场景。

### [#](https://react.iamkasong.com/hooks/usestate.html#mount时)mount时

`mount`时，`useReducer`会调用[mountReducer (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L638)，`useState`会调用[mountState (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1143)。

我们来简单对比这这两个方法：

```js
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  // 创建并返回当前的hook
  const hook = mountWorkInProgressHook();

  // ...赋值初始state

  // 创建queue
  const queue = (hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });

  // ...创建dispatch
  return [hook.memoizedState, dispatch];
}

function mountReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 创建并返回当前的hook
  const hook = mountWorkInProgressHook();

  // ...赋值初始state

  // 创建queue
  const queue = (hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: reducer,
    lastRenderedState: (initialState: any),
  });

  // ...创建dispatch
  return [hook.memoizedState, dispatch];
}
```

其中`mountWorkInProgressHook`方法会创建并返回对应`hook`，对应`极简Hooks实现`中`useState`方法的`isMount`逻辑部分。

可以看到，`mount`时这两个`Hook`的唯一区别为`queue`参数的`lastRenderedReducer`字段。

`queue`的数据结构如下：

```js
const queue = (hook.queue = {
  // 与极简实现中的同名字段意义相同，保存update对象
  pending: null,
  // 保存dispatchAction.bind()的值
  dispatch: null,
  // 上一次render时使用的reducer
  lastRenderedReducer: reducer,
  // 上一次render时的state
  lastRenderedState: (initialState: any),
});
```

其中，`useReducer`的`lastRenderedReducer`为传入的`reducer`参数。`useState`的`lastRenderedReducer`为`basicStateReducer`。

`basicStateReducer`方法如下：

```js
function basicStateReducer<S>(state: S, action: BasicStateAction<S>): S {
  return typeof action === 'function' ? action(state) : action;
}
```

可见，`useState`即`reducer`参数为`basicStateReducer`的`useReducer`。

`mount`时的整体运行逻辑与`极简实现`的`isMount`逻辑类似，你可以对照着看。

### [#](https://react.iamkasong.com/hooks/usestate.html#update时)update时

如果说`mount`时这两者还有区别，那`update`时，`useReducer`与`useState`调用的则是同一个函数[updateReducer (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L665)。

```js
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 获取当前hook
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;
  
  queue.lastRenderedReducer = reducer;

  // ...同update与updateQueue类似的更新逻辑

  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```

整个流程可以概括为一句话：

> 找到对应的`hook`，根据`update`计算该`hook`的新`state`并返回。

`mount`时获取当前`hook`使用的是`mountWorkInProgressHook`，而`update`时使用的是`updateWorkInProgressHook`，这里的原因是：

- `mount`时可以确定是调用`ReactDOM.render`或相关初始化`API`产生的`更新`，只会执行一次。
- `update`可能是在事件回调或副作用中触发的`更新`或者是`render阶段`触发的`更新`，为了避免组件无限循环`更新`，后者需要区别对待。

举个`render阶段`触发的`更新`的例子：

```js
function App() {
  const [num, updateNum] = useState(0);
  
  updateNum(num + 1);

  return (
    <button onClick={() => updateNum(num => num + 1)}>{num}</button>  
  )
}
```

在这个例子中，`App`调用时，代表已经进入`render阶段`执行`renderWithHooks`。

在`App`内部，调用`updateNum`会触发一次`更新`。如果不对这种情况下触发的更新作出限制，那么这次`更新`会开启一次新的`render阶段`，最终会无限循环更新。

基于这个原因，`React`用一个标记变量`didScheduleRenderPhaseUpdate`判断是否是`render阶段`触发的更新。

`updateWorkInProgressHook`方法也会区分这两种情况来获取对应`hook`。

获取对应`hook`，接下来会根据`hook`中保存的`state`计算新的`state`，这个步骤同[Update一节](https://react.iamkasong.com/state/update.html)一致。

## [#](https://react.iamkasong.com/hooks/usestate.html#调用阶段)调用阶段

调用阶段会执行[dispatchAction (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1662)，此时该`FunctionComponent`对应的`fiber`以及`hook.queue`已经通过调用`bind`方法预先作为参数传入。

```js
function dispatchAction(fiber, queue, action) {

  // ...创建update
  var update = {
    eventTime: eventTime,
    lane: lane,
    suspenseConfig: suspenseConfig,
    action: action,
    eagerReducer: null,
    eagerState: null,
    next: null
  }; 

  // ...将update加入queue.pending
  
  var alternate = fiber.alternate;

  if (fiber === currentlyRenderingFiber$1 || alternate !== null && alternate === currentlyRenderingFiber$1) {
    // render阶段触发的更新
    didScheduleRenderPhaseUpdateDuringThisPass = didScheduleRenderPhaseUpdate = true;
  } else {
    if (fiber.lanes === NoLanes && (alternate === null || alternate.lanes === NoLanes)) {
      // ...fiber的updateQueue为空，优化路径
    }

    scheduleUpdateOnFiber(fiber, lane, eventTime);
  }
}
```

整个过程可以概括为：

> 创建`update`，将`update`加入`queue.pending`中，并开启调度。

这里值得注意的是`if...else...`逻辑，其中：

```js
if (fiber === currentlyRenderingFiber$1 || alternate !== null && alternate === currentlyRenderingFiber$1)
```

`currentlyRenderingFiber`即`workInProgress`，`workInProgress`存在代表当前处于`render阶段`。

触发`更新`时通过`bind`预先保存的`fiber`与`workInProgress`全等，代表本次`更新`发生于`FunctionComponent`对应`fiber`的`render阶段`。

所以这是一个`render阶段`触发的`更新`，需要标记变量`didScheduleRenderPhaseUpdate`，后续单独处理。

再来关注：

```js
if (fiber.lanes === NoLanes && (alternate === null || alternate.lanes === NoLanes))
```

`fiber.lanes`保存`fiber`上存在的`update`的`优先级`。

`fiber.lanes === NoLanes`意味着`fiber`上不存在`update`。

我们已经知道，通过`update`计算`state`发生在`声明阶段`，这是因为该`hook`上可能存在多个不同`优先级`的`update`，最终`state`的值由多个`update`共同决定。

但是当`fiber`上不存在`update`，则`调用阶段`创建的`update`为该`hook`上第一个`update`，在`声明阶段`计算`state`时也只依赖于该`update`，完全不需要进入`声明阶段`再计算`state`。

这样做的好处是：如果计算出的`state`与该`hook`之前保存的`state`一致，那么完全不需要开启一次调度。即使计算出的`state`与该`hook`之前保存的`state`不一致，在`声明阶段`也可以直接使用`调用阶段`已经计算出的`state`。

> 你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1727)看到这段提前计算`state`的逻辑

## [#](https://react.iamkasong.com/hooks/usestate.html#小tip)小Tip

我们通常认为，`useReducer(reducer, initialState)`的传参为初始化参数，在以后的调用中都不可变。

但是在`updateReducer`方法中，可以看到`lastRenderedReducer`在每次调用时都会重新赋值。

```js
function updateReducer(reducer, initialArg, init) {
  // ...

  queue.lastRenderedReducer = reducer;

  // ...
```

也就是说，`reducer`参数是随时可变的。

<details class="custom-block details" style="display: block; position: relative; border-radius: 2px; margin: 1.6em 0px; padding: 1.6em; background-color: rgb(238, 238, 238);"><summary style="outline: none; cursor: pointer;">reducer可变Demo</summary><p style="line-height: 1.7;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code></p><p style="line-height: 1.7;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: rgb(71, 101, 130); padding: 0.25rem 0.5rem; margin: 0px; font-size: 0.85em; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px;"></code></p><p style="line-height: 1.7; margin-bottom: 0px; padding-bottom: 0px;"><a href="https://react.iamkasong.com/me.html" class="" style="font-weight: 500; text-decoration: none; color: rgb(62, 175, 124);"></a><strong style="font-weight: 600;"></strong></p></details>

# useEffect

在[架构篇commit阶段流程概览](https://react.iamkasong.com/renderer/prepare.html)我们讲解了`useEffect`的工作流程。

其中我们谈到

> 在`flushPassiveEffects`方法内部会从全局变量`rootWithPendingPassiveEffects`获取`effectList`。

本节我们深入`flushPassiveEffects`方法内部探索`useEffect`的工作原理。

## [#](https://react.iamkasong.com/hooks/useeffect.html#flushpassiveeffectsimpl)flushPassiveEffectsImpl

`flushPassiveEffects`内部会设置`优先级`，并执行`flushPassiveEffectsImpl`。

> 你可以从[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.old.js#L2458)看到`flushPassiveEffects`的代码

`flushPassiveEffectsImpl`主要做三件事：

- 调用该`useEffect`在上一次`render`时的销毁函数
- 调用该`useEffect`在本次`render`时的回调函数
- 如果存在同步任务，不需要等待下次`事件循环`的`宏任务`，提前执行他

本节我们关注前两步。

在`v16`中第一步是同步执行的，在[官方博客 (opens new window)](https://zh-hans.reactjs.org/blog/2020/08/10/react-v17-rc.html#effect-cleanup-timing)中提到：

> 副作用清理函数（如果存在）在 React 16 中同步运行。我们发现，对于大型应用程序来说，这不是理想选择，因为同步会减缓屏幕的过渡（例如，切换标签）。

基于这个原因，在`v17.0.0`中，`useEffect`的两个阶段会在页面渲染后（`layout`阶段后）异步执行。

> 事实上，从代码中看，`v16.13.1`中已经是异步执行了

接下来我们详细讲解这两个步骤。

## [#](https://react.iamkasong.com/hooks/useeffect.html#阶段一-销毁函数的执行)阶段一：销毁函数的执行

`useEffect`的执行需要保证所有组件`useEffect`的`销毁函数`必须都执行完后才能执行任意一个组件的`useEffect`的`回调函数`。

这是因为多个`组件`间可能共用同一个`ref`。

如果不是按照“全部销毁”再“全部执行”的顺序，那么在某个组件`useEffect`的`销毁函数`中修改的`ref.current`可能影响另一个组件`useEffect`的`回调函数`中的同一个`ref`的`current`属性。

在`useLayoutEffect`中也有同样的问题，所以他们都遵循“全部销毁”再“全部执行”的顺序。

在阶段一，会遍历并执行所有`useEffect`的`销毁函数`。

```js
// pendingPassiveHookEffectsUnmount中保存了所有需要执行销毁的useEffect
const unmountEffects = pendingPassiveHookEffectsUnmount;
  pendingPassiveHookEffectsUnmount = [];
  for (let i = 0; i < unmountEffects.length; i += 2) {
    const effect = ((unmountEffects[i]: any): HookEffect);
    const fiber = ((unmountEffects[i + 1]: any): Fiber);
    const destroy = effect.destroy;
    effect.destroy = undefined;

    if (typeof destroy === 'function') {
      // 销毁函数存在则执行
      try {
        destroy();
      } catch (error) {
        captureCommitPhaseError(fiber, error);
      }
    }
  }
```

其中`pendingPassiveHookEffectsUnmount`数组的索引`i`保存需要销毁的`effect`，`i+1`保存该`effect`对应的`fiber`。

向`pendingPassiveHookEffectsUnmount`数组内`push`数据的操作发生在`layout阶段` `commitLayoutEffectOnFiber`方法内部的`schedulePassiveEffects`方法中。

> `commitLayoutEffectOnFiber`方法我们在[Layout阶段](https://react.iamkasong.com/renderer/layout.html#commitlayouteffectonfiber)已经介绍

```js
function schedulePassiveEffects(finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      const {next, tag} = effect;
      if (
        (tag & HookPassive) !== NoHookEffect &&
        (tag & HookHasEffect) !== NoHookEffect
      ) {
        // 向`pendingPassiveHookEffectsUnmount`数组内`push`要销毁的effect
        enqueuePendingPassiveHookEffectUnmount(finishedWork, effect);
        // 向`pendingPassiveHookEffectsMount`数组内`push`要执行回调的effect
        enqueuePendingPassiveHookEffectMount(finishedWork, effect);
      }
      effect = next;
    } while (effect !== firstEffect);
  }
}
```

## [#](https://react.iamkasong.com/hooks/useeffect.html#阶段二-回调函数的执行)阶段二：回调函数的执行

与阶段一类似，同样遍历数组，执行对应`effect`的`回调函数`。

其中向`pendingPassiveHookEffectsMount`中`push`数据的操作同样发生在`schedulePassiveEffects`中。

```js
// pendingPassiveHookEffectsMount中保存了所有需要执行回调的useEffect
const mountEffects = pendingPassiveHookEffectsMount;
pendingPassiveHookEffectsMount = [];
for (let i = 0; i < mountEffects.length; i += 2) {
  const effect = ((mountEffects[i]: any): HookEffect);
  const fiber = ((mountEffects[i + 1]: any): Fiber);
  
  try {
    const create = effect.create;
   effect.destroy = create();
  } catch (error) {
    captureCommitPhaseError(fiber, error);
  }
}
```

# useRef

`ref`是`reference`（引用）的缩写。在`React`中，我们习惯用`ref`保存`DOM`。

事实上，任何需要被"引用"的数据都可以保存在`ref`中，`useRef`的出现将这种思想进一步发扬光大。

在[Hooks数据结构一节](https://react.iamkasong.com/hooks/structure.html#memoizedstate)我们讲到：

> 对于`useRef(1)`，`memoizedState`保存`{current: 1}`

本节我们会介绍`useRef`的实现，以及`ref`的工作流程。

由于`string`类型的`ref`已不推荐使用，所以本节针对`function | {current: any}`类型的`ref`。

## [#](https://react.iamkasong.com/hooks/useref.html#useref)useRef

与其他`Hook`一样，对于`mount`与`update`，`useRef`对应两个不同`dispatcher`。

```js
function mountRef<T>(initialValue: T): {|current: T|} {
  // 获取当前useRef hook
  const hook = mountWorkInProgressHook();
  // 创建ref
  const ref = {current: initialValue};
  hook.memoizedState = ref;
  return ref;
}

function updateRef<T>(initialValue: T): {|current: T|} {
  // 获取当前useRef hook
  const hook = updateWorkInProgressHook();
  // 返回保存的数据
  return hook.memoizedState;
}
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.old.js#L1208-L1221)看到这段代码

可见，`useRef`仅仅是返回一个包含`current`属性的对象。

为了验证这个观点，我们再看下`React.createRef`方法的实现：

```js
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  return refObject;
}
```

> 你可以从[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react/src/ReactCreateRef.js)看到这段代码

了解了`ref`的数据结构后，我们再来看看`ref`的工作流程。

## [#](https://react.iamkasong.com/hooks/useref.html#ref的工作流程)ref的工作流程

在`React`中，`HostComponent`、`ClassComponent`、`ForwardRef`可以赋值`ref`属性。

```js
// HostComponent
<div ref={domRef}></div>
// ClassComponent / ForwardRef
<App ref={cpnRef} />
```

其中，`ForwardRef`只是将`ref`作为第二个参数传递下去，不会进入`ref`的工作流程。

所以接下来讨论`ref`的工作流程时会排除`ForwardRef`。

```js
// 对于ForwardRef，secondArg为传递下去的ref
let children = Component(props, secondArg);
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.old.js#L415)看到这段代码

我们知道`HostComponent`在`commit阶段`的`mutation阶段`执行`DOM`操作。

所以，对应`ref`的更新也是发生在`mutation阶段`。

再进一步，`mutation阶段`执行`DOM`操作的依据为`effectTag`。

所以，对于`HostComponent`、`ClassComponent`如果包含`ref`操作，那么也会赋值相应的`effectTag`。

```js
// ...
export const Placement = /*                    */ 0b0000000000000010;
export const Update = /*                       */ 0b0000000000000100;
export const Deletion = /*                     */ 0b0000000000001000;
export const Ref = /*                          */ 0b0000000010000000;
// ...
```

> 你可以在[ReactSideEffectTags文件 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js#L24)中看到`ref`对应的`effectTag`

所以，`ref`的工作流程可以分为两部分：

- `render阶段`为含有`ref`属性的`fiber`添加`Ref effectTag`
- `commit阶段`为包含`Ref effectTag`的`fiber`执行对应操作

## [#](https://react.iamkasong.com/hooks/useref.html#render阶段)render阶段

在`render阶段`的`beginWork`与`completeWork`中有个同名方法`markRef`用于为含有`ref`属性的`fiber`增加`Ref effectTag`。

```js
// beginWork的markRef
function markRef(current: Fiber | null, workInProgress: Fiber) {
  const ref = workInProgress.ref;
  if (
    (current === null && ref !== null) ||
    (current !== null && current.ref !== ref)
  ) {
    // Schedule a Ref effect
    workInProgress.effectTag |= Ref;
  }
}
// completeWork的markRef
function markRef(workInProgress: Fiber) {
  workInProgress.effectTag |= Ref;
}
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.old.js#L693)看到`beginWork`的`markRef`、[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCompleteWork.old.js#L153)看到`completeWork`的`markRef`

在`beginWork`中，如下两处调用了`markRef`：

- `updateClassComponent`内的[finishClassComponent (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.old.js#L958)，对应`ClassComponent`

注意`ClassComponent`即使`shouldComponentUpdate`为`false`该组件也会调用`markRef`

- [updateHostComponent (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.old.js#L1156)，对应`HostComponent`

在`completeWork`中，如下两处调用了`markRef`：

- `completeWork`中的[HostComponent (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCompleteWork.old.js#L728)类型
- `completeWork`中的[ScopeComponent (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCompleteWork.old.js#L1278)类型

> `ScopeComponent`是一种用于管理`focus`的测试特性，详见[PR(opens new window)](https://github.com/facebook/react/pull/16587)

总结下`组件`对应`fiber`被赋值`Ref effectTag`需要满足的条件：

- `fiber`类型为`HostComponent`、`ClassComponent`、`ScopeComponent`（这种情况我们不讨论）
- 对于`mount`，`workInProgress.ref !== null`，即存在`ref`属性
- 对于`update`，`current.ref !== workInProgress.ref`，即`ref`属性改变

## [#](https://react.iamkasong.com/hooks/useref.html#commit阶段)commit阶段

在`commit阶段`的`mutation阶段`中，对于`ref`属性改变的情况，需要先移除之前的`ref`。

```js
function commitMutationEffects(root: FiberRoot, renderPriorityLevel) {
  while (nextEffect !== null) {

    const effectTag = nextEffect.effectTag;
    // ...

    if (effectTag & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        // 移除之前的ref
        commitDetachRef(current);
      }
    }
    // ...
  }
  // ...
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.old.js#L2342)看到这段代码

```js
function commitDetachRef(current: Fiber) {
  const currentRef = current.ref;
  if (currentRef !== null) {
    if (typeof currentRef === 'function') {
      // function类型ref，调用他，传参为null
      currentRef(null);
    } else {
      // 对象类型ref，current赋值为null
      currentRef.current = null;
    }
  }
}
```

接下来，在`mutation阶段`，对于`Deletion effectTag`的`fiber`（对应需要删除的`DOM节点`），需要递归他的子树，对子孙`fiber`的`ref`执行类似`commitDetachRef`的操作。

在[mutation阶段一节](https://react.iamkasong.com/hooks/renderer/mutation.html#commitmutationeffects)我们讲到

> 对于`Deletion effectTag`的`fiber`，会执行`commitDeletion`。

在`commitDeletion`——`unmountHostComponents`——`commitUnmount`——`ClassComponent | HostComponent`类型`case`中调用的`safelyDetachRef`方法负责执行类似`commitDetachRef`的操作。

```js
function safelyDetachRef(current: Fiber) {
  const ref = current.ref;
  if (ref !== null) {
    if (typeof ref === 'function') {
      try {
        ref(null);
      } catch (refError) {
        captureCommitPhaseError(current, refError);
      }
    } else {
      ref.current = null;
    }
  }
}
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberCommitWork.old.js#L183)看到这段代码

接下来进入`ref`的赋值阶段。我们在[Layout阶段一节](https://react.iamkasong.com/renderer/layout.html#commitlayouteffects)讲到

> `commitLayoutEffect`会执行`commitAttachRef`（赋值`ref`）

```js
function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    // 获取ref属性对应的Component实例
    const instance = finishedWork.stateNode;
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent:
        instanceToUse = getPublicInstance(instance);
        break;
      default:
        instanceToUse = instance;
    }

    // 赋值ref
    if (typeof ref === 'function') {
      ref(instanceToUse);
    } else {
      ref.current = instanceToUse;
    }
  }
}
```

至此，`ref`的工作流程完毕。

## [#](https://react.iamkasong.com/hooks/useref.html#总结)总结

本节我们学习了`ref`的工作流程。

- 对于`FunctionComponent`，`useRef`负责创建并返回对应的`ref`。
- 对于赋值了`ref`属性的`HostComponent`与`ClassComponent`，会在`render阶段`经历赋值`Ref effectTag`，在`commit阶段`执行对应`ref`操作。

# useMemo和useCallback

在了解其他`hook`的实现后，理解`useMemo`与`useCallback`的实现非常容易。

本节我们以`mount`与`update`两种情况分别讨论这两个`hook`。

## [#](https://react.iamkasong.com/hooks/usememo.html#mount)mount

```js
function mountMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  // 创建并返回当前hook
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  // 计算value
  const nextValue = nextCreate();
  // 将value与deps保存在hook.memoizedState
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  // 创建并返回当前hook
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  // 将value与deps保存在hook.memoizedState
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

可以看到，与`mountCallback`这两个唯一的区别是

- `mountMemo`会将`回调函数`(nextCreate)的执行结果作为`value`保存
- `mountCallback`会将`回调函数`作为`value`保存

## [#](https://react.iamkasong.com/hooks/usememo.html#update)update

```js
function updateMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  // 返回当前hook
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;

  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      // 判断update前后value是否变化
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 未变化
        return prevState[0];
      }
    }
  }
  // 变化，重新计算value
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  // 返回当前hook
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;

  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      // 判断update前后value是否变化
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 未变化
        return prevState[0];
      }
    }
  }

  // 变化，将新的callback作为value
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

可见，对于`update`，这两个`hook`的唯一区别也是**是回调函数本身还是回调函数的执行结果作为value**。
