# Inject

`inject` 可以通过导入文件，替换全局变量。对于适配全局环境非常有用。
比如，我们有如下目录结构：

```
/project_root
|___/scripts
    |___run.ts
    |___process_shim.ts
|___/src
    |___app.ts
|___package.json
```

源码文件 `app.ts` 如下：
```typescript
declare var process: any;
console.log(process.cwd());
```

脚本文件 `run.ts` 如下：
```typescript
import { build } from 'esbuild';

async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        outdir: 'out',
        inject: ['./scripts/process_shim.ts']
    }) 
}
run();
```

`inject` 配置的 `process_shim.ts` 的如下：
```typescript
export const process = {
    cwd: () => "process_shim_cwd"
}
```

运行脚本文件，生成的 `app.js` 文件内容如下：
```javascript
(() => {
  // scripts/process_shim.ts
  var process = {
    cwd: () => "process_shim_cwd"
  };

  // src/app.ts
  console.log(process.cwd());
})();
```
从 `./out/app.js` 文件看出，esbuild 将 `inject` 插入的文件一起打包了。

下面更改 `process_shim.ts` 文件如下：
```typescript
const cwd = "./src/";
const author = "Mango";
export const process = {
    cwd: () => cwd
}
```
在 `process_shim.ts` 文件中导出了 `process`，其引用了 `cwd` 变量，而 `author` 变量没有导出。
生成的代码如下：
```javascript
(() => {
  // scripts/process_shim.ts
  var cwd = "./src/";
  var process = {
    cwd: () => cwd
  };

  // src/app.ts
  console.log(process.cwd());
})();
```
从打包文件可看出，如果没有打包 `author`，这是 esbuild 的自动 `Tree Shaking` 功能，也可以设置 `treeShaking: false` 来加入 `author` 变量。

## 与 `define` 结合使用
仍使用上面的例子，此时 `process_shim.ts` 改为：
```typescript
export function dummy_process_cwd() {
    return "dummy_process_cwd";
}
```

脚本文件 `run.ts` 如下：
```typescript
import { build } from 'esbuild';
async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        outdir: 'out',
        define: {
            "process.cwd": "dummy_process_cwd"
        },
        inject: ['./scripts/process_shim.ts'],
        treeShaking: false
    }) 
}
run();
```
生成的文件 `out/app.js` 如下：
```javascript
(() => {
  // scripts/process_shim.ts
  function dummy_process_cwd() {
    return "dummy_process_cwd";
  }

  // src/app.ts
  console.log(dummy_process_cwd());
})();
```

## 注入没有导入语法的文件
也可以配置没有 `export` 导出语法的文件。这种情况下，被注入的文件会被插入开头，就像所有文件都有 `import "./file.ts` 一样。

> TODO: 自己试验时，没有 `export` 的文件，不会被打包，需要继续探索。

## 条件注入
如果只有源代码中使用到导出的变量时，才将这个文件导入，需要将这个文件标记为 `sideEffects: false`。
这个文件需要加入一个 npm 包，并且在 `package.json` 中设置 `sideEffects: false`。
