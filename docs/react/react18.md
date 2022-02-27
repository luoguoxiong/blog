# React18新特性

https://zh-hans.reactjs.org/blog/2021/12/17/react-conf-2021-recap.html

### Automatic batching(自动批量处理)

 在React18之前会触发三次更新

```js
 fetch('/something').then(()=>{
   setIsFetching(false)
   setError(null)
   setFormStatus('success')
 })
```

### Suspense on the server（服务端渲染按需渲染）

首屏渲染时，渲染用户需要的界面，提高用户体验，和首屏渲染性能

```jsx
<Suspense fallback={<Spinner/>}>
  <Comments></Comments>
</Suspense>
```

### New Api(concurrent features)(并发功能的Api)

1. startTransition()
2. useTransition()
3. useDeferredValue()
4. useId()
5. useSyncExternalStore()

### 怎么升级到18

1.  Install react and react-dom from npm
2. ReactDom.render 替换为 ReatDOM.createRoot

```jsx
let container = document.getElementByLd('app')
let root = ReactDOM.createRoot(container)
root.render(<App/>)
```

 