# webpack 打包优化

## 一、 分析打包速度

我们可以通过**speed-measure-webpack-plugin** 插件去分析webpack 插件、loader各个阶段花费的时间、各个module的数量。

```js
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

const smp = new SpeedMeasurePlugin();

//将默认的webpack配置文件包裹起来
const webpackConfig = smp.wrap({
  plugins: [
    new MyPlugin(),
    new MyOtherPlugin()
  ]
});
```

![](/Users/peroluo/github/blog/docs/img/speed.jpg)

## 二、分析影响打包速度环节

从webpack的打包原理，可以了解到构建的过程是递归地构建一个依赖关系图(dependency graph),其中包含应用程序需要的每个模块,然后将所有这些模块打包成一个或多个 bundle。

1. 分析所有的依赖项，这需要占用一定的时间，即**搜索时间**。
2. 编译所有的模块，解析成浏览器能运行的时间，既**编译时间**。
3. 依赖分析打包成bundle后，需要压缩代码量，既**压缩时间**。
4. 并不是每次构建，都需要重新打包，既**二次打包时间**。

## 三、优化搜索时间

#### 优化loader配置

在使用loader时，可以通过配置 test、include、exlude 三个配置项来提高loader的命中率。

```js
{
  test: /\.js[x]?$/,
    exclude: /node_modules/,
    use: {
       loader: 'babel-loader',
         options: {
           presets: [
             '@babel/preset-env',
             '@babel/preset-react',
             { plugins: ['@babel/plugin-proposal-class-properties'] }
           ]
       }
    }
},
```

#### 优化resolve.module

`resolve.modules` 用于配置 webpack 去哪些目录下寻找第三方模块，`resolve.modules` 的默认值是 `['node_modules']` ，含义是先去当前目录下的 `./node_modules` 目录下去找想找的模块，如果没找到就去上一级目录 `../node_modules` 中找，再没有就去 `../../node_modules` 中找，以此类推。

```js
resolve: {
  modules: [
    'src/components', 
    'node_modules'
  ],
},
```

####  优化 resolve.alias 配置

`resolve.alias` 配置项通过别名来把原导入路径映射成一个新的导入路径，减少耗时的递归解析操作。

```js
 resolve: {
    alias: {
      '@compontents': 'src/components',
    }
 }
```

## 四、优化编译时间

由于webpack构建时是单线程模式，webpack只能逐步处理文件，在进行构建时，不能充分理由cpu，从而延长构建时间。

#### thread-loader 线程池优化

```js
{
  test: /\.js$/,
  exclude: /node_modules/,
  // 创建一个 js worker 池
  use: [ 
        'thread-loader',
        'babel-loader'
  ] 
}
```

#### happyPack 多进程打包

HappyPack 可利用多进程对文件进行打包(默认cpu核数-1)，对多核cpu利用率更高。HappyPack 可以让 Webpack 同一时间处理多个任务，发挥多核 CPU 的能力，将任务分解给多个子进程去并发的执行，子进程处理完后，再把结果发送给主进程。

**注意，当项目较小时，多进程打包反而会使打包速度变慢。**

## 五、优化压缩时间

#### terser 启动多进程

webpack4 默认内置使用 `terser-webpack-plugin` 插件压缩优化代码，而该插件使用 `terser` 来缩小 `JavaScript` 。

使用多进程并行运行来提高构建速度。并发运行的默认数量为 `os.cpus().length - 1` 。

```js
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true,
      }),
    ],
  },
};
```

**可以显著加快构建速度，因此强烈推荐开启多进程**

## 六、优化二次打包时间

在日常开发中，可能只需要新增或修改某些模块的代码，并不需要全量进行webpack的构建，所以webpack可以通过缓存进行构建优化。常见的缓存方案有：`cache-loader`、`HardSourceWebpackPlugin`

#### cache-loader

cache-loader 和 thread-loader 一样，使用起来也很简单，仅仅需要在一些性能开销较大的 loader 之前添加此 loader，以将结果缓存到磁盘里，显著提升二次构建速度。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.ext$/,
        use: ['cache-loader', ...loaders],
        include: path.resolve('src'),
      },
    ],
  },
};
```

 **请注意，保存和读取这些缓存文件会有一些时间开销，所以请只对性能开销较大的 loader 使用此 loader。**

#### HardSourceWebpackPlugin

- 第一次构建将花费正常的时间
- 第二次构建将显着加快（大概提升90%的构建速度）。

**注意：仅支持开发环境**

## 七、按需编译

能不能通过某种方式实现开发环境只编译需要的模块，类似于next开发环境的那种体验？后续再webpack插件开发补充。