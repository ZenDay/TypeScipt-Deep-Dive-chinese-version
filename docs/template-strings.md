### 模版字符串
语法上来说这些是使用了反引号（即 \`）而不是单（'）或者双（"）印号。模版字符串的动机有三个：

* 多行字符串
* 字符串插值
* 标签模版

#### 多行字符串
曾经想要在 JavaScript 字符串中放入一个新行？可能你想要嵌入一些歌词？你可能需要使用我们的转义字符 `\` *转义字面换行* ，然后手动把新行 `\n` 放在下一行上。如下所示：

```ts
var lyrics = "Never gonna give you up \
\nNever gonna let you down";
```

在 TypeScript 中你可以只使用一个模版字符串：

```ts
var lyrics = `Never gonna give you up
Never gonna let you down`;
```

#### 字符串插值
另一个普遍的用例是你想要由一些静态字符串＋一些变量生成一些字符串。为了这些，你会需要使用*模版逻辑*，这就是*模版字符串*名字的由来。这可能是你之前生成 html 字符串的方式：

```ts
var lyrics = 'Never gonna give you up';
var html = '<div>' + lyrics + '</div>';
```
现在有了模版字符串，你只要这样做：

```ts
var lyrics = 'Never gonna give you up';
var html = `<div>${lyrics}</div>`;
```

需要注意的是任何在插值（`${` 和 `}`）里的占位符都会被当成 JavaScript 表达式来执行，例如：你可以做数学计算。

```ts
console.log(`1 and 1 make ${1 + 1}`);
```

#### 标签模版

你可以在模版字符串前放置一个函数（叫做 `tag`），它可以得到预处理模版字符串和所有占位符表达式的机会并且返回一个结果。注意事项：

* 所有静态词会被作为一个数组传到第一个变量里。
* 所有占位符表达式的值会被传到其余参数中。大多数情况下你会使用剩余参数来把它们也转换成一个数组。

这里是一个从所有标识符里转义 html 的标签函数（叫做 `htmlEscape`）的例子：

```ts
var say = "a bird in hand > two in the bush";
var html = htmlEscape `<div> I would just like to say : ${say}</div>`;

// 示例标签函数
function htmlEscape(literals, ...placeholders) {
    let result = "";

    // 交错静态词和占位符
    for (let i = 0; i < placeholders.length; i++) {
        result += literals[i];
        result += placeholders[i]
            .replace(/&/g, '&amp;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;');
    }

    // 加上最后一个静态词
    result += literals[literals.length - 1];
    return result;
}
```

#### 生成的 JS
对于 ES6 之前的编译目标来说，代码相当简单。多行字符串变成转义字符串。字符串插值变成*字符串连接*。标签模版变成函数调用。

#### 总结
在任何语言中多行字符串和字符串都是很棒的存在。更棒的是，你现在可以在你的 JavaScript 中使用了（感谢 TypeScript！）。标签模版允许你创造强大的字符串效用。
