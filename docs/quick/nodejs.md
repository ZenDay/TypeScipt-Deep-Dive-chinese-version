# TypeScript 同 NodeJS
TypeScript 自成立以来就有了对 NodeJS 的*一等*支持。下面是怎么在 TypeScript 中开始一个 NodeJS 项目：

1. 添加 `node.d.ts` (`npm install @types/node --save-dev`) 到你的[编译上下文中](../project/compilation-context.md)。
2. 把 `--module` 设为 `"commonjs"` 进行编译。
3. 通过简单地在你的 tsconfig 添加 node 作为 `types`  来加到全局解决方案里面。

所以你的 tsconfig 会看起来像这样：

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "types": [
            "node"
        ]
    }
}
```

现在你可以伴随着 TypeScript 的安全性和开发工程性使用所有 node 模块（例如 `import fs = require('fs')`）！


## 创建 TypeScript node 模块

你甚至可以使用其他以 TypeScript 编写的 node 模块。作为一个模块开发者，一件你应该真正去做的事是：

* 你应该在你的 `package.json` 中拥有一个 `typings` 域（例如：`src/index`）类似于 `main` 域来指向默认的 TypeScript 定义出口。例如 [`package.json` for csx](https://github.com/basarat/csx/blob/gh-pages/package.json)。


示例包：`npm install csx` 来安装 [csx](https://www.npmjs.com/package/csx)，使用方式：`import csx = require('csx')`。


## 另外

这样的 NPM 模块在 browserify（使用 tsify）或者 webpack（使用 ts-loader）也工作的很好。