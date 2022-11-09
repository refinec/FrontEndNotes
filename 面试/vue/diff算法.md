# vue2 双端diff算法

## 什么是虚拟DOM？

> 虚拟DOM就是一个JS对象，包含的字段有**标签名称**、**标签属性**、**子标签的名称**、**子标签属性**、**文本节点**

![虚拟DOM](../../assets/%E9%9D%A2%E8%AF%95/image-20221108193025809.png)

## key有什么作用？

> **diff 算法的目的是根据 key 复用 dom 节点，通过移动节点而不是创建新节点来减少 dom 操作。**

## diff 算法干什么用？

> 用于对比**新旧**两个虚拟DOM，找出差异，最小化更新视图。
>
> diff的过程就是调用名为`patch`的函数，比较新旧节点，一边比较一边给**真实的DOM**打补丁。

diff 算法约定了两种处理原则：**<u>只做同层的对比，type 变了就不再对比子节点</u>。**

![img](../../assets/%E9%9D%A2%E8%AF%95/diff.png)

![image-20221108194206202](../../assets/%E9%9D%A2%E8%AF%95/image-20221108194206202.png)

## **双端diff算法**(能减少节点的移动次数)比对流程

1. 需要 4 个指针，分别指向**新旧**两个 `vnode` 数组的首尾。
2. 先`oldS`和`newS`比（首首）、再`oldE`和`newE`比(尾尾)、接着`oldS`和`newE`比(首尾)、然后`oldE`和`newS`比(尾首)
3. 找到了，就进行`patchVnode`。
4. 如果没找到，就遍历`oldVNode`,找出与`newS`拥有相同的`key`值的元素，进行 `patchVnode`
5. 头和尾的指针向中间移动，直到 `oldStartIdx > oldEndIdx && newStartIdx <= newEndIdx` 或者 `newStartIdx > newEndIdx && oldStartIdx <= oldEndIdx `，说明就处理完了全部的节点。
6. 还剩下旧的 `vnode` 就批量删除，剩下新的 `vnode` 就批量新增

![image-20221108194430868](../../assets/%E9%9D%A2%E8%AF%95/image-20221108194430868.png)

