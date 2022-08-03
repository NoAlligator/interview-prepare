# beginWork

- mount的时候主要做React转换成Fiber节点并挂载到父fiber的工作
- update时会进行Diff来决定是否复用或者重新创建fiber并挂载到父fiber的工作

# completeWork

- mount的时候主要时间Fiber节点生成对应的DOM节点，并将子孙DOM节点插入到刚生成的DOM节点中，并处理props（updateQueue）、ref
- update的时候主要处理props（updateQueue）、ref

# beforeMutaion

- 遍历`effectList`并调用`commitBeforeMutationEffects`函数处理。
  - 处理`DOM节点`渲染/删除后的 `autoFocus`、`blur` 逻辑。
  - 调用`getSnapshotBeforeUpdate`生命周期钩子。
  - 调度`useEffect`。

# mutation

- `mutation阶段`也是遍历`effectList`，执行函数。这里执行的是`commitMutationEffects`。
  - 根据`ContentReset effectTag`重置文字节点
  - 更新`ref`
  - 根据`effectTag`分别处理，其中`effectTag`包括(`Placement` | `Update` | `Deletion` | `Hydrating`)

> 当`fiber.tag`为`FunctionComponent`，会调用`commitHookEffectListUnmount`。该方法会遍历`effectList`，执行所有`useLayoutEffect hook`的销毁函数。

# switch current to workInProgress

# layout

- `layout阶段`也是遍历`effectList`，执行函数。具体执行的函数是`commitLayoutEffects`。
  - commitLayoutEffectOnFiber（调用`生命周期钩子`和`hook`相关操作）
    - 对于`ClassComponent`，他会通过`current === null?`区分是`mount`还是`update`，调用[`componentDidMount`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L538)或[`componentDidUpdate`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCommitWork.new.js#L592)。
    - 对于`FunctionComponent`及相关类型，他会调用`useLayoutEffect hook`的`回调函数`，调度`useEffect`的`销毁`与`回调`函数
  - commitAttachRef（赋值 ref）
    - 获取`DOM`实例，更新`ref`。

# Diff

`React`的`diff`会预设三个限制：

1. 只对同级元素进行`Diff`。如果一个`DOM节点`在前后两次更新中跨越了层级，那么`React`不会尝试复用他。
2. 两个不同类型的元素会产生出不同的树。如果元素由`div`变为`p`，React会销毁`div`及其子孙节点，并新建`p`及其子孙节点。
3. 开发者可以通过 `key prop`来暗示哪些子元素在不同的渲染下能保持稳定。

# 单点Diff

- 当`child !== null`且`key相同`且`type不同`时执行`deleteRemainingChildren`将`child`及其兄弟`fiber`都标记删除。
- 当`child !== null`且`key不同`时仅将`child`标记删除，继续遍历。

# 多点Diff

- 第一轮遍历：处理`更新`的节点。
  - `let i = 0`，遍历`newChildren`，将`newChildren[i]`与`oldFiber`比较，判断`DOM节点`是否可复用。
  - 如果可复用，`i++`，继续比较`newChildren[i]`与`oldFiber.sibling`，可以复用则继续遍历。
  - 如果不可复用，分两种情况：
    - `key`不同导致不可复用，立即跳出整个遍历，**第一轮遍历结束。**
    - `key`相同`type`不同导致不可复用，会将`oldFiber`标记为`DELETION`，并继续遍历
  - 如果`newChildren`遍历完（即`i === newChildren.length - 1`）或者`oldFiber`遍历完（即`oldFiber.sibling === null`），跳出遍历，**第一轮遍历结束。**

- 第二轮遍历：处理剩下的不属于`更新`的节点。

# 更新

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

# Hook

> `hook`与`FunctionComponent fiber`都存在`memoizedState`属性，不要混淆他们的概念。
>
> - `fiber.memoizedState`：`FunctionComponent`对应`fiber`保存的`Hooks`链表。
> - `hook.memoizedState`：`Hooks`链表中保存的单一`hook`对应的数据。
> - `hook.queue`：`Hook`中的更新链表（单向环形链表）。

