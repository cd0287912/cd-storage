### 什么是 Babel?

`babel`是一个工具链，主要用于将采用`ECMAScript 2015+`语法编写的代码转换为向后兼容的 `JavaScript` 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。 `babel`的工作是：

- 语法转换
- 通过`Polyfill`的方式在目标环境中添加缺失的特性
- 源码转换

### 基本使用

我们安装 babel 的三个库，才能实现基本的代码转换:

- @babel/core babel 的核心库
- @babel/cli babel 的命令行工具
- @babel/preset-env 预设代码 es6 转为 es5 的规则

定义配置文件:

```javascript
module.exports = {
  presets: ["@babel/preset-env"],
};
```

源代码:

```javascript
const add = (x, y) => {
  return x + y;
};
```

命令行输入:

```javascript
npx babel main.js -o main.build.js
// 生成 main.build.js
"use strict";
var add = function add(x, y) {
  return x + y;
};
```

我们通过`babel`把`es6`的箭头函数转换为普通函数，以上这就是`babel`的基本使用方式。

### 预设与插件

`babel`是如何转换的？或者说`babel`转换规则是什么？通过预设`presets`与插件`plugins`
我们可以把预设理解为一组插件的集合，我们只需要配置一个简单的预设就能达到配置多个插件的目的
常见的预设有：

#### @babel/preset-env

#### @babel/preset-react

#### @babel/preset-typescript

不同的预设来处理不同的文件类型或者代码

### 目标环境

默认`babel`通过预设`@babel/preset-env`把代码编译为`ES5`的版本。浏览器的版本不同所支持的语法也不同，我们想把代码编译为特定版本的浏览器环境下，我们该怎么做？
第一种，我们可以配置`@babel/preset-env`的`targets`参数

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 60,
        },
      },
    ],
  ],
};
```

由配置可以知道，`presets`是一个数组，预设是其中一个个元素。这个元素可以是一个字符串也可以是一个数组，我们想对预设传递更多的配置信息我们就要指定元素是数组，并且该元素数组的第二项才是配置，如上代码所示。
我们在`targets`的对象中指定要适配目标浏览器的版本。

第二种，我们可以配置`package.json`的`browserslist`，它是一个数组

```json
"browserslist": [
  "chrome 60"
]
```

我们更推荐第二种方式，因为可以搭配其他的插件，如`Autoprefixer`来处理`css`
注意：如果浏览器的版本实现了一些`ES6`的语法，那么`babel`就不会对其进行更低版本的代码转换。
我们来举个例子：

```javascript
const getInfo = () => {};
new Promise();
```

我们对`main.js`来进行`babel`编译一下

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 50,
        },
      },
    ],
  ],
};
```

我们使用如上配置，结果如下：

```javascript
"use strict";

const getInfo = () => {};

new Promise();
```

我们查阅相关资料，发现`chrome 50`是支持了箭头函数的，也是支持`Promise`，所以`babel`并没有对其处理。
我们在把`chrome`版本改低点：

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 18,
        },
      },
    ],
  ],
};
```

结果：

```javascript
"use strict";

var getInfo = function getInfo() {};

new Promise();
```

我们查阅相关资料，发现`chrome 18`是不支持了箭头函数的，也不支持`Promise`，但是`babel`只处理了箭头函数，并没有处理`Promise`。那这是什么情况？

### Polyfill

经过我们查阅后发现，`babel`默认只会转换新的语法，不会转换新的`api`。
`ES6`新增的语法包括箭头函数，`async`函数，`class`语法，解构语法等等。
`ES6`新增的`api`包括`Promise`，`Map`，`Symbol`， `Proxy`，`Iterator`和一些实例上的方法如数组的`find`等等。
那咋办嘛？引入`Polyfill`垫片的概念，目的是磨平浏览器之间的版本实现差异。
那既然一些老的浏览没有这些`api`，那`babel`就引用`core-js`为目标浏览器提供实现，这样老的浏览器就能认识这些`api`了。
我们使用`Polyfill`需要配合`@babel/preset-env`来实现，我们有两种实现的方式：

#### entry 引入

配置文件：

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 18,
        },
        useBuiltIns: "entry",
        corejs: 3,
      },
    ],
  ],
};
```

在我们代码入口文件中引入：

```javascript
// 引入这两个包，无需下载
import "core-js/stable";
import "regenerator-runtime/runtime";
[].find(() => {});
new Promise();
```

我们需要在配置文件中新增`useBuiltIns`属性，值为`entry`。同时指定`corejs`的版本。

#### usage 引入

配置文件：

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 18,
        },
        useBuiltIns: "usage",
        corejs: 3,
      },
    ],
  ],
};
```

代码入口：

```javascript
[].find(() => {});
new Promise();
```

我们发现，使用`usage`的方式时，去掉了代码入口的包引入，`babel`会自动引入相关`core-js`代码。
那么这两种方式有什么区别？

1. `entry`方式只会考虑到目标浏览器缺失的`api`进行引入，不会考虑实际代码用到的`api`进行针对性引入。
1. `usage`方式会考虑到目标浏览器缺失的`api`，以及实际代码用到的`api`进行针对性引入。
1. `entry`方式需要在代码入口手动引入相关的`polyfill`，而`usage`不需要。

### modules

我们通过`@babel/preset-env`这个预设，学习到了如何配置目标环境，了解了`polyfill`，这里还有个`modules`需要了解一下，其他属性，可以查阅官网。
常见的模块化语法有两种，`es6 module`的`import export`和`commonjs`的`require module.exports`。
我们对`modules`的值不设置，或者值为`auto`的时候，会发现转码前的代码里`import`都被转为`require`了。若设置为`false`，那么就不对`es6`模块进行更改，还是使用`import`引入模块。
使用`es6`模块化的好处就是在使用`webpack`一类的打包工具时，可以进行静态分析，可以做`tree shaking`等等优化的措施。

### runtime

到目前为止，我们会发现`babel`在进行转换的时候会采取几种方法：

1. 一些特殊的语法，`babel`会注入一些辅助函数
1. 不认识的`api`时会引入`core-js`里的`@babel/runtime`实现来抹平浏览器的差异

那么会出现什么问题呢？
第一种会增加文件的体积大小，第二种会污染全局(涉及到了原型对象的重写之类)
我们需要安装`@babel/plugin-transform-runtime`，来解决这些问题。

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: 18,
        },
        useBuiltIns: "usage",
        corejs: 3,
      },
    ],
  ],
  plugins: [
    [
      "@babel/plugin-transform-runtime",
      {
        corejs: 3,
      },
    ],
  ],
};
```

这样通过辅助函数就会统一引用`@babel/runtime`里面定义的函数，解决了问题 1。
然后定义`@babel/plugin-transform-runtime`里面`corejs`的版本，把原本引入的`@babel/runtime`和`core-js`更换为`@babel/runtime-corejs3`，完成了对`api`的实现进行转换，这样就不会污染全局了，解决了问题 2。

这个包依赖了`@babe/runtime`，辅助函数都定义在这个包里面。
使用插件`@babel/plugin-transform-runtime` 时，我们还可以定义其配置项。具体可以查阅[官网](https://babeljs.io/docs/en/babel-plugin-transform-runtime)。
