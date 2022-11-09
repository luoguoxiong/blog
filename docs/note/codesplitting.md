# CodeSplitting

## code splitting 的策略方案

1. bigVendors 是大 vendors 方案，会将 async chunk 里的 node_modules 下的文件打包到一起，可以避免重复。同时缺点是，`单文件的尺寸过大`、`毫无缓存效率可言`

2. depPerChunk 和 bigVendors 类似，不同的是把依赖按 package name + version 进行拆分，算是解了 bigVendors 的尺寸和缓存效率问题。但同时带来的潜在问题是，可能导致请求较多。我的理解是，对于非大型项目来说其实还好，因为`单个页面的请求不会包含非常多的依赖`、`基于 HTTP/2，几十个请求不算问题。但是，对于大型项目或巨型项目来说，需要考虑更合适的方案`。

3. granularChunks 自定义某些包进行分组，对与固定版本的npm可以进行抽离，`充分发挥缓存效率`。

### bigVendors
```js
{
    cacheGroups: {
        vendors: {
        test: /[\\/]node_modules[\\/]/,
        priority: 10,
        name: 'vendors',
        chunks: 'async',
        },
    },
}
```
### depPerChunk
```js
{
    vendors: {
        test: /[\\/]node_modules[\\/]/,
        priority: 10,
        chunks: 'async',
        name(module: any) {
            // e.g. node_modules/.pnpm/lodash-es@4.17.21/node_modules/lodash-es
            const path = module.context.replace(/.pnpm[\\/]/, '');
            const packageName = path.match(
            /[\\/]node_modules[\\/](.*?)([\\/]|$)/,
            )[1];
            return `npm.${packageName
            .replace(/@/g, '_at_')
            .replace(/\+/g, '_')}`;
        },
    },
}
```
### granularChunksk
可参考：https://github.com/luoguoxiong/hulljs/commit/c43dfbd#diff-eb36d034e9374e3fed16218dd11a36d4fe4d2179693fc716187f1a0921731505
```js
const createGranlarChunks = (library: Record<string, Array<string>>) => {
  const cacheGroups: Record<string, any> = {};
  for(const libraryName in library){
    const packages = library[libraryName];
    cacheGroups[libraryName] = {
      name: libraryName,
      chunks: 'all',
      test: new RegExp(
        `[\\\\/]node_modules[\\\\/](${packages.join(
          '|',
        )})[\\\\/]`,
      ),
      priority: 40,
      enforce: true,
    };
  }
  return {
    default: false,
    defaultVendors: false,
    ...cacheGroups,
  };
};
```