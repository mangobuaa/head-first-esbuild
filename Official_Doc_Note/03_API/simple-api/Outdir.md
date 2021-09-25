# Outdir

> 配置： `build`

`outdir` 设置打包后输出文件的目录。
下面代码，会在项目根目录生成一个 `/out` 目录存放打包后的文件。

```javascript
require('esbuild').buildSync({
  entryPoints: ['app.js'],
  bundle: true,
  outdir: 'out',
})
```

打包输出到 `outdir` 目录时，
- 如果该目录不存在，会创建；
- 如果已经存在，不会清除该目录内的其他文件
- 如果有同名文件，会覆盖

如果多个入口点分散在不同的目录中，默认情况下，会以入口点的第一个共同目录为基准生成打包目录。
比如，有如下入口点 `entryPoints: ['./src/page/home/home.ts', './src/page/about/about.ts]`，生成的文件如下：
```
/out_dir
|___/home
    |___home.js
|___about
    |___about.js
```
因为两个入口的第一个共同目录是 `/page`，此时 esbuild 会以此为基准生成。

如果想改变这个默认行为，可以配置 `outbase` 选项。
比如将上面案例，配置 `outbase: './src'`，生成的文件如下：

```
/out_dir
|___/page
    |___/home
        |___home.js
    |___about
        |___about.js
```