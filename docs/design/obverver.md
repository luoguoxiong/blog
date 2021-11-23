# 观察者与发布订阅模式

### 观察者模式
> 观察者模式可以类比为：`房东-用户模型`。房东有单间、一房一厅、二房一厅的户型。其中Observer为`租户`，Subject为`房东`。
```js
// 观察者（租客）
class Observer {
  constructor(subject) {
    this.subject = subject;
  }
  notify() {
    console.log(`收到一条房东的消息，${this.subject}空了！！！`);
  }
}

// 主题（房东）
class Subject {
  // 根据户型的不同收集相应的订阅者
  constructor() {
    this.subjectList = {};
  }
  // 订阅
  add(subject, observer) {
    if (!this.subjectList[subject]) {
      this.subjectList[subject] = [];
    }
    this.subjectList[subject].push(observer);
  }
  // 解除订阅
  remove(subject, observer) {
    this.subjectList[subject].forEach((item, index) => {
      if (item === observer) {
        this.subjectList[subject].splice(index, 1);
      }
    });
  }
  // 触发事件
  fire(subject) {
    this.subjectList[subject].forEach((item) => item.notify());
  }
}

const observerA = new Observer("单间");
const observerB = new Observer("一房一厅");
const observerC = new Observer("二房一厅");

// 把这3个观察者添加到相应的分组
const subjects = new Subject();
subjects.add("单间", observerA);
subjects.add("一房一厅", observerB);
subjects.add("二房一厅", observerC);

// 房东主动告知各个户型
subjects.fire("单间");
subjects.fire("一房一厅");
subjects.fire("二房一厅");
```
### 发布订阅模式
> 发布订阅模式可以类比为：`房东-中介-租客模型`。 

```js
class EventBus {
  constructor() {
    this.events = {};
  }

  on(name, func) {
    if (!this.events[name]) {
      this.events[name] = [];
    }
    this.events[name].push(func);
  }

  emit(name, ...args) {
    if (!this.events[name]) {
      return;
    }
    this.events[name].forEach((item) => {
      item.apply(this, args);
    });
  }

  remove(name) {
    if (this.events[name]) {
      delete this.events[name];
    }
  }
}

const bus = new EventBus();

// 租客订阅了不同户型
bus.on("单间", function (value) {
  console.log(`A收到了一条消息：${value}`);
});

bus.on("一房一厅", function (value) {
  console.log(`B收到了一条消息：${value}`);
});

bus.on("二房一厅", function (value) {
  console.log(`C收到了一条消息：${value}`);
});

// 房东通过中介发布消息
bus.emit("单间", "单间有房了");
bus.emit("一房一厅", "一房一厅有房了");
bus.emit("二房一厅", "二房一厅有房了");
```

### 总结
> 1. 在观察者模式中，观察者是知道Subject的，Subject一直保持对观察者进行记录。然而，在发布订阅模式中，发布者和订阅者不知道对方的存在。它们只有通过消息代理进行通信。
> 2. 在发布订阅模式中，组件是松散耦合的，正好和观察者模式相反。
> 3. 观察者模式大多数时候是同步的，比如当事件触发，Subject就会去调用观察者的方法。而发布-订阅模式大多数时候是异步的（使用消息队列）。
> 4. 观察者 模式需要在单个应用程序地址空间中实现，而发布-订阅更像交叉应用模式。