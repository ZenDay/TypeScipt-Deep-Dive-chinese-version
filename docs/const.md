### const

`const` 是 ES6 / TypeScript 提供的一个非常受欢迎的特性。它允许你创建不可变的变量。无论是对于文档还是运行时视角而言，这都是非常好的。要使用 const，只要用 `const` 来替代 `var`：

```ts
const foo = 123;
```

> 这种语法比其他强制用户输入像是 `let constant foo` —— 即变量 ＋ 行为说明 —— 的东西要好得多（恕我直言）。

`const` 在可读性和可维护性方面是好的实践，以及避免了使用*魔法字面量*。例如：

```ts
// 可读性低
if (x > 10) {
}

// 更好！
const maxRows = 10;
if (x > maxRows) {
}
```

#### const 声明必须初始化
下面的例子会导致编译错误：

```ts
const foo; // ERROR: const declarations must be initialized
```

#### 赋值操作的左侧不可以是常量
常量在创建之后就是不可变的了，所以如果你尝试去赋一个新的值，这是一个编译错误：

```ts
const foo = 123;
foo = 456; // ERROR: Left-hand side of an assignment expression cannot be a constant
```

#### 块级作用域
`const` 就像 [`let`](./let.md) 一样是块级作用域：

```ts
const foo = 123;
if (true) {
    const foo = 456; // Allowed as its a new variable limited to this `if` block
}
```

#### 深不可变性
`const` 对于对象字面量也有效，考虑到了保护变量*引用*：

```ts
const foo = { bar: 123 };
foo = { bar: 456 }; // ERROR : Left hand side of an assignment expression cannot be a constant
```

然而它仍然允许对象子属性可变，就像下面展示的那样：

```ts
const foo = { bar: 123 };
foo.bar = 456; // Allowed!
console.log(foo); // { bar: 456 }
```

因此我推荐使用 `const` 在字面量或者不可变的数据结构上。
