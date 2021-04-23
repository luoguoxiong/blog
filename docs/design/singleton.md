# 单例模式

> 确保（一个类）仅有一个实例，并提供全局访问。
>
> 常用的创建单例的方法就是：通过一个可控变量，判断是否已经生成实例。

## 一、单例模式的实现

1. 对象字面量

```js
var Sinleton = {
  attr: 1,
  show() {
    return this.attr
  }
}
```

> 这样创建的单例对象 ：
>
> 1. 对象 Sinleton 确实是独一无二的。
> 2. 如果 Sinleton 变量被声明在全局作用域下，那么我们可以在代码中的任何位置使用这个变量。
>
> 这样就满足了单例模式的两个条件。
>
> 优点：简单且实用
>
> 缺点：
>
> 1. 没有什么封装性，所有的属性方法都是暴露的。对于一些需要使用私有变量的情况就显得心有余而力不足了
> 2. 对于 this 的问题也有一定弊端。

2. 构造函数内部判断

```js
function Singleton(name, age) {
  if(Singleton.unique) {
    return Singleton.unique
  }
  this.name = name
  this.age = age
  Singleton.unique = this
}
```

> 缺点：提出一个属性来做判断，但是也没有安全性，一旦外部修改了Construct的unique属性，那么单例模式也就被破坏了。

3. 闭包

```js
const Singleton = (function() {
  let constance = null
  return function Construt() {
    if(!constance) {
      constance = this
    }
    return constance
  }
})()
```

## 二、常用的使用场景

1. 闭包封装私有变量

```js
const createCache = (()=>{
  let cacheData= {}
  return {
    getData:()=>cacheData,
    setData:(value)=>{
      cacheData = value
    }
  }
})()
```

