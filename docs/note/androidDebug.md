# Android调试技巧

## 普通浏览器H5页面

1. 手机打开USB调试

2. 打开谷歌浏览器 输入`chrome://inspect/#devices`，找到第二个选项`Discover network targets`,点击后面的`Discover network targets`,加入预设的Port位置

3. 找到 `Remote Target #LOCALHOST`，点击对应的网站的`inspect`

## 微信内置浏览器 （要打开科学上网工具）

1. 手机开启USB调试

2. 微信打开http://debugxweb.qq.com/?inspector=true，然后退出

3. 打开谷歌浏览器 输入`chrome://inspect/#devices`，找到第二个选项`Discover network targets`,点击后面的`Discover network targets`,加入预设的Port位置

4. 找到 `Remote Target #LOCALHOST`，点击对应的网站的`inspect`