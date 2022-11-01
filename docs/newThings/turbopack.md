# Turbopack
gitHub地址：https://github.com/vercel/turbo

文档地址：https://turbo.build/pack/docs

Turbopack 是针对 JavaScript 和 TypeScript 优化的增量打包器，由 Vercel 的 Webpack 和 Next.js 的创建者用 Rust 编写的构建工具。

在大型应用程序上，Turbopack 的更新速度比 Vite 快 10 倍，比 Webpack 快 700 倍。对于最大的应用程序，差异变得更加明显，更新速度比 Vite 快 20 倍。

Turbopack 性能的秘诀是双重的：高度优化的机器代码和低级增量计算引擎，可以缓存到单个函数的级别。一旦 Turbopack 执行了一项任务，它就再也不会这样做了。

## Turbopack 为啥快？
1、基于 Rust 二进制语言

2、内置增量计算引擎。该引擎结合 Turborepo 以及 Google 的 Bazel 的增量计算的创新，可以将缓存提高到单个函数的水平

3、缓存。但现在只支持内存缓存，未来会支持持久化缓存，存文件系统或远程服务器

4、基于请求的按需编译。

## 为啥要写 Turbopack？
1、Vite 这种 Bundless 的方式，由于只要编译一个文件，所以响应式更新速度极快。但在处理模块较多的大型应用时，大量级联的网络请求会导致相对缓慢的启动时间。对于浏览器来说，更多请求也意味着更慢的速度。

2、Turbopack 内置了可重复使用的 Turbo 构建引擎，这类似一个函数调用的调度器，允许并行执行，更重要的是，他会缓存所有调度的函数的结果，这意味着同样的工作永远不需要做两次。

3、为啥不用 esbuild？因为 esbuild 虽然快，但是，1）不支持 HMR，2）没有缓存，这意味着你需要重复做同样的工作，3）不支持懒打包，比如基于路由和请求的按需打包方案。

4、Turbopack 支持在 dev 模式下做懒打包，当一个请求过来，会建立懒模块依赖图，只计算用到的模块，然后打包出单一的块返回给浏览器。作者认为，在大型应用里，这会比 Native ESM 快地多。

## Turbopack 的功能？

1、JavaScript 和 TypeScript 的打包基于 SWC，支持在 package.json 中定义 browserslist 声明需要支持的浏览器，暂不支持 Babel 及其插件，未来会以插件形式支持 Babel

2、TypeScript 支持 tsconfig.json 中声明的 paths 和 baseUrl，类似 webpack 的 alias，但不支持 TypeScript 的类型校验，需要单独执行 tsc --watch 或者依赖 IDE 的 TypeScript 集成能力

3、支持 React 的 JSX 和 RSC，暂不支持 next/dynamic，暂不支持 Vue 和 Svelte，未来会以插件形式支持

4、CSS 基于 SWC（swc_css），支持全局 CSS 和 CSS Modules，约定 .module.css 即为 CSS Modules，内置支持 postcss-nested 语法，所以可以在 CSS 里直接用嵌套语法，支持 @import 语法来引入额外 CSS 文件，暂不支持 PostCSS 插件，但可通过手动执行 postcss-cli 绕过，不支持 SCSS 和 LESS，暂不支持 Tailwind CSS

5、Dev Server 支持 HMR 和 Fast Refresh

6、静态资源支持 import assets 资源、public 目录和 JSON

7、import 语法上支持 CJS 和 ESM，部分支持 AMD，支持动态 require()、type import 和动态 import()

8、环境变量支持 .env 文件并同时支持实时 reload（无需手动重启），支持 process.env 做变量注入

## Turbopack 现状和规划
1、已内置到 NextJS 13，但只支持 dev 模式，未来会支持 build 模式

2、计划集成到 SvelteKit

3、计划支持持久化缓存，包括本地和远程的缓存方式

4、目前还不能从 webpack 迁移到 Turbopack，未来会为所有 webpack 用户提供平滑的迁移路径

5、正在用 Rust 重写 Turborepo，未来会于 Turbopack 合并