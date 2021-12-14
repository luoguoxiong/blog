# 源码分析- 渲染阶段

> 渲染阶段：
>
> 	1. 执行完所有的副作用
> 	2. 判断finishedWork是否有PassiveMask副作用，如果有以一个优先级调用flushPassiveEffects。
> 	3. 判断finishedWork是否有相关副作用（BeforeMutationMask | MutationMask | LayoutMask | PassiveMask）。如果有则进行commitBeforeMutationEffects、commitMutationEffects、commitLayoutEffects三个阶段。

```js
function commitRootImpl(root, renderPriorityLevel) {
  do {
    // 执行完所有的副作用
    flushPassiveEffects();
  } while (rootWithPendingPassiveEffects !== null);

  const finishedWork = root.finishedWork;
  const lanes = root.finishedLanes;
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      pendingPassiveEffectsRemainingLanes = remainingLanes;
      scheduleCallback(NormalSchedulerPriority, () => {
        flushPassiveEffects();
        return null;
      });
    }
  }
  const subtreeHasEffects =
    (finishedWork.subtreeFlags &
      (BeforeMutationMask | MutationMask | LayoutMask | PassiveMask)) !==
    NoFlags;
  const rootHasEffect =
    (finishedWork.flags &
      (BeforeMutationMask | MutationMask | LayoutMask | PassiveMask)) !==
    NoFlags;

  if (subtreeHasEffects || rootHasEffect) {
    const prevTransition = ReactCurrentBatchConfig.transition;
    ReactCurrentBatchConfig.transition = 0;
    const previousPriority = getCurrentUpdatePriority();
    setCurrentUpdatePriority(DiscreteEventPriority);

    const prevExecutionContext = executionContext;
    executionContext |= CommitContext;

    ReactCurrentOwner.current = null;

    // before mutation
    const shouldFireAfterActiveInstanceBlur = commitBeforeMutationEffects(
      root,
      finishedWork,
    );
    // mutation
    commitMutationEffects(root, finishedWork, lanes);

    root.current = finishedWork;

    // layout
    commitLayoutEffects(finishedWork, root, lanes);
    // Reset the priority to the previous non-sync value.
    ReactCurrentBatchConfig.transition = prevTransition;
  } else {
    root.current = finishedWork;
  }

  if (rootDoesHavePassiveEffects) {
    // This commit has passive effects. Stash a reference to them. But don't
    // schedule a callback until after flushing layout work.
    rootDoesHavePassiveEffects = false;
    rootWithPendingPassiveEffects = root;
    pendingPassiveEffectsLanes = lanes;
  } else {
    // There were no passive effects, so we can immediately release the cache
    // pool for this render.
    releaseRootPooledCache(root, remainingLanes);
  }

  remainingLanes = root.pendingLanes;

  // Always call this before exiting `commitRoot`, to ensure that any
  // additional work on this root is scheduled.
  ensureRootIsScheduled(root, now());

  if (
    includesSomeLane(pendingPassiveEffectsLanes, SyncLane) &&
    root.tag !== LegacyRoot
  ) {
    flushPassiveEffects();
  }

  // If layout work was scheduled, flush it now.
  flushSyncCallbacks();

  if (enableSchedulingProfiler) {
    markCommitStopped();
  }
  return null;
}
```

### 一、commitBeforeMutationEffects

> 主要处理渲染前的相关副作用处理主要由commitBeforeMutationEffects_begin、commitBeforeMutationEffectsDeletion、commitBeforeMutationEffects_complete、commitBeforeMutationEffectsOnFiber几个函数执行。

#### 1. commitBeforeMutationEffects_begin

```js
function commitBeforeMutationEffects_begin() {
  while (nextEffect !== null) {
    const fiber = nextEffect;

    // This phase is only used for beforeActiveInstanceBlur.
    // Let's skip the whole loop if it's off.
    if (enableCreateEventHandleAPI) {
      // TODO: Should wrap this in flags check, too, as optimization
      const deletions = fiber.deletions;
      if (deletions !== null) {
        for (let i = 0; i < deletions.length; i++) {
          const deletion = deletions[i];
          commitBeforeMutationEffectsDeletion(deletion);
        }
      }
    }

    const child = fiber.child;
    if (
      (fiber.subtreeFlags & BeforeMutationMask) !== NoFlags &&
      child !== null
    ) {
      ensureCorrectReturnPointer(child, fiber);
      nextEffect = child;
    } else {
      commitBeforeMutationEffects_complete();
    }
  }
}
```

