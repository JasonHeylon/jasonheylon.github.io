---
layout: post
title: "一次API封装的回顾 - 模仿ActiveRecord的API调用库"
date: 2020-03-01 00:00:00
categories: frontend
comments: true
---

本文为一次对API接口请求层的封装记录。

## 为什么要后台API进行封装

在当时的团队内，前端团队把项目根据业务类型进行了拆分，但后台接口为单一服务，所以每次添加新前端项目或者添加新功能时，都需要做一些与接口调用相关的重复工作。同时还有如下问题
- 项目A依赖接口apiA, 项目B也依赖接口apiA, 由于项目B的需求接口apiA做了修改(新增字段),项目A经常不能同步更新
- 对于接口返回的数据类型我们使用Typescript进行定义，这样在多个项目间会有相同的Model无法同步。
- 接口管理分散，每个项目单独定义API地址等，没法统一进行升级等。

## 对封装后的期望

- 使用Typescript，对后台接口数据进行统一类型定义。
- 后台接口为resutful规范，前端只要定义主目录，自动计算增删改查对应地址。类似：[Backbone Model urlRoot](https://backbonejs.org/#Model-urlRoot)、[Rails ResourcesRouting](https://guides.rubyonrails.org/v3.2/routing.html#resource-routing-the-rails-default)
- 请求返回Promise 支持Async-Await语法。

对于Rails出身的前端，还是对ActiveRecord类语法有些偏爱的，所以我们期望api的使用上可以实现如下的样子。

```javascript

import API from 'API'

const api = new API({
  baseUrl: 'https://www.example.com/api/',
  getToken: () => cookies.get('my-cookie-key'),
  // ...
})

// ...

const article = await api.articles.get(route.params.id)
return <Article article={article} />


// ...

const articles = await api.articles.query({ category: route.params.category })
                              .sort({ createdAt: -1 })
                              .page(1)
                              .perPage(15)
return <collection articles={articles} />
```

## 主要的实现逻辑

对于以上语法主，首先返回的类型为Promise。

1. fetchData接受url参数，返回promise like的对象。
2. 定义 params对象，为请求的queryString对象，会在方法返回的对象通过闭包方式访问并修改。
3. 通过`Promise.reslove().then()`向`microtasks queue`插入真实的API请求方法。真实的请求会使用params生成queryString。
4. 向上一步返回的Promise添加`page`方法，向形成闭包的作用域中的`params`对象设置`page`属性的值。

```javascript

function fetchData(url) {
  const params = {};

  const realRequest = () => {
    const searchParams = new URLSearchParams(params);
    const fullUrl = [url, '?', searchParams.toString()].join('');
    return fetch(fullUrl).then(resp => resp.json());
  }

  const promise = Promise.resolve().then(() => realRequest());

  promise.page = (page) => {
    params.page = page;
    return promise;
  };

  return promise;
}

fetchData('http://www.example.com').page(1).then(resp => console.log(resp));
```

这样可以发现，`fetchData` 方法的是一个Promise，可以直接通过`.then()` 或者`await`语法处理这个Promise被resolve的结果，也可以通过`.page()`方法设置异步请求的参数，只要我们往返回的promise上加入更多的方法和配置项就可以完成我们期望的样子了。

## 一些优化

直接对Promise对象进行修改并不是一个好方式，使用继承(原型链)或者说是装饰器。

首先创建Queryable类，包括构造方法，http的请求方法，以及一些设置请求参数的方法。

在构造函数中，创建一个Promise实例，并设置为原型，由于`class`语法本身就会创建并设置`Queryable.prototype`对象，将实例方法存入这个对象，所以需要将这个Promise实例放置在`Queryable.prototype.__proto__`中。


```javascript
// queryable.js

class Queryable {
  // 构造方法接受参数
  // client： 请求客户端，可以是axios的实例
  // path: resource的地址
  // method: 请求动词verb
  // params: 请求的queryString
  constructor(client, path, method, params = {}) {
    this.params = params;
    // ...

    const p: Promise = Promise.resolve().then(() => this.realRequest());
    Object.setPrototypeOf(Object.getPrototypeOf(this), p);
  }

  realRequest(){
    // 实际的请求
    return this.client(this.method, this.path, this.params);
    // ...
  }

  setParams(params) {
    this.parms = {...this.params, ...params};
    return this;
  }

  page(page) {
    return this.setParams({ page: page });
  }

  // ...
}

```


当调用`then`方法时，`receiver`并不是Promise对象本身，这会影响`this`的指向，运行环境也会抛出异常，我们在`Queryable`类定义一个`then`方法并显示绑定为`Queryable.prototype.__proto__`。

```javascript
// queryable.js

class Queryable {
  // ...
  then(...args: any): Promise<any> {
    const prototype = Object.getPrototypeOf(Object.getPrototypeOf(this));
    return prototype.then.call(prototype, ...args);
  }
  // ...
}

```

这样Queryable对象就基本完成了，之后在领域Model的父类(Base Model)中定义`query`方法，方法内创建一个`Queryable`实例，然后返回。

```javascript
// baseModel.js

class Base {
  // ...
  query (conditions) {
    return new Queryable(client, this.basePath, 'get', conditions);
  }
  // ...
}


// user.js

class User extends Base {
  static basePath = 'users'
  // ...
}
```

到此就基本完成了我们期望的API调用方式了。

```javascript
const maleUsers = await User.query(gender: 'male', age: 18).page(10);
const femaleUsers = await User.query(gender: 'female', age: 18);
```
