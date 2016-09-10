### 类
在 JavaScript 中类作为一等公民很重要的原因在于：

1. [类提供了一个有用的结构化抽象](./tips/classesAreUseful.md)
2. 提供了一个为开发者使用类的持续化方式而不是每一个框架（emberjs，reactjs 等等）都存在着自己的版本。
3. 面向对象的开发者已经理解了类。

最终 JavaScript 开发者可以*拥有 `class`*。这里我们有一个叫做 Point 的基本类：

```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```
这个类会在 ES5 中生成这样的 JavaScript 代码：

```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```
现在作为一等语言构造，这是一个相当地道的传统 JavaScript 类模式了。

### 继承
TypeScript 中的类（像其他语言那样）支持*单*继承，只要像下面这样使用 `extends` 关键字：

```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```
如果你的类有构造函数，那么你*必须*在你的构造函数中调用父级构造函数（TypeScript 会为你指出这一点）。这保证了需要需要设置在 `this` 上的东西能够得到设置。在调用了 `super` 以后你可以添加任何而外的你想要在你的构造函数中做的东西（这里我们添加了额外的成员 `z`）。

需要注意的是你可以轻易地重载父级成员函数（这里我们重载了 `add`）而在你的成员中仍然使用父类的功能（使用 `super.` 语法）。

### 静态
TypeScript 的类支持 `static` 属性，这会被所有这个类的实例共享。一个自然的地方来放置（和访问）它们就是类本身，而 TypeScript 就是这么做的：

```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```

你可以拥有静态成员或者静态函数。

### 访问修饰符
TypeScript 支持访问修饰符 `public`、`private` 和 `protected`，它们决定了一个 `class` 成员的可访问性，如下所示：

| 可访问在 | `public` | `private` | `protected` |
|---------|----------|-----------|-------------|
| 类的实例 | 是		 | 否        | 否		       |
| 类	   | 是		 | 是        | 是		       |
| 类的孩子 | 是		 | 否        | 是		       |

Note that at runtime (in the generated JS) these have no significance but will give you compile time errors if you use them incorrectly. An example of each is shown below:需要注意的是在运行时（在生成的 JS 里）这些是没有意义的，但是如果你不正确使用它们，会出现编译时错误。对于每种情况的

```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// EFFECT ON INSTANCES
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// EFFECT ON CHILD CLASSES
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

跟之前一样，这些修饰符同时在成员属性和成员函数上有效。

### 抽象
`abstract` 可以看成一个访问修饰符。我们把之所以它单独介绍，是因为跟之前提到的修饰符不同，它可以在 `class` 上也可以在类的成员上。拥有一个 `abstract` 修饰符主要意味着这个功能*不能直接被调用*而子类必须提供它。

* `abstract` 的**类**不能直接被实例化。用户必须创造一些 `class` 来继承`abstract class`。
* `abstract` 的**成员**不能直接被访问，而且子类必须提供这个功能。

### 构造函数是可选的

类不是必须要拥有一个构造函数。例如：

```ts
class Foo {}
let foo = new Foo();
```

### 使用构造函数来定义

在一个类中拥有一个成员并且像下面那样去初始化：

```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```
是这样一种普遍的模式，因此 TypeScript 提供了一种缩写形式，你可以用一个*修饰符*作为作为成员的前缀然后它会在类中自动被声明和从构造函数中被复制。所以前面的例子可以被重写为（留意 `public x:number`）：

```ts
class Foo {
    constructor(public x:number) {
    }
}
```

### 属性初始化器
这是由 TypeScript（实际上是从 ES7 ）提供的一个俏皮的特性。你可以在类的构造函数之外初始化任何类的成员，对于提供默认值（留意 `members = []`）很有用

```ts
class Foo {
    members = [];  // Initialize directly
    add(x) {
        this.members.push(x);
    }
}
```
