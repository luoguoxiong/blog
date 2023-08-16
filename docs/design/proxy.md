# 代理模式

## 一、虚拟代理实现图片预加载
```js
var myImage = (function(){ 
  var imgNode = document.createElement( 'img' ); 
  document.body.appendChild( imgNode ); 
  return { 
    setSrc: function( src ){ 
    imgNode.src = src; 
  } 
  } 
})();

var proxyImage = (function(){ 
  var img = new Image; 
  img.onload = function(){ 
    myImage.setSrc( this.src ); 
  } 
 return { 
  setSrc: function( src ){ 
    myImage.setSrc( 'file:// /C:/Users/svenzeng/Desktop/loading.gif' ); 
    img.src = src; 
  } 
 } 
})(); 
proxyImage.setSrc( 'http:// imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg' );

// 不使用代理
var MyImage = (function(){ 
  var imgNode = document.createElement( 'img' ); 
  document.body.appendChild( imgNode ); 
  var img = new Image; 
  img.onload = function(){ 
  
  }; 
 return { 
  setSrc: function( src ){ 
    imgNode.src = 'file:// /C:/Users/svenzeng/Desktop/loading.gif'; 
    img.src = src; 
  } 
 } 
})(); 
MyImage.setSrc( 'http:// imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg' );
```
为了说明代理的意义，下面我们引入一个面向对象设计的原则——单一职责原则。

单一职责原则指的是，就一个类（通常也包括对象和函数等）而言，应该仅有一个引起它变
化的原因。如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起它变化的原因可
能会有多个。面向对象设计鼓励将行为分布到细粒度的对象之中，如果一个对象承担的职责过多，
等于把这些职责耦合到了一起，这种耦合会导致脆弱和低内聚的设计。当变化发生时，设计可能
会遭到意外的破坏。 

职责被定义为“引起变化的原因”。上段代码中的 MyImage 对象除了负责给 img 节点设置 src
外，还要负责预加载图片。我们在处理其中一个职责时，有可能因为其强耦合性影响另外一个职
责的实现。

另外，在面向对象的程序设计中，大多数情况下，若违反其他任何原则，同时将违反开放—
封闭原则。如果我们只是从网络上获取一些体积很小的图片，或者 5 年后的网速快到根本不再需
要预加载，我们可能希望把预加载图片的这段代码从 MyImage 对象里删掉。这时候就不得不改动
MyImage 对象了。

实际上，我们需要的只是给 img 节点设置 src，预加载图片只是一个锦上添花的功能。如果
能把这个操作放在另一个对象里面，自然是一个非常好的方法。于是代理的作用在这里就体现出
来了，代理负责预加载图片，预加载的操作完成之后，把请求重新交给本体 MyImage。

*简单来说：myImage仅仅只是赋值图片地址，proxyImage是让myImage具有图片预加载功能，两者职能是单一，且不交叉*

## 二、虚拟代理合并请求
多选框选中后发送改文件
```js
var synchronousFile = function( id ){ 
 console.log( '开始同步文件，id 为: ' + id ); 
}; 
var proxySynchronousFile = (function(){ 
  var cache = [], // 保存一段时间内需要同步的 ID 
  timer; // 定时器
  return function( id ){ 
    cache.push( id ); 
    if ( timer ){ // 保证不会覆盖已经启动的定时器
      return; 
    } 
    timer = setTimeout(function(){ 
      synchronousFile( cache.join( ',' ) ); // 2 秒后向本体发送需要同步的 ID 集合
      clearTimeout( timer ); // 清空定时器
      timer = null; 
      cache.length = 0; // 清空 ID 集合
    }, 2000 ); 
  } 
})(); 

var checkbox = document.getElementsByTagName( 'input' ); 
for ( var i = 0, c; c = checkbox[ i++ ]; ){
  c.onclick = function(){ 
    if ( this.checked === true ){ 
      proxySynchronousFile( this.id ); 
    } 
  } 
};
```