[**`updateChildren`方法源码**：](https://github.com/vuejs/vue/blob/bd6cea0973247e2a8e1d4a2250614c0bf44f0b26/src/core/vdom/patch.js#L404-L474)

```js
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh)
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) { // 首首
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) { // 尾尾
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // 首尾，Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { //尾首， Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        // 在oldCh中遍历找出与newStartVnode相同的元素
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }

function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}

function sameInputType (a, b) {
  if (a.tag !== 'input') return true
  let i
  const typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type
  const typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type
  return typeA === typeB || isTextInputType(typeA) && isTextInputType(typeB)
}

function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    if (oldVnode === vnode) {
      return
    }

    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // clone reused vnode
      vnode = ownerArray[index] = cloneVNode(vnode)
    }

    const elm = vnode.elm = oldVnode.elm

    if (isTrue(oldVnode.isAsyncPlaceholder)) {
      if (isDef(vnode.asyncFactory.resolved)) {
        hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
      } else {
        vnode.isAsyncPlaceholder = true
      }
      return
    }

    // reuse element for static trees.
    // note we only do this if the vnode is cloned -
    // if the new node is not cloned it means the render functions have been
    // reset by the hot-reload-api and we need to do a proper re-render.
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance
      return
    }

    let i
    const data = vnode.data
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode)
    }

    const oldCh = oldVnode.children
    const ch = vnode.children
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }
    if (isUndef(vnode.text)) {
      if (isDef(oldCh) && isDef(ch)) {
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } else if (isDef(ch)) {
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch)
        }
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
      } else if (isDef(oldCh)) {
        removeVnodes(oldCh, 0, oldCh.length - 1)
      } else if (isDef(oldVnode.text)) {
        nodeOps.setTextContent(elm, '')
      }
    } else if (oldVnode.text !== vnode.text) {
      nodeOps.setTextContent(elm, vnode.text)
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
    }
  }
```

## 总结

1. diff算法用于对比新旧两个虚拟DOM，找出差异，最小化的更新视图。

2. 再对比的过程中，只做同层的对比，节点的type(类型)变了，那就不再对比子节点

3. 如果节点相同，那就对比子节点，对比的时候，需要 4 个指针，分别指向**新旧**两个 `vnode` 数组的首尾

   1. 先`oldS`和`newS`比（首首）、再`oldE`和`newE`比(尾尾)、接着`oldS`和`newE`比(首尾)、然后`oldE`和`newS`比(尾首)

   2. 找到了，就进行`patchVnode`。

   3. 如果没找到，就遍历`oldVNode`,找出与`newS`拥有相同的`key`值(`key`没有的话，就对比`tag`)的元素，再进行 `patchVnode`

      > `patchVnode`方法内的对比是：
      >
      > 1. `newVNode`和`oldVNode`都有文本节点，那么就用**新文本替换旧文本**
      > 2. `newVNode`有子节点，但`oldVNode`没有子节点，那么增加新的子节点
      > 3. `newVNode`没有子节点，但`oldVNode`有子节点，那么删除旧的子节点
      > 4. `newVNode`和`oldVNode`都有子节点，那么调用`updateChildren`方法继续比较(`updateChildren`方法内包含`patchVnode`，`patchVnode`方法内也包含`updateChildren`)

4. 对比过的头和尾指针要向中间移动，直到 `oldStartIdx > oldEndIdx && newStartIdx <= newEndIdx` 或者 `newStartIdx > newEndIdx && oldStartIdx <= oldEndIdx `，说明就处理完了全部的节点。

5. 还剩下旧的 `vnode` 就批量删除，剩下新的 `vnode` 就批量新增

# vue3 快速diff算法

## 最长递增子序列

> vue3的diff算法用到了**最长递增子序列算法**

求解最长递增子序列 [leetcode300](https://leetcode.cn/problems/longest-increasing-subsequence/)给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

> 子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。


示例 1：

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

示例 2：

```
输入：nums = [0,1,0,3,2,3]
输出：4
```

示例 3：

```
输入：nums = [7,7,7,7,7,7,7]
输出：1
```

### **动态规划：O(n²)**

> 定义：`dp[i]`代表以`num[i]`结尾的最长子序列的长度
>
> - 双层遍历：对比`num[i]`和`num[j]`之前的数据
> - 当`num[j]`< `num[i]`时，`num[i]`就可以拼接在`num[j]`后，此时`num[i]`位置的上升子序列长度为：`dp[i]+1`
> - 当`num[j]` > `num[i]`时，`num[i]`和`num[j]`无法构成上升子序列，跳过
> - 计算选出`dp[i]`中最大的值即为计算结果

```ts
function lengthOfLIS(nums: number[]): number {
    const len: number = nums.length;
    if (len <= 1) return len;
    const dp: number[] = new Array(len).fill(1);
    for(let i = 0; i < len; i++) {
        for(let j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[j] + 1, dp[i]);
            }
        }
    }
    return Math.max(...dp);
};
```

![最长递增子序列](../../assets/%E9%9D%A2%E8%AF%95/lengthOfLIS.png)

### 贪心 + 二分查找：O(nlogn)

> 要使上升子序列的长度尽可能的长，就要使序列上升的速度尽可能的慢，因此需要让序列内末尾数字尽可能的小。
> 我们可以维护一个result数组，用来存放单调递增序列结果，然后依次遍历nums数组；
>
> 如果`nums[i] > result[len]`, 则直接插入到result末尾
> 否则，在result数组中通过二分查找的方式，找到第一个比`nums[i]`大的值`result[j]`;并更新`result[j] = nums[i]`

```ts
function lengthOfLIS(nums: number[]): number {
  const n:number = nums.length
  if (n <=1 ) return n;
  let result:number[] = [nums[0]]
  let len = result.length	// 最大长度
  for (let i = 1; i < n; i++) {
    if (nums[i] > result[len-1]) { //大于末尾的值， 直接近栈
      result.push(nums[i]) 
      ++len
    } else {
      let left = 0, right = len;
      while(left < right) { // 二分查找序列内第一个大于nums[i]的值
        const mid = (left + right) >> 1
        if (result[mid] < nums[i]) {
          left = mid + 1
        } else {
          right = mid
        }
      }
      result[left] = nums[i] // 替换
    }
  }
  return len
}
```

![最长递增子序列](../../assets/%E9%9D%A2%E8%AF%95/lengthOfLIS2.png)

注意：这个方案中的result得到的长度是正确的，但是顺序并不一定是正确结果需要的顺序，比如`[10, 9, 2, 5, 3, 7, 1, 18]`得到的result为`[1, 3, 7, 18]`

**那么为什么贪心算法可以得到正确的长度呢？**

要想得到最长上升子序列的正确长度，首先必须保证result内存放的数值增速尽可能稳和慢，所以要使用增长空间大、有潜力的值来组合；

比如1,50,5,……当我们遍历到50的时候，并不知道后面是否还有值，此时先将数据放入栈中存起来是明智的，继续往后遍历遇到了5，显然选用1,5比选用1,50更让人放心也更有潜力，因为后面的数再往栈内存放的几率更大，即使后面没有更多值了，那么选用1,5还是1,50其实最后长度是一样的。

那如果使用了更小的值，已经在栈内的值应该如何处理呢？比如我们栈中存放了1,3,9,10,再往后遍历的时候遇到了5，显然5比9,10都更有潜力，如果将栈直接变成1,3,5又不太可能，因为如果后面没有更多值了，长度由4变成3，结果是错误的；但如果不去管5的话，后面又碰到了 6,7,8那不就JJ了；

所以我们可以考虑既不能放弃有潜力的值，也不能错失正确的长度结果，因此我们不妨鱼和熊掌都兼得一下，比如将第一个大于5的值9替换掉变成1,3,5,10，这样在放弃栈内容顺序正确性的情况下保证了栈长度的正确性，接下来，再往后遍历会遇到3种情况：

后面没有更多值了，此时结果长度为4，是没问题的

如果后面遇到50，则可以直接插入到栈中，变为1,3,5,10,50，长度为5也是没问题的，因为我们并没有将最后的值替换掉，所以我们可以将栈想象成为9做了个替身5，真正的值还是替换前的1,3,9,10

如果后面遇到了6，则按照一开始的规则，将10替换掉变成1,3,5,6，长度为4也是没问题的，因为我们将最后的值都做了替换，所以此时替身5就变成了真身，同时我们也发现，得到的栈中的值就是最后的最优解

可以发现，在没有替换完栈中的值时，栈中被替换的的值，起到的是占位的效果，为后面遍历数字提供参照的作用；

**要想得到正确的序列，首先要对上面的代码做一些改动：**

- 将result修改为存储下标（最后回溯是会改成真正的值）；为下面的chain提供参考

- 增加chain变量，存放每一位在被加入到result时其对应的前一位的下标值，进行关系绑定
- 回溯chain，覆盖result的值。因为result内，最后一位一定是正确的，所以可以从后往前进行修正

上面我们说过在对栈内某个值进行替换后，变动的值后面的所有的值如果都没有变过的话，那么替换的值只是一个替身，无法作为最后结果进行输出，只有替换值后面的都变动过了，才会由替身变为真身。那么在没有全部替换前，我们是需要有一种方法去保存原来顺序的：

比如`3,5,7`，可以想象成`7->5->3`他们之间是强绑定，7前面绑定的永远都是5，5前面永远都是3

- 如果此时遇到了4，栈会变成`3,4,7`，5虽然变成了4，但是`7->5->3`这个绑定关系是不会变的

- 如果此时又遇到了15，栈变成了`3,4,7,15`，则绑定和回溯关系就变成了`15->7->5->3`

那么什么时候4能生效呢？那就是在4后面的值都被替换了，比如又遇到了6和8，则栈变为了`3，4，6，8`,绑定和回溯关系就变成了`8->6->4->3`

![最长递增子序列](../../assets/%E9%9D%A2%E8%AF%95/resize_w_500.png)

```ts
function getOfLIS(nums: number[]):number[]  {
  const n:number = nums.length
  if (n <=1 ) return nums;
  let result:number[] = [0]  // 由原来存储具体值改为存储下标
  let chain = new Map() // 通过下标存储映射关系
  for (let i = 0; i < n; i++) {
    const j = result[result.length - 1]
    if (nums[i] > nums[j]) {
      chain.set(i,{val: i, pre: j})
      result.push(i)
    } else {
      let left = 0, right = result.length;
      while(left < right) {
        const mid = (left + right) >> 1
        if (nums[result[mid]] < nums[i]) {
          left = mid + 1
        } else {
          right = mid
        }
      }
      chain.set(i,{val: i, pre: result[left - 1]})
      result[left] = i
    }
  }
  let preIdx = result[result.length - 1]
  let len = result.length
 // 从后往前进行回溯，修正覆盖result中的值，找到正确的顺序
  while(chain.get(preIdx)) {  
  	let lastObj = chain.get(preIdx)  
    result[--len] = nums[lastObj.val]
    preIdx = lastObj.pre
  }
  return result
}; 

const test= [9,2,5,3,7,101,4,18,1]
console.log(getOfLIS(test)); // [2,3,4,18]
```

## diff算法

> vue3中的diff和上面的思想其实是一样的，都是基于下标来绑定数字在被插入result内时和其前面一个数字的关系。但是它看起来会更加难以理解，因为它是通过数组（P）来绑定回溯关系的，返回的是最长递增子序列的下标值
>

```js
  function getSequence(arr) {
    const p = arr.slice() // 回溯专用
    const result = [0]
    let i, j, u, v, c
    const len = arr.length
    for (i = 0; i < len; i++) {
      const arrI = arr[i]
      // 排除了等于0的情况，原因是0并不代表任何dom元素，只是用来做占位的
      if (arrI !== 0) {
        j = result[result.length - 1]
        // 当前值大于子序列最后一项
        if (arr[j] < arrI) {
          // p内存储当前值的前一位下标
          p[i] = j
          // 存储当前值的下标
          result.push(i)
          continue
        }
        u = 0
        v = result.length - 1
        // 当前数值小于子序列最后一项时，使用二分法找到第一个大于当前数值的下标
        while (u < v) {
          c = ((u + v) / 2) | 0
          if (arr[result[c]] < arrI) {
            u = c + 1
          } else {
            v = c
          }
        }
        if (arrI < arr[result[u]]) {
          // 第一位不需要操作，一位它没有前一项
          if (u > 0) {
            // p内存储找到的下标的前一位
            p[i] = result[u - 1]
          }
          // 找到下标，直接替换result中的数值
          result[u] = i
        }
      }
    }
    u = result.length
    v = result[u - 1]
    // 回溯，从最后一位开始，将result全部覆盖，
    while (u-- > 0) {
      result[u] = v
      v = p[v]
    }
    return result
  }
```

