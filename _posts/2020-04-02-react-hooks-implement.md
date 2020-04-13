---
layout: post
title: "从零实现React Hooks"
date: 2020-04-02 00:00:00
categories: frontend
comments: true
---

React Hooks解决了以前代码重用方法(Mixin, HOC, Render Props)的短板，目前已经被广泛推崇并应用，很多人刚开始看到React Hooks的写法感觉很酷炫，但他的实现原理很多人有点摸不着头脑。下面我们一起尝试实现React Hooks。

## 实现 useState

第一个就是从最基本的`useState`开始了, 为了方便验证，我们直接使用`console.log`打印出state

```javascript
function useState(initalValue) {
  let state = initalValue;
  const setState = (newValue) => (state = newValue);
  return [state, setState];
}

const [count, setCount] = useState(1);

console.log(count);
setCount(2);
console.log(count);
```

```bash
# console
> 1
> 1
```

state 没法变化, 因为方法返回值为值传递，内部的`state`改变将不会体现到之前外部的`count`值。

我们把state 改用方法返回，形成闭包，方法中将内部状态`_state`返回。

```javascript
function useState(initalValue) {
  let _state = initalValue;

  const state = () => _state;
  const setState = (newValue) => (_state = newValue);

  return [state, setState];
}

const [count, setCount] = useState(1);

console.log(count());
setCount(2);
console.log(count());
```

```bash
# console
> 1
> 2
```

这样打印出了`setCount`后的`state`了

## React & Component
我们将`useState`放入一个立即执行函数中，立即执行函数返回值为包含`setState`方法的对象，并命名为React。

```javascript
const React = (function () {
  function useState(initalValue) {
    let _state = initalValue;

    const state = () => _state;
    const setState = (newValue) => (_state = newValue);

    return [state, setState];
  }

  return { useState };
})();

const [count, setCount] = React.useState(1);

console.log(count());
setCount(2);
console.log(count());
```

以方法的方式写一个模拟组件, 由于还不支持DOM，我们在方法中返回一个包含`render`方法和`click`方法的对象，模拟可操作和渲染的React组件。

```javascript
function Component() {
  const [count, setCount] = React.useState(1);
  return {
    // 模拟render
    render: () => console.log(count),
    // 模拟点击事件
    click: () => setCount(count + 1),
  };
}
```

- 将`_state` 提出到useState外层，以保留之前的状态，在下次调用`useState(initalState)`时可以优先使用外层闭包的_state，避免每次调用`useState`都会被默认值覆盖。
- 实现`React.render`方法，方法接受一个参数为将要渲染的组件。内部将调用组件方法和方法返回对象的`render`方法。

```javascript
const React = (function () {
  // 将内部状态提取出到立即执行函数
  let _state;
  function useState(initalValue) {
    // 优先获取内部闭包的状态
    const state = _state || initalValue;
    const setState = (newValue) => (_state = newValue);

    return [state, setState];
  }

  // render 方法模拟将组件渲染出来
  function render(Component) {
    const c = Component();
    c.render();
    return c;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  return {
    render: () => console.log({ count }),
    click: () => setCount(count + 1),
  };
}

// 测试代码
let App = React.render(Component);
App.click();
App = React.render(Component);
```

```bash
# console
> {count: 1}
> {count: 2}
```
这样在模拟点击后再次render可以得到更新后的`state`


## 在组件内多次使用`useState`

一个组件中大都会管理着多个state, 所以在一次render过程中调用多次`useState`在所难免。

```javascript
function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState('Hello');

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  };
}

let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
# console
> {count: 1, text: "Hello"}
> {count: 2, text: 2}
> {count: "World", text: "World"}
```

由于内部只保存了一个`_state`变量，所以每个`setState`都会修改同一个状态变量。

### 使用数组存储hooks

我们用数组`hooks`替换原来的`state`, 并且使用`hookIndex`当指针标记当前需要使用第几个hook，每次useState之后会将`hookIndex`指针后移一位。

