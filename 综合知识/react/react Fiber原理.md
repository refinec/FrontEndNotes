## Fiber是什么？

React Fiber 是 React 16 引入的一种新的架构，旨在解决以前版本中更新机制的问题，特别是长时间更新过程中主线程被阻塞的问题。

Fiber 是一个 JavaScript 对象，代表 React 的一个工作单元，包含了与组件相关的信息（组件的类型、键、属性、状态、子元素、兄弟元素和父元素等）

```json
{
  type: 'h1', // 组件类型
  key: null, // React key
  props: { ... }, // 输入的props
  state: { ... }, // 组件的state
  child: Fiber | null, // 第一个子元素的Fiber
  sibling: Fiber | null, // 下一个兄弟元素的Fiber
  return: Fiber | null, // 父元素的Fiber
  // ...其他属性
}
```

这种结构使得 React 可以在处理任何 Fiber 之前判断是否有足够的时间完成该工作，并在必要时中断和恢复工作。

## Fiber解决了什么？作用

在React 16之前的版本中，是使用递归的方式处理组件树更新，称为**堆栈调和（Stack Reconciliation）**，这种方法一旦开始就不能中断，直到整个组件树都被遍历完。这种机制在处理大量数据或复杂视图时可能导致主线程被阻塞，从而使应用无法及时响应用户的输入或其他高优先级任务。

Fiber的引入改变了这一情况。Fiber可以理解为是React自定义的一个带有链接关系的DOM树，每个Fiber都代表了一个工作单元，React可以在处理任何Fiber之前判断是否有足够的时间完成该工作，并在必要时中断和恢复工作。

https://segmentfault.com/a/1190000044468085

## Fiber的工作原理

Fiber 的核心特点是可以中断和恢复，这增强了 React 的并发性和响应性。React 在更新时，会根据现有的 Fiber 树创建一个新的临时树（Work-in-progress Tree），并在后台进行更新。更新完成后，新的树会替换掉旧的树。

工作流程

1. **Reconciliation（调和）阶段**：确定哪些部分的 UI 需要更新。这个阶段会比较新的 props 和旧的 Fiber 树来确定哪些部分需要更新。

   调和阶段分为三个小阶段：

   1. **创建与标记更新节点**：判断 Fiber 节点是否要更新。
   2. **收集副作用列表**：遍历 Fiber 节点，同时记录有副作用节点的关系。
   3. **处理副作用**：根据标记的副作用类型，执行相应的操作

2. **Commit（提交）阶段**：更新 DOM 并执行任何副作用。这个阶段会遍历在调和阶段创建的副作用列表进行更新

   1. **遍历副作用列表**：处理节点删除和确认节点在 before mutation 阶段是否有要处理的副作用。
   2. **正式提交**：递归遍历 Fiber，更新副作用节点。
   3. **处理 layout effects**：处理由 useLayoutEffect 创建的 layout effects

React Fiber 通过引入调度器和双缓冲技术，使得 React 可以在处理大量数据或复杂视图时避免主线程被阻塞，从而提高应用的响应性和性能