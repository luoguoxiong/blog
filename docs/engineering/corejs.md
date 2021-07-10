# 你了解core-js吗

core-js是一个JavaScript标准库，包含了ECMAScript2020在内的多项特性的polyfills,以及ECMAScript在proposals阶段特性、W3C新特性等，它是一个现代化前端项目的“标准套件”。

core-js是由[core-js-builder](https://github.com/zloirock/core-js/tree/master/packages/core-js-builder)、[core-js-bundle](https://github.com/zloirock/core-js/tree/master/packages/core-js-bundle)、[core-js-compat](https://github.com/zloirock/core-js/tree/master/packages/core-js-compat)、[core-js-pure](https://github.com/zloirock/core-js/tree/master/packages/core-js-pure)、[core-js](https://github.com/zloirock/core-js/tree/master/packages/core-js)五个包组成。

core-js是实现基础垫片的能力，也是整个core-js的逻辑核心。

core-js-compat用于通过 browserslist 查询获取所需的 core-js 模块列表,一般用于Babel生态，通过业务需求配置需要的按需加载垫片。

```js
const {
  list,                  // array of required modules
  targets,               // object with targets for each module
} = require('core-js-compat')({
  targets: '> 2.5%',     // browserslist query or object of minimum environment versions to support
  filter: /^(es|web)\./, // optional filter - string-prefix, regexp or list of modules
  version: '3.15',       // used `core-js` version, by default - the latest
});
```

core-js-builder用于生成制定浏览器的 polyfill。这个 API 有助于有条件地包含或排除 core-js 的某些部分，使用来自 core-js-compat 包的浏览器列表查询。

```js
require('core-js-builder')({
  modules: ['es', 'esnext.reflect', 'web'],         // modules / namespaces, by default - all `core-js` modules
  exclude: ['es.math', 'es.number.constructor'],    // a blacklist of modules / namespaces, by default - empty list
  targets: '> 0.5%',                                // optional browserslist query
  filename: './my-core-js-bundle.js',               // optional target filename, if it's missed a file will not be created
}).then(code => {                                   // code of result polyfill
  // ...
}).catch(error => {
  // ...
});
```

## 最佳的Profill方案

```json
{
  "presets": [
    ["@babel/env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      },
      "useBuiltIns": "usage" 
    }],
    "@babel/stage-2"
  ],
  "plugins": ["@babel/transform-runtime"]
}
```

关键点是`useBuiltIns`这一配置项，它的值有三种：

- `false`: 不对`polyfills`做任何操作
- `entry`: 根据`target`中浏览器版本的支持，将`polyfills`拆分引入，仅引入有浏览器不支持的`polyfill`
- `usage(新)`：检测代码中`ES6/7/8`等的使用情况，仅仅加载代码中用到的`polyfills`