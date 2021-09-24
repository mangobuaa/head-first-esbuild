# Getting Started

## 1. 安装 esbuild
```bash
# 初始化仓库
npm init -y

# 安装 esbuild
npm install esbuild
```

可通过检测版本，查看是否已经正确安装：
```bash
# 使用 npx
npx esbuild --version

# 或者通过路径找到可执行文件
./node_modules/.bin/esbuild --version

# 0.12.28
```

## 2. 第一个 bundle
使用一个现实中的例子，展示 esbuild 可以做些什么。

安装 `react` 和 `react-dom`:
```bash
npm i react react-dom
```

构建如下目录结构：
```
/project_root
|___/src
    |___app.jsx
|___package.json
```

app.jsx 的代码如下：
```javascript
import React from "react";
import { renderToString } from 'react-dom/server';

let Greet = () => <h1>hello esbuild!</h1>;

console.log(renderToString(<Greet />));
```

使用 CLI 打包 app.jsx：
```bash
npx esbuild src/app.jsx --bundle --outfile=out.js
```
运行命令后，会在根目录下生成 `out.js` 文件。
使用 node 运行 `out.js`：
```bash
node out.js
```
会得到如下输出：
```
<h1 data-reactroot="">hello esbuild!</h1>
```

从上面的例子中可以看出，esbuild 会自动转换 JSX 语法。esbuild 提供了面向大部分场景的默认配置。如果需要更改配置，详情请看配置篇。

## 3. 配置 npm script
> 配置 npm 脚本，与使用其他 CLI 相同。

可以在 package.json 文件中配置脚本：

```json
{
  "scripts": {
    "build": "esbuild app.jsx --bundle --outfile=out.js"
  }
}
```

运行如下命令，即可构建：
```bash
npm run build
```

## 4. 自定义脚本
> 推荐这种方式，后面将以此方式为主

使用 esbuild 提供的 API，编写脚本，用 node 运行。如下是一个简单的脚本：
构建如下目录结构：
```
/project_root
|___/scripts
    |___run.js
|___/src
    |___app.jsx
|___package.json
```

run.js 的代码如下：
```javascript
const esbuild = require("esbuild");
esbuild.build({
    entryPoints: ['./src/app.jsx'],
    bundle: true,
    outfile: 'out.js'
}).catch(() => process.exit(1));
```
运行 run.js:
```bash
node scripts/run.js
```
效果与使用 CLI 相同。

与 `build()` 相对应的异步方法，还有一个 `buildAsync()` 同步方法。不过不推荐使用这个方法，因为plugins(插件) 只能在 `build()` 方法中使用。

## 构建浏览器版本
esbuild 默认配置就是支持浏览器的，生成的是一个iife(立即执行函数)格式的代码。
esbuild 也支持 sourcemap，minify 等配置，如下所示：

```javascript
const esbuild = require("esbuild");
esbuild.build({
    entryPoints: ['./src/app.jsx'],
    bundle: true,
    sourcemap: true,    // 生成 sourcemap
    minify: true,       // 压缩代码
    target: ['chrome58', 'firefox57', 'safari11', 'edge16'],    // 需支持的浏览器，不是必须的
    outfile: 'out.js'
}).catch(() => process.exit(1));
```

有时我们使用的包只支持在 node 环境下运行，比如这个包使用了 node 内置的 `path`。这个情况下可以在 package.json 中配置该 `path` 的替代包：

```json
// package.json
{
  "browser": {
    "path": "path-browserify"
  }
}
```

关于 `browser` 字段的详情，可查看[此处](https://github.com/defunctzombie/package-browser-field-spec)。

有些 npm 包的设计初衷不是在浏览器环境下运行的。一般我们可以使用 esbuild 的配置文件即可成功打包。
对于包中使用的全局变量，有时在浏览器中是未定义的，可以使用 `define` 和 `inject` 特性。这两个特性后续会介绍。

## 构建 node 版本

一般使用 node 时，是不需要打包的。不过经过打包，可以使发布的包体积更小等特性。
打包过程中，会自动去除 ts 类型、转换 esm 到 cjs、转换新语法等特性。

如果打包的代码运行在 node 平台，应该设置 `platform` 为 `node`。这个配置会修改其他的默认配置。比如，对于 `fs` 包会自动标记为 `external`，这样打包时，就不会将这个包打包进去。同时，也会忽略 `package.json` 文件中 `browser` 配置。

```javascript
require('esbuild').buildSync({
  entryPoints: ['app.js'],
  bundle: true,
  platform: 'node',
  target: ['node10.4'],
  outfile: 'out.js',
})
```

有时我们打包时，因为某些原因，不想将某个包进行打包。比如， `fsevents` 包含子本地扩展。可以使用 `external` 字段。

```javascript
require('esbuild').buildSync({
  entryPoints: ['app.jsx'],
  bundle: true,
  platform: 'node',
  external: ['fsevents'],
  outfile: 'out.js',
});
```

## 其他安装方式
> 现阶段主要学习 esbuild 使用，其他安装方式后续再补充。作为前端，暂时不用不到。
