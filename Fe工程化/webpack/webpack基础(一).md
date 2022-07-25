### webpack 基础(一)

`webpack` 是一个静态模块化打包工具。它通过一个或多个入口为起点构建一个依赖图，将程序所用的资源打包成一个或多个静态 `bundle` 文件。

我们可以使用一个配置文件来灵活的配置 `webpack`。

```js
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "bundle.js",
    clean: true,
  },
  module: {},
  plugins: [],
};
```

`webpack`一共有五个核心的概念，如上所示。

- mode 模式：生产模式与开发模式
- entry 入口：指明打包的入口以及一些配置项
- output 出口：指明打包后的文件输出目录以及一些配置项
- module loader：默认`webpack`只能处理`js`和`json`文件，要处理其他类型的文件就需要 `loader` 转换器
- plugins 插件：增强`webpack`的能力
