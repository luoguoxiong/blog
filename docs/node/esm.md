# Node.js 12 支持最新 ECMAScript 模块
Node.js 12(发布于 2019/04/23)带来 ECMAScript 模块的改进支持。这一支持在通常的 flag 后面添加 --experimental-modules 开启。


简单说明：文件名扩展 .mjs 更方便，但是 ES 模块也允许 .js。

CommonJS 模块(CJS): 最早适用于 Node.js 模块的标准。

ECMAScript 模块(ES 模块, ESM): ECMAScript 规范提供的模块标准。

package.prop 意思是 package.json 的属性 prop。

## 1. 模块说明符
模块说明符是标识模块的字符串。它们在 CommonJS 模块和 ES 模块中的工作方式略有不同。在我们查看差异之前，我们需要了解模块说明符的不同类别。

### 1.1 模块说明符的分类
在 ES 模块中，我们把说明符分为下面的类别。这些分类起源于 CommonJS 模块。

1. 相对路径：以小数点开始。举例：
```shell
'./some/other/module.mjs'
'../../lib/counter.mjs'
```
2. 绝对路径：以斜杠开始。举例：
```shell
'/home/jane/file-tools.mjs'
```
3. URL：包含协议(从技术上说，路径也是 URLs)。举例：
```shell
'https://example.com/some-module.mjs'
'file:///home/john/tmp/main.mjs'
```
5. 裸路径：不是以小数点、斜杠或协议开始，并且由不包含扩展名的单文件名组成。举例：
```shell
'lodash'
'the-package'
```
6. 深入导入路径：以裸路径开始，且至少有一个斜杠。举例：
```shell
'the-package/dist/the-module.mjs'
```

### 1.2 CommonJS 模块说明符
CommonJS 怎么处理模块说明符：

* CommonJS 不提供 URLs 作为说明符。
* 相对路径和绝对路径按照预期处理。
* 可以加载一个目录 foo 作为模块：
    * 如果目录中存在文件 foo/index.js
    * 如果目录存在文件 foo/package.json，且它的属性 "main" 指向一个模块文件。
* 裸路径和深导入路径的解析最近的目录 node_modules：
    * 与导入模块相同的目录
    * 在目录的父目录中
    * 其它

### 1.3 文件扩展名
Node.js 支持下面的默认文件扩展名：

* ES 模块的 .mjs
* CommonJS 模块的 .cjs
文件扩展名 .js 在 ESM 或者 CommonJS 两者之中任一都有效。具体取决于 package.type 属性，有两种设置：
* CommonJS(默认情况)：扩展名 .js 或者没有扩展名的文件被解析为 CommonJS。
```js 
"type": "commonjs"
```
* 模块：扩展名 .js 或者没有扩展名的文件被解析为 ESM。
```js
"type": "module"
```

### 2. 互操作性
### 2.1 在 ESM 中导入 CommonJS
在此时，你有两个选项从 ES 模块中导入 CommonJS 模块：

考虑下面的 CommonJS 模块：

```js
// common.cjs
module.exports = {
  foo: 123,
};
```
第一个选项是默认导入(未来可能添加命名导入)：

```js
// es1.mjs
import common from './common.cjs'; // default import
console.log(common.foo); // output: 123
```

第二个选项是使用 createRequire()：
```js
// es2.mjs
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

const common = require('./common.cjs');
console.log(common.foo); // output: 123
```
### 2.1 在 CommonJS 中导入 ESM
如果你想要在 CommonJS 模块中导入 ES 模块，你可以使用 import() 操作符。
使用下面的 ES 模块作为例子：

```js
// es.mjs
export const bar = 'abc';
```

这里我们在 CommonJS 模块中导入它：

```js
// common.cjs
async function main() {
  const es = await import('./es.mjs');
  console.log(es.bar); // output: 'abc'
}
main();
```

在 Node.js 中使用 ES 模块
在使用 Node.js 12 之前，你在 Node.js 中有以下选项来使用 ES 模块：

通过库： John-David Dalton 所作的 esm。esm 也支持 Node.js 的旧版本

开启 flag --experimental-modules 

为了支持 ESM 的该 flag 可能在 2019 年十月被移除，当 Node.js 12 到达 LTS 状态的时候。