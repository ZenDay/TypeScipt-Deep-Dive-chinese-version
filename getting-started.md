* [开始使用 TypeScript](#getting-started-with-typescript)
* [TypeScript 版本](#typescript-version)

# 开始使用 TypeScript{#getting-started-with-typescript}

TypeScript 会被编译成 JavaScript。JavaScipt 是你实际要执行的代码（在浏览器或者服务器上）。所以你需要下面的这些东西：

* TypeScript 编译器（开源可用：[源代码](https://github.com/Microsoft/TypeScript/) 以及 [NPM](https://www.npmjs.com/package/typescript)）
* 一个 TypeScript 编辑器（如果你喜欢你也可以用 notepad 但是我使用 [alm 🌹](http://alm.tools)）


![](https://raw.githubusercontent.com/alm-tools/alm-tools.github.io/master/screens/main.png)


## TypeScript 版本{#typescript-version}

不使用*固定版本*的 TypeScript 编译器，我们将在这本书里介绍很多新的特性，这些特性可能不会与某个版本号联系到一起。我通常推荐人们使用 nightly 版本，因为**编译器测试套件只会随着时间的推移发现更多的 bugs**。

你可以通过命令行安装：

```
npm install -g typescript@next
```

然后现在命令  `tsc` 将会是最新而且最棒的。不同的 IDE 也都支持它，例如：

* `alm` 总是装载着最新的 TypeScript 版本。
* 你可以通过创建 `.vscode/settings.json` 以及写入以下内容来告诉 vscode 使用这个版本：

```json
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

## TypeScript 定义
TypeScript 有一个外部 JavaScript 代码库*声明文件*的概念。将近 90% 的优秀 JavaScript 库都会使用在一个名叫 [DefinitelyTyped](http://definitelytyped.org/) 的项目中的*高质量*文件。你需要 `typings` 来获取这些定义。别担心，之后我们会解释这意味着什么…现在只需要安装：

```
npm install -g typings
```

## 获取源代码
本书的源代码可以在 github 仓库 https://github.com/basarat/typescript-book/tree/master/code 中找到。绝大部分的示例代码可以被复制到 alm 中，然后你就可以愉快地和它们玩耍了。对于那些需要额外设置（例如 npm 模块）的示例代码，我们会在呈现代码之前给予你代码的链接。

`this/will/be/the/link/to/the/code.ts`
```ts
// This will be the code under discussion
```

随着开发配置完成，让我们进入 TypeScript 语法的部分。