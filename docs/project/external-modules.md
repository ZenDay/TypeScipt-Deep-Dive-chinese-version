## 外部模块
TypeScript 外部模块模式有着强大的能力和可用性。这里我们讨论它的能力和一些需要反映到真实世界使用的模式。

### 文件查找
下面的语句：

```ts
import foo = require('foo');
```

告诉 TypeScript 编译器去寻找这种形式的外部模块声明：

```ts
declare module "foo" {
    /// 一些变量定义

    export var bar:number; /*例子*/
}
```
一个相对路径的引入例子：

```ts
import foo = require('./foo');
```
告诉 TypeScript 编译器在相对于当前文件的路径 `./foo.ts` 或者 `./foo.d.ts` 去寻找 TypeScript 文件。

这不是完整的规范，但它是一个正确的思想模型来拥有和使用。我们稍后会描述本质的细节。

### 编译器模块选项
下面的语句：

```ts
import foo = require('foo');
```

基于编译器*模块*选项（`--module commonjs` 或 `--module amd` 或 `--module umd` 或 `--module system`）会生成*不同的* JavaScript。

个人推荐：使用 `--module commonjs`，然后你的代码就可以在 NodeJS 和使用了类似于 `webpack` 的前端上工作了。

### 只引入类型
下面的语句

```ts
import foo = require('foo');
```

实际上做了*两*件事

* 引入 foo 模块的类型信息。
* 指定 foo 模块上的运行时依赖。

你可以进行挑选以便只有*类型信息*被加载而没有运行时依赖发生。在继续下去之前，你可能想要重温下这本书的 [*声明空间*](../project/declarationspaces.md) 这章。

如果你没有使用变量声明空间中的引入名称，那么引入会完全从生成的 JavaScript 中被移除。最好通过例子来解释这些东西。一旦你理解它，我们就会给你呈现一些用例。

#### 例子 1
```ts
import foo = require('foo');
```
会生成 JavaScript：

```js

```
没错，因为 foo 没被使用，生成了*文件*。

#### 例子 2
```ts
import foo = require('foo');
var bar: foo;
```
会生成 JavaScript：

```js
var bar;
```
这是因为 `foo`（或者它的任意属性如：`foo.bas`）从未被作为变量使用。

#### 例子 3
```ts
import foo = require('foo');
var bar = foo;
```
会生成 JavaScript（假设是 commonjs）：

```js
var foo = require('foo');
var bar = foo;
```
这是因为 `foo` 被用做变量。


### 用例：懒加载
类型推断需要在*早期*完成。这意味着如果你想要在文件 `bar` 中使用一些来自文件 `foo` 的类型，你必须要这样做：

```ts
import foo = require('foo');
var bar: foo.SomeType;
```
然而你可能想在运行时处于特定状态时才加载文件 `foo`。这些情况下你应该只在*类型注解*下使用 `import` 的名称，而且**不**作为*变量*使用。这移除了所有早期被 TypeScript 注入的运行时依赖代码。 Then *manually import* the actual module using code that is specific to your module loader.然后*手动的引入*特定于你的模块加载器的真正模块使用代码。

作为例子，下面的基于 `commonjs` 的代码里，我们只在特定的函数调用上加载一个模块

```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation这里懒加载 `foo` 以及使用了原始的模块*只*作为类型注解。
    var _foo: typeof foo = require('foo');
    // 现在使用 `_foo` 作为 `foo` 的替代变量。
}
```

一个类似的 `amd`（使用了 requirejs）例子会是：

```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation
    require(['foo'], (_foo: typeof foo) => {
        // Now use `_foo` as a variable instead of `foo`.
    });
}
```

这种模式通常用于：

* 在特定路由下加载特定的 JavaScript 的 web 应用
* 为了加速应用的启动按需加载特定模块的 node 应用。

### 用例：打破循环依赖

类似于懒加载用例，特定的模块加载器（commonjs/node 和 amd/requirejs）在循环依赖下不能正常工作。在这个例子里，一个方向上的*懒加载*代码和另一个方向上的提前加载模块就十分有用了。

### 用例：确保引入

有的时候，你想要只是为了副作用加载一个文件（例如模块可能通过一些像是 [CodeMirror addons](https://codemirror.net/doc/manual.html#addons) 的库注册了自己等）。然而如果你仅仅做了 `import/require`，转换出来的 JavaScript 将不包括模块上的依赖，而你的模块加载器（例如 webpack）可能会完全忽略引入。在这个例子里你可以使用 `ensureImport` variable 来确保编译出来的 JavaScript 拿到了模块的依赖，例如：

```ts
import foo = require('./foo');
import bar = require('./bar');
import bas = require('./bas');
const ensureImport: any =
    foo
    || bar
    || bas;
```
The key advantage of using `import/require` instead of just `var/require` is that you get file path completion / checking / goto definition navigation etc.使用 `import/require` 替代仅仅是 `var/require` 的关键好处在于你可以获得文件路径补全／检查／跳转定义导航等。

[](// TODO: es6 modules)
