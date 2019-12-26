---
layout: post
title: "Lazyman实现方法"
date: 2019-12-12 00:00:00
categories: frontend
comments: true
---

## ES 5

```javascript
function LazyMan (name) {

  function _lazyMan (name) {
    this.name = name;
    this.events = [];
    this.sleepFirstTime = 0;

    var that = this;

    this.sleepFirst = function (time) {
      that.events.unshift(function () {
        setTimeout(that.runNextEvent, time * 1000);
      });

      return that;
    };

    this.sleep = function (time) {
      that.events.push(function () {
        setTimeout(that.runNextEvent, time * 1000);
      });

      return that;
    };

    this.eat = function (food) {
      that.events.push(function () {
        console.log('eat ' + food);
        that.runNextEvent();
      });

      return that;
    };

    this.runNextEvent = function () {
      var event = that.events.shift();
      if (event) event();
    };

    this.events.push(function () {
      console.log('Hello this is ' + that.name);
      that.runNextEvent();
    })

    setTimeout(this.runNextEvent, 0);
  }

  return new _lazyMan(name);
}

LazyMan('Mike')
  .sleep(4)
  .eat('apple')
  .sleep(6)
  .eat('orange');

// LazyMan('Mike')
//   .sleepFirst(4)
//   .eat('apple')
//   .sleep(6)
//   .eat('orange');

```

## Promise

```javascript

```


## generator

```javascript
function LazyMan (name) {
  let events = [() => {console.log('This is ' + name)}]
  let runner = _lazyMan()

  function sleep (time){
    setTimeout(() => runner.next(), time * 1000)
  }

  function * _lazyMan() {
    for (const event of events) {
      typeof event === 'number' ? yield sleep(event) : event()
    }
  }

  Promise.resolve().then(() => runner.next())

  const man = {
    sleepFirst (time) {
      events.unshift(time)
      return man
    },

    sleep (time) {
      events.push(time)
      return man
    },

    eat(food) {
      events.push(() => console.log(`eat ${food}`))
      return man
    }
  }

  return man
}

LazyMan('Mike')
  .sleep(4)
  .eat('apple')
  .sleep(6)
  .eat('orange');

// LazyMan('Mike')
//   .sleepFirst(4)
//   .eat('apple')
//   .sleep(6)
//   .eat('orange');
```
