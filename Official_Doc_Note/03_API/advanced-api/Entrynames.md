# EntryNames

> 配置 `build`

> 注意，其优先级高于 `outbase`。 

`entryNames` 通过设置模板，设置输出文件目录与名称，名称与之对应的入口点文件的格式。

默认情况下，输出文件目录与名称与入口点相同。
其基本使用如下所示：
```javascript
require('esbuild').buildSync({
  entryPoints: ['src/main-app/app.js'],
  entryNames: '[dir]/[name]-[hash]',
  outbase: 'src',
  bundle: true,
  outdir: 'out',
})
```
`entryNames` 可使用的模板标注有三个：
- `[dir]`
- `[name`
- `[hash]`

使用 `entryNames` 时，不用加扩展名，esbuild 会根据文件内容提供合适的扩展名。

## `[dir]`
`[dir]` 是入口点相对 `outbase` 的**相对目录**，其目的是为了防止不同目录下同名入口点的名称冲突。

比如有两个入口点 `src/pages/home/index.ts` 和 `src/pages/about/index.ts`，`outbase` 设置为 `src`，`entryNames` 设置为 `[dir]/[name]`，则生成的代码为：
```
/out_dir
|___/pages
    |___/home
        |___index.js
    |___/about
        |___index.js
```
如果将 `entryNames` 设置为 `[name]`，打包会失败，如果不失败，会生成下面的目录结构：
```
/out_dir
|___index.js
|___index.js
```
同一目录下，两个文件命名冲突，所以打包失败。
如果将 `entryNames` 设置为 `[name]-[hash]`，则可以打包成功，生成的目录结构为：
```
/out_dir
|___index-Adfaadfa.js
|___index-Bddfalko.js
```
但 `outbase` 的设置相当于失效了，而且两个文件不好区分。

## `[name]`
`[name]` 是入口点名字。比如入口点是`./src/xxx/app.js`，则 `[name]` 为 `app`。
如果入口点采用如下设置，`[name]` 则为 `appEntry`：
```javascript
{
    entryNames: {
        appEntry: "./src/xxx/app.js"
    }
}
```
## `[hash]`

`[hash]` 值与文件内容相关的随机数，只有当文件更改时，才会发生变化。



