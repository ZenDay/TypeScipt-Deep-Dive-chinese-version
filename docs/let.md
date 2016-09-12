### let

`var` 变量在 JavaScript 中是*函数作用域的*。这跟很多变量是*块级作用域*的其他语言（C#、Java 等等）不一样。如果你用*块级作用域*的心态来看 JavaScript，你会认为下面这段代码会输出 `123`，而实际上会输出 `456`。

```ts
var foo = 123;
if (true) {
    var foo = 456;
}
console.log(foo); // 456
```
这是因为 `{` 没有创建一个新的*变量作用域*。在 if *块*里的变量 `foo` 跟在 if 块外面的是同一个。这通常是 JavaScript 程序的一个普遍错误原因。这就是为什么 TypeScript（和 ES6）提出 `let` 关键字来允许你定义真正*块级作用域*的变量。即如果你使用 `let` 而不是 `var` ，你会获得跟你可能已经在作用域外定义的变量断开链接的一个真的单独元素。使用 `let` 来展示同一个例子：

```ts
let foo = 123;
if (true) {
    let foo = 456;
}
console.log(foo); // 123
```

`let` 会把你从错误中拯救的另一个地方是循环中。 

```ts
var index = 0;
var array = [1, 2, 3];
for (let index = 0; index < array.length; index++) {
    console.log(array[index]);
}
console.log(index); // 0
```
真诚地说，我们发现只要可以，使用 `let` 是更好的选择，因为它对于新人和现在的多语言开发者来说会产生更少的惊奇，

#### 函数创建了新的作用域
既然提到了，我们想要证明 JavaScript 中函数创建了一个新的变量作用于。考虑下面这种情况：

```ts
var foo = 123;
function test() {
    var foo = 456;
}
test();
console.log(foo); // 123
```
这就像你预期的那样表现。如果没有了这一点，要在 JavaScript 中写代码会变得非常困难。

#### 生成的 JS
由 TypeScript 生成的 JS 在如果有一个相同的名字已经存在于周围的作用域时，会简单重命名 `let` 变量。例如：下面这个例子生成代码简单地把 `let` 替换成 `var`：

```ts
if (true) {
    let foo = 123;
}

// 变成 //

if (true) {
    var foo = 123;
}
```
然而如果变量名已经被周围的作用域使用了，那么一个新的变量名会像下面这样被生成（留意 `_foo`）：

```ts
var foo = '123';
if (true) {
    let foo = 123;
}

// 变成 //

var foo = '123';
if (true) {
    var _foo = 123; // 重命名
}
```

#### 闭包中的 let
一个常见的 JavaScript 开发者的编程面试题是这个简单文件的输出：

```ts
var funcs = [];
// 创建一堆函数
for (var i = 0; i < 3; i++) {
    funcs.push(function() {
        console.log(i);
    })
}
// 调用它们
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
有的人会认为答案是 `0,1,2`。出人意料的是对于所有三个函数来说都是输出 `3`。原因在于所有三个函数都在使用外部作用域的变量 `i`，而当我们执行它们（在第二个循环中）的时候，`i` 的值会是 `3`（那是第一个循环的最终状态）。

一个修复方法是在每次循环的时候创建一个具体到循环迭代的新变量。就像我们之前学习的那样，我们可以像下面这样通过创建一个新的函数以及马上执行它（即类里面的 IIFE 模式 `(function() { /* body */ })();`）来创建一个新的变量作用域：

```ts
var funcs = [];
// 创建一堆函数
for (var i = 0; i < 3; i++) {
    (function() {
        var local = i;
        funcs.push(function() {
            console.log(local);
        })
    })();
}
// 调用它们
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
在这里函数关闭了（于是叫做 `closure`）*本地*变量（方便地命名为 `local`） 以及使用它来代替循环变量 `i`。

> 需要注意的是闭包会有性能影响（它们需要储存周围状态）

ES6 `let` 关键字在循环中会跟之前的例子有一样的表现。

```ts
var funcs = [];
// 创建一堆函数
for (let i = 0; i < 3; i++) { // Note the use of let
    funcs.push(function() {
        console.log(i);
    })
}
// 调用它们
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```

使用 `let` 来代替 `var` 创建单独于每一个循环迭代的变量 `i`。 

#### 总结
对于绝大多数代码来说 `let` 是非常有用的。他能大大提升你的代码可读性和减少程序错误的几率。



[](https://github.com/olov/defs/blob/master/loop-closures.md)
