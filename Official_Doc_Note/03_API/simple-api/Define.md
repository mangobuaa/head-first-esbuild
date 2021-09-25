# Define

`Define` 可以定义一个常量表达式，替换全局变量。通过这种方式，可以在不改变源代码的情况下，改变生成的代码。

官方示例：
```typescript
// 源代码
let js = 'DEBUG && require("hooks")'

// 将 Debug 替换为 true
require('esbuild').transformSync(js, {
  define: { DEBUG: 'true' },
})
// 结果
{
  code: 'require("hooks");\n',
  map: '',
  warnings: []
}

// 将 Debug 替换为 false
require('esbuild').transformSync(js, {
  define: { DEBUG: 'false' },
})
// 结果
{
  code: 'false;\n',
  map: '',
  warnings: []
}
```

自己编的示例：
目录结构如下：
```
/project_root
|___/scripts
    |___run.ts
|___/src
    |___app.ts
|___package.json
```
运行 esbuild 的脚本 `run.ts` 如下：
```typescript
// run.ts
import { build } from 'esbuild';
async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        outfile: 'out.js',
        define: { 
            __Text: "String"    // 定义全局变量 __Text 替换为 String
        }
    });
} 
run();
```
源代码 `app.ts` 如下：
```typescript
// app.ts
const obj = {
    ___Text(str: string) {
        return str;
    }
}

console.log(__Text("one"));     // 全局变量 __Text
console.log(obj.___Text("one"));// 非全局变量 __Text
```

运行脚本后，打包出的文件 `out.js` 如下：
```javascript
(() => {
  // src/app.ts
  var obj = {
    ___Text(str) {
      return str;
    }
  };
  console.log(String("one"));
  console.log(obj.___Text("one"));
})();
```
从 `out.js` 可以看出，只替换了全局变量的 `__Text`。


替换表达式需要是一个 JSON 对象（null, number, string, array, object）或者一个标识符。
除了 array 和 object，其他类型会简单替代；而 array 和 object 会使用一个标识符引用这个值，然后使用这个值去替换。
如果想替换成字符串，需要再加个引号。

下面例子展示了 array、object、字符串的使用。

代码结构与上面例子相同：

脚本代码 run.ts:
```typescript
// run.ts


import { build } from 'esbuild';

const globalObj = {
    getBuildDate:  new Date().toString()
}
const globalArray = ["one", "two"];

async function run() {
    await build({
        entryPoints: ['./src/app.ts'],
        bundle: true,
        outfile: 'out.js',
        define: {
            __Text: "String",
            author: '"Mango"',
            globalObj: JSON.stringify(globalObj),
            globalArray: JSON.stringify(globalArray)
        }
    });
}
run();
```

源代码：
```typescript
// app.ts
// author 是一个全局字符串，使用 esbuild 动态代替
console.log(author);    
// __Text 是一个全局标识符，使用 esbuild 动态代替
console.log(__Text("one"));

// globalObj 是一个全局 object，使用 esbuild 动态代替
const buildDate = globalObj.getBuildDate;
console.log(buildDate);

// globalArray 是一个全局 array，使用 esbuild 动态代替
globalArray.forEach(item => console.log(item));
```

生成的代码 `out.js` 如下：
```javascript
(() => {
  // <define:globalArray>
  var define_globalArray_default = ["one", "two"];

  // <define:globalObj>
  var getBuildDate = "Sat Sep 25 2021 18:47:17 GMT+0800 (\u4E2D\u56FD\u6807\u51C6\u65F6\u95F4)";
  var define_globalObj_default = { getBuildDate };

  // src/app.ts
  console.log("Mango");
  console.log(String("one"));
  var buildDate = define_globalObj_default.getBuildDate;
  console.log(buildDate);
  define_globalArray_default.forEach((item) => console.log(item));
})();
```

对于级联的对象替换有两种方式：以 `process.env.NODE_ENV` 为例：

第一种方式：直接使用字符串替换：
```typescript
define: {
    // 
    "process.env.NODE_ENV": '"production"'
}
```

第二种方式：使用对象替换的方式：
```typescript
const myProcess = {
    env: {
        NODE_ENV: "development"
    }
}

define: {
    process: JSON.stringify(myProcess)
}
```


