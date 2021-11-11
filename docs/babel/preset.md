# 搭建统一的Babel preset

1. 依循命名约定 babel-preset-* 来创建一个新项目（请务必对这个命名约定保持责任心，也就是说不要滥用这个命名空间）

2. 创建一个 package.json，包括针对预设所必要的 dependencies。
```json
{
    "name": "babel-preset-my-awesome-preset",
    "version": "1.0.0",
    "main": "./index.js",
    "dependencies": {
        "babel-preset-es2015": "^6.3.13",
        "babel-preset-react": "^6.3.13",
        "babel-plugin-transform-flow-strip-types": "^6.3.15"
    }
}
```

3. 创建 index.js 文件用于导出 .babelrc 的内容，使用对应的 require 调用来替换 plugins／presets 字符串。
```js
module.exports = {
    presets: [
        require("babel-preset-es2015"),
        require("babel-preset-react")
    ],
    plugins: [
        require("babel-plugin-transform-flow-strip-types")
    ]
};
```