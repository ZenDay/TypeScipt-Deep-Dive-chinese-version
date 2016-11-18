# `@types`

[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped) 绝对是 TypeScript 最棒的优势之一。社区已经高效地**编档**了接近 90％ 的顶尖 JavaScript 项目的生态。

这意味着你可以通过一种交互探索性的方式使用这些项目，而不需要为这些文档开启一个单独的窗口并且确认你没有打错字。

## 使用 `@types`

安装的过程很简单，因为它就工作于 `npm` 之上。下面是一个你可以简单地安装 `jquery` 的类型定义的例子：

```
npm install @types/jquery --save-dev
```

`@types` 支持*全局*和*模块*类型定义。


### 全局 `@types`

默认任何支持全局消费的定义会被自动包括进来。例如：对于 `jquery` 你应该能够在你的项目中直接*全局*使用 `$`。

然而对于*库*（例如 `jquery`），我推荐使用*模块*：

### 模块 `@types`

在安装以后，没有特别的配置会被真正地引入。你只要像模块一样使用它：

```
import * as $ from "jquery";

// 在这个模块中尽情使用 $ :)
```

## 控制全局

可以看出拥有一个支持全局自动泄漏的定义可能会导致一些团队的问题，所以你可以选择明确地增加有意义的类型，通过使用 `tsconfig.json` `compilerOptions.types` 例如：

```
{
    "compilerOptions": {
        "types" : [
            "jquery"
        ]
    }
}
```

上面展示了一个只有 `jquery` 允许被使用的例子。即时某个人像这样 `npm install @types/node` 安装了另外一个定义是全局的（例如：[`process`](https://nodejs.org/api/process.html)），也不会泄漏到你的代码中，除非你把它们添加到 `tsconfig.json` 类型选项里。
