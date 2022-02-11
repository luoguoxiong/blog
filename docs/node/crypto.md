# crypto模块 资源加解密

> crypto模块的目的是为了提供通用的加密和哈希算法。用纯JavaScript代码实现这些功能不是不可能，但速度会非常慢。
>
> Nodejs用C/C++实现这些算法后，通过cypto这个模块暴露为JavaScript接口，这样用起来方便，运行速度也快。

### 对Buffer进行加解密

```js
'use strict';
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const ALGOKEY = 'aes-256-ctr',
  CIPHERKEY = '926c1cb66873c42de5de5409c04d68ab',
  BINARYIV = '1bae14caf86c43de';

// 数据加解密 isDecrypt 是否加密
const cryptFileWithSalt = (
  dataBuffer,
  isDecrypt = false,
) => {
  if (!isDecrypt) {
    const cipher = crypto.createCipheriv(ALGOKEY, CIPHERKEY, BINARYIV);
    return Buffer.concat([cipher.update(dataBuffer), cipher.final()]);
  } else {
    const cipher = crypto.createDecipheriv(ALGOKEY, CIPHERKEY, BINARYIV);
    return Buffer.concat([cipher.update(dataBuffer), cipher.final()]);
  }
};
```

### 远程资源转换为Buffer

```js
// 从远程连接获取资源Buffer
const getBufferFormLink = (url) => new Promise((resolve, reject) => {
  try {
    const chunks = [];
    let size = 0;
    const httpStream = request({
      method: 'GET',
      url,
    });

    httpStream.on('data', (chunk) => {
      chunks.push(chunk);
      size += chunk.length;
    });

    httpStream.on('close', () => {
      const dataBuffer = Buffer.concat(chunks, size);
      resolve(dataBuffer);
    });
  } catch (error) {
    reject(error);
  }
});
```

### 远程资源进行加密

```js
const cryptFileFormUrl = (url) => getBufferFormLink(url).then((originBuffer) => cryptFileWithSalt(originBuffer, true));
```

