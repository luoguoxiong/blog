# 前端构建工具的区别

## 一、gulp

前端构建工具，gulp是基于Nodejs，自动化地完成 javascript、coffee、sass、less、html/image、css 等文件的测试、检查、合并、压缩、格式化、浏览器自动刷新、部署文件生成，并监听文件在改动后重复指定的这些步骤。
借鉴了管道（pipe）思想，前一级的输出，直接变成后一级的输入，使得在操作上非常简单。
gulp基于流式操作，通过各种 Transform Stream 来实现文件不断处理 输出。

**优点：**
gulp文档简单，学习成本低，使用简单；对大量源文件可以进行流式处理，借助插件，可以对文件类型进行多种操作处理。

**缺点：**
不支持tree-shaking、热更新、代码分割等。 gulp 对 js 模块化方案无能为力，只是对静态资源做流式处理，处理之后并未做有效的优化整合。

**适用场景：**
静态资源密集操作型场景，主要用于css、图片等静态资源的处理操作。

## 二、Webpack

Webpack 是一种前端资源模块化 管理和打包 工具。它可以将许多松散的模块按照依赖和规则打包成符合生产环境部署的前端资源。还可以将按需加载的模块进行代码分割，等到实际需要的时候再异步加载。

**优点：**
基本之前gulp 可以进行的操作处理，现在webpack也都可以做。同时支持热更新，支持tree shaking 、Scope Hoisting、动态加载、代码拆分、文件指纹、代码压缩、静态资源处理等，支持多种打包方式等。

**缺点：**
不支持 打包出esm格式的代码 (打包后的代码再次被引用时tree shaking 困难)， 打包后亢余代码多，配置较为复杂。

**适用场景：**
应用程序开发。

## 三、tsc、babel

tsc/babel 可以将 ts 代码编译 js 代码。支持编译成 esm、cjs、amd 格式的文件

**优点：**
编译速度快，可以保留原有的目录相对位置，分目录保存各模块的代码，便于按需引用加载；

**缺点：**
只对语言本身进行编译转换，不支持tree shaking 等高级功能、tsc/babel常与其他工具配合使用。

## 四、Rollup
Rollup 是一个模块打包工具, 可以将我们按照 ESM (ES2015 Module) 规范编写的源码构建输出如下格式:

IIFE: 自执行函数, 可通过 script 标签加载

AMD: 通过 RequireJS 加载

CommonJS: Node 默认的模块规范, 可通过 Webpack 加载

UMD: 兼容 IIFE, AMD, CJS 三种模块规范

ESM: ES2015 Module 规范, 可用 Webpack, Rollup 加载

**优点：**
支持动态导入。

支持tree shaking。仅加载模块里用得到的函数以减小文件大小。

Scope Hoisting。 rollup可以将所有小文件生成到一个大文件中，所有代码都在同一个函数作用域里:， 不会像 Webpack 那样用很多函数来包装模块。

没有其他冗余代码, 执行很快。除了必要的 cjs, umd 头外，bundle 代码基本和源码差不多，也没有奇怪的 __webpack_require__, Object.defineProperty 之类的东西,

**缺点：**
不支持热更新功能；对于commonjs模块，需要额外的插件将其转化为es2015供rollup 处理；无法进行公共代码拆分。

**适用场景：**
由纯js开发的第三方库； 需要生成单一的umd文件的场景

## 五、Esbuild
esbuild 是一个用 Go 语言编写的用于打包，压缩 Javascript 代码的工具库。它最突出的特点就是打包速度极快 （extremely fast）

**优点：**
构建速度快

**缺点：**
esbuild 没有提供 AST 的操作能力,没有插件机制

**适用场景：**
生产环境使用 esbuild 的可行性
