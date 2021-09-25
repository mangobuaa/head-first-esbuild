# Target

> 配置 `build` 和 `transform`

`target` 设置生成的 `js`、`css` 的运行环境。
- 可以使用 ES 版本号，如 `es2015`, `es2016`, `esnext`
- 可以使用浏览器版本号，如 `chrome58`
- 可以使用 node 版本号，如 `node10`

默认使用 `esnext`，esbuild 会使用所有已发布的新语法。

下面是一个例子，在实际实用时，一般会使用这个配置的一个子集。

```javascript
require('esbuild').buildSync({
  entryPoints: ['app.js'],
  target: [
    'es2020',
    'chrome58',
    'firefox57',
    'safari11',
    'edge16',
    'node12',
  ],
  outfile: 'out.js',
})
```

注意，当 esbuild 对 js 做降级处理时，如果无法降级成功，则构建失败。在转换为 es5 时，时有发生；转换为 es6 基本没问题。
