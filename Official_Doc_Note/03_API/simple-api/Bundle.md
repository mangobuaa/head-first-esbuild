# Bundle

bundle 意味着将导入的依赖都内置到1个文件。这个过程会递归完成。
默认情况下，不会打包，需要指定。

```javascript
require('esbuild').buildSync({
  entryPoints: ['in.js'],
  bundle: true,         // 需要指定
  outfile: 'out.js',
})
```

打包与文件串联不同。esbuild 可以打包出多个独立的文件。

只有静态定义的 import 才可以，比如使用字符常量。如果需要运行时确定的 import，esbuild 不会打包。

> 此时的静态导入和动态导入，可以简单认为导入语法中有变量，注意与异步导入区分。

```javascript
// 静态导入
import 'pkg';
import('pkg');
require('pkg');

// 动态导入
import(`pkg/${foo}`);
require(`pkg/${foo}`);
['pkg'].map(require);
```