#### IIFE 怎么了
从类生成的 js 可以长这样：
```ts
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.add = function (point) {
    return new Point(this.x + point.x, this.y + point.y);
};
```

它被包裹在立即执行函数表达式（IIFE），即：

```ts
(function () {

    // BODY

    return Point;
})();
```

的原因是需要继承。这使得 TypeScript 通过变量 `_super` 获得基类。例如：

```ts
var Point3D = (function (_super) {
    __extends(Point3D, _super);
    function Point3D(x, y, z) {
        _super.call(this, x, y);
        this.z = z;
    }
    Point3D.prototype.add = function (point) {
        var point2D = _super.prototype.add.call(this, point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    };
    return Point3D;
})(Point);
```

需要注意的是 IIFE 允许 TypeScript 很容易地在 `_super` 变量中获得基类 `Point`，并且在类体中可以持续地使用。

### `__extends`
你会注意到在你继承一个类的时候，TypeScript 还声称了下面的函数：
```ts
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```
这里 `d` 引用了派生类而 `b` 引用了基类。这个函数做了两件事：

1. 从基类中复制静态成员到子类，即 `for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];`
2. 把子类函数的原型设置到父类的 `proto` 的可选查找成员，即高效地 `d.prototype.__proto__ = b.prototype`

人们对于理解 1 很少会存在困难，但是很多人会陷于理解 2。因此一个解释是适当的。

#### `d.prototype.__proto__ = b.prototype`

在帮助了很多人理解这件事之后，我找到下面这种最简单的解释。首先我们会解释在 `__extends` 中的代码是怎么等价于 `d.prototype.__proto__ = b.prototype` 的，然后解释为什么这一行它自身是意义重大的。为了理解这点，你需要知道这几样东西：

1. `__proto__`
2. `prototype`
3. `new` 对于在被调用函数里 `this` 的作用
4. `new` 对于`prototype` 和 `__proto__` 的作用

JavaScript 中所有的对象都包含 `__proto__` 成员。这个成员在旧浏览器中是常常不可访问的（有的时候文档引用这个魔法属性为 `[[prototype]]`）它有一个目的：如果一个属性在查找时在一个对象上找不到（例如 `obj.property`）那么它会去查找 `obj.__proto__.property`。如果还是找不到那么去找 `obj.__proto__.__proto__.property` 直到两者之一被满足：*找到了*或者是*最后的`.__proto__`自身为 null*。这解释了为什么 JavaScript 表示支持自带的*原型继承*。下面的这个例子展示了这一点，你可以在 chrome 控制台或者 nodejs 中运行：

```ts
var foo = {}

// 在 foo 和 foo.__proto__ 中设置
foo.bar = 123;
foo.__proto__.bar = 456;

console.log(foo.bar); // 123
delete foo.bar; // 从对象中删除
console.log(foo.bar); // 456
delete foo.__proto__.bar; // 从 foo.__proto__ 中删除
console.log(foo.bar); // undefined
```

这很酷，因此你现在理解了 `__proto__`。另外一个有用的信息是所有在 JavaScript 中的 `function` 有一个属性叫 `prototype`，而它有一个成员 `constructor` 反指向这个函数。下面是展示：

```ts
function Foo() { }
console.log(Foo.prototype); // {} 即它存在而且不是 undefined
console.log(Foo.prototype.constructor === Foo); // 有一个成员叫做 `constructor` 指向这个函数
```

现在让我们看看*`new` 对于在被调用函数里 `this` 的作用*。基本上被调用函数里的 `this` 会指向从函数中被返回的新创建的对象。如果你在函数中变化 this 上的一个属性，就很容易看得到这一点了：

```ts
function Foo() {
    this.bar = 123;
}

// 用 new 操作符调用
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

现在你唯一需要知道的就是在一个函数上调用 `new` 复制了函数的 `prototype` 到函数调用返回的新创建对象的 `__proto__` 上。这里是你可以运行来完全理解的代码：

```ts
function Foo() { }

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // True!
```

就是这样。现在在 `__extends` 之外直接地看下面的这些代码。我已经加入了行数编号：

```ts
1  function __() { this.constructor = d; }
2   __.prototype = b.prototype;
3   d.prototype = new __();
```

反过来阅读这个函数，第三行 `d.prototype = new __()` 即是指 `d.prototype = {__proto__ : __.prototype}`（因为 `new` 对于`prototype` 和 `__proto__` 的作用），再看前一行（即第二行 `__.prototype = b.prototype;`）你就得到了`d.prototype = {__proto__ : b.prototype}`。

等一下，我们想要的是 `d.prototype.__proto__`，即只是 proto 改变和维持旧的  `d.prototype.constructor`。这就是第一行（即 `function __() { this.constructor = d; }`）的意义出现的地方了。这里我们会有 `d.prototype = {__proto__ : b.prototype, constructor: d}`（因为 `new` 对于在被调用函数里 `this` 的作用）。因此因为我们回复了 `d.prototype.constructor`，我们实际上唯一改变的东西就是 `__proto__` 于是 `d.prototype.__proto__ = b.prototype`。

#### `d.prototype.__proto__ = b.prototype` 的意义

意义在于它允许你为子类添加成员函数以及从基类中继承其他。这通过下面的简单例子来演示：

```ts
function Animal() { }
Animal.prototype.walk = function () { console.log('walk') };

function Bird() { }
Bird.prototype.__proto__ = Animal.prototype;
Bird.prototype.fly = function () { console.log('fly') };

var bird = new Bird();
bird.walk();
bird.fly();
```
基本上 `bird.fly` 会从 `bird.__proto__.fly` 中查找（记得 `new` 使 `bird.__proto__` 指向 `Bird.prototype`）而 `bird.walk`（一个继承过来的成员）会从 `bird.__proto__.__proto__.walk` 中查找（因为 `bird.__proto__ == Bird.prototype` 和 `bird.__proto__.__proto__ == Animal.prototype`）
