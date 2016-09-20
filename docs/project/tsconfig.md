### 基本
tsconfig.json 非常容易上手，因为你需要的基本文件就是：

```json
{}
```
即，在你项目的*根目录*下的空白 JSON 文件。这样做的话，TypeScript 会包括*所有*在这个目录（和子目录）下的 `.ts` 文件到编译上下文的一部分。它还会选择一些明智的默认编译选项。

### compilerOptions
你可以使用 `compilerOptions` 客制化编译器选项。

```json
{
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "declaration": false,
        "noImplicitAny": false,
        "removeComments": true,
        "noLib": false
    }
}
```

这些（以及更多）编译器选项会在稍后讨论。

### TypeScript 编译器
好的 IDE 内置了动态的 `ts` 到 `js` 的编译。然而，如果你在使用 `tsconfig.json` 时想要手动地从命令行运行 TypeScript 编译器，有几种方法可以做到。

* 运行 `tsc`，然后他会在当前目录和父级目录寻找 `tsconfig.json` 直到找到为止。
* 运行 `tsc -p ./path-to-project-directory`。当然路径既可以是完整的或者相对于当前目录的。

你甚至可以通过使用 `tsc -w` 来开启 TypeScript 编译器的观察模式，而它会观察你的 TypeScript 项目文件的变化。
