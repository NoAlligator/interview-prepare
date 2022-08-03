# React设计理念

> React 是用 JavaScript 构建**快速响应**的大型 Web 应用程序的首选方式。

React设计理念的关键是实现`快速响应`。

## 制约快速响应的二因素

- JS计算
  - CPU限制/瓶颈
- 网络请求
  - IO瓶颈/带宽限制

## 处理CPU瓶颈

> 主流浏览器刷新频率为60Hz，即每（1000ms / 60Hz）16.6ms浏览器刷新一次。JS可以操作DOM，`GUI渲染线程`与`JS线程`是互斥的。所以**JS脚本执行**和**浏览器布局、绘制**不能同时执行，在每个16.6ms时间内**浏览器需要完成以上三个操作**，如果仅就JS脚本执行而言，如果它的执行时间过长就会挤压甚至错过样式布局。

### React的解决方案

在浏览器**每一帧的时间中**，预留一些时间给JS线程，`React`利用这部分时间更新组件（时间切片）。当预留的时间不够用时，`React`将线程控制权交还给浏览器使其有时间渲染UI，`React`则等待下一帧时间到来继续被中断的工作。解决`CPU瓶颈`的关键是实现`时间切片`，而`时间切片`的关键是：将**同步的更新**变为**可中断的异步更新**。

## 处理IO瓶颈

短暂延迟无感化、长时延迟进入后备处理

