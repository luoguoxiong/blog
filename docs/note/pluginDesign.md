# 插件式设计

例如，当我们需要把我们前端代码中的 css 样式提取打包，我们可以用 webpack 的 mini-css-extract-plugin，或者你如果用 rollup 的话，可以选择 rollup-plugin-postcss。

再比如我们可以给 babel 配置 @babel/plugin-proposal-decorators 插件来支持装饰器语法；

除了上述打包编译相关的工具，我们使用的代码编辑器也都支持各式各样的插件，动态地给软件增加各种能力，例如通过 Prettier 插件来是 VsCode 支持 Prettier 的代码格式化，或者安装主题插件来改变软件样式等。

除此之外，一些前端领域的框架也都有插件的机制，譬如 Vue，DvaJS，Eggjs 等。React 也有一些插件化开发的框架，例如 DevExtreme Reactive （以下简称 DR），React Pluggable。

## 何如设计？
设计一个插件系统时，我们要考虑几个问题：

程序中哪些是易变的，哪些是相对稳定的。易变的部分应暴露出相应的能力由插件来完成。

插件如何影响程序。通常会以扩展行为，修改状态，变更展示的方式体现。

一个简易的计算器程序，支持加减法：
```js
class Calculator {
   construct(initial) {
     this.num = initial;
   }
   add(num) {
     this.num = this.num + num;
     return this;
   }
   subtract(num) {
     this.num = this.num - num;
     return this;
   }
   result() {
     return this.num;
   }
 }

 const myCalculator = new Calculator(5);
 myCalculator.add(5).subtract(2).result(); // 8
```
可以很容易想到，计算器的主要抽象是运算，即当前值和一个新值的运算过程，这部分是稳定的，而支持的运算逻辑是可扩展的，适合做成插件，因此我们可以做如下的改造：
```js
// 程序主体，定义程序核心逻辑是增加计算器运算能力
 class Calculator {
   plugins = [];
   construct(initial) {
     this.num = initial;
   }

   use(plugin) {
     this.plugins.push(plugin);
     this[plugin.name] = plugin.calculate.bind(this);
   }

   result() {
     return this.num;
   }
 }
 // 插件声明
 interface Plugin {
   name: string;
   calculate(num: number) => this;
 }
 // 插件实现
 class AddPlugin implements Plugin {
   name: 'add',
   calculate(num) {
     this.num = this.num + num;
     return this;
   }
 }

 class SubtractPlugin implements Plugin {
   name: 'subtract',
   calculate(num) {
     this.num = this.num - num;
     return this;
   }
 }

 const myCalculator = new Calculator(5);
 // 插件安装
 myCalculator.use(new AddPlugin());
 myCalculator.use(new SubtractPlugin());

 myCalculator.add(5).subtract(2).result(); // 8
```
通过上面的例子，从插件的角度可以分成几个部分：

程序主体（Program），即上面的 Calculator；

插件接口声明（Plugin Interface），即上面的 Plugin；

插件实现（Plugin Implementation），即 AddPlugin，SubtractPlugin，MultiplicatiPlugin；