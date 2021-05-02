# 建造者模式

> - 创建时有很多必填参数需要验证。
> - 创建时参数求值有先后顺序、相互依赖。
> - 创建有很多步骤，全部成功才能创建对象。

```ts
class Programmer {
  age: number
  username: string
  color: string
  area: string

  constructor(p) {
    this.age = p.age
    this.username = p.username
    this.color = p.color
    this.area = p.area
  }

  toString() {
    console.log(this)
  }
}

class Builder {
  age: number
  username: string
  color: string
  area: string

  build() {
    if (this.age && this.username && this.color && this.area) {
      return new Programmer(this)
    } else {
      throw new Error('缺少信息')
    }
  }

  setAge(age: number) {
    if (age > 18 && age < 36) {
      this.age = age
      return this
    } else {
      throw new Error('年龄不合适')
    }
  }

  setUsername(username: string) {
    if (username !== '小明') {
      this.username = username
      return this
    } else {
      throw new Error('小明不合适')
    }
  }

  setColor(color: string) {
    if (color !== 'yellow') {
      this.color = color
      return this
    } else {
      throw new Error('yellow不合适')
    }
  }

  setArea(area: string) {
    this.area = area
    return this
  }
}

// test
const p = new Builder()
  .setAge(20)
  .setUsername('小红')
  .setColor('red')
  .setArea('hz')
  .build()
  .toString()
```

