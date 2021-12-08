# webpack优化性能

### 一、[critters](https://github.com/GoogleChromeLabs/critters)

> 此webpack插件能够过滤掉没有使用到样式
>
> 1. Fast - no browser, few dependencies
> 2. Integrates with Webpack [critters-webpack-plugin](https://github.com/GoogleChromeLabs/critters/tree/main/packages/critters-webpack-plugin)
> 3. 支持预加载和/或内联关键字体
> 4. 修剪未使用的 CSS 关键帧和媒体查询
> 5. 从延迟加载的样式表中删除内联的 CSS 规则

```js
import Critters from 'critters';

const critters = new Critters({
  // optional configuration (see below)
});

const html = `
  <style>
    .red { color: red }
    .blue { color: blue }
  </style>
  <div class="blue">I'm Blue</div>
`;

const inlined = await critters.process(html);

console.log(inlined);
// "<style>.blue{color:blue}</style><div class=\"blue\">I'm Blue</div>"
```

### 二、[prerender-loader](https://github.com/GoogleChromeLabs/prerender-loader)

> 此webpack插件主要用于单页应用预渲染，提高首屏体验
>
> 1. Webpack 中工作 与 html-webpack-plugin 集成
> 2.  与 webpack-dev-server / webpack serve 一起使用
> 3.  支持 DOM 和字符串预渲染 
> 4. 通过 async/await 或 Promises 进行异步渲染

### 三、[fontmin](https://github.com/ecomfe/fontmin)

> fontmin 可以压缩字体，缩小字体包
>
> 可以封装成webpack插件，把本地的字体包进行压缩

```js
var Fontmin = require('fontmin');

var fontmin = new Fontmin()
    .src('fonts/*.ttf')
    .dest('build/fonts');

fontmin.run(function (err, files) {
    if (err) {
        throw err;
    }

    console.log(files[0]);
    // => { contents: <Buffer 00 01 00 ...> }
});
```

### 四、preload和prefetch

> 利用浏览器的preload和prefetch支持，实现资源的预加载。
>
> 通过给异步加载的资源设置webpackPrefetch、webpackPreload 为true
>
> `webpackPrefetch` 会在浏览器闲置下载文件，`webpackPreload` 会在父 chunk 加载时并行下载文件。

```js
const { default: _ } = await import(/* webpackChunkName: "lodash" */ /* webpackPrefetch: true */ 'lodash');
```

### 五、代码分割和代码压缩

> 代码分割：通过抽离某个模块的代码，来减少总包的大小，提高首屏加载性能
>
> 代码压缩：减少代码体积的大小

```json
optimization: {
    // 代码压缩
    minimizer: [
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourceMap: false
      }),
      new OptimizeCSSAssetsPlugin({})
    ],

    // 代码分割处理
    splitChunks: {
      cacheGroups: {
        // 把src/common公共文件打包到common下
        common: {
          test: /[\\/]Common[\\/]/,
          name: 'common',
          chunks: 'all',
          minSize: 0,
          priority: -20
        }
      }
    }
  }
```

