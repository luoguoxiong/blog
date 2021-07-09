# Hooks 坑位告警

### 1. useEffect 如何监听 引用类型中 useState 的改变？

**核心知识点**： React 组件的更新机制对 state 只进行**浅对比**，也就是更新某个复杂类型数据时只要它的引用地址没变，那就不会重新渲染组件。更新复杂 state 的时候必须传给它一个全新的对象，而不是复制了它引用地址再修改的对象。所以 state 中如果是 object 或者 array 类型，如果直接改其值，是无法监听到变化。

**错误示例** ：虽然点击两个按钮改变了 state 值，但是 useEffect 无法监听到 state 发生变化，html 也无法重新渲染

```javascript
const MyDemo1 = () => {
  const [counter, setCounter] = useState({ a: 1, b: 2 });
  const [number, setNumber] = useState([1, 2, 3]);
  useEffect(() => {
    console.log("监听到变化");
  }, [counter, number]);
  const changeCounter = () => {
    counter.b = 40; //直接改变对象的值
    setCounter(counter);
  };
  const changeNumber = () => {
    number[2] = 10; //直接改变数组的值
    number.push(4);
    setNumber(number);
  };
  return (
    <>
      <div>{"counter:" + counter.a + "---" + counter.b}</div>
      <button onClick={changeCounter}>change counter</button>
      <div>num:</div>
      {number.map((item, index) => {
        return <p key={index}>{item}</p>;
      })}
      <button onClick={changeNumber}>change number</button>
    </>
  );
};
```

**正确示例**：使用展开运算符返回一个新对象或者新数组

```javascript
const MyDemo2 = () => {
  const [counter, setCounter] = useState({ a: 1, b: 2 });
  const [number, setNumber] = useState([1, 2, 3]);
  useEffect(() => {
    console.log("监听到变化");
  }, [counter, number]);
  const changeCounter = () => {
    setCounter({ ...counter, b: 40 });
  };
  const changeNumber = () => {
    const newNumber = [...number];
    newNumber[2] = 20;
    newNumber.push(4);
    setNumber(newNumber);
  };
  return (
    <>
      <div>{"counter:" + counter.a + "---" + counter.b}</div>
      <button onClick={changeCounter}>change counter</button>
      <div>num:</div>
      {number.map((item, index) => {
        return <p key={index}>{item}</p>;
      })}
      <button onClick={changeNumber}>change number</button>
    </>
  );
};
```

### 2. 如何在异步改变 state 后拿到最新的 state 值？

**核心知识点** ：React 合成事件中改变状态是异步的，出于减少 render 次数，react 会收集所有状态变更，然后比对优化，最后做一次变更。

```javascript
const MyDemo3 = () => {
  const [counter, setCounter] = useState(0);
  function asyncData() {
    console.log("异步请求数据：", counter); //这里会使用num做一些处理，比如num作为参数请求数据
  }
  function changeCounter() {
    setCounter(10); //改变data后执行 asyncData 函数
    console.log("counter", counter); //num为0
    asyncData();
  }
  return (
    <>
      <div>{counter}</div>
      <button onClick={changeCounter}>改变num</button>
    </>
  );
};
```

在代码中可以看出，asyncData 的调用和 setstate 在同一个宏任务中，这时 react 还没有 render，所以直接使用 state 获取的是上一次闭包里的值 0。

**改进** 可以使用 useRef，useRef 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数。返回的 ref 对象在组件的整个生命周期内保持不变。

```javascript
const MyDemo5 = () => {
  const [counter, setCounter] = useState(0);
  const cunterRef = useRef(0);
  function asyncData() {
    console.log("异步请求数据：" + cunterRef.current); //这里会使用 num 做一些处理，比如请求数据
  }
  function changeCounter() {
    setCounter(10);
    cunterRef.current = 10;
    asyncData();
  }
  function pureChangeCounter() {
    cunterRef.current = 20;
    setCounter(20);
  }
  return (
    <>
      <button
        onClick={() => {
          changeCounter();
        }}
      >
        点击我执行异步函数
      </button>
      <div>{counter}</div>
      <button
        onClick={() => {
          pureChangeCounter();
        }}
      >
        点击我只是单纯改变 data
      </button>
    </>
  );
};
```

### 3. useState 函数式更新

**核心知识点**：

1. useState函数式更新，每次都能拿到最新的值，复制式更新，拿到的是运行时的值。

2. setState在异步任务执行时，每一次setState都会触发一次渲染。

如下面的代码 handleClickFn 中通过函数式改变 count，handleClick 是常规方法改变 count，多次点击按钮后，均是在 3s 之后改变 count。

可以看出常规的方法只变化了一次，count是当前闭包执行时的值，**0->1**

而函数式 usestate 则改变了多次，因为它可以获取之前的 state 值，也就是代码中的 **prevCount 每次都是最新的值**。

```js
function MyDemo6() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setTimeout(() => {
      setCount(count + 1);
    }, 3000);
  }
  function handleClickFn() {
    setTimeout(() => {
      setCount((prevCount) => {
        return prevCount + 1;
      });
    }, 3000);
  }
  return (
    <>
      Count: {count}
      <button onClick={handleClick}>点击常规改变count</button>
      <br />
      <button onClick={handleClickFn}>点击函数式改变count</button>
    </>
  );
}
```

下面的例子：
handleClick：渲染1次，count的值为1；

handleClickFn：渲染1次，count的值为3；

```js
import React, { useState } from 'react';

const i = 0;
export const Poster = () => {

  const [count, setCount] = useState(0);
  function handleClick() {
    // setTimeout(() => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // }, 1000);
  }
  function handleClickFn() {
    // setTimeout(() => {
    setCount((prevCount) => prevCount + 1);
    setCount((prevCount) => prevCount + 1);
    setCount((prevCount) => prevCount + 1);
    // }, 1000);

  }
  return (
    <>
        Count: {count}
      <button onClick={handleClick}>点击常规改变count</button>
      <br />
      <button onClick={handleClickFn}>点击函数式改变count</button>
    </>
  );
};
```

### 4. useEffect执行顺序

**核心知识点：**

1. 多个Effect按先后顺序执行。
2. effect在render之后执行。
3. effect return 在下次执行effect执行前会执行return。

```js
export default function MyDemo11() {
  console.log("init render");
  const [counter, setCounter] = useState(0);
  useEffect(() => {
    console.log(1);
  }, []);
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log("change");
      setCounter(counter + 2);
    }, 10000);
    console.log("effect:", timer);
    return () => {
      console.log("clear:", timer);
      clearTimeout(timer);
    };
  }, [counter]);
  console.log("before render");
  return (
    <div className="container">
      {console.log("render...")}
      <div className="el">{counter}</div>
    </div
  );
}
```

init render->  before render -> render... -> 1 -> effect ->**change ->init render ->before render->render... -> clear -> efect**

### 5. 父子组件中的 useEffect, useLayoutEffect 执行顺序

**核心知识点**：

1. useLayoutEffect执行顺序快于useEffect
2. useLayoutEffect同步执行，会阻塞渲染；useEffect异步执行，不会阻塞渲染。

**原因**：后续源码解读补充。