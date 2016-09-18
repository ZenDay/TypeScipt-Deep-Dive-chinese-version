## Async Await

> 注意：你不能在 TypeScript 中以有意义的方式使用 async await（ES5 的转换器正在进行中）。然而这将会很快改变，所以我们仍然有这一章。

作为一个思想实验，想像下面这段代码，这是一种方式去告诉 JavaScript 运行时在使用于 promise 上的 `await` 关键字上暂停执行代码，然后仅当（和如果）promise 从函数中返回时恢复执行。

```ts
// 非真实代码，一个思想实验
async function foo() {
    try {
        var val = await getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
}
```

当 promise 稳定后执行继续，

* 如果它是 fulfilled，那么 await 会返回值，
* 如果它 reject 了一个错误，那么错误会被同步地抛出使我们可以捕获。

这突然（而且神奇地）令异步编程跟同步编程一样容易。这个思想实验需要三样东西：

* *暂停函数*执行的能力。
* *把值放入*函数中的能力。
* *把错误抛到*函数中的能力。

这正是 generators 帮我们做到的事！这个思想实验*实际上是真实的*，而且就是 TypeScript / JavaScript 中 `async`/`await` 的实现。背后它实际上只使用了 generators。

### 生成的 JavaScript

你不需要去理解它，但如果你已经读了 [generators][generators]，就相当简单了。函数 `foo` 可以简单地被包裹为：

```ts
const foo = wrapToReturnPromise(function* () {
    try {
        var val = yield getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
})
```

`wrapToReturnPromise` 只是执行了 generator 函数来获得 `generator` 然后使用 `generator.next()`，如果值是一个 `promise`，它会 `then`+`catch` 这个 promise，以及取决于结果来调用 `genertor.next(result)` 或者 `genertor.throw(error)`。就是这样！

[generators]:./generators.md
