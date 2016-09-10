# JavaScript 中的糟粕

这里是一些你必须知道的 JavaScript 的糟糕的（引起误解的）部分。

> 注意：TypeScript 是 JavaScript 的一个超集。 只是有了可以实际被编译器或者 IDEs 使用的文档。;)

## Null 和 Undefined

事实是你将需要都处理这两者。只要使用 `==` 来检查两者。

```ts
/// 想像一下你要做 `foo.bar == undefined` 而 bar 可能是下面其中之一：
console.log(undefined == undefined); // true
console.log(null == undefined); // true
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```
推荐使用 `== null` 来检查 `undefined` 和 `null`。你通常不会想要区分这两者。

## undefined

还记得我说了你应该使用 `== null`。你当然记得啦（因为我刚刚才说过 ^）。别在根层次上使用它。在严格模式中如果你使用 `foo` 而 `foo` 是 undefined，你会得到一个 `ReferenceError` **错误** 以及整个调用堆栈的展开。

> 你应该使用严格模式 ... 实际上如果你使用模块的话 TS 编译器会为你插入 ... 在这本书后面会介绍更多所以你不需要对此十分清晰 :)

因此要检查一个*全局*层次的变量是否被定义，你通常使用 `typeof`：

```ts
if (typeof someglobal !== 'undefined') {
  // someglobal 现在可以安全地使用
  console.log(someglobal);
}
```

## this

任何在一个函数中对 `this` 的访问实际上由函数被调用的方式来控制。这通常被称为`调用上下文`。

这是一个例子：

```ts
function foo() {
  console.log(this);
}

foo(); // 输出 global 变量。例如在浏览器中是 `window`
let bar = {
  foo
}
bar.foo(); // 输出 `bar` 因为 `foo` 在 `bar` 上被调用
```

所以要注意你对于 `this` 的使用。如果你想要在一个类中从调用上下文里断开 `this`，可以使用箭头函数，[之后会讲述更多][arrow]。

[arrow]:../arrow-functions.md

## 接下来

就是这样。那些是 JavaScript 中引起误解的部分，它们仍然让这门语言的新手产生了很多的 bugs 🌹。