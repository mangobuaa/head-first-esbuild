# Entry Points

> 配置： `build`

入口点配置是一个字符串数组。每一个字符串是一个入口点。
其基本使用如下：

```javascript
require('esbuild').buildSync({
  entryPoints: ['home.ts', 'settings.ts'],
  bundle: true,
  write: true,
  outdir: 'out',
})
```

也可以给入口点起名字，如下所示，生成的文件为 `out/out1.js` 和 `out/out2.js`。
```javascript
require('esbuild').buildSync({
  entryPoints: {
    out1: 'home.js',
    out2: 'settings.js',
  },
  bundle: true,
  write: true,
  outdir: 'out',
})
```

如果想设置代码分割，需要配置 `splitting`

与入口点相关的配置，还可以参考：
- `splitting`，分割共用代码、异步导入
- `entryNames`，参考入口文件，设置生成文件的名称，优先级高于 `outbase`
- `outbase`，输入基准目录，默认以入口点的第一个共同目录为基准
- `outdir`，输出目录
- `outfile`，输出文件名称