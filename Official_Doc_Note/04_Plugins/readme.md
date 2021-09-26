# esbuild plugin

esbuild plugin 就是一个具有 `name` 和 `setup()` 的对象。 

They are passed in an array to the build API call. The setup function is run once for each build API call.

## 简单示例
构建一个可以从 `process.env` 中读取参数的插件。

目录结构如下：
```
/root
|___/src
    |___app.js
|___/plugins
    |___env-plugin.js
|___/scripts
    |___run-esbuild.js
```
各文件代码：
```javascript
// env-plugin.js
const envPlugin = {
    name: 'env',
    setup(build) {
        build.onResolve({ filter: /^env$/ }, args => {
            console.log('args: ', args);
            return {
                path: args.path,
                namespace: 'env-ns'
            } 
        });

        build.onLoad({ filter: /.*/, namespace: 'env-ns'}, () => {
            return {
                contents: JSON.stringify(process.env),
                loader: 'json'
            }
        });
    }
}
module.exports = envPlugin;
```
```javascript
// run-esbuild.js
const esbuild = require('esbuild');
const envPlugin = require('../plugins/env-plugin');

process.env.ENV = "dev";    // 开发环境标识

esbuild.build({
    entryPoints: ['./src/app.js'],
    bundle: true,
    outfile: './out/app.js',
    plugins: [envPlugin]
}).catch(() => process.exit(1));
```

```javascript
// app.js
import { ENV } from 'env';

console.log(`ENV is ${ENV}`);
```

运行:
```bash
node run scripts/run-esbuild.js
```
会生成 `out/app.js` 文件，运行：
```bash
node out/app.js
```
会得到其输出：
```
ENV is dev
```

## 从类型定义看插件各项参数
从下载的 esbuild 包中，可以找到 `lib/main.d.ts` 文件，内部定义了插件类型。

插件定义
```typescript
export interface Plugin {
  name: string;
  setup: (build: PluginBuild) => (void | Promise<void>);
}
```
由类型定义可看出，`setup` 就是一个函数，可以无返回值，也可以返回 Promise。

`setup` 的参数为 `PluginBuild` 类型，其类型定义为：

```typescript
export interface PluginBuild {
  initialOptions: BuildOptions;
  onStart(callback: () =>
    (OnStartResult | null | void | Promise<OnStartResult | null | void>)): void;
  onEnd(callback: (result: BuildResult) =>
    (void | Promise<void>)): void;
  onResolve(options: OnResolveOptions, callback: (args: OnResolveArgs) =>
    (OnResolveResult | null | undefined | Promise<OnResolveResult | null | undefined>)): void;
  onLoad(options: OnLoadOptions, callback: (args: OnLoadArgs) =>
    (OnLoadResult | null | undefined | Promise<OnLoadResult | null | undefined>)): void;
}
```
由类型定义可看出，`PluginBuild` 包含四个回调和一个配置项。最常用的是 `onResolve()` 和 `onLoad()`。
其参数具体类型，不在此做过多说明。

重点记录 `onResolve()` 和 `onLoad()` 。

### onResolve()
```typescript
// onResolve() 第一个参数的类型
// 必须有一个名为 filter 的正则参数
export interface OnResolveOptions {
  filter: RegExp;
  namespace?: string;
}
```
onResolve() 第二个参数是回调函数。

```typescript
// 回调函数的参数类型
export interface OnResolveArgs {
  path: string;         // 引用时写的标记
  importer: string;     // 哪个文件导入的这个文件
  namespace: string;    // 默认为file
  resolveDir: string;   // 
  kind: ImportKind;     // 导入类别有以下 7 种
  pluginData: any;      
}

// 导入类别
export type ImportKind =
  | 'entry-point'

  // JS
  | 'import-statement'
  | 'require-call'
  | 'dynamic-import'
  | 'require-resolve'

  // CSS
  | 'import-rule'
  | 'url-token'
```

回调函数返回的 `path` 应该是绝对路径。
一般可通过下列方法获取绝对路径：
```javascript
// args 为回调参数
path.resolve(args.resolveDir, args.path),
```

external 的使用。如果不打包，则返回 external: false

## 插件执行顺序


