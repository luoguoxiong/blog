# XSS 跨站脚本攻击

XSS ： Cross Site Scrit 跨站脚本攻击（为与 CSS 区别，所以在安全领域叫 XSS）。

## 攻击原理
代码被恶意注入到页面中（例如评论），然后其他用户在访问页面时，浏览器执行了代码逻辑。

## 防御策略
浏览器自带 X-XSS-Protection
介绍：为 0 是禁止 xss 过滤，为 1 是启用 xss 过滤。
缺点：兼容性、不能完全杜绝 xss
特殊字符转义
介绍：例如<变为 `&lt;`
标签过滤
介绍：仅对安全 html 标签进行渲染，其他不与渲染
内容安全策略（CSP）
介绍：Content Security Policy 入门教程