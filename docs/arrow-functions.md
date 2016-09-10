* [箭头函数](#arrow-functions)
* [提示：箭头函数的需要](#tip-arrow-function-need)
* [提示：箭头函数的危险](#tip-arrow-function-danger)
* [提示：使用了 `this` 的库](#tip-arrow-functions-with-libraries-that-use-this)
* [提示：箭头函数继承](#tip-arrow-functions-and-inheritance)

### 箭头函数{#arrow-functions}

可以可爱地叫做*胖箭头*（因为 `->` 是瘦箭头而 `=>` 是胖箭头）以及 *lambda 函数*（因为其他的语言是这么叫的），另一个常用的特性是胖箭头函数 `()=>something`。一个*胖箭头*的动机是：

1. 你不再需要键入 `function`
2. 它词意上地捕获了 `this` 的意义
3. 它词意上地捕获了 `arguments` 的意义

对于一个自称是函数式的语言来说，在 JavaScript 中你往往键入了太多的 `function`。胖箭头使你能简单地创建函数

```ts
var inc = (x)=>x+1;
```
`this` 在 JavaScript 中一直是一个痛点。就像一个智者曾经说过的那样，“我讨厌 JavaScript，因为它往往太轻易地失去了 `this` 的意义”。胖箭头通过从周围上下文捕获 `this` 的意义修复了它。考虑这个纯 JavaScript 类：

```ts
function Person(age) {
    this.age = age
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, 应该是 2
```
如果你在浏览器上运行这段代码，函数中的 `this` 会指向 `window` 因为 `window` 会是那个执行 `growOld` 函数的对象。修复的方案是使用箭头函数：

```ts
function Person(age) {
    this.age = age
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
这会有效的原因是 `this` 的引用被箭头函数从函数体外部捕获了。这等价于下面这段 JavaScript 代码（如果你没有 TypeScript 你也可以自己写）：

```ts
function Person(age) {
    this.age = age
    var _this = this;  // 捕获 this
    this.growOld = function() {
        _this.age++;   // 使用被捕获的 this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
注意到因为你在使用 TypeScript，你可以在语法上变的更甜，把箭头和类组合起来：

```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

#### 提示：箭头函数的需要{#tip-arrow-function-need}
通过简洁的语法，如果你想要把函数给其他人来调用，你只*需要*使用箭头函数。简单地：

```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```
如果你想要自己调用，即：

```ts
person.growOld();
```
那么 `this` 会是正确的上下文（在例子 `person` 中）。

#### 提示：箭头函数的危险{#tip-arrow-function-danger}

事实上如果你想要 `this` *是调用上下文*，你*不应该使用箭头函数*。这是在被像是 jquery，underscore，mocha 和其他这样的库使用的回调函数时的情况。如果文档提及了在 `this` 上的函数，那么你应该只使用 `function` 而不是胖箭头。类似的如果你打算使用 `arguments`，就别使用箭头函数。

#### 提示：箭头函数和使用了 `this` 的库{#tip-arrow-functions-with-libraries-that-use-this}
很多库都这样做了，例如 `jQuery` 迭代器（一个例子 http://api.jquery.com/jquery.each/）会使用 `this` 来传递正在迭代的对象给你。在这种情况下，如果你想要访问库传递的 `this` 和周围的上下文，只需要使用一个临时变量例如 `_self`，就像你在没有箭头函数的时候做的一样。

```ts
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

#### 提示：箭头函数和继承{#tip-arrow-functions-and-inheritance}

如果你有一个实例方法是箭头函数那么它会保持 `this`。既然只有一个 `this`，这种函数不能使用 `super` 调用（`super` 只对原型成员有效）。你可以简单地在子类中重载它之前创建一个方法的副本来获得它。

```ts
class Adder {
    // This function is now safe to pass around
    add = (b: string): string => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: string): string => {
        return this.superAdd(b);
    }
}
```
