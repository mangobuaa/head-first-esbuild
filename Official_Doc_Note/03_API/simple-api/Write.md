# Write

> 配置：`build`

`build()` 方法可以将打包后的文件直接写入磁盘，也可以返回内存中的文件。默认情况下，使用 JS API 是写入磁盘。
如果需要获取内存中的表示，需要 `write` 配置：

```typescript
import { build } from 'esbuild';

async function run() {
    const result = await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        write: false,
        outfile: 'out.js'
    });
    
    console.log('output files: ', result.outputFiles);
    for(let outputFile of result.outputFiles) {
        console.log(outputFile.path);   // 生成文件的路径
        console.log(outputFile.text);   // 文件的内容（文本）
        console.log(outputFile.contents);// 文件的内容（Uint8Array）
    }
}

run();
```

`outputFile.text` 是一个 `getter`，盲猜是由 `contents` 转化过来。

`Uint8Array` 与 `string` 的转换可用如下函数：

```typescript
function uint8Array2String(unit8Array: Uint8Array) {
    let str = '';
    for (let i = 0; i < unit8Array.length; i++) {
        str += String.fromCharCode(unit8Array[i]);
    }

    return str;
}
```


