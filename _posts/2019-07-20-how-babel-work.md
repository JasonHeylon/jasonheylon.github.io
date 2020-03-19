---
layout: post
title: "浅谈Babel"
date: 2019-07-20 00:00:00
categories: frontend
comments: true
---

## 什么是Babel

Babel的官网这样解释：

> Babel 是一个 JavaScript 编译器。

通常来说，编译器是一种将高级语言编译成机器码的程序，而Babel是将高版本的JavaScript编译成低版本的JavaScript，以得到更好的低版本兼容性。有了Babel，开发者就可以在开发时使用最新的语法特性，而(基本)不用考虑兼容性问题。

例如我们使用较新的语法定义了个方法，需要对老版本浏览器兼容
```javascript
// 源码:
const func = name => `hello ${name}`;

// Babel编译后输出:
var func = function func(name) {
  return "hello".concat(name);
}
```

这个例子中Babel做了以下处理
- const 被转换成了var
- 箭头方法 转换成了function关键字方法定义
- 模板字符串被定义了 字符串的concat方法

但是Babel是如何知道哪些语法需要转换的呢？

## 开始使用Babel
想要使用Babel需要安装Babel，Babel的主程序npm包名为: `@babel/core`

```bash
> mkdir sample
> npm init -y
> npm install @babel/core
```

创建`index.js`文件
```javascript
// index.js
const babel = require('@babel/core')

const babelOptions = {}
const sourceCode = "const func = name => `hello ${name}`;"

babel.transform(sourceCode, babelOptions, function (err, result) {
  // result: { code, map, ast }
  console.log(result.code)
})
```

