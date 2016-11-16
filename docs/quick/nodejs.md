# TypeScript 同 NodeJS
TypeScript 自成立以来就有了对 NodeJS 的*一等*支持。下面是如何快速开始一个 NodeJS 项目：

> 注意：这里的很多步骤实际上只是普通地练习 nodejs 开始步骤

1. 开始一个 nodejs 项目 `package.json`。快速的方法：`npm init -y`
2. 添加 TypeScript（`npm install typescript --save-dev`）
3. 添加 `node.d.ts` (`npm install @types/node --save-dev`)
4. 为 TypeScript 选项初始化一个 `tsconfig.json`（`node ./node_modules/.bin/tsc --init`）

就是这样！启动你的 IDE（例如 `alm -o`）然后玩耍起来。现在你可以使用所有以 node 模块构建的东西（例 `import fs = require('fs')`），并且享受 TypeScript 的安全性和开发者人体工程学。

## 另外：即时编译 + 运行
* 添加 `ts-node`，我们将会使用它来在 node 中进行即时编译 + 运行（`npm install ts-node --save-dev`）
* 添加 `nodemon`，它会在一个文件改变的时候调用 `ts-node`（`npm install nodemon --save-dev`）

现在只需要在你的 `package.json` 中基于你的应用入口如 `index.ts` 添加一个 `script` 目标即可：

```json
  "scripts": {
    "start": "npm run build:live",
    "build:live": "nodemon --exec ./node_modules/.bin/ts-node -- ./index.ts"
  },
```
所以现在你可以运行`npm start` 并且当你编辑 `index.ts` 时：

* nodemon 重新运行它的命令（ts-node）
* ts-node 自动获取 tsconfig.json 和当前安装的 typescript 版本转换编译
* ts-node 通过 node 运行输出的 javascript。

## 创建 TypeScript node 模块

你甚至可以使用其他以 TypeScript 编写的 node 模块。作为一个模块开发者，一件你应该真正去做的事是：

* 你应该在你的 `package.json` 中拥有一个 `typings` 域（例如：`src/index`）类似于 `main` 域来指向默认的 TypeScript 定义出口。例如 [`package.json` for csx](https://github.com/basarat/csx/blob/gh-pages/package.json)。


示例包：`npm install csx` 来安装 [csx](https://www.npmjs.com/package/csx)，使用方式：`import csx = require('csx')`。


## 另外

这样的 NPM 模块在 browserify（使用 tsify）或者 webpack（使用 ts-loader）也工作的很好。