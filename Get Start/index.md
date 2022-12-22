---
title: "Esbuild Get Start"
date: 2022-12-02T14:56:44+08:00
---

## Install esbuild

首先, 需要下载并安装 esbuild, 这里以 npm 为例

```shell
npm install esbuild
```

你也可以把 esbuild 安装在项目内, 就需要使用以下方式来确保 esbuild 可以正常使用

```shell
./node_modules/.bin/esbuild --verison
```

推荐使用 npm 安装 esbuild 的可执行文件, 如果你不想使用 npm, 这里有一些其他方式

## Your first bundle

这里有一个快速使用 esbuild 的案例. 首先, 安装`react`与`react-dom`包

```shell
npm install react react-dom
```

然后创建一个`app.jsx`文件, 内容如下:

```js
import * as React from "react";
import * as Server from "react-dom/server";

let Greet = () => <h1>Hello, world!</h1>;
console.log(Server.renderToString(<Greet />));
```

最后, 告诉 esbuild 让它来打包这个文件

```shell
./node_modules/.bin/esbuild app.jsx --bundle --outfile=out.js
```

这将创建一个名叫`out.js`的文件, 该文件包含了你的代码以及 react 的源码, 这些代码会包含在一个自执行函数中, 且不会依赖于 node_modules 中的文件, 通过`node out.js`命令后, 你可以看到如下内容:

```js
<h1 data-reactroot="">Hello, world!</h1>
```

<!-- 需要注意的是, esbuild在不需要任何配置的情况下, 也会把JSX语法转换成Javascript语法. 当esbuild拥有自定义配置项时, 它会自动尝试更为合理的值来保证进程的顺利运行,  -->

## Build scripts

你的打包命令通常会被重复使用, 通常会在`package.json`文件中添加一条`script`命令来快速使用 esbuild, 像这样:

```json
{
  "scripts": {
    "build": "esbuild app.jsx --bundle --outfile=out.js"
  }
}
```

这条执行语句可以通过以下命令执行:

```shell
npm run build
# 或者
yarn build
```

然而, 如果你需要传递更多配置项的时候, 使用命令行的方式会显得很笨重. 所以你可能更希望在一个 JavaScript 文件中, 通过 esbuild 的 API 来实现更为复杂的配置场景. 比如这样:

```javascript
require("esbuild")
  .build({
    entryPoints: ["app.jsx"],
    bundle: true,
    outfile: "out.js",
  })
  .catch(() => process.exit(1));
```

`build`函数将会在一个子进程中执行打包过程, 且返回一个 Promise 对象, 当打包进程完成后, 会标记为`resolved`状态. 需要注意的是, 上面的代码没有处理打包过程中的异常, 这是因为默认情况下进程中出现的错误, 都会被输出到控制台中.

esbuild 还提供了一个`buildSync`的同步 API, 但推荐使用异步方式来构建脚本, 因为大部分的插件仅支持异步的 API

## Bundling for the browser

默认情况下打包出来的脚本是用于在浏览器执行的, 所以你不需要任何额外的配置就可以使用它. 开发模式中, 你可能想要打开`source map`功能, 这需要添加`--sourcemap`参数; 生产模式中, 你可能想打开要功能, 这需要添加`--minify`参数. 如果你希望自定义打包后支持的目标环境, 这个功能会把一些新的 JavaScript 语法编译为旧的 JavaScript 语法, 比如像这样:

```shell
esbuild app.jsx --bundle --minify --sourcemap --target=chrome58,firefox57,safari11,edge16
```

某些 npm 的包可能不能浏览器中运行, 此时可以使用 esbuild 的配置项来保证打包的顺利. 对于未定义的全局变量, 也可以通过 esbuild 的 `define` 或者 `inject` 功能来替换它们

## Bundling for node

虽然在 nodeJS 中是不需要打包代码的, 但在某些时候, esbuild 依然是很有用的, 比如用来转义 Typescript, 或者把 ECMAScript 模块转为 CommonJS 模块, 或者把新的 JS 语法转为特定版本的旧语法, 在发布某些包之前使用 esbuild 也是很有用的,比如它可以压缩你的文件, 让脚本在被读取时花费更少的时间.

如果你想通过 esbuild 打包运行在 Node 环境中的脚本, 需要添加`--platform=node`参数, 这将让 esbuild 自动选择一些更为友好的配置项. 比如说, 如果你的源码中使用了`fs`模块中的某些方法, esbuild 不会把`fs`模块打包进最终的文件中.

```shell
esbuild app.js --bundle --platform=node --target=node10.4
```

如果你希望打包后的脚本中不想包含`node_modules`中的模块, 可以通过`external`参数来把他们标记为"外部的"模块, 这样可以有效减小打包后的脚本文件大小, 像这样:

```javascript
require("esbuild")
  .build({
    entryPoints: ["./demo1/main.js"],
    bundle: true,
    outfile: "out.js",
    external: ["./node_modules/*"],
  })
  .catch(() => process.exit(1));
```

这里提供一个`external`特性的例子:

```javascript
// demo1/main.js
import jquery from "jquery";

console.log(jquery);

function add() {
  console.log("add");
}

add();
```

打包后的文件内容如下所示:

```javascript
(() => {
  var __create = Object.create;
  var __defProp = Object.defineProperty;
  var __getOwnPropDesc = Object.getOwnPropertyDescriptor;
  var __getOwnPropNames = Object.getOwnPropertyNames;
  var __getProtoOf = Object.getPrototypeOf;
  var __hasOwnProp = Object.prototype.hasOwnProperty;
  var __require = /* @__PURE__ */ ((x) =>
    typeof require !== "undefined"
      ? require
      : typeof Proxy !== "undefined"
      ? new Proxy(x, {
          get: (a, b) => (typeof require !== "undefined" ? require : a)[b],
        })
      : x)(function (x) {
    if (typeof require !== "undefined") return require.apply(this, arguments);
    throw new Error('Dynamic require of "' + x + '" is not supported');
  });
  var __copyProps = (to, from, except, desc) => {
    if ((from && typeof from === "object") || typeof from === "function") {
      for (let key of __getOwnPropNames(from))
        if (!__hasOwnProp.call(to, key) && key !== except)
          __defProp(to, key, {
            get: () => from[key],
            enumerable:
              !(desc = __getOwnPropDesc(from, key)) || desc.enumerable,
          });
    }
    return to;
  };
  var __toESM = (mod, isNodeMode, target) => (
    (target = mod != null ? __create(__getProtoOf(mod)) : {}),
    __copyProps(
      isNodeMode || !mod || !mod.__esModule
        ? __defProp(target, "default", { value: mod, enumerable: true })
        : target,
      mod
    )
  );

  // demo1/main.js
  var import_jquery = __toESM(
    __require("./node_modules/jquery/dist/jquery.js")
  );
  console.log(import_jquery.default);
  function add() {
    console.log("add");
  }
  add();
})();
```

从文件中可以看出`jquery`的引用, 直接会从`./node_modules/jquery/dist/jquery.js`中取出, 这就要求在执行脚本前, 必须保证环境中已经安装过需要的模块