```javascript
const React = (function () {
  // 存入数组
  let hooks = [];
  let hookIndex = 0;

  function useState(initalValue) {
    const state = hooks[hookIndex] || initalValue;
    const setState = (newValue) => (hooks[hookIndex] = newValue);

    // 每次执行将游标后移
    hookIndex += 1;

    return [state, setState];
  }

  // ...
  return { useState, render };
})();

// ...
let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
# console
> {count: 1, text: "Hello"}
> {count: 2, text: "Hello"}
> {count: "World", text: "Hello"} # 我们期望的是 {count: 2, text: "world"}
```
这是由于我们的`hookIndex`没有复位到0的问题，运行过程如下

1. 第一次render之后hookIndex为2。
2. click方法调用后将hooks[2]设置为 1 + 1 = 2。
3. 第二次render中，第一个useState时发现hooks[2]已定义，所以依然返回hooks[2]中的值，在render后hookIndex值为4。
4. setText调用时将 "World"赋值到hooks[4]中
5. 第三次render中, 第一个useState时发现hooks[4]已定义为"World",所以此时`count`为"World"。


解决这个方法很简单，在每次`React.render`之前将hookIndex重置，保证组件render方法调用时hooksIndex始终从0开始。

```javascript
const React = (function () {
  // ...
  function render(Component) {
    // 重置
    hookIndex = 0;
    const c = Component();
    c.render();
    return c;
  }
  // ...
})();

function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState('Hello');

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  };
}

// ...
let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
> {count: 1, text: "Hello"}
> {count: 1, text: "Hello"}
> {count: 1, text: "Hello"}
```

因为每一次`render`之后`hookIndex`一直为2，不管是`type`还是`click`修改的都是hooks[2]中的值。

这里使用闭包将当前的指针位置保留。

```javascript
const React = (function () {
  // ...
  function useState(initalValue) {
    const state = hooks[hookIndex] || initalValue;
    // 闭包当前的指针
    const _hookIndex = hookIndex;

    const setState = (newValue) => {
      hooks[_hookIndex] = newValue;
    };

    hookIndex += 1;

    return [state, setState];
  }
  // ...
})();

// ...
let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
> {count: 1, text: "Hello"}
> {count: 2, text: "Hello"}
> {count: 2, text: "World"}
```

这样`useState`就支持多次调用了。上面的实现方式中 使用数组将hook进行缓存，限制了useState的使用顺序，也就是说必须要保证每次render中`useState`的调用顺序是不变的，这也就说明了为什么不能将`hook`放在条件循环等语句中。

## useEffect

组件中大都会有一些副作用，绑定解绑DOM事件，异步api调用等。hooks中的useEffect就提供了这样一个入口。

```javascript
function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState('Hello');

  React.useEffect(() => {
    console.log('msg in effect');
  }, []);
  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  };
}
```
我们在useEffect中打印字符串。


```javascript
function useEffect(cb, depArray) {
  let hasChange = true;
  // TODO 判断 depArray有改变
  if (hasChanged) cb();
}
```

`useEffect`接受第二个参数为一个数组，在`useEffect`中会判断第二个参数有没有改变，只有变化时才会执行第一个参数(回调方法)。

我们把第二个参数存入内部数组`hooks`, 每次调用时获取之前存入的依赖数组，新的数组进行比较，如果有不同时，调用回调。并且在最后将`hookIndex`后移。

```javascript
function useEffect(cb, depArray) {
  const oldDeps = hooks[hookIndex];
  let hasChange = true;
  // 如果oldDeps 不存在说明是首次调用 需要回调
  if (oldDeps) {
    // 比较dep是否改变
    hasChange = depArray.some((dep, i) => !Object.is(dep, oldDeps[i]));
  }
  if (hasChange) cb();
  // 将新的数组存入hooks
  hooks[hookIndex] = depArray;
  // 和setState一样也需要向后移动指针
  hookIndex++;
}
```

