# 微前端 JS沙箱

**JS 沙箱**简单点说就是，主应用有一套全局环境`window`，子应用有一套私有的全局环境`fakeWindow`，子应用所有操作都只在新的全局上下文中生效，这样的子应用好比被一个个箱子装起来与主应用**隔离**，因此主应用加载子应用便不会造成**JS 变量的相互污染**、**JS 副作用**、**CSS 样式被覆盖**等，每个子应用的全局上下文都是独立的。

### 一、快照沙箱  snapshotSandbox

```js
function iter(obj, callbackFn) {
  for (const prop in obj) {
    if (obj.hasOwnProperty(prop)) {
      callbackFn(prop);
    }
  }
}
class SnapshotSandbox {
  constructor(name) {
    this.name = name;
    this.proxy = window;
    this.type = 'Snapshot';
    this.modifyPropsMap = {};
    this.windowSnapshot = {};
  }

  active() {
    // 记录当前快照
    this.windowSnapshot = {};
    iter(window, (prop) => {
      this.windowSnapshot[prop] = window[prop];
    });

    // 恢复之前的变更
    Object.keys(this.modifyPropsMap).forEach((p) => {
      window[p] = this.modifyPropsMap[p];
    });

    this.sandboxRunning = true;
  }

  inactive() {
    this.modifyPropsMap = {};

    iter(window, (prop) => {
      if (window[prop] !== this.windowSnapshot[prop]) {
        // 记录变更，恢复环境
        this.modifyPropsMap[prop] = window[prop];
        window[prop] = this.windowSnapshot[prop];
      }
    });

    if (process.env.NODE_ENV === 'development') {
      console.info(`[qiankun:sandbox] ${this.name} origin window restore...`, Object.keys(this.modifyPropsMap));
    }

    this.sandboxRunning = false;
  }
}

const snapshotSandbox = new SnapshotSandbox('test');

snapshotSandbox.active();
// 子应用A
window.a = { c: 2 };
console.log('快照沙箱挂载后的a:', window.a); // 123
snapshotSandbox.inactive();
console.log('快照沙箱卸载后的a:', window.a); // undefined
snapshotSandbox.active();
console.log('快照沙箱再次挂载后的a:', window.a); // 123
```

- 优点
  - 兼容几乎所有浏览器
- 缺点
  - 无法同时有多个运行时快照沙箱，否则在 window 上修改的记录会混乱，一个页面只能运行一个单实例微应用

### 二、代理沙箱 proxySandbox

```js
function CreateProxySandbox(fakeWindow = {}) {
  const _this = this;
  _this.proxy = new Proxy(fakeWindow, {
    set(target, p, value) {
      if (_this.sandboxRunning) {
        target[p] = value;
      }

      return true;
    },
    get(target, p) {
      if (_this.sandboxRunning) {
        return target[p];
      }
      return undefined;
    },
  });

  _this.mountProxySandbox = () => {
    _this.sandboxRunning = true;
  };

  _this.unmountProxySandbox = () => {
    _this.sandboxRunning = false;
  };
}

const proxyA = new CreateProxySandbox({});
const proxyB = new CreateProxySandbox({});

proxyA.mountProxySandbox();
proxyB.mountProxySandbox();

(function(window) {
  window.a = 'a';
  console.log('代理沙箱 a:', window.a); // a
})(proxyA.proxy);

(function(window) {
  window.b = 'b';
  console.log('代理沙箱 b:', window.b); // b
})(proxyB.proxy);

proxyA.unmountProxySandbox();
proxyB.unmountProxySandbox();

(function(window) {
  console.log('代理沙箱 a:', window.a); // undefined
})(proxyA.proxy);

(function(window) {
  console.log('代理沙箱 b:', window.b); // undefined
})(proxyB.proxy);
```

- **优点**

1. 可同时运行多个沙箱
2. 不会污染 window 环境

- **缺点**

1. 不兼容 ie
2. 在全局作用域上通过 `var` 或 `function` 声明的变量和函数无法被代理沙箱劫持，因为代理对象 `Proxy` 只能识别在该对象上存在的属性，通过 `var` 或 `function` 声明声明的变量是开辟了新的地址，自然无法被 `Proxy` 劫持。