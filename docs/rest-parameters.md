### 剩余参数
剩余参数（表示为 `...argumentName` 出现在最后一个参数）允许你快速地接受在你函数中的多个参数以及把它们合成一个数组。下面这个例子展示了它们。

```ts
function iTakeItAll(first, second, ...allOthers) {
    console.log(allOthers);
}
iTakeItAll('foo', 'bar'); // []
iTakeItAll('foo', 'bar', 'bas', 'qux'); // ['bas','qux']
```

剩余参数可以在任何函数中使用，包括：`function`、`()=>`、`class member`。