在 React 模块最后别忘了返回`useEffect`

```javascript
return { useState, render, useEffect };
```

当 Comopnent useEffect 的第二个参数为空数组时

```bash
> msg in effect
> {count: 1, text: "Hello"}
> {count: 2, text: "Hello"}
> {count: 2, text: "World"}
```

Comopnent 中填入 useEffect 的第二个参数为`count`

```javascript
const [count, setCount] = React.useState(1);
const [text, setText] = React.useState('Hello');
React.useEffect(() => {
  console.log('msg in effect');
}, [count]);

// ...
let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
> msg in effect
> {count: 1, text: "Hello"}
> msg in effect
> {count: 2, text: "Hello"}
> {count: 2, text: "World"}
```

Comopnent 中填入 useEffect 的第二个参数为`text`

```javascript
const [count, setCount] = React.useState(1);
const [text, setText] = React.useState('Hello');
React.useEffect(() => {
  console.log('msg in effect');
}, [text]);

// ...
let App = React.render(Component);
App.click();
App = React.render(Component);
App.type('World');
App = React.render(Component);
```

```bash
> msg in effect
> {count: 1, text: "Hello"}
> {count: 2, text: "Hello"}
> msg in effect
> {count: 2, text: "World"}
```

## DOM支持

我们将组建改写为返回一个`JSX.Element`，由于我们使用了`babel`和`@babel/react` preset, 所以JSX将被编译为`React.createElement`。

```javascript
function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState('Hello');
  React.useEffect(() => {
    console.log('msg in effect');
  }, [count]);
  // React.createElement('h1', {}, 'Hello World')
  return <h1>Hello World</h1>;
}

React.render(<Component />, document.getElementById('app'));
```

### React.createElement && React.create
因为JSX会被babel编译为`React.createElement`, 接下来我们需要实现`React.createElement` 和`React.render`两个方法。让Component可以渲染到DOM

首先我们需要实现 createElement, 接受三个参数
1. type: HTML元素标签字符串
2. props: HTML元素的属性,
3. 子元素。

然后我们返回一个对象包含字段, 返回的对象我们就叫它"virtual DOM"吧，哈。

- type: HTML元素标签字符串
- props: 属性字段
  - children: 子元素或者文本元素


我们还需要定义一下文本元素的"Virtual DOM"类型：
- type: 'TEXT_ELEMENT' 固定值，标识文本
- props: 属性字段
  - nodeValue: 文本字符串
  - children: 空数组

```javascript

function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map((child) => (typeof child === 'object' ? child : createTextElement(child))),
    },
  };
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: [],
    },
  };
}

```

### React.render

render与Dom相关的操作，我们单独创建一个文件, render.js

