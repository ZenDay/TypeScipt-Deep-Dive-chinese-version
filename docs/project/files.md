## 哪些文件？

你可以使用 `files` 来明确指出：

```json
{
    "files":[
        "./some/file.ts"
    ]
}
```

或者 `include` 和 `exclude` 来指定文件，例如：


```json
{
    "include":[
        "./folder/**/*"
    ],
    "exclude":[
        "./folder/**/*.spec.ts",
        "./folder/someSubFolder"
    ]
}
```

一些笔记：

* 如果 `files` 被指定了，那么其他的选项会被忽略
* `**/*` 表示所有文件夹和所有文件（`.ts`/`.tsx` 扩展名会被包括进去，如果 `allowJs` 是 true 的话，`.js`/`.jsx` 也会被包括进去）。
