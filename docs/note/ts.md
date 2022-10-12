# typeScript 不常用的一些技巧

## 交叉类型
```typescript
type Fn<T, U> = (first:T, second:U) => T&U


const fn:Fn<{a?:string}, {b?:string}> = (first, second) => {
  const result = {};
  for (const id in first) {
    result[id] = first[id];
  }
  for (const id in second) {
    result[id] = second[id];
  }
  return result;
};
```
### 类型别名

```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

### 索引类型
```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]

function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}

let name: string = getProperty(person, 'name');
let age: number = getProperty(person, 'age');
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```
### 映射类型
```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```