```javascript
// render.js

let _Component = null;
let _root = null;
let _hooks = null;

export const render = (hooks) => (Component = _Component, root = _root) => {
  if (JSON.stringify(hooks) === _hooks) {
    return; // 简单的判断是否需要重新渲染
  } else {
    _hooks = JSON.stringify(hooks);
  }

  // 去掉root中的已经渲染的内容
  while (root.firstChild) {
    root.removeChild(root.firstChild);
  }

  const Comp = reconcile(Component, root);
  _Component = Component;
  _root = root;
  const dom = createDom(Comp);

  // mount 新的dom
  root.appendChild(dom);
};

// 递归调用创建DOM
export function createDom(fiber) {
  const dom = fiber.type === 'TEXT_ELEMENT' ? document.createTextNode('') : document.createElement(fiber.type);
  const props = fiber.props || {};

  updateDom(dom, {}, props);

  if (props.children) {
    props.children.forEach((child) => {
      // 递归
      if (Array.isArray(child)) {
        child.forEach((c) => {
          dom.appendChild(createDom(c));
        });
      } else {
        dom.appendChild(createDom(child));
      }
    });
  }
  return dom;
}

const isEvent = (key) => key.startsWith('on');
const isProperty = (key) => key !== 'children' && !isEvent(key);
const isNew = (prev, next) => (key) => prev[key] !== next[key];
const isGone = (prev, next) => (key) => !(key in next);

function updateDom(dom, prevProps, nextProps) {
  // 去掉event listener
  Object.keys(prevProps)
    .filter(isEvent)
    .filter((key) => !(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach((name) => {
      const eventType = name.toLowerCase().substring(2);
      dom.removeEventListener(eventType, prevProps[name]);
    });
  // 移除之前不用的属性
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach((name) => {
      dom[name] = '';
    });
  // 添加新的属性 设置修改的属性
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach((name) => {
      dom[name] = nextProps[name];
    });
  // 添加event listener
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach((name) => {
      const eventType = name.toLowerCase().substring(2);
      dom.addEventListener(eventType, nextProps[name]);
    });
}

// 递归 调和
export function reconcile(Component, root) {
  const type = Component.type;
  if (Array.isArray(Component)) {
    return Component.map((child) => reconcile(child, root));
  }

  const Comp = typeof type === 'string' ? Component : type();

  if (Comp.props && Comp.props.children) {
    Comp.props.children.forEach((child, idx) => {
      if (typeof child.type !== 'string') {
        // 子组件 递归
        Comp.props.children[idx] = reconcile(Comp.props.children[idx], root);
      }
    });
  }
  return Comp;
}

```

在React的立即执行函数中将`render` 和`createElement`返回
```javascript
import { render } from './render';

const React = (function () {
  // ...
  return { useState, render: render(hooks), createElement, useEffect };
})()

// ...
```

这样我们的Hello World就可以渲染了。

![hello-world](/assets/posts/2020-04-02/hello-world.jpg)

我们向页面添加按钮并添加点击事件增加`count`

```javascript
function Component() {
  const [count, setCount] = React.useState(1);
  React.useEffect(() => {
    console.log('msg in effect');
  }, [count]);

  return (
    <div>
      <h2>{count}</h2>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </div>
  );
}
```
我们发现点击按钮后，页面并没有更新

![button-count-failed](/assets/posts/2020-04-02/button-count-failed.jpg)

我们需要一个事件循环去重新渲染页面，我们在React立即执行函数中加入方法，每400ms重新渲染一次。
```javascript

  function workLoop() {
    hookIndex = 0;
    render(hooks)();
    setTimeout(workLoop, 400);
  }
  setTimeout(workLoop, 400);

```
这样 点击按钮后 数字可以正常改变了

![button-count-failed](/assets/posts/2020-04-02/button-count-success.jpg)


hooks的强大也是因为我们可以自定义hooks并对代码进行更好的封装与重用。
所以我们开始写一个获取CNode论坛帖子标题的hook: `useCnodeList`
```javascript
const useCnodeList = (page) => {
  const [articles, setArticles] = React.useState([]);

  React.useEffect(() => {
    fetch(`https://cnodejs.org/api/v1/topics?page=${page}`)
      .then((resp) => resp.json())
      .then((json) => {
        setArticles(json.data);
      });
  }, [page]);

  return articles;
};

function Component() {
  const [count, setCount] = React.useState(1);
  const articles = useCnodeList(count);

  return (
    <div>
      <h2>CNode帖子： 第{count}页</h2>
      <button onClick={() => setCount(count + 1)}>下一页</button>

      <ul>
        {articles.map((article) => (
          <li key={article.id}>{article.title}</li>
        ))}
      </ul>
    </div>
  );
}

React.render(<Component />, document.getElementById('app'));

```

![custom-hooks](/assets/posts/2020-04-02/custom-hooks.jpg)


当点击按钮改变count值后，会获取对也页数的CNode论坛的数据。到此，React Hooks基本功能就都已经实现了 yeah!!

### ref links:
- [react-hooks-implement](https://github.com/JasonHeylon/react-hooks-implement)
- [jsconf.asia 2005 - react-hooks](https://www.swyx.io/speaking/react-hooks)
