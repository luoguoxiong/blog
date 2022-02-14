# 页面生命周期

### DOMContentLoaded

> DOMContentLoaded浏览器已完全加载 HTML，并构建了 DOM 树，但像 img 和样式表之类的外部资源可能尚未加载完成。

```js
<script>
  function ready() {
    alert('DOM is ready');

    // 图片目前尚未加载完成（除非已经被缓存），所以图片的大小为 0x0
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  }

  document.addEventListener("DOMContentLoaded", ready);
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

> `DOMContentLoaded` 处理程序在文档加载完成后触发，所以它可以查看所有元素，包括它下面的 `<img>` 元素。
>
> 但是，它不会等待图片加载。因此，`alert` 显示其大小为零。

### onload

> 当整个页面，包括样式、图片和其他资源被加载完成时，会触发 `window` 对象上的 `load` 事件。可以通过 `onload` 属性获取此事件。

```js
<script>
  window.onload = function() { // 与此相同 window.addEventListener('load', (event) => {
    alert('Page loaded');

    // 此时图片已经加载完成
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  };
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

### unload

> 当访问者离开页面时，`window` 对象上的 `unload` 事件就会被触发。我们可以在那里做一些不涉及延迟的操作，例如关闭相关的弹出窗口。

```js
let analyticsData = { /* 带有收集的数据的对象 */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

> - 请求以 POST 方式发送。
> - 我们不仅能发送字符串，还能发送表单以及其他格式的数据，在 [Fetch](https://zh.javascript.info/fetch) 一章有详细讲解，但通常它是一个字符串化的对象。
> - 数据大小限制在 64kb。
>
> 当 `sendBeacon` 请求完成时，浏览器可能已经离开了文档，所以就无法获取服务器响应（对于分析数据来说通常为空）。
>
> 还有一个 `keep-alive` 标志，该标志用于在 [fetch](https://zh.javascript.info/fetch) 方法中为通用的网络请求执行此类“离开页面后”的请求。你可以在 [Fetch API](https://zh.javascript.info/fetch-api) 一章中找到更多相关信息。
>
> 如果我们要取消跳转到另一页面的操作，在这里做不到。但是我们可以使用另一个事件 —— `onbeforeunload`。

### onbeforeunload

```js
window.onbeforeunload = function() {
  return "There are unsaved changes. Leave now?";
};
```

> 如果访问者触发了离开页面的导航（navigation）或试图关闭窗口，beforeunload 处理程序将要求进行更多确认。
>
> 如果我们要取消事件，浏览器会询问用户是否确定。

### readystatechange

```js
// 当前状态
console.log(document.readyState);

// 状态改变时打印它
document.addEventListener('readystatechange', () => console.log(document.readyState));
```

* `loading` —— 文档正在被加载。

- `interactive` —— 文档已被解析完成，与 `DOMContentLoaded` 几乎同时发生，但是在 `DOMContentLoaded` 之前发生。
- `complete` —— 文档和资源均已加载完成，与 `window.onload` 几乎同时发生，但是在 `window.onload` 之前发生。
