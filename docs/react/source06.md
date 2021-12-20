
# 源码分析 diff算法

> Dom 渲染在commit阶段，而Dom渲染是由Fiber树决定如何更新，Fiber树的更新主要发生在render阶段，及beginWork那个流程。

### 1. reconcileChildFibers

> diff算法的入口

```js
 function reconcileChildFibers(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChild: any,
    lanes: Lanes,
  ): Fiber | null {
    const isUnkeyedTopLevelFragment =
      typeof newChild === 'object' &&
      newChild !== null &&
      newChild.type === REACT_FRAGMENT_TYPE &&
      newChild.key === null;
    if (isUnkeyedTopLevelFragment) {
      newChild = newChild.props.children;
    }

    // 单一节点处理
    if (typeof newChild === 'object' && newChild !== null) {
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
          return placeSingleChild(
            reconcileSingleElement(
              returnFiber,
              currentFirstChild,
              newChild,
              lanes,
            ),
          );
          // =====
      }

      // 多节点
      if (isArray(newChild)) {
        return reconcileChildrenArray(
          returnFiber,
          currentFirstChild,
          newChild,
          lanes,
        );
      }
      throwOnInvalidObjectType(returnFiber, newChild);
    }
  }
```

### 2. reconcileSingleElement 单节点

> 只有key相同和type相同，才能复用当前节点。

```js
  function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
     
  //child节点不为null执行对比
  while (child !== null) {
    // 1.比较key
    if (child.key === key) {
      // 2.比较type
      switch (child.tag) {
        //...
        default: {
          if (child.elementType === element.type) {
            // type相同则可以复用 返回复用的节点
            return existing;
          }
          // type不同跳出
          break;
        }
      }
      //key相同，type不同则把fiber及和兄弟fiber标记删除
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      //key不同直接标记删除该节点
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }
      
  //新建新Fiber
}
```

### 3.reconcileChildrenArray 多节点

> 第一轮遍历步骤如下：
>
> 1. `let i = 0`，遍历`newChildren`，将`newChildren[i]`与`oldFiber`比较，判断`DOM节点`是否可复用。
> 2. 如果可复用，`i++`，继续比较`newChildren[i]`与`oldFiber.sibling`，可以复用则继续遍历。
> 3. 如果不可复用，分两种情况：
>
> - `key`不同导致不可复用，立即跳出整个遍历，**第一轮遍历结束。**
> - `key`相同`type`不同导致不可复用，会将`oldFiber`标记为`DELETION`，并继续遍历
>
> 1. 如果`newChildren`遍历完（即`i === newChildren.length - 1`）或者`oldFiber`遍历完（即`oldFiber.sibling === null`），跳出遍历，**第一轮遍历结束。**
>
> 第二轮遍历步骤如下：
>
> 1. `newChildren`遍历完，`oldFiber`没遍历完，需要遍历剩下的`oldFiber`，依次标记`Deletion`。
> 2. `newChildren`没遍历完，`oldFiber`遍历完，需要遍历剩下的`newChildren`为生成的`workInProgress fiber`依次标记`Placement`。
>
> 第三轮遍历：
>
> 更新中节点是按`newChildren`的顺序排列。在遍历`newChildren`过程中，每个`遍历到的可复用节点`一定是当前遍历到的`所有可复用节点`中**最靠右的那个**，即一定在`lastPlacedIndex`对应的`可复用的节点`在本次更新中位置的后面。
>
> 只需要比较`遍历到的可复用节点`在上次更新时是否也在`lastPlacedIndex`对应的`oldFiber`后面，就能知道两次更新中这两个节点的相对位置改变没有。
>
> 用变量`oldIndex`表示`遍历到的可复用节点`在`oldFiber`中的位置索引。如果`oldIndex < lastPlacedIndex`，代表本次更新该节点需要向右移动。
>
> `lastPlacedIndex`初始为`0`，每遍历一个可复用的节点，如果`oldIndex >= lastPlacedIndex`，则`lastPlacedIndex = oldIndex`。

```js
function reconcileChildrenArray(
    returnFiber: Fiber,//父fiber节点
    currentFirstChild: Fiber | null,//childs中第一个节点
    newChildren: Array<*>,//新节点数组 也就是jsx数组
    lanes: Lanes,//lane相关
  ): Fiber | null {
  
    let resultingFirstChild: Fiber | null = null;//diff之后返回的第一个节点
    let previousNewFiber: Fiber | null = null;//新节点中上次对比过的节点
  
    let oldFiber = currentFirstChild;//正在对比的oldFiber
    let lastPlacedIndex = 0;//上次可复用的节点位置 或者oldFiber的位置
    let newIdx = 0;//新节点中对比到了的位置
    let nextOldFiber = null;//下次对比的oldFiber
    for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {//第一次遍历
      if (oldFiber.index > newIdx) {//nextOldFiber赋值
        nextOldFiber = oldFiber;
        oldFiber = null;
      } else {
        nextOldFiber = oldFiber.sibling;
      }
      //updateSlot 更新节点，如果key不同则newFiber=null
      const newFiber = updateSlot(returnFiber,oldFiber,newChildren[newIdx],lanes);
      if (newFiber === null) {
        if (oldFiber === null) {
          oldFiber = nextOldFiber;
        }
        break;//跳出第一次遍历
      }
      if (shouldTrackSideEffects) {//检查shouldTrackSideEffects
        // 如果update是新创建的则删除oldFiber（key相同，type不同）
        if (oldFiber && newFiber.alternate === null) {
          deleteChild(returnFiber, oldFiber);
        }
      }
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);//标记节点插入
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
      oldFiber = nextOldFiber;
    }
  
    // 如果更新处理完后，刚好等于newChildren最后一个，则删除oldFober兄弟节点
    if (newIdx === newChildren.length) {
      deleteRemainingChildren(returnFiber, oldFiber);//将oldFiber中没遍历完的节点标记为DELETION
      return resultingFirstChild;
    }
  
    // oldFiber处理完，newChildren还没遍历完，则全部添加进去
    if (oldFiber === null) {
      for (; newIdx < newChildren.length; newIdx++) {//第2次遍历
        const newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
        if (newFiber === null) {
          continue;
        }
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);//插入新增节点
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
      return resultingFirstChild;
    }
  
    // 将剩下的oldFiber加入map中
    const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
  
    for (; newIdx < newChildren.length; newIdx++) {//第三次循环 处理节点移动
      const newFiber = updateFromMap(
        existingChildren,
        returnFiber,
        newIdx,
        newChildren[newIdx],
        lanes,
      );
      if (newFiber !== null) {
        if (shouldTrackSideEffects) {
          if (newFiber.alternate !== null) {
            existingChildren.delete(//删除找到的节点
              newFiber.key === null ? newIdx : newFiber.key,
            );
          }
        }
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);//标记为插入的逻辑
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
    }
  
    if (shouldTrackSideEffects) {
      //删除existingChildren中剩下的节点
      existingChildren.forEach(child => deleteChild(returnFiber, child));
    }
  
    return resultingFirstChild;
  }

```



