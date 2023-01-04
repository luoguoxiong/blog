# IOS调试H5页面

1. 安装 `brew`

```shell
/usr/bin/ruby -e "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install)"
```
2. 安装 `ios-webkit-debug-proxy`、 `remotedebug-ios-webkit-adapter`

```shell
brew install ios-webkit-debug-proxy
npm install remotedebug-ios-webkit-adapter -g
```
3. 手机端开启safair web检查器

设置 --> safari --> 高级 --> web检查器

4. 打开chrome，然后输入以下地址进行连接（在次之前记得打开手机safair）

5. 打开谷歌浏览器 输入`chrome://inspect/#devices`，找到第二个选项`Discover network targets`,点击后面的`Discover network targets`,加入预设的Port位置

6. 执行 `remotedebug_ios_webkit_adapter`

7. 找到 `Remote Target #LOCALHOST`，点击对应的网站的`inspect`