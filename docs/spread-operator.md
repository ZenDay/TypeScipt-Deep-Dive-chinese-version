### 展开运算符

展开运算符的主要目标是去展开一个数组的对象。这最好通过例子来解释。

#### Apply
展开数组到函数参数是一个普遍用例。之前你必须要使用 `Function.prototype.apply`：

```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo.apply(null, args);
```

现在你可以像下面那样通过简单地在参数前缀加上 `...` 来实现：

```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo(...args);
```

这里我们把 `args` 数组展开到位置上的 `arguments` 中。

#### 解构
我们已经在*解构*里看过这种用法了

```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1, 2, [3,4]
```
这里的目标是简单地使你能在解构时轻易地获取到其余元素。

#### 数组赋值
展开操作符使你可以简单的把一个数组的*展开版本*放到另一个数组中。如下所示：

```ts
var list = [1, 2];
list = [...list, 3, 4];
console.log(list); // [1,2,3,4]
```

#### 总结
`apply` 是 JavaScript 中你必将会做的东西，所以有更好的语法使你不需要为 `this` 参数填入丑陋的 `null` 是一件极好的事。而且有用于移出（解构）或者移入（赋值）一个数组到另一个数组的专用语法使你在局部数组里处理数组时更整洁。


[](https://github.com/Microsoft/TypeScript/pull/1931)
