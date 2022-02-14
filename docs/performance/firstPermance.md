# 首屏性能捕获

> 对于当页应用如果通过DOMContentLoaded事件，是无法正确衡量首屏性能的。
>
> 针对单页这种情况，可以通过onLoad事件执行完后，再通过获取首屏内的img标签，当首屏内的img都加载完成后，才是正确的首屏渲染时间。

```js
// 首屏渲染时间计算
function logFirstScreen(){
  const images = document.getElementsByTagName('img');
  const iLen = images.length;
  let curMax = 0;
  let inScreenLen = 0;

  function imageBack(){
    this.removeEventListener('load', imageBack);
    if(++curMax === inScreenLen){
      log();
    }
  }

  function log(){
    const info = new Date() - window.performance.timing.navigationStart;
    console.log(info);
  }

  for(let i = 0;i < iLen;i++){
    const img = images[i];
    const offset = {
      top: 0,
    };

    let curImg = img;

    while(curImg.offsetParent){
      offset.top += img.offsetTop;
      curImg = curImg.offsetParent;
    }

    if(document.documentElement.clientHeight < offsetTop){
      continue;
    }

    if(!img.complete){
      inScreenLen++;
      img.addEventListener('load', imageBack);
    }
  }

  if(inScreenLen === 0){
    log();
  }
}
```