#### 2. commitBeforeMutationEffectsDeletion

> 此函数主要用于处理失去焦点等相关操作。

#### 3. commitBeforeMutationEffects_complete

> nextEffect 如果有兄弟节点则设置  nextEffect = sibling，没有nextEffect = fiber.return。

```js
function commitBeforeMutationEffects_complete() {
  while (nextEffect !== null) {
    const fiber = nextEffect;
    setCurrentDebugFiberInDEV(fiber);
    try {
      commitBeforeMutationEffectsOnFiber(fiber);
    } catch (error) {
      reportUncaughtErrorInDEV(error);
      captureCommitPhaseError(fiber, fiber.return, error);
    }
    resetCurrentDebugFiberInDEV();

    const sibling = fiber.sibling;
    if (sibling !== null) {
      ensureCorrectReturnPointer(sibling, fiber.return);
      nextEffect = sibling;
      return;
    }

    nextEffect = fiber.return;
  }
}
```

#### 4. commitBeforeMutationEffectsOnFiber

> ClassComponent：在存在alternate时，会调用getSnapshotBeforeUpdate生命周期
>
> HostRoot：会清空rootContainer的根节点的内容。

```js
function commitBeforeMutationEffectsOnFiber(finishedWork: Fiber) {
  const current = finishedWork.alternate;
  const flags = finishedWork.flags;
  if ((flags & Snapshot) !== NoFlags) {
    switch (finishedWork.tag) {
      case ClassComponent: {
        if (current !== null) {
          const prevProps = current.memoizedProps;
          const prevState = current.memoizedState;
          const instance = finishedWork.stateNode;
          const snapshot = instance.getSnapshotBeforeUpdate(
            finishedWork.elementType === finishedWork.type
              ? prevProps
              : resolveDefaultProps(finishedWork.type, prevProps),
            prevState,
          );
          instance.__reactInternalSnapshotBeforeUpdate = snapshot;
        }
        break;
      }
      case HostRoot: {
        if (supportsMutation) {
          const root = finishedWork.stateNode;
          clearContainer(root.containerInfo);
        }
        break;
      }
    }
  }
}
```

### 二、commitMutationEffects

> 主要负责节点的渲染：commitMutationEffects_begin

#### 1. commitMutationEffects_begin

> 1. 先处理需要fiber.deletions需要删除的节点
> 2. 判断当前节点的subtreeFlags是否存在MutationMask

```js
function commitMutationEffects_begin(root) {
  while (nextEffect !== null) {
    var deletions = fiber.deletions;
    if (deletions !== null) {
      for (var i = 0; i < deletions.length; i++) {
        var childToDelete = deletions[i];
          commitDeletion(root, childToDelete, fiber);
      }
    }

    var child = fiber.child;

    if ((fiber.subtreeFlags & MutationMask) !== NoFlags && child !== null) {
      ensureCorrectReturnPointer(child, fiber);
      nextEffect = child;
    } else {
      commitMutationEffects_complete(root);
    }
  }
}
```

#### 2. commitDeletion

> 删除相关子节点，并触发对应的Effect相关函数
>
> 重置fiber和fiber的return

```js
function commitDeletion(
  finishedRoot: FiberRoot,
  current: Fiber,
  nearestMountedAncestor: Fiber,
): void {
  unmountHostComponents(finishedRoot, current, nearestMountedAncestor);
  // 重置被删除的fiber和fiber.alternate 的return值为null
  detachFiberMutation(current);
}
```

#### 3. unmountHostComponents

