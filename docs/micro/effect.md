
# 清除副作用

`沙箱`中使用 `window.addEventListener`、`setInterval` 这些 `需异步监听的全局api` 时，要确保子应用在移除时也要移除对应的监听事件，否则会对其他应用造成副作用。下面以setInteval为例：

```ts
import { noop } from 'lodash';

const rawWindowInterval = window.setInterval;
const rawWindowClearInterval = window.clearInterval;

export default function patch(global: Window) {
  let intervals: number[] = [];

  global.clearInterval = (intervalId: number) => {
    intervals = intervals.filter((id) => id !== intervalId);
    return rawWindowClearInterval(intervalId);
  };

  global.setInterval = (handler: CallableFunction, timeout?: number, ...args: any[]) => {
    const intervalId = rawWindowInterval(handler, timeout, ...args);
    intervals = [...intervals, intervalId];
    return intervalId;
  };

  return function free() {
    intervals.forEach((id) => global.clearInterval(id));
    global.setInterval = rawWindowInterval;
    global.clearInterval = rawWindowClearInterval;

    return noop;
  };
}
```

```js
let mountingFreer;
const proxy2 = new CreateProxySandbox({});

function mountSandbox() {
  proxy2.mountProxySandbox();

  // 在沙箱环境中执行的代码
  (function (window, self) {
    with (window) {
      // 记录副作用
      mountingFreer = patchSideEffects(window);
      window.a = "this is proxySandbox2";
      console.log("代理沙箱2挂载后的a:", window.a); // undefined

      // 定时输出字符串
      setInterval(() => {
        console.log("Interval");
      }, 500);
    }
  }.bind(proxy2.proxy)(proxy2.proxy, proxy2.proxy));
}

/**
 * @param isPatch 是否关闭副作用
 */
function unmountSandbox(isPatch = false) {
  proxy2.mountProxySandbox();
  console.log("代理沙箱2卸载后的a:", window.a); // undefined
  if (isPatch) {
    mountingFreer();
  }
}
```

