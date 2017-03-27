for...of

菜鸟 JavaScript 程序员经历的一个普遍错误是对于数组的 `for...in` 不会遍历数组项。取而代之的是它遍历了你传过去对象的_键_。如下所示。这里你想要 `9,2,5` 但是获得了索引 `0,1,2`：

```ts
var someArray = [9, 2, 5];
for (var item in someArray) {
    console.log(item); // 0,1,2
}
```

这就是 `for...of` 存在于 TypeScript（和 ES6）的原因之一。下面正确地遍历了数组，跟设想一样输出了成员：

```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item); // 9,2,5
}
```

同样的 TypeScript 使用 `for...of` 一个一个字符地遍历字符串没有任何困难：

```ts
var hello = "is it me you're looking for?";
for (var char of hello) {
    console.log(char); // is it me you're looking for?
}
```

#### JS 生成

对于 ES6 之前的目标，TypeScript 会生成标准的 `for (var i = 0; i < list.length; i++)` 类型的循环。例如这里是我们之前的例子生成的代码：

```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item);
}

// 变成 //

for (var _i = 0; _i < someArray.length; _i++) {
    var item = someArray[_i];
    console.log(item);
}
```

你可以看到使用 `for...of` 使得_意图_更清晰，还减少了你不得不写的代码（和你需要思考的变量名）的数量。

#### 缺陷

如果你的编译目标不是 ES6 或以上，生成的代码假设属性 `length` 存在于对象中而且对象能够用数字来索引，例如 `obj[2]`。所以对于遗留的 JS 引擎，它只支持字符串和数组。

如果 TypeScript 能看到你不是在是用数组或者字符串，它会给你一个清晰的错误 _"is not an array type or a string type"_：

```ts
let articleParagraphs = document.querySelectorAll("article > p");
// Error: Nodelist is not an array type or a string type
for (let paragraph of articleParagraphs) {
    paragraph.classList.add("read");
}
```

只在那些_你知道_是数组或者字符串的东西上使用 `for...of`。需要注意的是这个缺陷在将来版本的 TypeScript 中可能会被移除。

#### 总结

你会对于你会在数组中遍历元素的次数感到惊讶。下一次你想要遍历的时候，让 `for...of` 一展身手。你会让下一个审查你代码的人很开心。

