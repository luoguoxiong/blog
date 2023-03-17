# Retina屏幕兼容

## 原因

一个图像在标准设备下全屏显示时，一位图像素对应的就是一设备像素，导致一个完全保真的显示，因为一个位置像素不能进一步分裂。而当在Retina屏幕下时，他要放大四倍来保持相同的物理像素的大小，这样就会丢失很多细节，造成失真的情形

## 准备两套图片
```js
$(document).ready(function () {
   if (window.devicePixelRatio > 1) {
      var images = $("img.pic");
      images.each(function(i) {
        var x1 = $(this).attr('src');
        var x2 = x1.replace(/(.*)(.w+)/, "$1@2x$2");
        $(this).attr('src', x2);
      });
  }
});
```

## Image-set控制
```css
#logo {
background: url(pic.png) 0 0 no-repeat;
background-image: -webkit-image-set(url(pic.png) 1x, url(pic@2x.png) 2x);
background-image: -moz-image-set(url(pic.png) 1x,url(images/pic@2x.png) 2x);
background-image: -ms-image-set(url(pic.png) 1x,url(images/pic@2x.png) 2x);
background-image: -o-image-set(url(url(pic.png) 1x,url(images/pic@2x.png) 2x);
}
或者使用HTML代码控制亦可：
![](pic.png)
```

### 媒体查询
```css
@media only screen and (-webkit-min-device-pixel-ratio: 1.5),
       only screen and (min--moz-device-pixel-ratio: 1.5), /* 注意这里的写法比较特殊 */
       only screen and (-o-min-device-pixel-ratio: 3/2),
       only screen and (min-device-pixel-ratio: 1.5) {
            #logo {
            background-image: url(pic@2x.png);
            background-size: 100px auto;
            }
}
```