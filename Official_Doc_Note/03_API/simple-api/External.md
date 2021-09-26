# External

> 配置：`build`

`external` 标记哪个模块不被打包。当遇到导入该模块时，保持`import` 导入语法（iife 和 cjs 格式使用 `require`，`esm` 使用 `import`），在运行时导入。