[参考](https://react.iamkasong.com/preparation/idea.html#io%E7%9A%84%E7%93%B6%E9%A2%88)

# 老的React架构（React 15）

## 架构分层

- Reconciler（协调器） —— 负责找出变化的组件
  - 每当有**更新**发生时，**Reconciler**会做如下工作：
  - （Render -> JSX解析 -> Virtual DOM）调用函数组件、或class组件的`render`方法，将返回的JSX转化为虚拟DOM；
  - （DOM Diff）将虚拟DOM和上次更新时的虚拟DOM对比，通过对比找出本次更新中变化的虚拟DOM；
  - （通知render）通知Renderer将变化的虚拟DOM渲染到页面上
- Renderer（渲染器） —— 负责将变化的组件渲染到页面上

 ![image-20220128161412747](https://github.com/NoAlligator/pico/blob/main/img/202203271805846.png?raw=true)

## 老架构的缺点

在**Reconciler**中，`mount`的组件会调用[mountComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，`update`的组件会调用[updateComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会**递归更新子组件**。递**归更新子组件是缺点的来源。**

### 递归更新的缺点

由于递归执行，所以**更新一旦开始，中途就无法中断**。当层级很深时，递归更新时间超过了16ms（1000ms / 60），用户交互就会卡顿。而且老的架构**从架构上无法实现异步可中断更新**。

### 递归更新的流程

**Reconciler**和**Renderer**是交替工作的，当第一个`li`在页面上已经变化后，第二个`li`再进入**Reconciler**。由于整个过程都是**同步**的，所以在用户看来**所有DOM是同时更新的**。

![image-20220128162424933](https://github.com/NoAlligator/pico/blob/main/img/202203271805847.png?raw=true)

### 老的架构不能支持异步可中断更新的原因

当第一个`li`完成更新时中断更新，即步骤3完成后中断更新，此时后面的步骤都还未执行。用户本来期望`123`变为`246`，可是实际上只有`1 -> 2`。所以这一种同步不可中断式的更新方式在进行中断的过程中会导致不完全更新。基于这个原因，`React`决定重写整个架构。

![image-20220128162712137](https://github.com/NoAlligator/pico/blob/main/img/202203271805848.png?raw=true)

# 新的React架构（React 16）

## 架构分层

- Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入协调器；
- Reconciler（协调器）—— 负责找出变化的组件并执行Diff等，由协调器来通知调度器更新；
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上。

### 调度器Scheduler

既然我们以浏览器是否有剩余时间作为任务中断的标准，那么我们需要一种机制，当浏览器有剩余时间时通知我们。部分浏览器已经原生实现了[requestIdleCallback (opens new window)](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)，它可以通知浏览器是否有剩余时间来作为任务中断的标准，但是其兼容性差、触发频率不稳定的特性使得React必须自己实现功能更完备的polyfill，这就是**Scheduler**。除了在空闲时触发回调的功能外，Scheduler还提供了多种调度优先级供任务设置。Scheduler是**单独于React的库**。

### 协调器Reconciler

在React15中**Reconciler**是递归处理虚拟DOM的。更新工作从递归变成了可以中断的循环过程。每次循环都会调用`shouldYield`**判断当前是否有剩余时间**。

```javascript
/** @noinline */
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  //当且仅当Scheduler通知我们有空的时候才会进入工作单元
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

React16是如何解决**中断更新时DOM渲染不完全**的问题呢？

在React16中，**Reconciler**与**Renderer**不再是**交替工作**。当**Scheduler**将任务交给**Reconciler**后，**Reconciler**会为变化的虚拟DOM打上代表增/删/更新的标记，类似这样：

```javascript
export const Placement = /*             */ 0b0000000000010;
export const Update = /*                */ 0b0000000000100;
export const PlacementAndUpdate = /*    */ 0b0000000000110;
export const Deletion = /*              */ 0b0000000001000;
```

[全部标记参考](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js)

整个**Scheduler**与**Reconciler**的工作**都在内存中进行**。**只有当所有组件都完成Reconciler的工作，才会统一交给Renderer。**所以在**正式完成这一轮的全部更新**之前，React是不会主动进入commit阶段的。

### 渲染器

**Renderer**根据**Reconciler**为虚拟DOM打的标记，**同步**执行对应的DOM操作。

![image-20220128164828094](https://github.com/NoAlligator/pico/blob/main/img/202203271805849.png?raw=true)

其中红框中的步骤随时可能由于以下原因被中断：

- 有其他**更高优任务需要先更新**
- 当前**帧没有剩余时间**

由于红框中的工作都在内存中进行，**不会更新页面上的DOM**，所以**即使反复中断，用户也不会看见更新不完全的DOM**。

> **Scheduler**和**Reconciler**都是**平台无关**的，`React`为他们单独发了一个包[react-Reconciler (opens new window)](https://www.npmjs.com/package/react-reconciler)。

# Fiber的心智模型

## 代数效应

`代数效应`是`函数式编程`中的一个概念，用于**将`副作用`从`函数`调用中分离，使函数关注点保持纯粹**。

## 代数效应与Generator

从`React15`到`React16`，协调器（`Reconciler`）重构的一大目的是：将老的`同步更新`的架构变为`异步可中断更新`。`异步可中断更新`可以理解为：`更新`在执行过程中可能会被**打断**（浏览器时间分片用尽或有更高优任务插队），当可以继续执行时恢复之前执行的中间状态。`Generator`是JS原生支持的协程方案，但是存在以下缺陷：

- 类似`async`，`Generator`也是`传染性`的，使用了`Generator`则上下文的其他函数也需要作出改变，这样心智负担比较重；
- `Generator`执行的`中间状态`是**上下文关联**的，想要**复用中间状态**必须引入**全局变量**。

在现有`Generator`的逻辑下，可以实现的是“单一优先级任务的中断与继续”，但是无法实现“高优先级任务插队”（因为插队就要实现暂停并保留中间状态，上述第二条缺陷限制了Generator的使用）。

## 代数效应与React

对于类似`useState`、`useReducer`、`useRef`这样的`Hook`，我们不需要关注`FunctionComponent`的`state`在`Hook`中是如何保存的，`React`会为我们处理（关注状态之间的逻辑而非状态本身）。比较贴切的例子就是`Suspense`组件，将具有传染性的异步逻辑抽离出来使得关注点放在了**取值本身**。

## 代数效应与Fiber

`Fiber`并不是计算机术语中的新名词，他的中文翻译叫做`纤程`，与进程（Process）、线程（Thread）、协程（Coroutine）同为程序执行过程。在很多文章中将`纤程`理解为`协程`的一种实现。在`JS`中，`协程`的实现便是`Generator`。所以，我们可以将`纤程`(Fiber)、`协程`(Generator)理解为`代数效应`思想在`JS`中的体现。

`React Fiber`可以理解为：`React`内部实现的**一套状态更新机制**。支持任务不同`优先级`，可中断与恢复，并且恢复后可以复用之前的`中间状态`（复用中间状态是`Generator`无法实现的）。

#  Fiber架构的实现原理

> **虚拟DOM**在`React`中有个正式的称呼——`Fiber`

## Fiber 三层含义

1. 作为架构来说，之前`React15`的`Reconciler`采用递归的方式执行，数据保存在递归调用栈中，所以被称为`stack Reconciler`。`React16`的`Reconciler`基于`Fiber节点`实现，被称为`Fiber Reconciler`。
2. 作为**静态的数据结构**来说，每个`Fiber节点`对应一个`React element`，保存了该组件的类型（函数组件/类组件/原生组件...）、对应的DOM节点等信息。
3. 作为**动态的工作单元**来说，每个`Fiber节点`保存了本次更新中该组件改变的状态、要执行的工作（需要被删除/被插入页面中/被更新...）。

## Fiber 根节点

注：首次执行`ReactDOM.render`会创建`fiberRootNode`（源码中叫`fiberRoot`）和`rootFiber`。

![Untitled.png](https://github.com/NoAlligator/pico/blob/main/img/7f6502636689492da29c8cd783c3b37a~tplv-k3u1fbpfcp-watermark.awebp?raw=true)

### fiberRootNode

`fiberRootNode`是整个应用的根节点，绑定在真实DOM节点的`_reactRootContainer`属性上,当对一个元素重复调用`ReactDOM.render`时`fiberRootNode`不会改变。

### rootFiber

`rootFiber`是`<App/>`所在**组件树的根节点**，`rootFiber`在每次重新渲染的时候会重新构建。

## Fiber 节点的属性

我们可以按三层含义将Fiber Node分类：**作为静态数据结构的属性、用于连接其他Fiber节点形成Fiber树的属性、作为动态的工作单元的属性**。

```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // 作为静态数据结构的属性(Fiber节点的静态数据结构)
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // 用于连接其他Fiber节点形成Fiber树（Fiber节点通过以下属性连接成Fiber树）
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  // 作为动态的工作单元的属性（Fiber节点的动态工作单元）
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

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
}
```

## Fiber 三层含义详解

### Fiber作为架构来说

每个Fiber节点有个对应的`React element`，多个`Fiber节点`通过指针形成指定的Fiber树。

```javascript
// 存在三种指针：return、child、sibling

// 指向父级Fiber节点
this.return = null;
// 指向子Fiber节点
this.child = null;
// 指向右边第一个兄弟Fiber节点
this.sibling = null;
```

以如下的组件结构为例：

```jsx
function App() {
  return (
    <div>
      i am
      <span>KaSong</span>
    </div>
  )
}
```

对应的`Fiber树`结构：

![image-20220128232418841](https://github.com/NoAlligator/pico/blob/main/img/image-20220128232418841.png?raw=true)

#### 为什么父级指针叫做`return`而不是`parent`或者`father`呢？

因为作为一个**工作单元**，`return`指节点执行完`completeWork`后会返回的下一个节点。子`Fiber节点`及其兄弟节点完成工作后会返回其父级节点来执行`compoleteWork`，所以用`return`指代父级节点。

### Fiber作为静态数据结构来说

```javascript
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

### Fiber作为动态的工作单元来说

```javascript
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

剩下的两个字段保存调度优先级相关的信息：

```js
// 调度优先级相关
this.lanes = NoLanes;
this.childLanes = NoLanes;
```



# Fiber架构的工作原理

## 双缓存技术

> 当我们用`canvas`绘制动画，每一帧绘制前都会调用`ctx.clearRect`清除上一帧的画面。如果当前帧画面计算量比较大，导致清除上一帧画面到绘制当前帧画面之间有较长间隙，就会出现**白屏**。为了解决这个问题，我们可以在内存中绘制当前帧动画，绘制完毕后直接用当前帧替换上一帧画面，由于省去了两帧替换间的计算时间，不会出现从白屏到出现画面的闪烁情况。

这种**在内存中构建并直接替换**的技术叫做[双缓存 (opens new window)](https://baike.baidu.com/item/双缓冲)。`React`使用“双缓存”来完成`Fiber树`的构建与替换——对应着`DOM树`的创建与更新。

## 双缓存Fiber树

在`React`中最多会同时存在两棵`Fiber树`。当前屏幕上显示内容对应的`Fiber树`称为`current Fiber树`，正在内存中构建的`Fiber树`称为`workInProgress Fiber树`。

`current Fiber树`中的`Fiber节点`被称为`current fiber`，`workInProgress Fiber树`中的`Fiber节点`被称为`workInProgress fiber`，他们通过`alternate`属性连接。

```js
//互为alternate
currentFiber.alternate === workInProgressFiber;
workInProgressFiber.alternate === currentFiber;
```

`React`应用的根节点**通过使`current`指针在不同`Fiber树`的`rootFiber`间切换**来完成`current Fiber`树指向的切换。即当`workInProgress Fiber树`构建完成交给`Renderer`渲染在页面上后，应用根节点的`current`指针指向`workInProgress Fiber树`，此时`workInProgress Fiber树`就变为`current Fiber树`。**每次状态更新都会产生新的`workInProgress Fiber树`，通过`current`与`workInProgress`的替换，完成`DOM`更新。**挂载和更新的最大区别在这里体现为挂载的时候`rootFiber`没有`子Fiber节点`，也**不会进行Diff操作**。

## 挂载

考虑案例：

```jsx
function App() {
  const [num, add] = useState(0);
  return (
    <p onClick={() => add(num + 1)}>{num}</p>
  )
}

ReactDOM.render(<App/>, document.getElementById('root'));
```

### 创建FiberRootNode & rootFiber

首次执行`ReactDOM.render`会创建`fiberRootNode`（源码中叫`fiberRoot`）和`rootFiber`。其中`fiberRootNode`是整个应用的根节点，`rootFiber`是`<App/>`所在组件树的根节点。之所以要区分`fiberRootNode`与`rootFiber`，是因为***在应用中我们可以多次调用`ReactDOM.render`渲染不同的组件树，他们会拥有不同的`rootFiber`。但是整个应用的根节点只有一个，那就是`fiberRootNode`。***`fiberRootNode`的`current`会指向当前页面上已渲染内容对应`Fiber树`，即`current Fiber树`。

![image-20220130175530578](https://github.com/NoAlligator/pico/blob/main/img/202203271805850.png?raw=true)

```javascript
fiberRootNode.current = rootFiber;
```

由于是首屏渲染，页面中还没有挂载任何`DOM`，所以**`fiberRootNode.current`指向的`rootFiber`没有任何`子Fiber节点`**（即`current Fiber树`为空）。

#### 进入render阶段，构建workInProgress树

接下来进入`render阶段`，根据组件返回的`JSX`在内存中依次创建`Fiber节点`并连接在一起构建`Fiber树`，被称为`workInProgress Fiber树`。（下图中右侧为内存中构建的树，左侧为页面显示的树）。在构建`workInProgress Fiber树`时会尝试复用`current Fiber树`中已有的`Fiber节点`内的属性，在`首屏渲染`时只有`rootFiber`存在对应的`current fiber`（即`rootFiber.alternate`）。

![image-20220130175805014](https://github.com/NoAlligator/pico/blob/main/img/202203271805851.png?raw=true)

#### 构建完成，渲染

图中右侧已构建完的`workInProgress Fiber树`在`commit阶段`渲染到页面。此时`DOM`更新为右侧树对应的样子。`fiberRootNode`的`current`指针指向`workInProgress Fiber树`使其变为`current Fiber 树`。

![image-20220130175847702](https://github.com/NoAlligator/pico/blob/main/img/202203271805852.png?raw=true)



## 更新

#### 构建workInProgress树

接下来我们点击`p节点`触发状态改变，这会开启一次新的`render阶段`并构建一棵新的`workInProgress Fiber 树`。和`mount`时一样，`workInProgress fiber`的创建可以复用`current Fiber树`对应的节点数据。这个决定是否复用的过程就是Diff算法。

![image-20220130180238227](https://github.com/NoAlligator/pico/blob/main/img/202203271805853.png?raw=true)

#### 构建完成，渲染

`workInProgress Fiber 树`在`render阶段`完成构建后进入`commit阶段`渲染到页面上。**渲染完毕后**，`workInProgress Fiber 树`变为`current Fiber 树`。

![image-20220130180414333](https://github.com/NoAlligator/pico/blob/main/img/202203271805854.png?raw=true)

## 总结

- `Reconciler`工作的阶段被称为`render`阶段。因为**在该阶段会调用组件的`render`方法**。
- `Renderer`工作的阶段被称为`commit`阶段。`commit`阶段会把`render`阶段提交的信息渲染在页面上。
- `render`与`commit`阶段统称为`work`，即`React`在工作中。相对应的，如果任务正在`Scheduler`内调度，就不属于`work`。

# 源码的文件结构

## 重要文件夹

```
根目录
├── fixtures        # 包含一些给贡献者准备的小型 React 测试项目
├── packages        # 包含元数据（比如 package.json）和 React 仓库中所有 package 的源码（子目录 src）
├── scripts         # 各种工具链的脚本，比如git、jest、eslint等
```

## packages文件夹

### react文件夹

React的核心，包含所有全局 React API，如：

- React.createElement
- React.Component
- React.Children

这些 API 是全平台通用的，它不包含`ReactDOM`、`ReactNative`等平台特定的代码。在 NPM 上作为[单独的一个包 (opens new window)](https://www.npmjs.com/package/react)发布。

### scheduler文件夹

Scheduler（调度器）的实现。

### shared文件夹

源码中其他模块公用的**方法**和**全局变量**，比如在[shared/ReactSymbols.js (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/shared/ReactSymbols.js)中保存`React`不同组件类型的定义。

```jsx
// ...
export let REACT_ELEMENT_TYPE = 0xeac7;
export let REACT_PORTAL_TYPE = 0xeaca;
export let REACT_FRAGMENT_TYPE = 0xeacb;
// ...
```

## Renderer文件夹

#### 渲染包

如下几个文件夹为对应的**Renderer**

```text
- react-art
- react-dom                 # 注意这同时是DOM和SSR（服务端渲染）的入口
- react-native-renderer
- react-noop-renderer       # 用于debug fiber（后面会介绍fiber）
- react-test-renderer
```

#### 试验性包的文件夹

`React`将自己流程中的一部分抽离出来，形成可以独立使用的包，由于他们是试验性质的，所以不被建议在生产环境使用。包括如下文件夹：

```
- react-server        # 创建自定义SSR流
- react-client        # 创建自定义的流
- react-fetch         # 用于数据请求
- react-interactions  # 用于测试交互相关的内部特性，比如React的事件模型
- react-reconciler    # Reconciler的实现，你可以用他构建自己的Renderer
```

#### 辅助包

`React`将一些辅助功能形成单独的包。包括如下文件夹：

```
- react-is       # 用于测试组件是否是某类型
- react-client   # 创建自定义的流
- react-fetch    # 用于数据请求
- react-refresh  # “热重载”的React官方实现
```

### [react-reconciler (opens new window)](https://github.com/facebook/react/tree/master/packages/react-reconciler)文件夹

我们需要重点关注**react-reconciler**，在接下来源码学习中 **80%**的代码量都来自这个包。虽然他是一个实验性的包，内部的很多功能在正式版本中还未开放。但是他一边对接**Scheduler**，一边对接不同平台的**Renderer**，构成了整个 React16 的架构体系。

# JSX

## JSX 与 React 语法的转换

### React17下的JSX新特性

在浏览器中无法直接使用 JSX，所以大多数 React 开发者需依靠 Babel 或 TypeScript 来**将 JSX 代码转换为 JavaScript**。在React17中，已经***不需要显式导入React了***，全新的转换，你可以**单独使用 JSX 而无需引入 React**。**旧的 JSX 转换**会把 JSX 转换为 `React.createElement(...)` 调用。新的 JSX 转换**不会将 JSX 转换为 `React.createElement`**，而是自动从 React 的 package 中引入新的入口函数并调用。现在Babel源代码**无需引入 React** 即可使用 JSX（但仍需引入 React，以便使用 React 提供的 Hook 或其他导出。）

### JSX非只面向浏览器环境

JSX并不是只能被编译为`React.createElement`方法，你可以通过[@babel/plugin-transform-react-jsx (opens new window)](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)插件显式告诉`Babel`编译时需要将`JSX`编译为什么函数的调用（默认为`React.createElement`）。

## `React.createElement() `的原理

仅就React17以下版本而言，babel会将JSX直接转换成`React.createElement()`。

### `React.createElement()`函数调用

`React.createElement(type: ReactComponent, config, children) => ReactElement`

`ReactElement`方法尾调用`ReactElement()`返回一个React Element，该对象有个参数`$$typeof: REACT_ELEMENT_TYPE`标记了该对象是个React Element。在React中，所有JSX在运行时的返回结果（即`React.createElement()`的返回值）都是React Element。

```javascript
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

该函数返回的对象一定是`React Element`，因为`React`提供了验证合法`React Element`的全局API [React.isValidElement ](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react/src/ReactElement.js#L547)，满足`React Element`的条件就是合法对象 + 存在`$$typeof`属性且值为 `REACT_ELEMENT_TYPE`。

```javascript
export function isValidElement(object) {
    return (
        typeof object === 'object' &&
        object !== null &&
        object.$$typeof === REACT_ELEMENT_TYPE
    );
}
```

### React Component  VS.  React Element

`React.createElement()`的第一个参数`type`指的是`React Component`（如果是`function component`，`type`对应的是`function`本身；如果是`class component`，`type`对应的就是该`class`），调用`React.createElement()`返回`React Element`。

```jsx
//以下代码用于比较 class/function 两种形式的 React Component 和 React Element
class AppClass extends React.Component {
    render() {
        return <p>KaSong</p>
    }
}
console.log('这是ClassComponent：', AppClass);	  //返回类本身
console.log('这是Element：', <AppClass/>);		  //返回元素对象


function AppFunc() {
    return <p>KaSong</p>;
}
console.log('这是FunctionComponent：', AppFunc); //返回函数本身
console.log('这是Element：', <AppFunc/>);		  //返回元素对象
```

```jsx
// 类式React Element
{   
    type: ƒ AppClass() {}
    key: null
    ref: null
    props: Object
    _owner: null
    _store: Object
}
```

```jsx
// 函数式React Element
{
    type: ƒ AppFunc() {}
    key: null
    ref: null
    props: Object
    _owner: null
    _store: Object
}
```

#### React如何判断组件类型

由于类式组件和函数式组件都满足：

```js
AppClass instanceof Function === true;
AppFunc instanceof Function === true;
```

所以无法通过引用类型区分`ClassComponent`和`FunctionComponent`。`React`通过`ClassComponent`实例原型上的`isReactComponent`变量判断是否是`ClassComponent`。

```js
ClassComponent.prototype.isReactComponent = {};
```

## `JSX`与`Fiber`节点

`JSX`是一种描述当前组件内容的数据结构（静态），他不包含组件**schedule**、**reconcile**、**render**所需的相关信息，比如如下信息就不包括在`JSX`中：

- 组件在更新中的`优先级`
- 组件的`state`
- 组件被打上的用于**Renderer**的`标记`

这些内容都包含在`Fiber节点`中。所以，**在组件`mount`时，`Reconciler`根据`JSX`描述的组件内容生成组件对应的`Fiber节点`。在`update`时，`Reconciler`将`JSX`与`Fiber节点`保存的数据对比，生成组件对应的`Fiber节点`，并根据对比结果为`Fiber节点`打上`标记`。**

# render阶段详解

## 作用

render阶段执行的工作主要创建（更新）Fiber节点并构建（更新）Fiber树。

## 主要函数

`render阶段`开始于`performSyncWorkOnRoot`或`performConcurrentWorkOnRoot`方法的调用。这取决于本次更新是**同步更新还是异步更新**。

```javascript
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    // workInProgress代表当前已创建的workInProgress fiber。
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
    //  performUnitOfWork方法会创建下一个Fiber节点并赋值给workInProgress，并将workInProgress与已创建的Fiber节点连接起来构成Fiber树。
  }
}

//  workLoopSync 和 workLoopConcurrent的唯一的区别是是否调用shouldYield
//  workLoopConcurrent代表当前的更新是受到 调度器 控制的（可中断的递归）
//  如果当前浏览器帧没有剩余时间，shouldYield会 中止 循环，直到浏览器有 空闲时间 后再继续遍历。
```

`Fiber Reconciler`是从`Stack Reconciler`重构而来，通过遍历的方式实现**可中断的递归**，所以`performUnitOfWork`的工作可以**分为两部分：“递”和“归”**。

## “递”阶段

“递”阶段首先从`rootFiber`开始向下深度优先遍历。为遍历到的每个`Fiber节点`调用[beginWork方法 (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3058)。该方法会**根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来**。当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。

## “归”阶段

在“归”阶段会调用[completeWork (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L652)处理`Fiber节点`（根据`Fiber节点`类型来创建`DOM`元素或者跳过，并挂载到父节点上）。当某个`Fiber节点`执行完`completeWork`，如果**其存在`兄弟Fiber节点`（即`fiber.sibling !== null`），会进入其`兄弟Fiber`的“递”阶段。**如果不存在`兄弟Fiber`，会进入`父级Fiber`的“归”阶段。“递”和“归”阶段会交错执行直到“归”到`rootFiber`。至此，`render阶段`的工作就结束了。

## 举例

```jsx
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

对应的`Fiber树`结构：

![image-20220131162134971](https://github.com/NoAlligator/pico/blob/main/img/202203271805855.png?raw=true)

`render阶段`会依次执行：

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

> 注：之所以没有 “KaSong” Fiber 的 beginWork/completeWork，是因为作为一种性能优化手段，针对只有单一文本子节点的`Fiber`，`React`会特殊处理。

# render / “递” 阶段

## beginWork

在源码中，“递”阶段执行的方法就是`beginWork()`，该方法会**根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来**。当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。

```typescript
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  // ...省略函数体
}
```

其中传参：

- current：当前组件对应的`Fiber节点`在上一次更新时的`Fiber节点`，即`workInProgress.alternate`;
- workInProgress：当前组件对应的`Fiber节点`;
- renderLanes：优先级相关。

从[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html)我们知道，除[`rootFiber`](https://react.iamkasong.com/process/doubleBuffer.html#mount时)以外， 组件`mount`时，由于是首次渲染，是**不存在**当前组件对应的`Fiber节点`在上一次更新时的`Fiber节点`，即`mount`时`current === null`。组件`update`时，由于之前已经`mount`过，所以`current !== null`。

> 所以我们可以通过`current === null ?`来区分组件是处于`mount`还是`update`。

基于此原因，`beginWork`的工作可以**分为两部分**：

- `update`时：如果`current`存在，在满足一定条件时可以复用`current`节点，这样就能克隆`current.child`作为`workInProgress.child`，而不需要新建`workInProgress.child`。
- `mount`时：除`fiberRootNode`以外，`current === null`。会根据`fiber.tag`不同，创建不同类型的`子Fiber节点`。

```typescript
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

### update时

我们可以看到，满足如下情况时`didReceiveUpdate === false`（即可以直接复用前一次更新的`子Fiber`，不需要新建`子Fiber`）

1. `oldProps === newProps && workInProgress.type === current.type`，即`props`与`fiber.type`不变；
2. `!includesSomeLane(renderLanes, updateLanes)`，即当前`Fiber节点`优先级不够。

```typescript
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

### mount时

当不满足优化路径时，我们就进入第二部分，新建`子Fiber`。我们可以看到，根据`fiber.tag`不同，进入不同类型`Fiber`的创建逻辑。

> 可以从[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactWorkTags.js)看到`tag`对应的组件类型

```typescript
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

对于我们常见的组件类型，如（`FunctionComponent`/`ClassComponent`/`HostComponent`），最终会进入[reconcileChildren (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L233)方法。

### reconcileChildren()方法

这是`Reconciler`模块的核心部分。他做了以下事情：

- 对于`mount`的组件，他会创建新的`子Fiber节点`；
- 对于`update`的组件，他会将当前组件与该组件在上次更新时对应的`Fiber节点`比较（也就是俗称的`Diff`算法），将比较的结果生成新`Fiber节点`。

```typescript
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

从代码可以看出，和`beginWork`一样，他也是通过`current === null ?`区分`mount`与`update`。不论走哪个逻辑，最终他会*<u>生成新的子`Fiber节点`并赋值给`workInProgress.child`</u>*，作为本次`beginWork`[返回值 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L1158)，并作为下次`performUnitOfWork`执行时`workInProgress`的[传参 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1702)。

> 值得一提的是，`mountChildFibers`与`reconcileChildFibers`这两个方法的**逻辑基本一致**。唯一的区别是：`reconcileChildFibers`会为生成的`Fiber节点`**带上`effectTag`属性**，而`mountChildFibers`不会。

### effectTag

我们知道，`render阶段`的工作是在内存中进行，当工作结束后会通知`Renderer`需要执行的`DOM`操作。要执行`DOM`操作的具体类型就保存在`fiber.effectTag`中。

> 你可以从[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js)看到`effectTag`对应的`DOM`操作

比如：

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

> 通过二进制掩码表示`effectTag`，可以方便的使用位操作为`fiber.effectTag`赋值多个`effect`。

那么，如果要*<u>通知`Renderer`将`Fiber节点`对应的`DOM节点`插入页面中</u>*，需要满足两个条件：

1. `fiber.stateNode`存在，即`Fiber节点`中保存了对应的`DOM节点`
2. `(fiber.effectTag & Placement) !== 0`，即`Fiber节点`存在`Placement effectTag`

⭐我们知道，`mount`时，`fiber.stateNode === null`，且在`reconcileChildren`中调用的`mountChildFibers`不会为`Fiber节点`赋值`effectTag`。那么首屏渲染如何完成呢？

`fiber.stateNode`会在`completeWork`中创建。假设`mountChildFibers`也会赋值`effectTag`，那么可以预见`mount`时整棵`Fiber树`所有节点都会有`Placement effectTag`。那么`commit阶段`在执行`DOM`操作时每个节点都会执行一次插入操作，这样大量的`DOM`操作是极低效的。为了解决这个问题，<u>*在`mount`时只有`rootFiber`会赋值`Placement effectTag`，在`commit阶段`只会执行一次插入操作。*</u>

### beginWork流程图

![image-20220202100928506](https://github.com/NoAlligator/pico/blob/main/img/image-20220202100928506.png)

## compeleteWork

类似`beginWork`，`completeWork`也是针对不同`fiber.tag`调用不同的处理逻辑。

```ts
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
    }
```

我们重点关注页面渲染所必须的`HostComponent`（即原生`DOM组件`对应的`Fiber节点`）。

### 处理HostComponent

和`beginWork`一样，我们根据`current === null ?`判断是`mount`还是`update`。同时针对`HostComponent`，判断`update`时我们还需要考虑`workInProgress.stateNode != null `（即该`Fiber节点`是否存在对应的`DOM节点`）

```jsx
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

### update时

当`update`时，`Fiber节点`已经存在对应`DOM节点`，所以不需要生成`DOM节点`。需要做的主要是<u>*处理`props`*</u>，比如：

- `onClick`、`onChange`等**回调函数的注册**；
- 处理`style prop`；
- 处理`children prop`；
- 处理`DANGEROUSLY_SET_INNER_HTML prop`。

我们去掉一些当前不需要关注的功能（比如`ref`）。可以看到最主要的逻辑是调用`updateHostComponent`方法。

```typescript
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

在`updateHostComponent`内部，被处理完的`props`会被赋值给`workInProgress.updateQueue`，并最终会在`commit阶段`被渲染在页面上。其中`updatePayload`为数组形式，他的偶数索引的值为变化的`prop key`，奇数索引的值为变化的`prop value`。

### mount时

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

原因就在于`completeWork`中的`appendAllChildren`方法。由于`completeWork`属于“归”阶段调用的函数，每次调用`appendAllChildren`时都会将已生成的子孙`DOM节点`插入当前生成的`DOM节点`下。那么当“归”到`rootFiber`时，我们已经有一个构建好的离屏`DOM树`（内存中）。

### effectList

作为`DOM`操作的依据，`commit阶段`需要找到所有有`effectTag`的`Fiber节点`并依次执行`effectTag`对应操作。难道需要在`commit阶段`再遍历一次`Fiber树`寻找`effectTag !== null`的`Fiber节点`么？

这显然是很低效的。为了解决这个问题，在`completeWork`的上层函数`completeUnitOfWork`中，每个执行完`completeWork`且存在`effectTag`的`Fiber节点`会被**保存在一条被称为`effectList`的单向链表中**。`effectList`中第一个`Fiber节点`保存在`fiber.firstEffect`，最后一个元素保存在`fiber.lastEffect`。类似`appendAllChildren`，在“归”阶段，所有有`effectTag`的`Fiber节点`都会被追加在`effectList`中，最终形成一条以`rootFiber.firstEffect`为起点的单向链表：

```js
                       nextEffect         nextEffect
rootFiber.firstEffect -----------> fiber -----------> fiber
```

这样，在`commit阶段`只需要遍历`effectList`就能执行所有`effect`了。`effectList`相较于`Fiber树`，就像圣诞树上挂的那一串彩灯。

#### 链表演示代码：

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

### completeWork流程图

![img](https://github.com/NoAlligator/pico/blob/main/img/completeWork.png)

# commit阶段

`commitRoot`方法是`commit阶段`工作的起点。`fiberRootNode`会作为传参。

```js
commitRoot(root);
```

在`rootFiber.firstEffect`上保存了一条需要执行`副作用`的`Fiber节点`的单向链表`effectList`，这些`Fiber节点`的`updateQueue`中保存了变化的`props`。这些`副作用`对应的`DOM操作`在`commit`阶段执行。除此之外，一些生命周期钩子（比如`componentDidXXX`）、`hook`（比如`useEffect`）需要在`commit`阶段执行。

`commit`阶段的主要工作（即`Renderer`的工作流程）分为三部分：

- before mutation阶段（执行`DOM`操作前）
- mutation阶段（执行`DOM`操作）
- layout阶段（执行`DOM`操作后）

在`before mutation阶段`之前和`layout阶段`之后还有一些额外工作，涉及到比如`useEffect`的触发、`优先级相关`的重置、`ref`的绑定/解绑。

### 一、beforeMutation

`before mutation阶段`的代码很短，整个过程就是遍历`effectList`并调用`commitBeforeMutationEffects`函数处理。

```typescript
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

我们重点关注`beforeMutation`阶段的主函数`commitBeforeMutationEffects`做了什么。

#### commitBeforeMutationEffects

```jsx
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

整体可以分为三部分：

1. 处理`DOM节点`渲染/删除后的 `autoFocus`、`blur` 逻辑。
2. 调用`getSnapshotBeforeUpdate`生命周期钩子。
3. 调度`useEffect`。

##### 调用getSnapshotBeforeUpdate

`commitBeforeMutationEffectOnFiber`是`commitBeforeMutationLifeCycles`的别名。在该方法内会调用`getSnapshotBeforeUpdate`。从`React`v16开始，`componentWillXXX`钩子前增加了`UNSAFE_`前缀。究其原因，是因为`Stack Reconciler`重构为`Fiber Reconciler`后，`render阶段`的任务可能**中断/重新**开始，对应的组件在`render阶段`的生命周期钩子（即`componentWillXXX`）可能触发多次。为此，`React`提供了替代的生命周期钩子`getSnapshotBeforeUpdate`。我们可以看见，<u>*`getSnapshotBeforeUpdate`是在`commit阶段`内的`before mutation阶段`调用的，由于`commit阶段`是同步的，所以不会遇到多次调用的问题。*</u>

##### 调度useEffect

在这几行代码内，`scheduleCallback`方法由`Scheduler`模块提供，用于以某个优先级异步调度一个回调函数。

```jsx
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

在此处，被异步调度的回调函数就是触发`useEffect`的方法`flushPassiveEffects`。我们接下来讨论`useEffect`<u>*如何被异步调度，以及为什么要异步（而不是同步）调度*</u>。

##### 如何异步调度

在`flushPassiveEffects`方法内部会从全局变量`rootWithPendingPassiveEffects`获取`effectList`。关于`flushPassiveEffects`的具体讲解参照[useEffect与useLayoutEffect一节](https://react.iamkasong.com/hooks/useeffect.html)。在[completeWork一节](https://react.iamkasong.com/process/completeWork.html#effectlist)我们讲到，`effectList`中保存了需要执行副作用的`Fiber节点`。其中副作用包括：

- 插入`DOM节点`（Placement）
- 更新`DOM节点`（Update）
- 删除`DOM节点`（Deletion）

除此外，当一个`FunctionComponent`含有`useEffect`或`useLayoutEffect`，他对应的`Fiber节点`也会被赋值`effectTag`。在`flushPassiveEffects`方法内部会遍历`rootWithPendingPassiveEffects`（即`effectList`）执行`effect`回调函数。如果在此时直接执行，`rootWithPendingPassiveEffects === null`。

那么`rootWithPendingPassiveEffects`会在何时赋值呢？在上一节`layout之后`的代码片段中会根据`rootDoesHavePassiveEffects === true`决定是否赋值`rootWithPendingPassiveEffects`。

```jsx
const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;
if (rootDoesHavePassiveEffects) {
    rootDoesHavePassiveEffects = false;
    rootWithPendingPassiveEffects = root;
    pendingPassiveEffectsLanes = lanes;
    pendingPassiveEffectsRenderPriority = renderPriorityLevel;
}
```

所以整个`useEffect`异步调用分为三步：

1. `before mutation阶段`在`scheduleCallback`中调度`flushPassiveEffects`；
2. `layout阶段`之后将`effectList`赋值给`rootWithPendingPassiveEffects`；
3. `scheduleCallback`触发`flushPassiveEffects`，`flushPassiveEffects`内部遍历`rootWithPendingPassiveEffects`。

##### 为什么异步调用

> **与 componentDidMount、componentDidUpdate 不同**的是，在浏览器完成布局与绘制之后，传给 useEffect 的函数会延迟调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因此不应在函数中执行阻塞浏览器更新屏幕的操作。

`useEffect`异步执行的原因主要是防止同步执行时阻塞浏览器渲染。

#### 总结

在`before mutation阶段`，会遍历`effectList`，依次执行：

1. 处理`DOM节点`渲染/删除后的 `autoFocus`、`blur`逻辑
2. 调用`getSnapshotBeforeUpdate`生命周期钩子
3. 调度`useEffect`

### 二、mutation

类似`before mutation阶段`，`mutation阶段`也是遍历`effectList`，执行函数。这里执行的是`commitMutationEffects`。

```jsx
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

#### commitMutationEffects

代码如下：

```jsx
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

`commitMutationEffects`会遍历`effectList`，对每个`Fiber节点`执行如下三个操作：

1. 根据`ContentReset effectTag`重置文字节点；
2. 更新`ref`；
3. 根据`effectTag`分别处理，其中`effectTag`包括(`Placement` | `Update` | `Deletion` | `Hydrating`)。

我们关注步骤三中的`Placement` | `Update` | `Deletion`。`Hydrating`作为服务端渲染相关，我们先不关注。

#### Placement effect

当`Fiber节点`含有`Placement effectTag`，意味着该`Fiber节点`对应的`DOM节点`需要插入到页面中。调用的方法为`commitPlacement`。

该方法所做的工作分为三步：

1.获取父级`DOM节点`。其中`finishedWork`为传入的`Fiber节点`。

```js
const parentFiber = getHostParentFiber(finishedWork);
// 父级DOM节点
const parentStateNode = parentFiber.stateNode;
```

2.获取`Fiber节点`的`DOM`兄弟节点

```js
const before = getHostSibling(finishedWork);
```

3.根据`DOM`兄弟节点是否存在决定调用`parentNode.insertBefore`或`parentNode.appendChild`执行`DOM`插入操作。

```js
// parentStateNode是否是rootFiber
if (isContainer) {
  insertOrAppendPlacementNodeIntoContainer(finishedWork, before, parent);
} else {
  insertOrAppendPlacementNode(finishedWork, before, parent);
}
```

值得注意的是，`getHostSibling`（获取兄弟`DOM节点`）的执行很耗时，**当在同一个父`Fiber节点`下依次执行多个插入操作，`getHostSibling`算法的复杂度为指数级。**这是由于`Fiber节点`不只包括`HostComponent`，所以`Fiber树`和渲染的`DOM树`节点并不是一一对应的。要从`Fiber节点`找到`DOM节点`很可能跨层级遍历。

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
rootFiber -----> App -----> div -----> Item -----> li

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
fiberP.sibling.child
```

即`fiber p`的`兄弟fiber` `Item`的`子fiber` `li`。

#### Update effect

当`Fiber节点`含有`Update effectTag`，意味着**该`Fiber节点`需要更新**。调用的方法为`commitWork`，他会根据`Fiber.tag`分别处理。这里我们主要关注`FunctionComponent`和`HostComponent`。

##### FunctionalComponent Mutation

当`fiber.tag`为`FunctionComponent`，会调用`commitHookEffectListUnmount`。该方法会遍历`effectList`，执行所有`useLayoutEffect hook`的销毁函数。

所谓“销毁函数”，见如下例子：

```js
useLayoutEffect(() => {
  // ...一些副作用逻辑

  return () => {
    // ...这就是销毁函数
  }
})
```

##### HostComponent Mutation

当`fiber.tag`为`HostComponent`，会调用`commitUpdate`。最终会在[`updateDOMProperties`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-dom/src/client/ReactDOMComponent.js#L378)中将[`render阶段 completeWork`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L229)中为`Fiber节点`赋值的`updateQueue`对应的内容渲染在页面上。

```jsx
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

#### Deletion effect

当`Fiber节点`含有`Deletion effectTag`，意味着该`Fiber节点`对应的`DOM节点`需要从页面中删除。调用的方法为`commitDeletion`。

该方法会执行如下操作：

1. 递归调用`Fiber节点`及其子孙`Fiber节点`中`fiber.tag`为`ClassComponent`的[`componentWillUnmount`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L920)生命周期钩子，从页面移除`Fiber节点`对应`DOM节点`
2. 解绑`ref`
3. 调度`useEffect`的销毁函数

#### 总结

`mutation阶段`会遍历`effectList`，依次执行`commitMutationEffects`。该方法的主要工作为“根据`effectTag`调用不同的处理函数处理`Fiber`。

### 三、layout

`layout阶段`也是遍历`effectList`，执行函数。具体执行的函数是`commitLayoutEffects`。

```jsx
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

#### commitLayoutEffects

```jsx
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

#### commitLayoutEffectOnFiber(commitLifeCycles)

`commitLayoutEffectOnFiber`方法会根据`fiber.tag`对不同类型的节点分别处理。

⭐对于`ClassComponent`，他会通过`current === null?`区分是`mount`还是`update`，调用[`componentDidMount` (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L538)或[`componentDidUpdate` (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L592)。

触发`状态更新`的`this.setState`如果赋值了第二个参数`回调函数`，也会在此时调用。

```js
this.setState({ xxx: 1 }, () => {
    console.log("i am update~");
});
```

⭐对于`FunctionComponent`及相关类型，他会调用`useLayoutEffect hook`的`回调函数`，调度`useEffect`的`销毁`与`回调`函数。

> 相关类型`指特殊处理后的`FunctionComponent`，比如`ForwardRef`、`React.memo`包裹的`FunctionComponent

```js
  switch (finishedWork.tag) {
    // 以下都是FunctionComponent及相关类型
    case FunctionComponent:
    case ForwardRef:
    case SimpleMemoComponent:
    case Block: {
      // 执行useLayoutEffect的回调函数
      commitHookEffectListMount(HookLayout | HookHasEffect, finishedWork);
      // 调度useEffect的销毁函数与回调函数
      schedulePassiveEffects(finishedWork);
      return;
    }
```

**在上一节介绍[Update effect](https://react.iamkasong.com/renderer/mutation.html#update-effect)时介绍过，`mutation阶段`会执行`useLayoutEffect hook`的`销毁函数`。结合这里我们可以发现，`useLayoutEffect hook`从上一次更新的`销毁函数`调用到本次更新的`回调函数`调用是同步执行的。而`useEffect`则需要先调度，在`Layout阶段`完成后再异步执行。这就是`useLayoutEffect`与`useEffect`的区别。**

⭐对于`HostRoot`，即`rootFiber`，如果赋值了第三个参数`回调函数`，也会在此时调用。

```jsx
ReactDOM.render(<App />, document.querySelector("#root"), function() {
    console.log("i am mount~");
});
```

#### commitAttachRef

```jsx
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

代码逻辑很简单：获取`DOM`实例，更新`ref`。

#### current Fiber树切换

至此，整个`layout阶段`就结束了。

我们关注下这行代码：

```js
root.current = finishedWork;
```

在[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html#什么是-双缓存)我们介绍过，`workInProgress Fiber树`在`commit阶段`完成渲染后会变为`current Fiber树`。这行代码的作用就是切换`fiberRootNode`指向的`current Fiber树`。

那么这行代码为什么在这里呢？（在`mutation阶段`结束后，`layout阶段`开始前。）

我们知道`componentWillUnmount`会在`mutation阶段`执行。此时`current Fiber树`还指向前一次更新的`Fiber树`，在生命周期钩子内获取的`DOM`还是更新前的。`componentDidMount`和`componentDidUpdate`会在`layout阶段`执行。此时`current Fiber树`已经指向更新后的`Fiber树`，在生命周期钩子内获取的`DOM`就是更新后的。

#### 总结

`layout阶段`会遍历`effectList`，依次执行`commitLayoutEffects`。该方法的主要工作为“根据`effectTag`调用不同的处理函数处理`Fiber`并更新`ref`。

## 阶段 & 操作

![image-20220202112809451](https://github.com/NoAlligator/pico/blob/main/img/202203271805856.png)

# Diff算法

## 单点Diff

对于单个节点，我们以类型`object`为例，会进入`reconcileSingleElement`

```jsx
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

这个函数会做如下事情：![diff](https://react.iamkasong.com/img/diff.png)

让我们看看第二步**判断DOM节点是否可以复用**是如何实现的：

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

还记得我们刚才提到的，React预设的限制么，从代码可以看出，React通过先判断`key`是否相同，如果`key`相同则判断`type`是否相同，只有都相同时一个`DOM节点`才能复用。

这里有个细节需要关注下：

- 当`child !== null`且`key相同`且`type不同`时执行`deleteRemainingChildren`将`child`及其兄弟`fiber`都标记删除。
- 当`child !== null`且`key不同`时仅将`child`标记删除。

为什么`type`（暗示了此时`key`相同）不同要将兄弟节点都删除？

考虑如下例子：

当前页面有3个`li`，我们要全部删除，再插入一个`p`。

```jsx
// 当前页面显示的
ul > li * 3

// 这次需要更新的
ul > p
```

由于本次更新时只有一个`p`，属于单一节点的`Diff`，会走上面介绍的代码逻辑。

在`reconcileSingleElement`中遍历之前的3个`fiber`（对应的`DOM`为3个`li`），寻找本次更新的`p`是否可以复用之前的3个`fiber`中某个的`DOM`。当`key相同`且`type不同`时，代表我们已经找到本次更新的`p`对应的上次的`fiber`，但是`p`与`li` `type`不同，不能复用。既然唯一的可能性已经不能复用，则剩下的`fiber`都没有机会了，所以都需要标记删除。当`key不同`时只代表遍历到的该`fiber`不能被`p`复用，后面还有兄弟`fiber`还没有遍历到，所以**仅仅标记该`fiber`删除。**

## 多点Diff

该如何设计算法呢？如果让我设计一个`Diff算法`，我首先想到的方案是：

1. 判断当前节点的更新属于哪种情况；
2. 如果是`新增`，执行新增逻辑；
3. 如果是`删除`，执行删除逻辑；
4. 如果是`更新`，执行更新逻辑；

按这个方案，其实有个隐含的前提——**不同操作的优先级是相同的**

但是`React团队`发现，在日常开发中，相较于`新增`和`删除`，`更新`组件发生的频率更高。所以`Diff`会优先判断当前节点是否属于`更新`。

> **⚠注意**
>
> 在我们做数组相关的算法题时，经常使用**双指针**从数组头和尾同时遍历以提高效率，但是这里却不行：
>
> 虽然本次更新的`JSX对象` `newChildren`为数组形式，但是和`newChildren`中每个组件进行比较的是`current fiber`，同级的`Fiber节点`是由`sibling`指针链接形成的**单链表**，即不支持双指针遍历，即 `newChildren[0]`与`fiber`比较，`newChildren[1]`与`fiber.sibling`比较，所以无法使用**双指针**优化。

基于以上原因，`Diff算法`的整体逻辑会经历两轮遍历：

第一轮遍历：处理`更新`的节点。

第二轮遍历：处理剩下的不属于`更新`的节点。

### 第一轮遍历

第一轮遍历步骤如下：

1. `let i = 0`，遍历`newChildren`，将`newChildren[i]`与`oldFiber`比较，判断`DOM节点`是否可复用；
2. 如果可复用，`i++`，继续比较`newChildren[i]`与`oldFiber.sibling`，可以复用则继续遍历；
3. 如果不可复用，分两种情况：
   1. **`key`不同**导致不可复用，立即跳出整个遍历，**第一轮遍历结束**；
   2. **`key`相同`type`不同**导致不可复用，会将`oldFiber`标记为`DELETION`，并继续遍历。
4. 如果`newChildren`遍历完（即`i === newChildren.length - 1`）或者`oldFiber`遍历完（即`oldFiber.sibling === null`），跳出遍历，**第一轮遍历结束**。

当遍历结束后，会有两种结果：

### 步骤3跳出的遍历（key diff）

此时`newChildren`没有遍历完，`oldFiber`也没有遍历完。举个例子，考虑如下代码：

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

第一个节点可复用，遍历到`key === 2`的节点发现`key`改变，不可复用，跳出遍历，等待第二轮遍历处理。

此时`oldFiber`剩下`key === 1`、`key === 2`未遍历，`newChildren`剩下`key === 2`、`key === 1`未遍历。

### 步骤4跳出的遍历

可能`newChildren`遍历完，或`oldFiber`遍历完，或他们同时遍历完。举个例子，考虑如下代码：

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

带着第一轮遍历的结果，我们开始第二轮遍历。

### 第二轮遍历

对于第一轮遍历的结果，我们分别讨论：

#### `newChildren`与`oldFiber`同时遍历完

那就是最理想的情况：只需在第一轮遍历进行组件[`更新`](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L825)。此时`Diff`结束。

#### `newChildren`没遍历完，`oldFiber`遍历完

已有的`DOM节点`都复用了，这时还有新加入的节点，意味着本次更新有新节点插入，我们只需要遍历剩下的`newChildren`为生成的`workInProgress fiber`依次标记`Placement`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L869)看到这段源码逻辑

#### `newChildren`遍历完，`oldFiber`没遍历完

意味着本次更新比之前的节点数量少，有节点被删除了。所以需要遍历剩下的`oldFiber`，依次标记`Deletion`。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L863)看到这段源码逻辑

#### `newChildren`与`oldFiber`都没遍历完

这意味着有节点在这次更新中改变了位置。

这是`Diff算法`最精髓也是最难懂的部分。我们接下来会重点讲解。

> 你可以在[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L893)看到这段源码逻辑

#### 处理移动的节点

由于有节点改变了位置，所以不能再用位置索引`i`对比前后的节点，那么如何才能将同一个节点在两次更新中对应上呢？

我们需要使用`key`。

为了快速的找到`key`对应的`oldFiber`，我们将所有还未处理的`oldFiber`存入以`key`为key，`oldFiber`为value的`Map`中。

```javascript
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
```

> 你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactChildFiber.new.js#L890)看到这段源码逻辑

接下来遍历剩余的`newChildren`，通过`newChildren[i].key`就能在`existingChildren`中找到`key`相同的`oldFiber`。

#### 标记节点是否移动

既然我们的目标是寻找移动的节点，那么我们需要明确：

节点是否移动是以什么为参照物？

我们的参照物是：最后一个可复用的节点在`oldFiber`中的位置索引（用变量`lastPlacedIndex`表示）。

由于本次更新中节点是按`newChildren`的顺序排列。在遍历`newChildren`过程中，每个`遍历到的可复用节点`一定是当前遍历到的`所有可复用节点`中**最靠右的那个**，即一定在`lastPlacedIndex`对应的`可复用的节点`在本次更新中位置的后面。

那么我们只需要比较`遍历到的可复用节点`在上次更新时是否也在`lastPlacedIndex`对应的`oldFiber`后面，就能知道两次更新中这两个节点的相对位置改变没有。

我们用变量`oldIndex`表示`遍历到的可复用节点`在`oldFiber`中的位置索引。如果`oldIndex < lastPlacedIndex`，代表本次更新该节点需要向右移动。

`lastPlacedIndex`初始为`0`，每遍历一个可复用的节点，如果`oldIndex >= lastPlacedIndex`，则`lastPlacedIndex = oldIndex`。

## Demo1

在Demo中我们简化下书写，每个字母代表一个节点，字母的值代表节点的`key`

```jsx
// 之前
abcd

// 之后
acdb

===第一轮遍历开始===
a（之后）vs a（之前）  
key不变，可复用
此时 a 对应的oldFiber（之前的a）在之前的数组（abcd）中索引为0
所以 lastPlacedIndex = 0;

继续第一轮遍历...

c（之后）vs b（之前）  
key改变，不能复用，跳出第一轮遍历
此时 lastPlacedIndex === 0;
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === cdb，没用完，不需要执行删除旧节点
oldFiber === bcd，没用完，不需要执行插入新节点

将剩余oldFiber（bcd）保存为map

// 当前oldFiber：bcd
// 当前newChildren：cdb

继续遍历剩余newChildren

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index;
此时 oldIndex === 2;  // 之前节点为 abcd，所以c.index === 2
比较 oldIndex 与 lastPlacedIndex;

如果 oldIndex >= lastPlacedIndex 代表该可复用节点不需要移动
并将 lastPlacedIndex = oldIndex;
如果 oldIndex < lastplacedIndex 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右移动

在例子中，oldIndex 2 > lastPlacedIndex 0，
则 lastPlacedIndex = 2;
c节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：bd
// 当前newChildren：db

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
oldIndex 3 > lastPlacedIndex 2 // 之前节点为 abcd，所以d.index === 3
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：b
// 当前newChildren：b

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index;
oldIndex 1 < lastPlacedIndex 3 // 之前节点为 abcd，所以b.index === 1
则 b节点需要向右移动
===第二轮遍历结束===

最终acd 3个节点都没有移动，b节点被标记为移动
```

## Demo2

```jsx
// 之前
abcd

// 之后
dabc

===第一轮遍历开始===
d（之后）vs a（之前）  
key改变，不能复用，跳出遍历
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === dabc，没用完，不需要执行删除旧节点
oldFiber === abcd，没用完，不需要执行插入新节点

将剩余oldFiber（abcd）保存为map

继续遍历剩余newChildren

// 当前oldFiber：abcd
// 当前newChildren dabc

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
此时 oldIndex === 3; // 之前节点为 abcd，所以d.index === 3
比较 oldIndex 与 lastPlacedIndex;
oldIndex 3 > lastPlacedIndex 0
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：abc
// 当前newChildren abc

key === a 在 oldFiber中存在
const oldIndex = a（之前）.index; // 之前节点为 abcd，所以a.index === 0
此时 oldIndex === 0;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 0 < lastPlacedIndex 3
则 a节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：bc
// 当前newChildren bc

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index; // 之前节点为 abcd，所以b.index === 1
此时 oldIndex === 1;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 1 < lastPlacedIndex 3
则 b节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：c
// 当前newChildren c

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index; // 之前节点为 abcd，所以c.index === 2
此时 oldIndex === 2;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 2 < lastPlacedIndex 3
则 c节点需要向右移动

===第二轮遍历结束===
```

可以看到，我们以为从 `abcd` 变为 `dabc`，只需要将`d`移动到前面。但实际上React保持`d`不变，将`abc`分别移动到了`d`的后面。**从这点可以看出，考虑性能，我们要尽量减少将节点从后面移动到前面的操作。**

# 状态更新

## 流程概览

### render阶段的开始

`render阶段`开始于`performSyncWorkOnRoot`或`performConcurrentWorkOnRoot`方法的调用。这取决于本次更新是同步更新还是异步更新。

### commit阶段的开始

`commit阶段`开始于`commitRoot`方法的调用。其中`rootFiber`会作为传参，`render阶段`完成后会进入`commit阶段`。

```sh
触发状态更新（根据场景调用不同方法）

    |
    |
    v

    ？

    |
    |
    v

render阶段（`performSyncWorkOnRoot` 或 `performConcurrentWorkOnRoot`）

    |
    |
    v

commit阶段（`commitRoot`）
```

### 创建Update对象

在`React`中，有如下方法可以触发状态更新（排除`SSR`相关）：

- ReactDOM.render
- this.setState
- this.forceUpdate
- useState
- useReducer

#### 这些方法调用的场景各不相同，他们是如何接入同一套**状态更新机制**呢？

每次`状态更新`都会创建一个保存**更新状态相关内容**的对象，我们叫他`Update`。在`render阶段`的`beginWork`中会根据`Update`计算新的`state`。

### 从Fiber到Root

现在`触发状态更新的fiber`上已经包含`Update`对象。我们知道，`render阶段`是从`rootFiber`开始向下遍历。那么如何从`触发状态更新的fiber`得到`rootFiber`呢？

答案是：调用`markUpdateLaneFromFiberToRoot`方法。该方法做的工作可以概括为：从`触发状态更新的fiber`一直向上遍历到`rootFiber`，并返回`rootFiber`。由于不同更新优先级不尽相同，所以过程中还会更新遍历到的`fiber`的优先级。

### 调度更新

现在我们拥有一个`rootFiber`，该`rootFiber`对应的`Fiber树`中某个`Fiber节点`包含一个`Update`。接下来通知`Scheduler`根据**更新**的优先级，决定以**同步**还是**异步**的方式调度本次更新。这里调用的方法是`ensureRootIsScheduled`。

以下是`ensureRootIsScheduled`最核心的一段代码：

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

其中，`scheduleCallback`和`scheduleSyncCallback`会调用`Scheduler`提供的调度方法根据`优先级`调度回调函数执行。

可以看到，这里调度的回调函数为：

```js
performSyncWorkOnRoot.bind(null, root);
performConcurrentWorkOnRoot.bind(null, root);
```

即`render阶段`的入口函数。至此，`状态更新`就和我们所熟知的`render阶段`连接上了。

### 总结

梳理下`状态更新`的整个调用路径的关键节点：

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

## 心智模型

### 同步更新的心智模型

我们可以将`更新机制`类比`代码版本控制`。在没有`代码版本控制`前，我们在代码中逐步叠加功能。一切看起来井然有序，直到我们遇到了一个紧急线上bug（红色节点）。为了修复这个bug，我们需要首先将之前的代码提交。在`React`中，所有通过`ReactDOM.render`创建的应用（其他创建应用的方式参考[ReactDOM.render一节](https://react.iamkasong.com/state/reactdom.html#react的其他入口函数)）都是通过类似的方式`更新状态`。即没有`优先级`概念，`高优更新`（红色节点）需要排在其他`更新`后面执行。![流程1](https://react.iamkasong.com/img/git1.png)

### 并发更新得到心智模型

当有了`代码版本控制`，有紧急线上bug需要修复时，我们暂存当前分支的修改，在`master分支`修复bug并紧急上线。

![流程2](https://react.iamkasong.com/img/git2.png)bug修复上线后通过`git rebase`命令和`开发分支`连接上。`开发分支`基于`修复bug的版本`继续开发。

![流程3](https://react.iamkasong.com/img/git3.png)

在`React`中，通过`ReactDOM.createBlockingRoot`和`ReactDOM.createRoot`创建的应用会采用`并发`的方式`更新状态`。`高优更新`（红色节点）中断正在进行中的`低优更新`（蓝色节点），先完成`render - commit流程`。待`高优更新`完成后，**`低优更新`基于`高优更新`的结果`重新更新`**。

## Update对象

> `状态更新`流程开始后首先会`创建Update对象`，可以将`Update`类比`心智模型`中的一次`commit`。

### 分类

我们将可以触发更新的方法所隶属的组件分类：

- ReactDOM.render —— HostRoot
- this.setState —— ClassComponent
- this.forceUpdate —— ClassComponent
- useState —— FunctionComponent
- useReducer —— FunctionComponent

可以看到，一共三种组件（`HostRoot` | `ClassComponent` | `FunctionComponent`）可以触发更新。由于**不同类型组件工作方式不同，所以存在两种不同结构的`Update`**，其中`ClassComponent`与`HostRoot`共用一套`Update`结构，`FunctionComponent`单独使用一种`Update`结构。虽然他们的结构不同，但是他们工作机制与工作流程大体相同。

### 结构

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

字段意义如下：

- eventTime：任务时间，通过`performance.now()`获取的毫秒数。由于该字段在未来会重构，当前我们不需要理解他。
- lane：优先级相关字段。当前还不需要掌握他，只需要知道不同`Update`优先级可能是不同的。

> 你可以将`lane`类比`心智模型`中`需求的紧急程度`。

- suspenseConfig：`Suspense`相关，暂不关注。
- tag：更新的类型，包括`UpdateState` | `ReplaceState` | `ForceUpdate` | `CaptureUpdate`。
- payload：更新挂载的数据，不同类型组件挂载的数据不同。对于`ClassComponent`，`payload`为`this.setState`的第一个传参。对于`HostRoot`，`payload`为`ReactDOM.render`的第一个传参。
- callback：更新的回调函数。即在[commit 阶段的 layout 子阶段一节](https://react.iamkasong.com/renderer/layout.html#commitlayouteffectonfiber)中提到的`回调函数`。
- next：与其他`Update`连接形成链表。

### Update & Fiber

#### `Update`存在一个连接其他`Update`形成链表的字段`next`。联系`React`中另一种以链表形式组成的结构`Fiber`，他们之间有什么关联么？

答案是肯定的。从[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html)我们知道，`Fiber节点`组成`Fiber树`，页面中最多同时存在两棵`Fiber树`：

- 代表当前页面状态的`current Fiber树`
- 代表正在`render阶段`的`workInProgress Fiber树`

类似`Fiber节点`组成`Fiber树`，`Fiber节点`上的多个`Update`会组成链表并被包含在`fiber.updateQueue`中。

> 什么情况下一个Fiber节点会存在多个Update？
>
> 你可能疑惑为什么一个`Fiber节点`会存在多个`Update`。这其实是很常见的情况。
>
> 在这里介绍一种最简单的情况：
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
> 在一个`ClassComponent`中触发`this.onClick`方法，方法内部调用了两次`this.setState`。这会在该`fiber`中产生两个`Update`。

#### `Fiber节点`最多同时存在两个`updateQueue`：

- `current fiber`保存的`updateQueue`即`current updateQueue`
- `workInProgress fiber`保存的`updateQueue`即`workInProgress updateQueue`

在`commit阶段`完成页面渲染后，`workInProgress Fiber树`变为`current Fiber树`，`workInProgress Fiber树`内`Fiber节点`的`updateQueue`就变成`current updateQueue`。

### Update Queue

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

字段说明如下：

- baseState：本次更新前该`Fiber节点`的`state`，`Update`基于该`state`计算更新后的`state`。

> 你可以将`baseState`类比`心智模型`中的`master分支`。

- `firstBaseUpdate`与`lastBaseUpdate`：本次更新前该`Fiber节点`已保存的`Update`。以链表形式存在，链表头为`firstBaseUpdate`，链表尾为`lastBaseUpdate`。之所以在更新产生前该`Fiber节点`内就存在`Update`，是由于某些`Update`优先级较低所以在上次`render阶段`由`Update`计算`state`时被跳过。

> 你可以将`baseUpdate`类比`心智模型`中执行`git rebase`基于的`commit`（节点D）。

- `shared.pending`：触发更新时，产生的`Update`会保存在`shared.pending`中形成单向环状链表。当由`Update`计算`state`时这个环会被剪开并连接在`lastBaseUpdate`后面。

> 你可以将`shared.pending`类比`心智模型`中本次需要提交的`commit`（节点ABC）。

- effects：数组。保存`update.callback !== null`的`Update`。

### 例子

`updateQueue`相关代码逻辑涉及到大量链表操作，比较难懂。在此我们举例对`updateQueue`的工作流程讲解下。假设有一个`fiber`刚经历`commit阶段`完成渲染。该`fiber`上有两个由于优先级过低所以在上次的`render阶段`并没有处理的`Update`。他们会成为下次更新的`baseUpdate`。

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

`shared.pending` 会保证始终指向最后一个插入的`update`，你可以在[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.new.js#L208)看到`enqueueUpdate`的源码

更新调度完成后进入`render阶段`。

此时`shared.pending`的环被剪开并连接在`updateQueue.lastBaseUpdate`后面：

```js
fiber.updateQueue.baseUpdate: u1 --> u2 --> u3 --> u4
```

接下来遍历`updateQueue.baseUpdate`链表，以`fiber.updateQueue.baseState`为`初始state`，依次与遍历到的每个`Update`计算并产生新的`state`（该操作类比`Array.prototype.reduce`）。

在遍历时如果有优先级低的`Update`会被跳过。

当遍历完成后获得的`state`，就是该`Fiber节点`在本次更新的`state`（源码中叫做`memoizedState`）。

> `render阶段`的`Update操作`由`processUpdateQueue`完成，你可以从[这里 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactUpdateQueue.new.js#L405)看到`processUpdateQueue`的源码

`state`的变化在`render阶段`产生与上次更新不同的`JSX`对象，通过`Diff算法`产生`effectTag`，在`commit阶段`渲染在页面上。

渲染完成后`workInProgress Fiber树`变为`current Fiber树`，整个更新流程结束。

## 优先级

`状态更新`由`用户交互`产生，用户心里对`交互`执行顺序有个预期。`React`根据`人机交互研究的结果`中用户对`交互`的预期顺序为`交互`产生的`状态更新`赋予不同优先级。

具体如下：

- 生命周期方法：同步执行。
- 受控的用户输入：比如输入框内输入文字，同步执行。
- 交互事件：比如动画，高优先级执行。
- 其他：比如数据请求，低优先级执行。

### 调度优先级

我们在[新的React结构一节](https://react.iamkasong.com/preparation/newConstructure.html)讲到，`React`通过`Scheduler`调度任务。具体到代码，每当需要调度任务时，`React`会调用`Scheduler`提供的方法`runWithPriority`。该方法接收一个`优先级`常量与一个`回调函数`作为参数。`回调函数`会以`优先级`高低为顺序排列在一个`定时器`中并在合适的时间触发。对于更新来讲，传递的`回调函数`一般为[状态更新流程概览一节](https://react.iamkasong.com/state/prepare.html#render阶段的开始)讲到的`render阶段的入口函数`。

### 例子

![优先级如何决定更新的顺序](https://react.iamkasong.com/img/update-process.png)

在这个例子中，有两个`Update`。我们将“关闭黑夜模式”产生的`Update`称为`u1`，输入字母“I”产生的`Update`称为`u2`。其中`u1`先触发并进入`render阶段`。其优先级较低，执行时间较长。此时：

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

其中`u2`优先级高于`u1`。接下来进入`u2`产生的`render阶段`。在`processUpdateQueue`方法中，`shared.pending`环状链表会被剪开并拼接在`baseUpdate`后面。需要明确一点，`shared.pending`指向最后一个`pending`的`update`，所以实际执行时`update`的顺序为：

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

***我们可以看见，`u2`对应的更新执行了两次，相应的`render阶段`的生命周期勾子`componentWillXXX`也会触发两次。这也是为什么这些勾子会被标记为`unsafe_`。***

### 如何保证`Update`不丢失？

在[上一节例子](https://react.iamkasong.com/state/update.html#例子)中我们讲到，在`render阶段`，`shared.pending`的环被剪开并连接在`updateQueue.lastBaseUpdate`后面。

实际上`shared.pending`会被同时连接在`workInProgress updateQueue.lastBaseUpdate`与`current updateQueue.lastBaseUpdate`后面。

当`render阶段`被中断后重新开始时，会基于`current updateQueue`克隆出`workInProgress updateQueue`。由于`current updateQueue.lastBaseUpdate`已经保存了上一次的`Update`，所以不会丢失。

当`commit阶段`完成渲染，由于`workInProgress updateQueue.lastBaseUpdate`中保存了上一次的`Update`，所以 `workInProgress Fiber树`变成`current Fiber树`后也不会造成`Update`丢失。

### 如何保证状态依赖的连续性？

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

通过以上例子我们可以发现，`React`保证最终的状态一定和用户触发的`交互`一致，但是中间过程`状态`可能由于设备不同而不同。

## React.render

### 创建fiber

从[双缓存机制一节](https://react.iamkasong.com/process/doubleBuffer.html#mount时)我们知道，首次执行`ReactDOM.render`会创建`fiberRootNode`和`rootFiber`。其中`fiberRootNode`是整个应用的根节点，`rootFiber`是要渲染组件所在组件树的`根节点`。这一步发生在调用`ReactDOM.render`后进入的`legacyRenderSubtreeIntoContainer`方法中：

```js
// container指ReactDOM.render的第二个参数（即应用挂载的DOM节点）
root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
  container,
  forceHydrate,
);
fiberRoot = root._internalRoot;
```

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

### 创建update

我们已经做好了组件的初始化工作，接下来就等待创建`Update`来开启一次更新。这一步发生在`updateContainer`方法中：

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

值得注意的是其中`update.payload = {element};`这就是我们在[Update一节](https://react.iamkasong.com/state/update.html#update的结构)介绍的，对于`HostRoot`，`payload`为`ReactDOM.render`的第一个传参（JSX）。

### 流程概览

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

## this.setState

### 流程概览

可以看到，`this.setState`内会调用`this.updater.enqueueSetState`方法。

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

这里值得注意的是对于`ClassComponent`，`update.payload`为`this.setState`的第一个传参（即要改变的`state`）。

## this.forceUpdate

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

#### 赋值`update.tag = ForceUpdate;`有何作用呢？

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

- checkHasForceUpdateAfterProcessing：内部会判断本次更新的`Update`是否为`ForceUpdate`。即如果本次更新的`Update`中存在`tag`为`ForceUpdate`，则返回`true`。
- checkShouldComponentUpdate：内部会调用`shouldComponentUpdate`方法。以及当该`ClassComponent`为`PureComponent`时会浅比较`state`与`props`。

所以，当某次更新含有`tag`为`ForceUpdate`的`Update`，那么当前`ClassComponent`不会受其他`性能优化手段`（`shouldComponentUpdate`|`PureComponent`）影响，一定会更新。

# Hooks

## 极简Hooks模拟

使用案例

```jsx
import { useState } frfom 'react'
function App() {
	const [count, setCount] = useState(0)
    const handleClickUpdate = () => setCount
    return (
    	<Fragment>
        	<h1>{count}</h1>
            <button onClick={handleClickUpdate}>Update</button>
        </Fragment>
    )
}
```

更新对象由每一次setCount形成，被挂载在hook对象的queue上形成环状链表：

```javascript
const update = {
	//setCount需要执行的函数
    action,
    //指向同一个Hook的下一个更新（形成环状链表）
    next: null
}
```

调用更新，解释了update对象是如何形成的：

```js
function dispatchAction(queue, action) {
	//创建每一次调用setCount形成的update对象
    const update = {
        action,
        next: null
    }
    //没有形成环，是第一次进行更新
    if(queue.pending === null) {
		update.next = update
    } else {
        //针对该状态已经有了setCount更新操作被加入queue中形成环
        update.next = queue.pending.next
        queue.pending.next = update
    }
    //将queue指向最新的一次更新，这样使得queue.pending.next指向的必定是最早的的一次更新
    queue.pending = update
    //模拟React开始调度更新
    schedule()
}
```

函数式组件的queue保存在其对应的的Fiber节点对象中，其基本结构：

```js
const fiber =  {
    // 保存该函数式组件对应的Hooks链表（无环）
	memorizedState: null,
    stateNode: f App()
}
```

函数式组件对应的Fiber内部memorizedState形成了Hooks链表，Hook的基本结构如下：

```js
let hook = {
    //保存update对象的queue，最终会形成环状链表
    queue: {
        pending: null
    },
    //保存hook对应的状态
    memorizedState: intialState,
    //与下一个Hook连接形成单向无环链表
    next: null
}
```

在通过dispatchAction方法末尾通过schedule方法模拟React调度更新流程：

```js
function dispatchAction(queue, action) {
  // ...创建update
  
  // ...环状单向链表操作

  // 模拟React开始调度更新
  schedule();
}
```

schedule方法的模拟实现：

```javascript
//首次render是mount
isMount = true
function schedule() {
    // 更新前将workInProgress重置为fiber保存得到第一个Hook
	workInProgressHook = fiber.memorizedState
    // 触发组件render
    fiber.stateNode()
	//在首次执行render后，后续的render操作由mount变为update
    isMount = false
}
```

实现useState方法：

```js
function useState(initialState) {
    /*---根据mount/update调用 -> 生成hook对象/从workInProgress取出对应hook对象---*/
    
    //创建hook对象
    let hook
    if(isMount) {
		// mount时需要生成hook对象
        hook = {
            queue: {
                pending: null
            },
            memorizedState: initialState,
            next: null
        }
        // 将hook插入fiber.memoizedState链表末尾
        if(!fiber.memorizedState) {
            fiber.memorizedState = hook
        } else {
            workInProgressHook.next = hook;
        }
    } else {
		// update时从workInProgressHook中取出该useState对应的hook
        // update时找到对应hook
        hook = workInProgressHook;
        // 移动workInProgressHook指针
        workInProgressHook = workInProgressHook.next;
    }
    
    /* ------------执行update-------------- */
    let baseState = hook.memorizedState
    if(hook.queue.pending) {
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
    /* ------------更新state-------------- */
    hook.memorizedState = baseState
    return [baseState, dispatchAction.bind(null, hook.queue)]
}
```

### 与真实Hook实现的区别

1. 没有使用`isMount`变量：`React Hooks`没有使用`isMount`变量，而是在不同时机使用不同的`dispatcher`。换言之，`mount`时的`useState`与`update`时的`useState`不是同一个函数。
2. 跳过更新：`React Hooks`有中途跳过`更新`的优化手段。
3. `React Hooks`有`batchedUpdates`，当在`click`中触发三次`updateNum`，`精简React`会触发三次更新，而`React`只会触发一次。
4. `React Hooks`的`update`有`优先级`概念，可以跳过不高优先的`update`。

## 真实Hooks

### dispatcher

在React源码中，Hooks的实现是：组件mount / update两种情况下使用的hook函数的来源是**不同的对象**，这类对象在源码中被称为diapatcher。[dispatcher源码🔗](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1775)

### 区分mount | update

在`FunctionComponent` `render`前，会根据`FunctionComponent`对应`fiber`的以下条件区分`mount`与`update`:

```js
current === null || current.memoizedState === null
```

并将不同情况对应的`dispatcher`赋值给全局变量`ReactCurrentDispatcher`的`current`属性：



源码中的ContextOnlyDispatcher解读，useState的回调会自动被绑定到该
