# IOS调试H5页面

1. 安装 `brew`

```shell
/usr/bin/ruby -e "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install)"
```
2. 安装 `ios-webkit-debug-proxy`

```shell
brew install ios-webkit-debug-proxy
```
3. 手机端开启safair web检查器

设置 --> safari --> 高级 --> web检查器

4. 开启调试

```shell
ios_webkit_debug_proxy
```
5. 打开chrome，然后输入以下地址进行连接（在次之前记得打开手机safair）

6. 先输入地址：http://127.0.0.1:9222  点击对应的链接