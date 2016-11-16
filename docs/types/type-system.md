# TypeScript 类型系统
在我们讨论*为什么选择 TypeScript*的时候就已经谈论过了 TypeScript 类型系统的主要特性。下面是从讨论中提取出来的一些不需要更深层解释的关键点：

* TypeScript 中的类型系统被设计为可选的，因此*你的 JavaScript 就是 TypeScript*。
* 当类型错误存在时，TypeScript 不会停止*生成 JavaScript*，允许你*渐进把 JS 更新成 TS*。

现在让我们开始讲述 TypeScript 类型系统的*语法*。这次你可以开始马上在代码中使用这些注解并且看到它们的优点。这会为接下来的深入挖掘做准备。

## 基本注解
就像先前提到的那样，类型会被注解以 `:TypeAnnotation` 语法。任何在类型声明空间可用的东西都能用作类型注解。

下面的例子展示了类型注解能被用在变量，函数参数和函数返回值上。

```
var num: number = 123;
function identity(num: number): number {
    return num;
}
```

### 原始类型
JavaScript 原始类型很好地体现在 TypeScript 类型系统中。即 `string`，`number` 和 `boolean`，如下所示：

```ts
var num: number;
var str: string;
var bool: boolean;

num = 123;
num = 123.456;
num = '123'; // Error

str = '123';
str = 123; // Error

bool = true;
bool = false;
bool = 'false'; // Error
```

### 数组
TypeScript 为数组提供了专用的类型语法来使你更简单地注解和编档你的代码。语法是后置 `[]` 于任意有效的类型注解上（例如 `:boolean[]`）。这允许你安全地做任何你通常会做的数组操作，以及从像是赋值一个成员以错误的类型的错误中保护你。如下所示：

```ts
var boolArray: boolean[];

boolArray = [true, false];
console.log(boolArray[0]); // true
console.log(boolArray.length); // 2
boolArray[1] = true;
boolArray = [false, false];

boolArray[0] = 'false'; // Error!
boolArray = 'false'; // Error!
boolArray = [true, 'false']; // Error!
```

### 接口
接口在 TypeScript 中是核心的方法去组合多个类型注解到一个单独的命名注解上。考虑下面的例子：

```ts
interface Name {
    first: string;
    second: string;
}

var name: Name;
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```
这里我们组合了注解 `first: string` + `second: string` 到一个新的注解 `Name` 上，以强制类型检查单独的成员。接口在 TypeScript 有着很多能力，而我们会使用一整章来讲述你可以怎么使用它来获得好处。

### 行内类型注解
取创建一个新的 `interface` 而代之，你可以在行内使用 `:{ /*Structure*/ }` 标注任何东西。之前的例子用行内的方法重写：

```ts
var name: {
    first: string;
    second: string;
};
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```
行内的类型用于快速提供一次性类型标注。这可以把你从想一个（可能并不好）的类型名称中解救出来。然而，如果你发现你自己多次放置相同的行内类型标注，最好考虑重构它为一个接口（或者一个 `type alias`，后面会讲到）。

## 特殊类型
除了之前提到的原始类型以外，还有一些类型在 TypeScript 里有着特殊意义。它们分别是 `any`，`null`，`undefined` 和 `void`。

### any
`any` 类型在 TypeScript 类型系统中保持一个特殊地位。它给了你一个逃离类型系统告诉编译器出现 bug 的方式。`any` 兼容类型系统中的*任意所有*类型。这意味着*任何东西可以赋值给它*而*它可以赋值给任何东西*。例子如下所示：

```ts
var power: any;

// 获取任意所有类型
power = '123';
power = 123;

// 兼容所有类型
var num: number;
power = num;
num = power;
```

如果你正在把 JavaScript 代码移植到 TypeScript，在开始时你会跟 `any` 有一段友情。然而，别太把这段友情当真因为*确保类型安全的决定权在于你*。你基本上在告诉编译器*别做任何有意义的静态分析*。

### `null` 和 `undefined`

`null` 和 `undefined` JavaScript 关键字会被类型系统简单地看作与类型 `any` 一样。这些关键字能被赋值到任意其他类型。例子如下所示：

```ts
var num: number;
var str: string;

// 这些关键字能被赋值到任何东西
num = null;
str = undefined;
```

### `:void`
使用 `:void` 来标识函数不会有任何返回值。

```ts
function log(message): void {
    console.log(message);
}
```

