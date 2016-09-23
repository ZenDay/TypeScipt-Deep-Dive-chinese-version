## 命名空间
命名空间为你提供了一个关于使用在 JavaScript 中的普遍模式的一个方便的语法：

```ts
(function(something) {

    something.foo = 123;

})(something || something = {})
```

基本上 `something || something = {}` 允许匿名函数 `function(something) {}` 去*添加东西到一个现存项目中*（`something ||` 部分）或者*创建一个新对象然后把东西添加到这个对象里面去*（`|| something = {}` 部分）。这意味着你可以拥有两个这样被执行边界分开的块：

```ts
(function(something) {

    something.foo = 123;

})(something || something = {})

console.log(something); // {foo:123}

(function(something) {

    something.bar = 456;

})(something || something = {})

console.log(something); // {foo:123, bar:456}

```

这在  JavaScript 中很常使用，以确保东西不会泄漏到全局命名空间里。有了基于文件的模块，你不再需要担心这些，但是这个模式仍然对于一堆函数的*逻辑分组*非常有用。因此 TypeScript 提供了 `namespace` 关键字来分组它们，例如：

```ts
namespace Utility {
    export function log(msg) {
        console.log(msg);
    }
    export function error(msg) {
        console.error(msg);
    }
}

// 使用方式
Utility.log('Call me');
Utility.error('maybe!');
```
`namespace` 关键字生成了跟我们之前看到的一样的 JavaScript：

```ts
(function (Utility) {

// 添加东西到 Utility 里

})(Utility || (Utility = {}));
```

一点需要注意的是命名空间可以是嵌套的，因此你可以做像是 `namespace Utility.Messaging` 的东西来嵌套一个 `Messaging` 命名空间到 `Utility`。

对于大部分的项目来说，我们推荐使用外部模块，而对于快速演示和接入旧的 JavaScript 代码使用 `namespace`。
