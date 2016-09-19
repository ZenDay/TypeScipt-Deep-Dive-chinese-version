## Which files?

You can either use `files` to be explicit:

```json
{
    "files":[
        "./some/file.ts"
    ]
}
```

or `include` and `exclude` to specify files. e.g.


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

Some notes:

* if `files` are specified other options are ignored
* `**/*` means all folder and any files (the extensions `.ts`/`.tsx` will be included and even `.js`/`.jsx` if `allowJs` is true).