```js
function unmountHostComponents(
  finishedRoot: FiberRoot, // 根节点FiberRoot
  current: Fiber, // 将要删除的节点
  nearestMountedAncestor: Fiber, // 父节点
): void {
  let node: Fiber = current;

  let currentParentIsValid = false;

  let currentParent;
  let currentParentIsContainer;

  while (true) {
    if (!currentParentIsValid) {
      // 找到父级的dom节点
      let parent = node.return;
      findParent: while (true) {
        const parentStateNode = parent.stateNode;
        switch (parent.tag) {
          case HostComponent:
            currentParent = parentStateNode;
            currentParentIsContainer = false;
            break findParent;
          case HostRoot:
            currentParent = parentStateNode.containerInfo;
            currentParentIsContainer = true;
            break findParent;
          case HostPortal:
            currentParent = parentStateNode.containerInfo;
            currentParentIsContainer = true;
            break findParent;
        }
        parent = parent.return;
      }
      currentParentIsValid = true;
    }

    if (node.tag === HostComponent || node.tag === HostText) {
      commitNestedUnmounts(finishedRoot, node, nearestMountedAncestor);
      if (currentParentIsContainer) {
        removeChildFromContainer(
          ((currentParent: any): Container),
          (node.stateNode: Instance | TextInstance),
        );
      } else {
        removeChild(
          ((currentParent: any): Instance),
          (node.stateNode: Instance | TextInstance),
        );
      }
    } else if (
      enableSuspenseServerRenderer &&
      node.tag === DehydratedFragment
    ) {
      if (enableSuspenseCallback) {
        const hydrationCallbacks = finishedRoot.hydrationCallbacks;
        if (hydrationCallbacks !== null) {
          const onDeleted = hydrationCallbacks.onDeleted;
          if (onDeleted) {
            onDeleted((node.stateNode: SuspenseInstance));
          }
        }
      }
      if (currentParentIsContainer) {
        clearSuspenseBoundaryFromContainer(
          ((currentParent: any): Container),
          (node.stateNode: SuspenseInstance),
        );
      } else {
        clearSuspenseBoundary(
          ((currentParent: any): Instance),
          (node.stateNode: SuspenseInstance),
        );
      }
    } else if (node.tag === HostPortal) {
      if (node.child !== null) {
        currentParent = node.stateNode.containerInfo;
        currentParentIsContainer = true;
        node.child.return = node;
        node = node.child;
        continue;
      }
    } else {
      commitUnmount(finishedRoot, node, nearestMountedAncestor);
      if (node.child !== null) {
        node.child.return = node;
        node = node.child;
        continue;
      }
    }
    if (node === current) {
      return;
    }
    while (node.sibling === null) {
      if (node.return === null || node.return === current) {
        return;
      }
      node = node.return;
      if (node.tag === HostPortal) {
        currentParentIsValid = false;
      }
    }
    node.sibling.return = node.return;
    node = node.sibling;
  }
}
```

#### 4.commitNestedUnmounts

> 对tag为HostComponent或者HostText的Fiber节点处理，并对子它的子节点执行commitUnmount函数。

```js
function commitNestedUnmounts(
  finishedRoot: FiberRoot,
  root: Fiber,
  nearestMountedAncestor: Fiber,
): void {
  let node: Fiber = root;
  while (true) {
    commitUnmount(finishedRoot, node, nearestMountedAncestor);
    if (
      node.child !== null &&
      (!supportsMutation || node.tag !== HostPortal)
    ) {
      node.child.return = node;
      node = node.child;
      continue;
    }
    if (node === root) {
      return;
    }
    while (node.sibling === null) {
      if (node.return === null || node.return === root) {
        return;
      }
      node = node.return;
    }
    node.sibling.return = node.return;
    node = node.sibling;
  }
}
```

#### 5.commitUnmount

> 1. 对于函数类组件的Fiber节点，对fiber.updateQueue。取lastEffect.next作为第一个节点，依次执行effect销毁函数。
> 2. class 组件，则是调用componentWillUnmount生命周期。
> 3. HostComponent节点，则是ref处理操作。

