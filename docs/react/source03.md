# 源码分析-Fiber树的构建

### 一、Fiber树概览图

```js
import React from 'react';
import ReactDOM from 'react-dom';

function App() {
  return (
    <div className="App">
      <header className="App-header">
          Learn React
      </header>
      <footer>footer</footer>
    </div>
  );
}

ReactDOM.render(
    <App />,
  document.getElementById('root')
);
```
在worKLoopSync执行完，会生成一个如图下的Fiber树的结构数据
![](../img/react/fiberTree.jpg)

### 二、render阶段Fiber树构建流程
