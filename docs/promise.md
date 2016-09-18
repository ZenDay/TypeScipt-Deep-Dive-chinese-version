## Promise

`Promise` 类存在于很多现代 JavaScript 引擎中，而且可以很容易地被 [polyfill][polyfill]。Promise 的主要目的是为异步／回调风格的代码带来同步风格的错误处理。

### 回调风格的代码

为了充分体会 promise 的好处，让我们来看看一个简单的例子，这个例子证明了只使用回调来创建可读异步代码的难处。考虑创作一个异步版本的从文件加载 JSON 的简单情况。同步的版本可以很简单

```ts
import fs = require('fs');

function loadJSONSync(filename: string) {
    return JSON.parse(fs.readFileSync(filename));
}

// 好的 json 文件
console.log(loadJSONSync('good.json'));

// 不存在的 json 文件
try {
    console.log(loadJSONSync('absent.json'));
}
catch (err) {
    console.log('absent.json error', err.message);
}

// 无效的 json 文件
try {
    console.log(loadJSONSync('bad.json'));
}
catch (err) {
    console.log('bad.json error', err.message);
}
```

这个简单的 `loadJSONSync` 函数有三种行为，有效的返回值，文件系统错误或者 JSON.parse 错误。我们通过简单的 try/catch 来处理错误，跟你在其他语言做同步编程时使用的一样。现在让我们为这个函数制作一个好的异步版本。一个有着琐碎错误检查逻辑的像样初步尝试如下所示，

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}
```

就这么简单，它获取一个回调，把任何文件系统错误传到回调中。如果没有文件系统错误，它返回 `JSON.parse` 的结果。一些在使用基于回调的异步函数时需要记住的点是

1. 不要调用两次回调。
2. 不要抛出错误。

然而这个简单的函数违背了第二点。实际上 `JSON.parse` 会在它被传入了错误的 JSON 时抛出错误，因而回调不会被调用，应用也随之崩溃。如下所示：

```ts
// 加载无效的 json
loadJSON('bad.json', function (err, data) {
    // 永远不会被调用！
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

一个天真的修复方案时把 `JSON.parse 包裹在 try catch 中，如下所示：

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// 加载无效的 json
loadJSON('bad.json', function (err, data) {
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

然而在这份代码中有着微妙的 bug。如果回调（`cb`），非 `JSON.parse`，抛出了一个错误，因为我们把它包裹在 `try`/`catch` 里，`catch` 会执行而我们再一次调用了回调，即回调被调用了两次！如下所示：

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// 一个正确的文件，但是是不正确的回调 ... 被再次调用！
loadJSON('good.json', function (err, data) {
    console.log('our callback called');

    if (err) console.log('Error:', err.message);
    else {
        // 让我们来通过访问一个 undefined 变量的属性来模拟一个错误
        var foo;
        console.log(foo.bar);
    }
});
```

```bash
$ node asyncbadcatchdemo.js
our callback called
our callback called
Error: Cannot read property 'bar' of undefined
```

这是因为我们的 `loadJSON` 函数错误地把回调包裹在 `try` 块中。有一个简单的教训要记住。

> 简单的教训：把你的所有同步代码放在 try catch 中，除了当你要调用回调的时候。

遵循上面的简单教训，我们有了一个完全实用的 `loadJSON` 异步版本，如下所示：

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) return cb(err);
        try {
            var parsed = JSON.parse(data);
        }
        catch (err) {
            return cb(err);
        }
        return cb(null, parsed);
    });
}
```
诚然，如果你已经做过好几次了，这并不难理解，但尽管如此，为了简单的错误处理，它需要写太多的模版代码了。现在让我们来看看一个使用了 promise 的更好方式来实现异步 JavaScript。

## 创建 Promise

一个 promise 可以变得 `pending` 或者 `resolved` 或者 `rejected`。

![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png)

让我们关注于创建 promise。简单地在 `Promise`（promise 构造器）上调用 `new` 即可。promise 构造器被传递了 `resolve` 和 `reject` 函数味了控制 promise 状态。

```ts
const promise = new Promise((resolve, reject) => {
    // resolve / reject 函数操控着 promise 的命运
});
```

### 订阅 promise 的命运

promise 的命运可以使用 `.then`（如果 resolved 的话）或者 `.catch`（如果 rejected 的话）来订阅。

```ts
const promise = new Promise((resolve, reject) => {
    resolve(123);
});
promise.then((res) => {
    console.log('I get called:', res === 123); // I get called: true
});
promise.catch((err) => {
    // 不被调用
});
```

```ts
const promise = new Promise((resolve, reject) => {
    reject(new Error("Something awful happened"));
});
promise.then((res) => {
    // 不被调用
});
promise.catch((err) => {
    console.log('I get called:', err.message); // I get called: 'Something awful happened'
});
```

### Promise 快速方法
* 快速创建一个已经 resolved 的 promise：`Promise.resolve(result)`
* 快速创建一个已经 rejected 的 promise：`Promise.reject(error)`

### Promise 的链式性
Promise 的链式性 **是 promise 提供好处的核心**。一旦你有了一个 promise，从那一个点开始，你可以使用 `then` 函数来创建 promise 链。

* 如果你从链中的任何函数返回了一个 promise，`.then` 只会在值是 resolved 的时候被调用

```ts
Promise.resolve(123)
    .then((res) => {
        console.log(res); // 123
        return 456;
    })
    .then((res) => {
        console.log(res); // 456
        return Promise.resolve(123);
    })
    .then((res) => {
        console.log(res); // 123：需要注意的是这个 `this` 跟 resolved 的值一起被调用
        return Promise.resolve(123);
    })
