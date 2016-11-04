## 声明空间

TypeScript 中有两个声明空间：*变量*声明空间和*类型*声明空间。这些概念会在接下来探讨。

### 类型声明空间
类型声明空间包含了哪些可以用作类型注解的东西。例如，下面是一些类型声明：

```ts
class Foo { }
interface Bar { }
type Bas = {}
```
这意味着你可以使用`Foo`，`Bar`，`Bas`等作为类型注解。例如

```ts
var foo: Foo;
var bar: Bar;
var bas: Bas;
```

需要注意的是即使你有 `interface Bar`，*你不能把它作为变量来使用*，因为它不属于*变量声明空间*。如下所示：

```ts
interface Bar {};
var bar = Bar; // ERROR: "cannot find name 'Bar'"
```

它说 `cannot find name` 的原因是名称 `Bar` *未定义*在*变量*声明空间里。这引出了我们的下一个话题“变量声明空间”。

### 变量声明空间
变量声明空间包含了你可以用作变量的东西。我们之前看到 `class Foo` 在*类型*声明空间中声明了类型 `Foo`。猜猜怎么样？它也在*变量声明空间*里声明了*变量* `Foo`，如下所示：

```ts
class Foo { }
var someVar = Foo;
var someOtherVar = 123;
```
这对于有时候想要把类作为变量传递的情况是很好的。记住

* 我们不能使用像是一个 `interface` 这样*只*存在于*类型*声明空间的东西作为变量。

同样的，你使用 `var` 声明的一些东西，*只*存在于*变量*声明空间而不能用做类型注解：

```ts
var foo = 123;
var bar: foo; // ERROR: "cannot find name 'foo'"
```
The reason why it says `cannot find name` is because the name `foo` *is not defined* in the *type* declaration space.它说 `cannot find name` 的原因是名称 `foo` *未定义*在*类型*声明空间里。

### 提示

#### 复制在类型声明空间中的东西

如果你想要移动一个类，你可能会这样做：

```ts
class Foo { }
var Bar = Foo;
var bar: Bar; // ERROR: "cannot find name 'Bar'"
```
这会导致错误，因为 `var` 仅仅复制 `Foo` 到*变量*声明空间中，因此你不能使用 `Bar` 作为类型注解。正确的方法是使用 `import` 关键字。需要注意的是如果你使用*命名空间*或者*模块*（之后会有更多关于它们的讨论），你只能以这种方法使用 `import` 关键字：

```ts
namespace importing {
    export class Foo { }
}

import Bar = importing.Foo;
var bar: Bar; // Okay
```

#### 捕获变量的类型

实际上你可以使用一个类型注解中使用了 `typeof` 操作符的变量。这允许你告诉编译器一个变量跟另一个是相同类型。如下所示：

```ts
var foo = 123;
var bar: typeof foo; // `bar` 跟 `foo` 有着相同类型（这里是 `number`）
bar = 456; // Okay
bar = '789'; // ERROR: Type `string` is not `assignable` to type `number`
```

#### 捕获类成员的类型

类似于捕获变量的类型，你可以仅仅为了类型捕获的目的声明一个变量：

```ts
class Foo {
  foo: number; // 一些我们想捕获它类型的成员
}

// 纯粹为了捕获类型
declare let _foo: Foo;

// 跟之前一样
let bar: typeof _foo.foo;
```

### 捕获魔法字符串的类型

很多 JavaScript 库和框架取代了原生 JavaScript 的字符串。你可以使用 `const` 变量来捕获它们的类型。如：

```ts
// 捕获魔法字符串的*类型*和*值*：
const foo = "Hello World";

// 使用捕获的类型
let bar: typeof foo;

// bar 只能被赋值为 `Hello World`
bar = "Hello World"; // Okay!
bar = "anything else "; // Error!
```

在这个例子中  `bar` 拥有文本类型 `"Hello World"`。我们会在文本类型的章节中详细讲述。