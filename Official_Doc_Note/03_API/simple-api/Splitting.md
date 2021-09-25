# Splitting 代码分割

> 配置： `build`，该功能仍在开发中，当前只支持 esm 导出。

主要有两个方面的作用：

- 分割多个入口点共享的代码。
- 分割异步加载的代码。

如果没有开启 splitting，`import()` 表达式会转为 `Promise.resolve().then(() => require())`。虽然有异步属性，但是所加载的代码在同一个文件中。

当使用 `splitting` 时还需要设置 `outdir`。

以下例子展示 splitting 的效果：

1. 分割多个入口点共享的代码示例：
代码目录结构如下：
```
/project_root
|___/scripts
    |___run.ts
|___/out        <== 输出目录
|___/src
    |___entry1.ts
    |___entry2.ts
    |___share.ts
|___package.json        
```

源代码 `entry1.ts`， `entry2.ts`，`share.ts` 代码如下：
```typescript
// share.ts
export const share = "share";

// entry1.ts
import { share } from './share';
console.log('entry1: ', share);

// entry2.ts
import { share } from './share';
console.log('entry2: ', share);
```

脚本 `run.ts` 代码如下：
```typescript
import { build } from 'esbuild';

async function run() {
    await build({
        entryPoints: ['./src/entry1.ts', './src/entry2.ts'],
        bundle: true,
        format: 'esm',
        outdir: 'out',
        splitting: true
    });
}

run();
```

运行脚本后，会在 `/out` 目录下生成3个文件：
```
/out
|___entry1.js
|___entry2.js
|___chunk-5OFS4OED.js    <=== 后面的 hash 值每次不同
```
内部代码为：

```javascript
// chunk-5OFS4OED.js
var share = "share";

export {
  share
};

// entry1.js
import {
  share
} from "./chunk-5OFS4OED.js";
console.log("entry1: ", share);


// entry2.js
import {
  share
} from "./chunk-5OFS4OED.js";
console.log("entry2: ", share);
```

从上面生成结果看，把共用代码 `share.ts` 提取成单个的 chunk。

2. 动态导入分割示例
目录结构如下：
```
/project_root
|___/scripts
    |___run.ts
|___/out        <== 输出目录
|___/src
    |___main.ts
    |___dynamic.ts
|___package.json
```
源代码 `main.ts` 和 `dynamic.ts` 如下：
```typescript
// dynamic.ts
export const name = "dynamic.ts";

// main.ts
async function main() {
    const dynamic = await import("./dynamic");
    console.log(dynamic.name);
}
main();
```
脚本 `run.ts` 代码：
```typescript
import { build } from 'esbuild';
async function run() {
    await build({
        entryPoints: ['./src/main.ts'],
        bundle: true,
        format: 'esm',
        outdir: 'out',
        splitting: true
    });
}
run();
```

运行脚本后，会在 `/out` 目录下生成2个文件：
```
/out
|___main.js
|___dynamic-5ADD4OED.js    <=== 后面的 hash 值每次不同
```

其中代码为：
```javascript
// dynamic-5ADD4OED.js
var name = "dynamic.ts";
export {
  name
};

// main.js
async function main() {
  const dynamic = await import("./dynamic-UWQ42AO6.js");
  console.log(dynamic.name);
}
main();
```

从上面可以看出，生成的代码与源代码变化不大。

下面再看一个没有分割的版本，在脚本中设置配置 `splitting: false`，打包结果只有一个文件 `main.js`，其内容为：
```javascript
var __defProp = Object.defineProperty;
var __markAsModule = (target) => __defProp(target, "__esModule", { value: true });
var __esm = (fn, res) => function __init() {
  return fn && (res = (0, fn[Object.keys(fn)[0]])(fn = 0)), res;
};
var __export = (target, all) => {
  __markAsModule(target);
  for (var name2 in all)
    __defProp(target, name2, { get: all[name2], enumerable: true });
};

// src/dynamic.ts
var dynamic_exports = {};
__export(dynamic_exports, {
  name: () => name
});
var name;
var init_dynamic = __esm({
  "src/dynamic.ts"() {
    name = "dynamic.ts";
  }
});

// src/main.ts
async function main() {
    // 重点在这一行
  const dynamic = await Promise.resolve().then(() => (init_dynamic(), dynamic_exports));
  console.log(dynamic.name);
}
main();
```
从上面生成的代码可以看出，所有代码都打包到一个文件中，异步导入使用的 `Promise.resolve().then(() => ())` 语法。

