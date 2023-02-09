# webpack按需加载原理

## 基本环境
```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```
```js
// a.js
const a = 'a模块';
export default a;

// index.js
import('./a').then((data) => {
  console.log(data);
});
```

## 打包产物
```js
//  bundle.js 把index.js中的import()语句变成了这个样子
__webpack_require__
  .e(/*! import() */ 0)
  .then(__webpack_require__.bind(null, /*! ./a */ './a.js'))
  .then((data) => {
    console.log(data);
  });
```

## 分析代码

*webpack_require.e*

```js
// 定义installedChunks，用来存储加载过的js信息
/******/ 	var installedChunks = {
/******/ 		"main": 0
/******/ 	};

/******/ 	__webpack_require__.e = function requireEnsure(chunkId) {
/******/     	        // 定义一个存储promise的数组
/******/ 		var promises = [];
/******/
/******/ 		// JSONP chunk loading for javascript
/******/		// installedChunks为一个对象，用来存储加载过的js信息
/******/ 		var installedChunkData = installedChunks[chunkId];
/******/ 		if(installedChunkData !== 0) { // 0代表已经加载过了
/******/
/******/ 			// 如果已经存在不为0，则代表正在加载
/******/ 			if(installedChunkData) {
    					// installedChunkData[2]存储的是正在加载中的promise
/******/ 				promises.push(installedChunkData[2]);
/******/ 			} else {
/******/ 				// 定义一个promise
/******/ 				var promise = new Promise(function(resolve, reject) {
/******/ 					installedChunkData = installedChunks[chunkId] = [resolve, reject];
/******/ 				});
    					// 存储promise
/******/ 				promises.push(installedChunkData[2] = promise);
/******/
/******/ 				// 创建script标签，开始加载js
/******/ 				var script = document.createElement('script');
/******/ 				var onScriptComplete;
/******/
/******/ 				script.charset = 'utf-8';
    					// 设置一个超时时间
/******/ 				script.timeout = 120;
/******/ 				if (__webpack_require__.nc) {
/******/ 					script.setAttribute("nonce", __webpack_require__.nc);
/******/ 				}
    					// 获取src，并赋值
/******/ 				script.src = jsonpScriptSrc(chunkId);
/******/
/******/ 				// 创建一个error，在加载出错后返回
/******/ 				var error = new Error();
    					// 定义加载完成后的时间
/******/ 				onScriptComplete = function (event) {
/******/ 					// avoid mem leaks in IE.
/******/ 					script.onerror = script.onload = null;
/******/ 					clearTimeout(timeout);
    						// 判断是否加载成功
/******/ 					var chunk = installedChunks[chunkId];
    						// 不成功，进行错误处理
/******/ 					if(chunk !== 0) {
/******/ 						if(chunk) {
/******/ 							var errorType = event && (event.type === 'load' ? 'missing' : event.type);
/******/ 							var realSrc = event && event.target && event.target.src;
/******/ 							error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
/******/ 							error.name = 'ChunkLoadError';
/******/ 							error.type = errorType;
/******/ 							error.request = realSrc;
/******/ 							chunk[1](error);
/******/ 						}
/******/ 						installedChunks[chunkId] = undefined;
/******/ 					}
/******/ 				};
/******/ 				var timeout = setTimeout(function(){
/******/ 					onScriptComplete({ type: 'timeout', target: script });
/******/ 				}, 120000);
    					// 加载成功和失败都走onScriptComplete，具体原因看下文
/******/ 				script.onerror = script.onload = onScriptComplete;
/******/ 				document.head.appendChild(script);
/******/ 			}
/******/ 		}
    			// 返回promise
/******/ 		return Promise.all(promises);
/******/ 	};
```

这段代码总共干了这几件事：

1. 定义一个promise数组，用来存储promise.
2. 判断是否已经加载过，如果加载过，返回一个空数组的promise.all().
3. 如果正在加载中，则返回存储过的此文件对应的promise.
4. 如果没加载过，先定义一个promise，然后创建script标签，加载此js，并定义成功和失败的回调
5. 返回一个promise

只看这个函数，我们可能还有一下疑问：

判断有无加载过是通过判断installedChunks[chunkId]的值是否为0，但在script.onerror/script.onload回调函数中并没有把installedChunks[chunkId]的值置为0
promise 把 resolve 和 reject 全部存入了 installedChunks 中， 并没有在获取异步chunk成功的onload 回调中执行 resolve，那么，resolve 是什么时候被执行的呢?

针对这两个问题，我们需要看一下打包后的另一个文件：

*0.bundle.js*

```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([[0],{

/***/ "./a.js":
/*!**************!*\
  !*** ./a.js ***!
  \**************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\nconst a = 'a模块';\r\n/* harmony default export */ __webpack_exports__[\"default\"] = (a);\n\n//# sourceURL=webpack:///./a.js?");

/***/ })

}]);
```
我们看到，在此文件中，会执行window["webpackJsonp"].push()方法，即每次加载完一个文件，就会执行全局的webpackJsonp数组的push方法，此push方法就是关键。

*bundle.js*

```js
/******/    // 定义全局数组window["webpackJsonp"]，并重写window["webpackJsonp"]的push方法 	
/******/    var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
/******/ 	var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
/******/ 	jsonpArray.push = webpackJsonpCallback;

		// 重写window["webpackJsonp"]的push方法 
/******/ 	function webpackJsonpCallback(data) {
/******/ 		var chunkIds = data[0];
/******/ 		var moreModules = data[1];
/******/
/******/
/******/ 		// add "moreModules" to the modules object,
/******/ 		// then flag all "chunkIds" as loaded and fire callback
/******/ 		var moduleId, chunkId, i = 0, resolves = [];
/******/ 		for(;i < chunkIds.length; i++) {
/******/ 			chunkId = chunkIds[i];
/******/ 			if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
    					// 获取此js文件对应的promise中的resolve方法数组
/******/ 				resolves.push(installedChunks[chunkId][0]);
/******/ 			}
    				// 把installedChunks[chunkId] 置为0，代表已经加载过
/******/ 			installedChunks[chunkId] = 0;
/******/ 		}
/******/ 		for(moduleId in moreModules) {
/******/ 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
/******/ 				modules[moduleId] = moreModules[moduleId];
/******/ 			}
/******/ 		}
/******/ 		if(parentJsonpFunction) parentJsonpFunction(data);
/******/		
    			// 执行此js文件对应的promise中的resolve方法
/******/ 		while(resolves.length) {
/******/ 			resolve.shift()();
/******/ 		}
/******/
/******/ 	};
```

1. 定义全局数组window["webpackJsonp"]，并重写window["webpackJsonp"]的push方法
2. 在新的push方法中，把installedChunks[chunkId]置为0，代表已经加载过，并执行js对应的promise的resolve方法