# React 性能优化
> demo仓库地址： https://github.com/luoguoxiong/react-performance-optimization
### 一、PureComponent
> PureComponent 负责 shouldComponentUpdate——它对状态和 props 数据进行浅层比较（shallow comparison）。
>
> PureComponent 如果浅层数据未发生改变，则不会重复渲染

```js
import React from 'react';

class PureComp extends React.PureComponent{
  render(){
    console.log('PureComp 渲染了');
    return<div>{this.props.name}</div>;
  }
}

class NoPureComp extends React.Component{
  render(){
    console.log('NoPureComp 渲染了');
    return<div>{this.props.name}</div>;
  }
}

export class PureComponentDemo extends React.Component {
  constructor() {
    super();
    this.state = {
      name: 'PureComponentDemo',
    };
  }
  componentDidMount() {
    setInterval(() => {
      this.setState({
        name: 'PureComponentDemo',
      });
    }, 1000);
  }

  render() {
    console.log('Render Called Again');
    return (
      <>
        <PureComp name={this.state.name} />
        <NoPureComp name={this.state.name} />
      </>
    );
  }
}
```
### 二、shouldComponentUpdate

> 在shouldComponentUpdate生命周期进行porps或state判断决定需要不要重新渲染
>
>  return false不会重新渲染

```js
import React from 'react';

export class ShouldComponentUpdateDemo extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      name: 'Mayank',
    };
  }

  componentDidMount() {
    setTimeout(() => {
      this.setState({
        name: 'Mayank',
      });
    }, 10);
  }

  shouldComponentUpdate(nextProps, nextState) {
    if(nextState.age !== this.state.age || nextState.name === this.state.name) {
      return false;
    }
    return true;
  }

  render() {
    console.log('ShouldComponentUpdateDemo 渲染了');
    return (
      <div>
        <b>User Name:</b> {this.state.name}
      </div>
    )
  }
}
```

### 三、React.memo

> 在class组件内，可以通过shouldComponentUpdate来决定是否要重新渲染，在函数式组件可以通过React.memo实现。
>
>  React.memo第二参数的执行函数返回true,则不会重复更新
>
> 如下：
>
> NoMemo call Render 打印N次
>
> isMemo call Render 只会打印一次

```js
import React, { useState, useEffect } from 'react';

const UserCom = (props) => {
  console.log(`${props.type} call Render`);
  return <div> {props.name}</div>;
};

const MemoUserCom = React.memo(UserCom, (prevProps, nextProps) => prevProps.name === nextProps.name);

export const MemoDemo = () => {

  const [_, setName] = useState('init');

  useEffect(() => {
    setInterval(() => {
      setName(Math.random());
    }, 1000);
  }, []);

  return (
    <>
      <UserCom
        name="NoMemo"
        type="NoMemo" />
      <MemoUserCom
        name="isMemo"
        type="isMemo" />
    </>
  );
};
```

### 四、巧用Diff策略

> 在条件渲染的时候，避免使用if else 进行节点渲染控制，这样会导致全部虚拟dom进行计算，并全量渲染
>
> 使用 && 操作符，充分利用React Diff 策略

```js
/* eslint-disable react/display-name */
import React, { useState, useRef } from 'react';

const A = React.memo(() => {
  console.log('A called render');
  return <div>A</div>;
});

const B = React.memo(() => {
  console.log('B called render');
  return <div>B</div>;
});

export const Diff = () => {
  const [isShow, setShow] = useState(false);

  const ref = useRef(false);

  React.useEffect(() => {
    setInterval(() => {
      ref.current = !ref.current;
      setShow(ref.current);
    }, 1000);
  }, []);

  //   return (
  //     <>
  //       {isShow && <A />}
  //       <B />
  //     </>
  //   );
  if(isShow){
    return (
      <>
        <A />
        <B />
      </>
    );
  }else{
    return <B />;
  }
};
```

### 五、React.useMemo

> 使用场景一：
>
> ​		如果存在某些值需要根据state进行复杂计算，可以通过监听state的值，只有当state的值发生改变，再去进行复杂的数据计算。
>
> 使用场景二：
>
> ​		缓存组件状态，避免重复渲染

```js
import React from 'react';

export const UseMemoDemo = () => {
  const [userId] = React.useState(0);

  const [otherStatus, setOther] = React.useState(0);

  const val = React.useMemo(() =>
    // todo 复杂数据计算
    userId
  , [userId]);

  React.useEffect(() => {
    setInterval(() => {
      setOther(Math.random());
    }, 1000);
  }, []);

  const comP = React.useMemo(() => {
    console.log('memo call Render');
    return <div>{userId}</div>;
  }, [userId]);

  return (
    <div>
      {val}
      {comP}
      {otherStatus}
    </div>
  );
};
```

### 六、React.useCallback 

> React.useCallback保存事件不被更新，来减少diff的计算成本。
>
> 当然useCallback是有使用成本的。
>
> 参考：https://zh-hans.reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render
>
> Hooks不会因为在渲染时创建函数而变慢
>
> 申请 useCallbck 第一个参数对应的函数所需要的内存，这一点的花费就和 noUseCallback 的开销一样了，就算我们会使用缓存，useCallback 第一个参数的内存的开销也是要的。
>
> 建议：如果组件的渲染是复杂的diff操作，可以考虑使用useCallback。

```js
import React from 'react';

// eslint-disable-next-line react/display-name
const Button = React.memo(({ onClick, children }) => {
  console.log('Button call Render');
  return <button onClick={onClick}>{children}</button>;
}, (prev, next) => prev.onClick === next.onClick);


export const UseCallbackDemo = () => {
  const [count, setCount] = React.useState(0);
  const [count2, setCount2] = React.useState(0);

  const clickBtn = () => {
    setCount2(count2 + 1);
  };

  const clickBtn2 = React.useCallback( () => {
    setCount(count + 1);
  }, [count]);

  return (
    <>
      <Button
        onClick={clickBtn}>
            noUseCallback
      </Button>

      <Button
        onClick={clickBtn2}>
              useCallback
      </Button>
    </>
  );
};
```

###  七、lazy 与 Suspense

> 组件按需加载的使用方法

```js
import React, { lazy, Suspense } from 'react';

const MyComponent = lazy(() => import('./button'));

export const LazyAndSuspenseDemo = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <MyComponent />
  </Suspense>
);
```

### 八、 不可变数据