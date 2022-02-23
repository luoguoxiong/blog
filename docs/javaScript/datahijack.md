# 数据劫持

### Object.defineProperty

不支持数据类型数据的劫持，需要重写数组操作方法进行劫持。

```js
const aryMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
const arrayAugmentations = [];

aryMethods.forEach((method) => {

  const original = Array.prototype[method];

  arrayAugmentations[method] = function() {
    console.log('数组', this, method);
    return original.apply(this, arguments);
  };

});

const observer = (data) => {
  if(Array.isArray(data)) {
    data.__proto__ = arrayAugmentations;
    return;
  };
  if(!data || typeof data !== 'object') return;
  Object.keys(data).forEach((key) => {
    let value = data[key];
    observer(value);
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: false,
      get(){
        console.log(`get=>${key}`, value);
        return value;
      },
      set(newValue){
        console.log(`set=>${key}`, newValue);
        value = newValue;
      },
    });
  });
};

const obj = {
  arr: [1, 2, 3],
  age: 8,
  type: {
    s: 2,
  },
};

observer(obj);

obj.age = 7;

obj.type.s = 4;

obj.arr.push(4);

console.log(obj.age);
```

### Proxy

性能更好的数据劫持处理方式

```js
const observer2 = (data) => {
  Object.keys(data).forEach((item) => {
    let value = data[item];
    if(typeof value === 'object'){
      data[item] = new Proxy(value, {
        set: (target, property, value, review) => {
          if(property !== 'length'){
            console.log('set key', item);
          }
          return Reflect.set(target, property, value, review);
        },
      });
    }else{
      Object.defineProperty(data, item, {
        enumerable: true,
        configurable: false,
        set(newValue){
          console.log('set key', item);
          value = newValue;
        },
      });
    }
  });
};

observer2(obj);

obj.age = 8;

obj.type.s = 4;

obj.arr.push(4);
```

