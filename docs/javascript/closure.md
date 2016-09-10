## 闭包

JavaScript 获得的最好的东西就是闭包了。JavaScript 中的一个函数可以访问任何定义在外部域的变量。关于闭包，最好用例子来解释：

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;

    function bar() {
        console.log(variableInOuterFunction); // 访问了外部域的变量
    }

    // 调用本地函数来演示它访问了 arg
    bar();
}

outerFunction("hello closure"); // 输出 hello closure！
```

你可以看到内部函数可以访问外部域的变量（variableInOuterFunction）。在外部函数的变量被关闭（或者说被约束）在内部函数中。这就是术语**闭包**的由来。这个概念本身是十分简单和直观的。

这是最妙的部分：内部函数可以访问外部域的变量，*尽管在外部函数已经返回了之后*。这是因为变量仍然被约束在内部函数中而不依赖外部函数。让我们再次看下一个例子：

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;
    return function() {
        console.log(variableInOuterFunction);
    }
}

var innerFunction = outerFunction("hello closure!");

// 注意 outerFunction 已经返回了
innerFunction(); // 输出 hello closure！
```

### 为什么说它绝妙
它使你更容易地组合对象。例如：暴露模块模式：

```ts
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```

高层次来说它也使像是 nodejs 这样的东西变得可能（如果它现在没能进入你的脑海中也不用担心。它终究会的🌹）：

```ts
// 解释这个概念的伪代码
server.on(function handler(req, res) {
    loadData(req.id).then(function(data) {
        // `res` 已经被关闭了进来而且是可用的
        res.send(data);
    })
});
```