@babel/core 的使用非常简单，通常我们使用transform方法对源码进行编译。
`babel#transform`方法接受三个参数分别是 源代码，编译参数 以及一个回调方法，回调方法中`result`参数中的`code`就是被编译后的代码，详细api可以查看[官方文档](https://babeljs.io/docs/en/babel-core#transform)

当我们在shell中运行这段代码中时会发现，result.code和源码一模一样。
```bash
> node index.js
=> const func = name => `hello ${name}`
```

## Babel Plugin

babel本身并不知道如何对源码进行编译，如果只babel编译几乎就等同于`(sourceCode) => sourceCode`。想要正确的把新语法编译到ES5语法就需要使用babel插件(Babel Plugin)了，基本上每一个ECMAScript新语法都会有一个对应的Babel插件。比如例子中代码，就包含了三个新的语法点
```javascript
const func = name => `hello ${name}`;
```

- [箭头方法(arrow function)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)(ES015)：需要使用插件[@babel/plugin-transform-arrow-functions](https://www.npmjs.com/package/@babel/plugin-transform-arrow-functions)
- [模板字符串(template literals)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)(ES015)：需要使用插件[@babel/plugin-transform-template-literals](https://www.npmjs.com/package/@babel/plugin-transform-template-literals)
- [块作用域常量(const statement)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)(ES015)：需要使用插件[@babel/plugin-transform-block-scoping](https://www.npmjs.com/package/@babel/plugin-transform-block-scoping)

我们把这三个插件安装并且配置到`transform` 方法的第二个参数中，就可以进行正确编译了。
```javascript
// index.js
const babel = require('@babel/core')

const babelOptions = {
  "@babel/plugin-transform-arrow-functions",
  "@babel/plugin-transform-block-scoping",
  "@babel/plugin-transform-template-literals"
}
const sourceCode = "const func = name => `hello ${name}`;"

babel.transform(sourceCode, babelOptions, function (err, result) {
  console.log(result.code)
})
```

运行结果:
```bash
> node index.js
=> var func = function (name) {
  return "hello ".concat(name);
};
```
此时就可以得到我们期望的ES5代码

## Babel Preset
我们想要使用ES2015来程序我们就需要把所有ES2015新语法的Babel插件安装进来，preset就是将一系列Babel插件打包在一起，我们可以使用[babel-preset-es2015](https://www.npmjs.com/package/babel-preset-es2015)后就可以肆无忌惮的使用ES2015提供的新语法，而且不用在考虑应该添加哪些Babel插件了。

babel-preset-es2015就包含一下插件
  - check-es2015-constants
  - transform-es2015-arrow-functions
  - transform-es2015-block-scoped-functions
  - transform-es2015-block-scoping
  - transform-es2015-classes
  - transform-es2015-computed-properties
  - transform-es2015-destructuring
  - transform-es2015-duplicate-keys
  - transform-es2015-for-of
  - transform-es2015-function-name
  - transform-es2015-literals
  - transform-es2015-modules-commonjs
  - transform-es2015-object-super
  - transform-es2015-parameters
  - transform-es2015-shorthand-properties
  - transform-es2015-spread
  - transform-es2015-sticky-regex
  - transform-es2015-template-literals
  - transform-es2015-typeof-symbol
  - transform-es2015-unicode-regex
  - transform-regenerator

### @babel/preset-env
Babel官网推荐使用[@babel/preset-env](https://www.npmjs.com/package/@babel/preset-env)，它会一直同步最新的ECMAScript版本，使用它就可以一直使用最新标准的语法进行开发了。
```bash
> npm install @babel/preset-env
```

```javascript
const babel = require('@babel/core')
// 这里我们只要使用preset-env, 就不用单独引入插件啦
const babelOptions = {
  presets: [
    "env"
  ]
}

const sourceCode = "const func = name => `hello ${name}`;"
babel.transform(sourceCode, babelOptions, function (err, result) {
  console.log(result.code)
})
```

当然在一个团队中也可以很容易的定义一个自己的preset，比如：
```javascript
module.exports = function() {
  return {
    plugins: [
      "@babel/plugin-transform-arrow-functions",
      "@babel/plugin-transform-block-scoping",
      "@babel/plugin-transform-template-literals"
    ]
  };
}
```

## Babel工作方式

在上面的例子中，源代码进入babel的`transform`方法后 babel会对源代码进行以下三步处理

[sourceCode] -> `parse` -> `transform` -> `generation` -> [compiledCode]

1. parse(解析)： 将JavaScript代码进行抽象化为一个抽象语法树: **AST**(Abstract Syntax Tree)。 [babel-parser(babylon)](https://github.com/babel/babel/tree/master/packages/babel-parser)
2. transform(转换)：将第一步生成的AST根据规则进行修改，得到修改后的AST。[babel-traverse](https://www.npmjs.com/package/babel-traverse)
3. generation(生成): 将第二步生成AST转回成JavaScript代码。[babel-generator](https://www.npmjs.com/package/babel-generator)

其中为了更好的理解AST可以使用[astexplorer](https://astexplorer.net/#/gist/5f425ac9eafc8c2610098c479af139c5/latest), 它可以实时展示JavaScript转换后等到的AST


### @babel/cli
上面的例子中都是在代码中进行编译，Babel也提供了cli工具：
```bash
> npm install @babel/cli
```
```javascript
// index.js
const func = name => `hello ${name}`
```

我们将编译完的代码输出到lib文件夹中, 并且指定preset:
```bash
> npx babel index.js --out-dir lib --presets=@babel/preset-env
```
```javascript
// lib/index.js
"use strict";

var func = function func(name) {
  return "hello ".concat(name);
};
```

参数可以使用babelrc
```javascript
// .bablrc.js
module.exports = function (api) {
  api.cache(true);

  const presets = [ '@babel/preset-env' ];
  const plugins = [  ];

  return {
    presets,
    plugins
  };
}
```


## babel-polyfill 和 core-js + @babel/plugin-transform-runtime

### 为什么需要polyfill

babel对新ECMAScript 的编译有一个问题就是像Set、Map、Promise、Symbol这种新特性，Balbel本事是无法进行向底版本转义。
```javascript
// lib/index.js
const s = new Set([1, 2, 3]);
```

在进行编译后得到：
```javascript
var s = new Set([1, 2, 3]);
```

这时就需要使用[polyfill](https://developer.mozilla.org/zh-CN/docs/Glossary/Polyfill)了, Babel7.4.0版本之后官方就是用`core-js` + `@babel/plugin-transform-runtime` 替代了原来的`@babel/polyfill`。

- [core-js](https://github.com/zloirock/core-js) 提供了新语法的polyfill
- [@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime) 提供一个沙盒环境，避免polyfill污染全局环境。

### 使用

在安装core-js(此处安装的是core-js@3)和@babel/plugin-transform-runtime后进行配置, 将`@babel/plugin-transform-runtime`加入到plugins数组中

> core-js@2 只支持全局变量的调用, 如`Promise`，core-js@3 支持实例属性的调用，如`[].includes`

```javascript
// .babelrc.js
module.exports = function (api) {
  api.cache(true);

  const presets = [ '@babel/preset-env' ];
  const plugins = [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": '3' // 指定core-js 版本,
      }
    ]
  ];

  return {
    presets,
    plugins
  };
}
```

编译后代码

```javascript
"use strict";

var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault");

var _set = _interopRequireDefault(require("@babel/runtime-corejs3/core-js-stable/set"));

var s = new _set["default"]([1, 2, 3])
```


## 如何开发一个 Babel 插件

babel插件模板：

```javascript
module.exports = function({ types: t }) {
  return {
    visitor: {
      Identifier(path) {

      },
      // ...
    }
  }
}
```
其中方法的参数types：[Babel Types](https://babeljs.io/docs/en/6.26.3/babel-types)

plugin的本质就是在visitor中对整个代码的AST进行调整。

### 例子

比如我们对于字符串变量进行更改变量名为 原变量名+"Str"后缀。
```javascript
// 源码
const name = 'Babel'
console.log('Hello', name);

// 预期编译后为
const nameStr = 'Babel'
console.log('Hello', nameStr);
```

首先完成plugin文件`strVariable.js`， 方案为：
1. 通过使用 VariableDeclarator 获取变量声明节点
2. 判断定义变量的初始化值为字符串
3. 将变量名追加"Str"后缀

```javascript
// ./src/strVariable.js
module.exports = function ({ type: t}) {
  return {
    visitor: {
      VariableDeclarator(path) {
        if (path.node.init.type === 'StringLiteral') {
          path.node.id.name = path.node.id.name + "Str"
        }
      }
    }
  }
}

```

配置使用plugin
```javascript
// ./src/.babelrc.js
const strVariable = require('./strVariable');
module.exports = function (api) {
  const presets = [
  ];
  const plugins = [
    strVariable,
  ];
  return {
    presets,
    plugins
  };
}

```
编译后的输出如下：

```javascript
const textStr = "Hello Babel";

function main() {
  console.log(text);
}
```

发现在变量声明时的变量名改动成功了，但是后面调用时的名称也需要修改，这时就需要scope.rename重命名了, 可以查询[Babel-handbook Scope](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#scope)

我们将`path.node.id.name = path.node.id.name + "Str"` 改为`path.scope.rename(path.node.id.name, path.node.id.name + 'Str')`

再次编译 => ok！

### 例子2

比如将正则表达式变量定义提升到文件最上面的Babel Plugin
```javascript
module.exports = function({ types: t }) {
  return {
    visitor: {
      RegExpLiteral(path) {
        const variableName = path.parent.id.name;
        const newIdentifier = path.scope.generateUidIdentifier(variableName);

        const variableDeclaration = t.variableDeclaration("const",
          [
            t.variableDeclarator(newIdentifier, path.node)
          ]
        );

        const program = path.findParent(parent => t.isProgram(parent));
        path.scope.rename(variableName, newIdentifier.name);
        program.node.body.unshift(variableDeclaration);
        path.parentPath.remove();
      }
    }
  };
};
```

源码
```javascript
getVersion('3.2.6')
function getVersion(versionString) {
  const versionRegex = /(\d+)\.(\d+).(\d+)/;
  const [, major, minor, path] = versionRegex.exec(versionString);

  return { major, minor, path };
}
```

编译后
```javascript
const _versionRegex = /(\d+)\.(\d+).(\d+)/;
getVersion('3.2.6');

function getVersion(versionString) {
  const _versionRegex = /(\d+)\.(\d+).(\d+)/;

  const [, major, minor, path] = _versionRegex.exec(versionString);

  return {
    major,
    minor,
    path
  };
}
```

[Babel Types](https://babeljs.io/docs/en/6.26.3/babel-types)


相关连接s:

- [JAVASCRIPT AST VISUALIZER](https://resources.jointjs.com/demos/javascript-ast) 一个做可视化插件的公司做的AST可视化演示。
- [ast explorer](https://astexplorer.net/) 实时AST编译器，包含eslint、babylon等解析器
- [babel handbook](https://github.com/jamiebuilds/babel-handbook) babel手册 有中文版
- [EmberConf 2016: How to Build a Compiler by James Kyle](https://www.youtube.com/watch?v=Tar4WgAfMr4) James Kyle 在EmberConf2016的演讲 如何构建一个编译器
- [ESLint: Working with Plugins](https://eslint.org/docs/developer-guide/working-with-plugins) eslint的插件开发
