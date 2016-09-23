# TypeScript 同 NodeJS
TypeScript 自成立以来就有了对 NodeJS 的*一等*支持。下面是怎么在 TypeScript 中开始一个 NodeJS 项目：

1. 使用设置为 `"commonjs"` 的 `--module` 编译（就像我们在[模块](../project/external-modules.md)中提到的那样）
2. 添加 `node.d.ts`（`typings install dt~node --global`）到你的[编译上下文中](../project/compilation-context.md)。

现在你可以伴随着 TypeScript 的安全性和开发工程性使用所有 node 模块（例如 `import fs = require('fs')`）！


## 创建 TypeScript node 模块

你甚至可以使用其他以 TypeScript 编写的 node 模块。作为一个模块开发者，一件你应该真正去做的事是：

* 你应该在你的 `package.json` 中拥有一个 `typings` 域（例如：`src/index`）类似于 `main` 域来指向默认的 TypeScript 定义出口。例如 [`package.json` for csx](https://github.com/basarat/csx/blob/gh-pages/package.json)。


示例包：`npm install csx` 来安装 [csx](https://www.npmjs.com/package/csx)，使用方式：`import csx = require('csx')`。


## 另外

这样的 NPM 模块在 browserify（使用 tsify）或者 webpack（使用 ts-loader）也工作的很好。
