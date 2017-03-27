### 解构

TypeScript 支持下面这几种形式的解构（字面上地因为解开结构而得名解构）：

1. 对象解构
2. 数组解构

很容易就能想到解构是_结构_的反转操作。在 JavaScript 中_结构_的方法是对象字面量：

```ts
var foo = {
    bar: {
        bas: 123
    }
};
```

没有了绝妙的_结构_支持，在 JavaScript 中创建新对象确实会变得非常笨重。解构带来了从结构中提取数据的相同层次的便捷性。

#### 对象解构

解构很有用，因为它允许你在一行内实现，而其他方法则需要多行。考虑下面这个例子：

```ts
var rect = { x: 0, y: 10, width: 15, height: 20 };

// 解构赋值
var {x, y, width, height} = rect;
console.log(x, y, width, height); // 0,10,15,20
```

没有解构，你将必须一个接着一个地从 `rect` 里把 `x,y,width,height` 拿出来。

想要把一个提取出来的变量赋值到一个新的变量名，你可以像下面这样做：

```ts
// 结构
const obj = {"some property": "some value"};

// 解构
const {"some property": someProperty} = obj;
console.log(someProperty === "some value"); // true
```

另外你可以使用解构来获取解构中的_深层_数据。如下所示：

```ts
var foo = { bar: { bas: 123 } };
var {bar: {bas}} = foo; // 简化的 `var bas = foo.bar.bas;`
```

#### 数组解构

一个普通的编程问题：不使用第三个变量来交换两个变量。TypeScript 的解决方案：

```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1
```

需要注意的是数组解构即编译器有效地为你做了 `[0], [1], ...` 等等。不能保证这些值一定存在。

#### 数组解构剩余

你可以从数组中拿出任意数量的元素然后获得_一个数组_，它包含了被解构数组剩余的所有元素。

```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1, 2, [3,4]
```

#### 数组解构忽略

你可以简单地通过让它的位置为空来忽略任意索引，即在赋值操作左侧的 `, ,`。例如：

```ts
var [x, , ...remaining] = [1, 2, 3, 4];
console.log(x, remaining); // 1, [3,4]
```

#### JS 生成

对于非 ES6 目标生成的 JavaScript 简单地涉及到创建临时变量，就像没有原生语言支持的解构时你不得不做的那样：

```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1

// 变成 //

var x = 1, y = 2;
_a = [y,x], x = _a[0], y = _a[1];
console.log(x, y);
var _a;
```

#### 总结

解构通过减少行数和明确意图来使你的代码更加可读和可维护。数组解构能够允许你像元组那样使用数组。

