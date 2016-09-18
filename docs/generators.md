## Generators

> 注意：你不能在 TypeScript 中以有意义的方式使用 generators（ES5 的转换器正在进行中）。然而这将会很快改变，所以我们仍然有这一章。

`function *` 是用来创建 *generator 函数*的语法。调用一个 generator 函数返回的是一个 *generator 对象*。Generator 函数有两个主要动机。Generator 对象遵循 [迭代器][iterator] 接口（即 `next`，`return` 和 `throw` 函数）。

### 懒迭代器

Generator 函数能够用于创建懒迭代器，例如下面的函数返回了一个需要的**无限**整数列表。

```ts
function* infiniteSequence() {
    var i = 0;
    while(true) {
        yield i++;
    }
}

var iterator = infiniteSequence();
while (true) {
    console.log(iterator.next()); // { value: xxxx, done: false } 一直持续下去
}
```

当然如果迭代器结束了，你可以获得 `{done:true}` 的结果，如下所示：

```ts
function* idMaker(){
  let index = 0;
  while(index < 3)
    yield index++;
}

let gen = idMaker();

console.log(gen.next()); // { value: 0, done: false }
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { done: true }
```

### 外部控制执行
这是 generators 中令人兴奋的部分。它实质上允许了一个函数暂停它的执行并且把它剩余函数执行的控制权（命运）传递给调用者。

在你调用时 generator 函数不会执行。它知识创建了一个 generator 对象。考虑下面这个例子的执行：

```ts
function* generator(){
    console.log('Execution started');
    yield 0;
    console.log('Execution resumed');
    yield 1;
    console.log('Execution resumed');
}

var iterator = generator();
console.log('Starting iteration'); // 这会在 generator 函数体的任何东西执行之前执行
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

如果你运行它，你会获得这样的输出：

```
$ node outside.js
Starting iteration
Execution started
{ value: 0, done: false }
Execution resumed
{ value: 1, done: false }
Execution resumed
{ value: undefined, done: true }
```

* 函数只在 generator 对象上调用 `next` 时开始执行。 
* 函数在遇到 `yield` 语句的时候*暂停*。
* 函数在 `next` 被调用的时候*恢复*。

> 所以实质上 generator 函数的执行是由 generator 对象控制的。

我们使用 generator 的交流方式通常是 generator 为迭代器返回值的一种方式。JavaScript 中 generators 提供的一个非常强大的特性是它们允许有两种交流的方式！

* 你可以使用 `iterator.next(valueToInject)` 控制 `yield` 表达式的结果值
* 你可以使用 `iterator.throw(error)` 来在 `yield` 表达式的点抛出错误

下面的例子展示了 `iterator.next(valueToInject)`：

```ts
function* generator() {
    var bar = yield 'foo';
    console.log(bar); // bar!
}

const iterator = generator();
// 开始执行直至我们获得了第一个 yield 值
const foo = iterator.next();
console.log(foo.value); // foo
// 插入 bar，恢复执行
const nextThing = iterator.next('bar');
```

下面的例子展示了 `iterator.throw(error)`：

```ts
function* generator() {
    try {
        yield 'foo';
    }
    catch(err) {
        console.log(err.message); // bar!
    }
}

var iterator = generator();
// 开始执行直至我们获得了第一个 yield 值
var foo = iterator.next();
console.log(foo.value); // foo
// 抛出错误 'bar'，恢复执行
var nextThing = iterator.throw(new Error('bar'));
```

这是几点总结：

* `yield` 允许一个 generator 函数暂停它的执行并且把控制权传递给外部系统
* 外部系统可以推入一个值到 generator 函数体中
* 外部系统可以抛出一个错误到 generator 函数体中

这有什么用处？跳到下一章 [**async/await**][async-await] 去寻找吧。

[iterator]:./iterators.md
[async-await]:./async-await.md
