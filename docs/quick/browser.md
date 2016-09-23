# 浏览器端 TypeScript
如果你正在使用 TypeScript 来创建 web 应用，这里是我的建议：

## 通用机器设置

* 安装 [NodeJS](https://nodejs.org/en/download/)

## 项目设置
* 创建一个项目文件夹
```
mkdir your-project
cd your-project
```
* 创建 `tsconfig.json`。这里我们讨论[模块](../project/external-modules.md)。同样也可以设置 `tsx` 编译。
```json
{
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "sourceMap": true,
        "jsx": "react"
    },
    "exclude": [
        "node_modules"
    ],
    "compileOnSave": false
}
```
* 创建一个 npm 项目：
```
npm init -y
```
* 安装 TypeScript-nightly，webpack，[`ts-loader`](https://github.com/TypeStrong/ts-loader/), typings
```
npm install typescript@next webpack ts-loader typings --save-dev
```
* 初始化 typings（创建 `typings.json` 文件）。
```
"./node_modules/.bin/typings" init
```
* 创建 `webpack.config.js` 来打包你的模块到一个单独的 `bundle.js` 文件，它包含了所有你的资源：
```js
module.exports = {
    entry: './src/app.tsx',
    output: {
        filename: './dist/bundle.js'
    },
    resolve: {
        // Add `.ts` and `.tsx` as a resolvable extension.
        extensions: ['', '.webpack.js', '.web.js', '.ts', '.tsx', '.js']
    },
    module: {
        loaders: [
            // all files with a `.ts` or `.tsx` extension will be handled by `ts-loader`
            { test: /\.tsx?$/, loader: 'ts-loader' }
        ]
    }
}
```
* 设置 npm 脚本来运行构建。同样使它在 `npm install` 时运行 `typings install`。在你的 `package.json` 里添加 `script` 部分：
```json
"scripts": {
    "prepublish": "typings install",
    "watch": "webpack --watch"
},
```

现在只要运行下面的（在包含了 `webpack.config.js` 的目录下）：

```
npm run watch
```

现在如果你编辑了你的 `ts` 或者 `tsx` 文件，webpack 会为你生成 `bundle.js`。使用你的 web 服务器托管这个文件🌹。

## 更多
如果你打算使用 React（我非常推荐你去看看它），这里还有一些步骤：

```
npm install react react-dom --save-dev
```

```
"./node_modules/.bin/typings" install dt~react --global --save
```

```
"./node_modules/.bin/typings" install dt~react-dom --global --save
```

示例的 `index.html`：

```
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello React!</title>
    </head>
    <body>
        <div id="root"></div>

        <!-- Main -->
        <script src="./dist/bundle.js"></script>
    </body>
</html>
```
示例的 `./src/app.tsx`

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";

const Hello = (props: { compiler: string, framework: string }) => {
    return (
        <div>
            <div>{props.compiler}</div>
            <div>{props.framework}</div>
        </div>
    );
}

ReactDOM.render(
    <Hello compiler="TypeScript" framework="React" />,
    document.getElementById("root")
);
```

你可以在这里克隆示例项目：https://github.com/basarat/react-typescript
