# 数据校验之 JSON Schema

数据校验是日常开发中的常见需求。在客户端进行数据校验可以有更好的交互体验，给予更清晰的反馈文案，并且提前预警，节省服务器端资源。
在开发脚手架时，经常会遇到比较多的JSON定义字段，可以通过JSON Schema与ajv 进行配置文件的校验.


### JSON Schema

JSON(JavaScript Object Notation)作为一种简单的数据交换格式被广泛应用。基于简单的数据类型可以表示各种结构化数据。 
JSON schema则是定义JSON数据的模式，即约束JSON数据有哪些字段、其值是如何表示的。
JSON schema本身用JSON编写，且遵循一定规范，需要使用其他语言编写好的程序来解析与验证。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
    "first_name": { "type": "string" },
    "last_name": { "type": "string" },
    "birthday": { "type": "string", "format": "date" },
    "address": {
      "type": "object",
      "properties": {
        "street_address": { "type": "string" },
        "city": { "type": "string" },
        "state": { "type": "string" },
        "country": { "type" : "string" }
      }
    }
  }
}
```
### ajv 校验Schema
```js
// or ESM/TypeScript import
import Ajv from "ajv"
// Node.js require:
const Ajv = require("ajv")

const ajv = new Ajv() // options can be passed, e.g. {allErrors: true}

const schema = {
  type: "object",
  properties: {
    foo: {type: "integer"},
    bar: {type: "string"}
  },
  required: ["foo"],
  additionalProperties: false,
}

const data = {
  foo: 1,
  bar: "abc"
}

const validate = ajv.compile(schema)
const valid = validate(data)
if (!valid) console.log(validate.errors)
```

### 参考文档
https://json-schema.org/

https://ajv.js.org/guide/getting-started.html

https://json-schema.apifox.cn/

https://segmentfault.com/a/1190000040652111
