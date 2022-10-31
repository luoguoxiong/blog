# IOS 下overflow:hidden 无效

原因： ios系统 认为网页内容是个整体，需要将所有的都显示出来，所有overflow就不该起作用，这是他们刻意的，不是bug，而且在更高版本的ios7,8,9中也是这样设定的.

对viewport我们没法改变，但是我们可以通过改变body自身的overflow：

改变html的overflow默认属性（visible）为auto或者hidden，哪个都行，对后面的效果都是一致的，这样body就不会继承viewport的overflow属性，然后通过设置body的overflow:hidden，大部分的溢出内容就会被隐藏掉
不过设置了position:absolute的除外，因为它默认是相对viewport进行绝对定位的，这个时候你要对body进行设置position:relative
总结起来一句话，body的范围要比viewport的范围要大，这样才能使body的overflow生效

```css
html {
  overflow: hidden;
  height: 100%;
  margin: 0;
  padding: 0;
  border: none;
}

body {
  overflow: hidden;
  position: relative;
  box-sizing: border-box;
  margin: 0;
  height: 100%;
}
```
