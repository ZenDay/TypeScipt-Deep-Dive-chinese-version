## 模块

### 全局模块

默认情况下当你开始在一个新的 TypeScript 文件里写代码时，你的代码时在*全局*命名空间里的。以 `foo.ts` 作为例子：

```ts
var foo = 123;
```

如果你现在创建一个*新的*文件 `bar.ts` 在相同的项目中，你会被 TypeScript 类型系统允许使用变量 `foo 因为它是全局可用的：

```ts
var bar = foo; // 允许
```
不用说全局命名空间是危险的，因为它会导致你的代码出现命名冲突。我们推荐使用文件模块，这将会在下面介绍。

### 文件模块
也叫*外部模块*。如果你在 TypeScript 文件的根层次有`import` 或 `export` 的话，它会在文件中创建一个*本地*作用域。所以如果我们把之前的 `foo.ts` 改成下面这样（注意 `export` 的使用）：

```ts
export var foo = 123;
```

我们会不再在全局命名空间里有 `foo`。这可以通过创建一个新的文件 `bar.ts` 得到证明：

```ts
var bar = foo; // ERROR: "cannot find name 'foo'"
```

如果你想要在 `bar.ts` 中使用 `foo.ts` 中的东西，*你需要明确地引入它*。下面是更新后的 `bar.ts`：

```ts
import {foo} from "./foo";
var bar = foo; // 允许
```
在 `bar.ts` 中使用一个 `import` 不仅允许你从其他文件中带来东西，还把文件 `foo.ts` 标记为一个*模块*，因此 `foo.ts` 不会污染全局命名空间。

使用了外部模块的给定 TypeScript 文件生成的 JavaScript 取决于编译器标识 `module`。
