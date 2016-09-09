# 为什么选择 TypeScript
TypeScript 有两个主要的目的：

* 为 JavaScript 提供一个可选的类型系统
* 为当前版本的 JavaScript 引擎提供将来 JavaScript 版本的新功能。

接下来我们来讨论这些目标的动机。

## TypeScript 类型系统

你可能想知道 “**为什么要往 JavaScript 中添加类型？**”

类型拥有着提高代码质量和可读性的可靠能力。大型团队（例如 google，microsoft，facebook）一直在不断得出这个结论。特别地：

* 在重构时，类型增加了你的敏捷性。 *让编译器捕获错误比在运行时崩溃更好*。
* 类型是你能拥有的最好的文档形式之一。 *函数签名是定理，而函数体则是证明过程*。

然而类型有着不必要的过分严谨。因此 TypeScript 讲究让门槛尽可能地低，通过：

### 你的 JavaScript 就是 TypeScript
TypeScript 为你的 JavaScript 代码提供了编译时的类型安全。这也难怪它有着这样的名字。厉害之处在于类型是完全可选的。你的 JavaScript 代码的 `.js` 文件能够被重命名为 `.ts` 文件，而 TypeScript 仍会给你有效的 `.js` 文件，这个文件跟你的原来的 JavaScript 文件一模一样。TypeScript 有意而且严格地成为了有着可选类型检查的 JavaScript 的超集。

### 类型可以是隐含的
TypeScript 会尝试去推断尽可能多的类型信息，因此它可以在代码开发时通过最小的生产力开销给予你类型安全。例如，在下面的例子中 TypeScript 会知道 foo 是 `number` 类型并且在第二行报错：

```ts
var foo = 123;
foo = '456'; // Error: cannot assign `string` to `number`

// Is foo a number or a string?
```
这种类型推断是动机良好的。如果你做了类似于例子展示的事情，那么在你剩余的代码中，你就不能确定 `foo` 是 `number` 还是 `string`。这种问题常常出现在大型的多文件代码仓库中。我们之后会对类型推断规则进行深入挖掘。

### 类型可以是清晰的
就像我们之前提到过的一样，TypeScript 会尽可能安全地去推断，然而你也可以使用标注来：

1. 帮助编译器，以及更重要地，为下一个不得不阅读你的代码的开发者（有可能就是未来的你）添加文档。
2. 强制让编译器看到的就是你认为它应该看到的。这就是你对于代码的理解能跟（编译器做的）代码的算法分析匹配。

TypeScript 使用了在其他*可选*标注语言（例如 ActionScript 和 F#）中很流行的后缀类型标注。

```ts
var foo: number = 123;
```
因此如果你做了错误的事，编译器会报错，例如：

```ts
var foo: number = '123'; // Error: cannot assign a `string` to a `number`
```

在下一章我们会讨论所有 TypeScript 支持的标注语法的细节。

### 类型是结构化的
在一些语言中（特别是那些名义上类型化的），静态类型导致了不必要的严谨，因为即使*你知道*代码能正常运行但语言语义强制你去复制这些东西。这就是为什么像是 [automapper](http://automapper.org/) 这样的东西在 C# 中很重要。在 TypeScript 中因为我们真的想要以最小的认知超载来让 JavaScript 开发者更容易上手，所以类型是*结构化的*。这意味着*鸭子类型*是第一级语言结构。参考下面的例子。函数 `iTakePoint2D` 会接受接受任何包含了所有预期事物（`x` 和 `y`）的东西。

```ts
interface Point2D {
    x: number;
    y: number;
}
interface Point3D {
    x: number;
    y: number;
    z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* do something */ }

iTakePoint2D(point2D); // exact match okay
iTakePoint2D(point3D); // extra information okay
iTakePoint2D({ x: 0 }); // Error: missing information `y`
```

### 类型错误不会阻止 JavaScript 生成
为了让你更容易地把 JavaScript 代码迁移到 TypeScript 上，尽管会有编译错误，默认情况下  TypeScript 会尽可能地*生成有效的 JavaScript*。例子：

```ts
var foo = 123;
foo = '456'; // Error: cannot assign a `string` to a `number`
```

会生成这样的 js：

```ts
var foo = 123;
foo = '456';
```

因此你可以增量更新你的 JavaScript 代码到 TypeScript。这跟其他语言编译器的工作机制有很大区别，也是另外一个使用 TypeScript 的原因。

### Types can be ambient
TypeScript 的一个主要的设计目标就是让你能够安全而且轻松地在 TypeScript 中使用现存的 JavaScript 库。TypeScript 通过*定义*来做到这点。TypeScript 为你提供你想要在你的定义上花费功夫多少的递加，你花费得更多，你就能得到更多的类型安全和代码智能。注意，绝大多数流行的 JavaScript 库的定义已经被 [DefinitelyTyped 社区](https://github.com/borisyankov/DefinitelyTyped) 写好了，所以对于绝大多数的目标而言：

1. 定义文件已经存在了。
2. 或者最起码，你有了一个大量的，被很好地审查过的已经可用的 TypeScript 定义模版的列表。

这里用一个 [jquery](https://jquery.com/) 的小例子来作为一个你如何编写你自己的定义文件的简单例子。默认情况下（也是好的 JS 代码希冀的），希望你在你使用变量之前先进行定义（例如在某些地方使用 `var` ）

```ts
$('.awesome').show(); // Error: cannot find name `$`
```

一个快速的修复方法是*你可以告诉 TypeScript* 的确有叫作 `$` 的东西：

```ts
declare var $:any;
$('.awesome').show(); // Okay!
```

如果你希望你能够基于这个基本的定义来构建以及提供更多信息来帮助保护你避免出错的话：

```ts
declare var $:{
    (selector:string)=>any;
};
$('.awesome').show(); // Okay!
$(123).show(); // Error: selector needs to be a string
```

我们稍后会在你知道更多有关 TypeScript 的东西（例如诸如 `interface` 和 `any` 之类的东西）后仔细讨论为现存 JavaScript 创建 TypeScript 定义的细节。

## 未来 JavaScript => 现在
TypeScript 为现在的 JavaScript 引擎（只支持 ES5 等等）提供了一系列在 ES6 中出现的特性。TypeScript 团队正积极地添加这些特性，这个列表只会随着时间的推移越来越大，而我们会在它们自己的章节讨论它。仅仅作为一个样品，这里是一个 class 的例子：

```ts
class Point {
    constructor(public x: number, public y: number) {
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```

以及我们可爱的箭头函数：

```ts
var inc = (x)=>x+1;
```

### 结语
在这节中我们为你提供了 TypeScript 的动机和设计目标。了解了这些之后，我们就可以深度挖掘 TypeScript 的细枝末节了。

[](Interfaces are open ended)
[](Type Inferernce rules)
[](Cover all the annotations)
[](Cover all ambients : also that there are no runtime enforcement)
[](.ts vs. .d.ts)