# Babel 配置文件
Babel的配置文件可以通过.babelrc，.babelrc.js，babel.config.js和package.json

## .babelrc
```json
{
    "presets": ["es2015", "react"],
    "plugins": ["transform-decorators-legacy", "transform-class-properties"]
}
```
## babel.config.js和.babelrc.js
```js
 module.exports = {
    "presets": ["es2015", "react"],
    "plugins": ["transform-decorators-legacy", "transform-class-properties"]
}
```
## package.json
```json
{
    "name": "demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "babel": {
      "presets": ["es2015", "react"],
      "plugins": ["transform-decorators-legacy", "transform-class-properties"]
    }
}
```
