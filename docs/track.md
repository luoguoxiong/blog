![](./img/coffee.jpg)

# React无痕埋点的实践

## 一、前言

**前端埋点：**通过前端页面上进去埋点，捕获用户行为，提供给数据分析人员。来获取产品的使用情况，进而促进产品的优化和迭代。

## 二、背景

常见的埋点方案：

**手动代码埋点：** 精确到用户某个行为进行数据上报。

* 优点：准确，满足产品自定义需求。

* 缺点：代码耦合度太高，不利于代码维护和服用。

**全埋点：**通过全局捕获事件进行数据上报。

* 优点：能够较大程度支持上报用户数据，降低业务代码耦合度。

* 缺点：对于组件化的前端工程，不能很好兼容。

针对上述描述的常见埋点方案，需要我们提供一种能减低代码耦合度、又能精确上报数据、还能支持组件化的前端工程。

## 三、埋点方案

结合团队使用的技术栈React、产品运营需求，采用**Porps传参**的方式实现**点击埋点**、**路由监听**实现**PV埋点**、**部分手动埋点**来实现埋点需求。

### 1. Props传参实现点击埋点

```ts
initReactTrack({
  onClickEvent: (message) => {
    console.log(message);
  },
});
```

```jsx
// Button组件
<Button track={{ a: 1 }}>测试无痕埋点</Button>
// html标签
<div track={{ a: 1 }}>测试无痕埋点</div>
```

期望值：

> initReactTrack函数中的onClickEvent，能够拿到组件或原生Dom中的track数据

实现思路：

> jsx语法中的组件、标签是其实是调用的方法是：**React.createElement**。
>
> 结合上述的使用情况，我们只需要重写*React.createElement*方法即可。只要组件的props中有track和onClick属性，并且track有值，则认为当前元素是需要上报的埋点。

实现代码：

```tsx
import React from 'react';

import { _history } from 'src/utils/history';

type OnClickEvent = (message:any) => void

type OnRouterChange = (url:string) => void

interface InitReactTrack {
    onClickEvent?:OnClickEvent;
    onRouterChange?:OnRouterChange;
}

const getReactFCInitializer = ({
  onClickEvent,
  onRouterChange,
}:InitReactTrack) => {
  const originalCreateElement = React.createElement;

  const propsWithTrackEvents = function(props:any) {
    if (props.track) {
      const reactClick = props.onClick;
      props.onClick = (e:any) => {
        onClickEvent && onClickEvent(props.track);
        reactClick && reactClick(e);
      };
    }
    return props;
  };

  React.createElement = function() {
    const args = Array.prototype.slice.call(arguments);

    let props = args[1];

    if (props && props.track) {
      props = propsWithTrackEvents(props || {});
    }

    return originalCreateElement.apply(null, args);
  };

  const routerChange = () => {
    onRouterChange && onRouterChange(window.location.href);
  };

  routerChange();
  _history.addEventListener(() => {
    routerChange();
  });
};

export const initReactTrack = ({ onClickEvent, onRouterChange }:InitReactTrack) => getReactFCInitializer({ onClickEvent, onRouterChange });
```

***注意:***使用ts开发的项目，给jsx元素添加track属性时会报错，需要手动修改@types/react 的 Attributes ts定义。如下

````ts
interface Attributes {
	key?: Key | null;
  track?:any;
}
````

### 2. **路由监听**实现PV埋点

```tsx
initReactTrack({
  onClickEvent: (message) => {
    console.log(message);
  },
  onRouterChange: (parmas) => {
    console.log(parmas);
  },
});
```

期望值：

> 当页面路由发生改变时能够在onRouterChange拿到的url路径。

实现思路：

> 1. 前端路由的两种模式：hash、history模式。
> 2. hash路由切换可以通过hashchange去捕获，这里就不再补充。
> 3. history模式，没有相关api去捕获路由切换，所以需要我们自己去实现啊类似hashchange的功能。既：当执行history的back, forward, go, pushState, replaceState能够执行回调函数。

hissory监听实现代码：

```ts
interface Historys {
    back():void;
    forward():void;
    go(delta?:number):void;
    pushState(data:any, title:string, url?:string | null):void;
    replaceState(data:any, title:string, url?:string | null):void;
}

type Listener = () => void

class HistoryListener{
    deeps:Array<Listener> = []

    constructor(){
      const methods = ['back', 'forward', 'go', 'pushState', 'replaceState'];
      methods.forEach((name:keyof Historys) => {
        this.registListner(name);
      });
    }

    notify(){
      this.deeps.forEach((listener:Listener) => listener());
    }

    registListner = (name:keyof Historys) => {
      const method = history[name];
      const _this = this;
      history[name] = function(...args:any[]) {
        method.apply(history, args);
        _this.notify();
      };
    };

    addEventListener(listner:Listener){
      this.deeps.push(listner);
      window.addEventListener('popstate', listner, false);
    }

    removeEventListener(listner:Listener){
      let i = 0;
      while(i < this.deeps.length){
        if(this.deeps[i] === listner){
          this.deeps.splice(i, 1);
          window.removeEventListener('popstate', listner);
          break;
        }
        i++;
      }
    }
}

export const _history = (() => {
  let constancs:HistoryListener;
  return () => constancs = constancs ? constancs : new HistoryListener();
})()();
```

## 四、小结

通过上述的实现方案，实现了点击埋点、和pv埋点，解决了与业务代码耦合的问题，在日常开发中没有太大成本，也挺高了埋点效率，只需要在入口引入initReactTrack方法即可。



本文只是对点击埋点和pv的无痕捕获，对于一些具体的业务场景，还需要结合手动埋点来实现。