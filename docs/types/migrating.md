## 从 JavaScript 迁移

通常来说这个过程包括了以下步骤：

* 添加 `tsconfig.json`
* 将你的源代码文件扩展名从 `.js` 改成 `.ts`。使用 `any` 来开始*抑止*错误。
* 使用 TypeScript 来编写新的代码并且尽可能少地使用 `any`。
* 返回到旧代码里并且开始加入类型标注和解决发现的 bugs。
* 为第三方 JavaScript 代码使用环境定义。

让我们来深入讨论这几个点。

注意所有 JavaScript 都是*有效的* TypeScript。那就是说，如果你给予 TypeScript 编译器一些 JavaScript -> 被 TypeScript  编译器释放的 JavaScript 会跟原始 JavaScript 表现地完全相同。这意味着改变 `.js` 扩展名为 `.ts` 将不会对你的代码库造成不良影响。

### 抑止错误
TypeScript 会马上开始对你的代码进行类型检查，并且你的原始 JavaScript 代码*可能不像你想象中的那么整洁*，因此你会得到诊断错误。对于其中的大部分错误你可以使用 `any` 来抑止，例如：

```ts
var foo = 123;
var bar = 'hey';

bar = foo; // ERROR: cannot assign a number to a string
```

即使**错误是合法的**（而且在大部分情况下推论信息会比代码库不同部分的原作者所想象的更好），你的关注点可能会在用 TypeScript 编写新代码的同时渐进地更新旧代码库。这里你可以使用类型断言来抑止错误，如下所示：

```ts
var foo = 123;
var bar = 'hey';

bar = foo as any; // Okay!
```

在其他地方你可能想要标注某些东西为 `any`，例如：

```ts
function foo() {
    return 1;
}
var bar = 'hey';
bar = foo(); // ERROR: cannot assign a number to a string
```

抑止：

```ts
function foo(): any { // 添加 `any`
    return 1;
}
var bar = 'hey';
bar = foo(); // Okay!
```

> 注意：抑止错误是危险的，但是它使你更注意*新* TypeScript 代码中的错误。你可以在你继续下去的时候留下 `// TODO:` 注释。

### 第三方 JavaScript
你可以把你的 JavaScript 改变成 TypeScript，但是你不能让整个世界都使用 TypeScript。这就是 TypeScript 的环境定义出场的时候。一开始我们推荐你创建 `vendor.d.ts`（`.d.ts` 扩展名说明这是一个*声明文件*）并且开始在里面添加一些肮脏的东西。或者为特定库创建特定的文件，例如 `jquery.d.ts` 对应 jquery。

> 注意：90% 的 JavaScript 库都有被良好地维护着的类型定义存在于一个叫做 [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped) 的 OSS 仓库中。我们推荐在像我们介绍的那样创建你自己的定义之前先在那里寻找。尽管如此，这个快速而邋遢的方法对于减少你在 TypeScript 上的初始冲突上至关重要。

考虑 `jquery` 的例子，你可以简单地为它创建一个*微小的*定义：

```ts
declare var $: any;
```

有的时候你可能想要在某些东西上添加清晰的的标注（如 `JQuery`）并且你需要*类型声明空间*中的某些东西。你可以很简单地使用 `type` 关键字来做到：

```ts
declare type JQuery = any;
declare var $: JQuery;
```

这为你提供了更轻松的未来更新道路。

再次强调，一个高质量的 `jquery.d.ts` 存在于 [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped) 中。但是你现在知道如何*快速地*解决任何 JavaScript -> TypeScript 冲突，当在使用第三方 JavaScript 的时候。我们接下来会看到环境定义的细节。


# 第三方 NPM 模块

类似于全局变量声明，你可以很轻松地声明一个全局模块。例如对于 `jquery`，如果你想要把它当成模块（https://www.npmjs.com/package/jquery）来使用，你可以这样写：

```ts
declare module "jquery";
```

然后你就可以在需要的时候引入它：

```ts
import * as $ from "jquery";
```

> 再次强调，一个高质量的 `jquery.d.ts` 存在于 [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped) 中，它提供了一个更高质量的 jquery 模块声明。但是对于你的库，它可能存在，因此现在你有了一个快速而低阻力的方式来继续迁移 🌹

# 外部非 js 资源

你甚至可以引入任意文件，例如 `.css` 文件（如果你正在使用像是 webpack 这样的东西），通过一个简单的 `*` 风格声明：

```ts
declare module "*.css";
```

现在人们可以 `import * as foo from "./some/file.css";`
