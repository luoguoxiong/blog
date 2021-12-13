# 源码分析- 渲染阶段

> 渲染阶段：
>
> 	1. 执行完所有的副作用
> 	1. 判断finishedWork是否有PassiveMask副作用，如果有以一个优先级调用flushPassiveEffects。
> 	1. 判断finishedWork是否有相关副作用（BeforeMutationMask | MutationMask | LayoutMask | PassiveMask）。如果有则进行commitBeforeMutationEffects、commitMutationEffects、commitLayoutEffects三个阶段。

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



### 三、commitLayoutEffects



### 四、flushPassiveEffects





