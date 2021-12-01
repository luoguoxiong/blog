# 源码分析-调试篇

### 一、拉去React 仓库代码
```shell
    git clone git@github.com:facebook/react.git
```

### 二、安装依赖并构建和创建软连
```shell
cd react

yarn

yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE

cd build/node_modules/react
# 申明react指向
yarn link
cd build/node_modules/react-dom
# 申明react-dom指向
yarn link
```

### 三、使用create-react-app 创建调试项目
```shell
npx create-react-app react-code

yarn link react react-dom
```

1. render 
    参数说明：
        element jsx 元素 
        container 挂载dom树的节点
        callback 回调函数

2. legacyRenderSubtreeIntoContainer
    参数说明：
        parentComponent: React组件
        children: jsx 元素 ,
        container: 挂载dom树的节点,
        forceHydrate: 是否是服务端渲染,
        callback: 回调函数,
    工作：
        判断container是否含有_reactRootContainer属性
        有：走updateContainer
        没有：走legacyCreateRootFromDOMContainer
    2.1 legacyCreateRootFromDOMContainer(mount阶段)
         1. 清除存在的页面内容
         2. createContainer
            containerInfo: 挂载dom树的节点,
            tag: RootTag(0或1）,
            hydrate: boolean,
            hydrationCallbacks: null | SuspenseHydrationCallbacks,
            isStrictMode: boolean,
            concurrentUpdatesByDefaultOverride: null | boolean,
            1. createFiberRoot (生成FiberRoot节点)
                1. createHostRootFiber 生成HostRootFiber,FiberRoot节点的current指向HostRootFiber
                2. initializeUpdateQueue 初始化HostRootFiber的一些属性HostRootFiber
         3. markContainerAsRoot
                1. 给当前挂载的根节点添加internalContainerInstanceKey属性为
         4. listenToAllSupportedEvents（React 事件机制）
                1. 给根节点添加allNativeEvents的事件
                2. listenToNativeEvent
                3. addTrappedEventListener
                    1. createEventListenerWrapperWithPriority
    2.2 updateContainer
        1. createUpdate
        2. enqueueUpdate
        3. scheduleUpdateOnFiber (构建fiber树)
            1. markUpdateLaneFromFiberToRoot
            2. markRootUpdated
            3. ensureRootIsScheduled
                1. markStarvedLanesAsExpired
        4. entangleTransitions
        <!-- 渲染阶段 -->