```js
function commitUnmount(
  finishedRoot: FiberRoot,
  current: Fiber,
  nearestMountedAncestor: Fiber,
): void {
  onCommitUnmount(current);

  switch (current.tag) {
    case FunctionComponent:
    case ForwardRef:
    case MemoComponent:
    case SimpleMemoComponent: {
      const updateQueue: FunctionComponentUpdateQueue | null = (current.updateQueue: any);
      if (updateQueue !== null) {
        const lastEffect = updateQueue.lastEffect;
        if (lastEffect !== null) {
          const firstEffect = lastEffect.next;

          let effect = firstEffect;
          do {
            const {destroy, tag} = effect;
            if (destroy !== undefined) {
              if ((tag & HookInsertion) !== NoHookEffect) {
                safelyCallDestroy(current, nearestMountedAncestor, destroy);
              } else if ((tag & HookLayout) !== NoHookEffect) {
                if (enableSchedulingProfiler) {
                  markComponentLayoutEffectUnmountStarted(current);
                }

                if (
                  enableProfilerTimer &&
                  enableProfilerCommitHooks &&
                  current.mode & ProfileMode
                ) {
                  startLayoutEffectTimer();
                  safelyCallDestroy(current, nearestMountedAncestor, destroy);
                  recordLayoutEffectDuration(current);
                } else {
                  safelyCallDestroy(current, nearestMountedAncestor, destroy);
                }

                if (enableSchedulingProfiler) {
                  markComponentLayoutEffectUnmountStopped();
                }
              }
            }
            effect = effect.next;
          } while (effect !== firstEffect);
        }
      }
      return;
    }
    case ClassComponent: {
      safelyDetachRef(current, nearestMountedAncestor);
      const instance = current.stateNode;
      if (typeof instance.componentWillUnmount === 'function') {
        safelyCallComponentWillUnmount(
          current,
          nearestMountedAncestor,
          instance,
        );
      }
      return;
    }
    case HostComponent: {
      safelyDetachRef(current, nearestMountedAncestor);
      return;
    }
    case HostPortal: {
      // TODO: this is recursive.
      // We are also not using this parent because
      // the portal will get pushed immediately.
      if (supportsMutation) {
        unmountHostComponents(finishedRoot, current, nearestMountedAncestor);
      } else if (supportsPersistence) {
        emptyPortalContainer(current);
      }
      return;
    }
    case DehydratedFragment: {
      if (enableSuspenseCallback) {
        const hydrationCallbacks = finishedRoot.hydrationCallbacks;
        if (hydrationCallbacks !== null) {
          const onDeleted = hydrationCallbacks.onDeleted;
          if (onDeleted) {
            onDeleted((current.stateNode: SuspenseInstance));
          }
        }
      }
      return;
    }
    case ScopeComponent: {
      if (enableScopeAPI) {
        safelyDetachRef(current, nearestMountedAncestor);
      }
      return;
    }
  }
}
```

#### 6.commitMutationEffects_complete

> 有关于nextEffect相关操作，当前节点执行完commitMutationEffectsOnFiber，如果有兄弟节点则会继续执行commitMutationEffects_begin，当没有兄弟节点了，则nextEffect赋值为fiber.return，执行commitMutationEffects_complete。

```js
function commitMutationEffects_complete(root: FiberRoot) {
  while (nextEffect !== null) {
    const fiber = nextEffect;
    commitMutationEffectsOnFiber(fiber, root);
    if (sibling !== null) {
      nextEffect = sibling;
      return;
    }
    nextEffect = fiber.return;
  }
}
```

#### 7.commitMutationEffectsOnFiber

> 根据Fiber的flags属性，进行节点的Placement、PlacementAndUpdate、Update等相关操作。（既dom节点的添加和修改）

```js
function commitMutationEffectsOnFiber(finishedWork: Fiber, root: FiberRoot) {
  const flags = finishedWork.flags;
  const primaryFlags = flags & (Placement | Update | Hydrating);
  outer: switch (primaryFlags) {
    case Placement: {
      commitPlacement(finishedWork);
      finishedWork.flags &= ~Placement;
      break;
    }
    case PlacementAndUpdate: {
      // Placement
      commitPlacement(finishedWork);
      finishedWork.flags &= ~Placement;
      // Update
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
    case Hydrating: {
      finishedWork.flags &= ~Hydrating;
      break;
    }
    case HydratingAndUpdate: {
      finishedWork.flags &= ~Hydrating;
      // Update
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
    case Update: {
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
  }
}
```

### 三、commitLayoutEffects



### 四、flushPassiveEffects