```

* 你可以合并之前链上任何部分的错误处理到一个单独的 `catch` 中

```ts
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .then((res) => {
        console.log(res); // not called
        return Promise.resolve(123);
    })
    .then((res) => {
        console.log(res); // not called
        return Promise.resolve(123);
    })
    .catch((err) => {
        console.log(err.message); // something bad happened        
    });
```

* `catch` 实际上返回一个新的 promise（高效地创建一个新的 promise 链）：

```ts
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
        return Promise.resolve(123);
    })
    .then((res) => {
        console.log(res); // 123
    })
```

* 任何在 `then`（或者 `catch`）中被抛出的错误会导致返回的 promise 失败

```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('something bad happened')
        return 456;
    })
    .then((res) => {
        console.log(res); // never called
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
    })
```

> 错误跳到尾部的 `catch`（并且跳过了中间的 `then`），伴随着同步错误也能高效的被捕获的事实提供给我们一个异步编程范例，它允许了比原生回调更好的错误处理。


### TypeScript 和 promise
TypeScript 的伟大之处在于它知道 promise 链抛出来的值流。

```ts
Promise.resolve(123)
    .then((res)=>{
         // res is inferred to be of type `number`
         return true;
    })
    .then((res) => {
        // res is inferred to be of type `boolean`

    });
```

当然它也知道展开可能返回一个 promise 的任何函数调用：

```ts
function iReturnPromiseAfter1Second():Promise<string> {
    return new Promise((resolve)=>{
        setTimeout(()=>resolve("Hello world!"), 1000);
    });
}

Promise.resolve(123)
    .then((res)=>{
         // res is inferred to be of type `number`
         return iReturnPromiseAfter1Second();
    })
    .then((res) => {
        // res is inferred to be of type `string`
        console.log(res); // Hello world!
    });
```


### 把一个回调风格的函数转换成返回一个 promise

只要把函数调用放在 promise 中，并且在错误发生时  `reject`，在没有错误时 `resolve` 就行了。例如，让我们来包裹 `fs.readFile`

```ts
import fs = require('fs');
function readFileAsync(filename:string):Promise<any> {
    return new Promise((resolve,reject)=>{
        fs.readFile(filename,(err,result) => {
            if (err) reject(err);
            else resolve(result);
        });
    });
}
```


### 回过来看那个 JSON 的例子
现在让我们回国来看我们的 `loadJSON` 例子，并且使用 promise 来重写异步版本。所有我们需要去做就是作为一个 promise 去文件内容，然后把它们解析成 JSON，然后就完成啦。如下所示：

```ts
function loadJSONAsync(filename: string): Promise<any> {
    return readFileAsync(filename)
                .then(function (res) {
                    return JSON.parse(res);
                });
}
```

使用的方法（注意它跟这章最开始介绍的原始同步版本是多么相似啊 🌹）：

```ts
// 好的 json 文件
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // never called
    })

// 不存在的 json 文件
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// 无效的 json 文件
    .then(function () {
        return loadJSONAsync('bad.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```

### 并行控制流
我们已经看到了使用 promise 来做一系列顺序的异步任务的便利性。这只是简单的调用链式的 `then`。

然而你可能想要运行一系列异步的任务，然后得到所有结果来做些什么。`Promise` 提供了静态的 `Promise.all` 函数，你可以使用它来等待 n 个 promise 完成。你提供给它一个包含了 `n` 个 promise 的数组，而它返回给你包括了 `n` 个 resolved 值的数组。如下所示

```ts
// 一个模拟从服务器加载东西的异步函数
function loadItem(id: string): Promise<{id: string}> {
    return new Promise((resolve)=>{
        console.log('loading item', id);
        setTimeout(() => { // simulate a server delay
            resolve({ id: id });
        }, 1000);    
    });
}

// 链式化
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loaditem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // overall time will be around 2s

// 并行
Promise.all([loadItem(1),loaditem(2)])
    .then((res) => {
        [item1,item2] = res;
        console.log('done')    
    }); // overall time will be around 1s
```


[polyfill]:https://github.com/stefanpenner/es6-promise
