# External

> 配置：`build`

`external` 标记哪个模块不被打包。当遇到导入该模块时，保持`import` 导入语法（iife 和 cjs 格式使用 `require`，`esm` 使用 `import`），在运行时导入。

- 可以不打包那些永远不会被执行的代码；
- 有些包不能被打包，比如 `fsevents`，其包含了一些 native 代码，需要在使用时动态导入。

官方示例如下：
```javascript
// 源码
require('fs').writeFileSync('app.js', 'require("fsevents")')

// 脚本
require('esbuild').buildSync({
  entryPoints: ['app.js'],
  outfile: 'out.js',
  bundle: true,
  platform: 'node',
  external: ['fsevents'],
})

// 输出
{ errors: [], warnings: [] }
```
在 `external` 也可以使用 `*` 通配符。例如 `*.png` 将匹配所有的 `.png` 文件， `/images/*` 将匹配所有路径以 `/images/` 开头的文件。这个路径匹配的是源代码中的路径，不是经过 `resolve` 后的真实路径。

## 自己编的案例
有如下目录结构，不打包 `external.self.js` 文件：
```
/project_root
|___/scripts
    |___run.ts
|___/src
    |___app.ts
    |___external.self.js
|___package.json
```

源码文件 `app.ts`：
```typescript
import { name } from './external.self.js';
console.log(name);
```
源码文件 `external.self.js`：
```javascript
module.exports = {
    name: "external"
}
```
脚本文件 `run.ts`：
```typescript
async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        external: ['*.self.js'],
        outdir: 'out',
        format: 'cjs'
    })
}
run();
```

生成的文件如下：
```javascript
var __create = Object.create;
var __defProp = Object.defineProperty;
var __getOwnPropDesc = Object.getOwnPropertyDescriptor;
var __getOwnPropNames = Object.getOwnPropertyNames;
var __getProtoOf = Object.getPrototypeOf;
var __hasOwnProp = Object.prototype.hasOwnProperty;
var __markAsModule = (target) => __defProp(target, "__esModule", { value: true });
var __reExport = (target, module2, desc) => {
  if (module2 && typeof module2 === "object" || typeof module2 === "function") {
    for (let key of __getOwnPropNames(module2))
      if (!__hasOwnProp.call(target, key) && key !== "default")
        __defProp(target, key, { get: () => module2[key], enumerable: !(desc = __getOwnPropDesc(module2, key)) || desc.enumerable });
  }
  return target;
};
var __toModule = (module2) => {
  return __reExport(__markAsModule(__defProp(module2 != null ? __create(__getProtoOf(module2)) : {}, "default", module2 && module2.__esModule && "default" in module2 ? { get: () => module2.default, enumerable: true } : { value: module2, enumerable: true })), module2);
};

// src/app.ts
var import_external_self = __toModule(require("./external.self.js"));
console.log(import_external_self.name);
```
看最后两行，`require` 的路径是 `./external.self.js`，当运行该文件时，会报无法找到该模块的错误。

### 修复
这种情况下，可以使用插件，将 `require` 的导入路径改为绝对路径，同时将 `*.self.js` 标记为 `external`，就不用使用配置中的 `external` 字段了。
新的脚本文件如下：
```typescript
import { build, Plugin } from 'esbuild';
import { resolve } from 'path';

const externalPlugin: Plugin = {
    name: 'externalPlugin',
    setup(build) {
        build.onResolve({filter: /.self.js$/}, (args) => {
            args.path
            return {
                path: resolve(args.resolveDir, args.path),
                external: true
            }
        });
    }
}

async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        outdir: 'out',
        format: 'cjs',
        plugins: [externalPlugin]
    })
}
run();
```

生成的文件最后两行变为：
```javascript
var import_external_self = __toModule(require("/Users/xxxxxx/Documents/02-study/02-learn/17-vite/esbuild-code/06_external/src/external.self.js"));
console.log(import_external_self.name);
```
这样引入绝对路径，运行是可以的。当前也可以通过插件，改为相对生成文件的路径。