## 泛型
计算机科学中很多算法和数据结构不依赖于对象的*实际类型*。然而你仍然想要在不同的变量中制定约束。一个简单的玩具例子是，一个获取一个列表然后返回一个反向列表的函数。这里的约束在于被传入函数的东西和函数返回的东西之间：

```ts
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// 安全!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

这里基本上就是说函数 `reverse` 拿了一个数组（`items: T[]`），它是*某个*类型 `T` 的（注意 `reverse<T>` 里的类型参数），并且返回了一个类型为 `T`（注意 `: T[]`）的数组。因为 `reverse` 函数返回了跟它拿到的同样类型的东西，TypeScript 知道 `reversed` 变量同样也是类型  `number[]` 并且会给予它类型安全。同样地如果你传一个 `string[]` 的数组给 reverse 函数，返回的结果同样也是一个 `string[]` 的数组，并且你得到了同样的类型安全，如下所示：

```ts
var strArr = ['1', '2'];
var reversedStrs = reverse(strArr);

reversedStrs = [1, 2]; // Error!
```

实际上 JavaScript 数组已经有一个 `.reverse` 函数，TypeScript 的确使用了泛型来定义它的结构：

```ts
interface Array<T> {
 reverse(): T[];
 // ...
}
```

这意味着你在调用 `.reverse` 于任何数组时都能得到类型安全，如下所示：

```ts
var numArr = [1, 2];
var reversedNums = numArr.reverse();

reversedNums = ['1', '2']; // Error!
```
我们稍后在 **Ambient Declarations** 这章展示 `lib.d.ts` 的时候会讨论更多关于 `Array<T>` 的话题。

## 并类型
在 JavaScript 中很普遍的状况是，你想要允许一个属性是多个类型如*`string ` 或者 `number`*中的一个。在这里*并类型*（在类型注解中用 `|` 标记，例如 `string|number`）就派上用场了。一个普遍的用例是，能够拿一个单独的对象或者一个对象数组的函数，例如：

```ts
function formatCommandline(command: string[]|string) {
    var line = '';
    if (typeof command === 'string') {
        line = command.trim();
    } else {
        line = command.join(' ').trim();
    }

    // 用 line:string 做些事
}
```

## 交类型
`extend` 是一种 JavaScript 中的常见模式，用于你拿到两个对象，然后创建一个新的同时拥有这两个对象特性的对象。**交类型**允许你以安全的方式去使用这个模式，如下所示：

```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U> {};
    for (let id in first) {
        result[id] = first[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            result[id] = second[id];
        }
    }
    return result;
}

var x = extend({ a: "hello" }, { b: 42 });

// x 现在同时有 `a` 和 `b`
var a = x.a;
var b = x.b;
```

## 元组类型
JavaScript 没有一等元组支持。人们通常使用数组当作元组。这正是 TypeScript 类型系统所支持的。元组可以使用 `:[typeofmember1, typeofmember2]` 等来标注。一个元祖可以有任意数量的成员。元组的示例如下：

```ts
var nameNumber: [string, number];

// Okay
nameNumber = ['Jenny', 8675309];

// Error!
nameNumber = ['Jenny', '867-5309'];
```

将这与 TypeScript 中的解构支持合并起来，元组看起来就是一等的了，尽管底层是使用数组实现的。

```ts
var nameNumber: [string, number];
nameNumber = ['Jenny', 8675309];

var [name, num] = nameNumber;
```

## 类型别名
TypeScript 提供了方便的语法来为你将会不只一次使用的类型注解提供名字。别名是使用 `type SomeName = someValidTypeAnnotation` 语法创建的。例子如下所示：

```ts
type StrOrNum = string|number;

// 使用方法：就像其他类型那样
var sample: StrOrNum;
sample = 123;
sample = '123';

// 检查
sample = true; // Error!
```

不像 `interface`，你可以给一个类型以字面意义上的任何类型注解（对于像是并和交类型的东西特别有用）。这里是一些例子使你更熟悉它的语法：

```ts
type Text = string | { text: string };
type Coordinates = [number, number];
type Callback = (data: string) => void;
```

> 提示：如果你需要有深层的类型注解，使用 `interface`。为简单的对象结构（例如 `Coordinates`）使用类型别名只是为了给他们一个语义的名字。

## 总结
现在你可以开始标注你的大部分 JavaScript 代码，我们可以跳到 TypeScript 类型系统中所有能力的深度细节了。
