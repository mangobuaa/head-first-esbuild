# Outfile

> 配置：`build`

`outfile` 指定输出文件的名字。不过这个字段只有一个入口点时有效。当有多个入口点时，需要使用 `outdir` 配置输出目录。

以下是使用示例：
```javascript
require('esbuild').buildSync({
  entryPoints: ['app.js'],
  bundle: true,
  outfile: 'out.js',
})
```