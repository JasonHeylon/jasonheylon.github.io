---
layout: post
title: "Advanced features of vue.js"
date: 2019-02-10 12:00:00
categories: frontend
tags: vue
comments: true
---


## Reactive

### getter and setter

```javascript
const obj = { foo: 123 }

convert(obj)

obj.foo // should log: 'getting key "foo": 123'

obj.foo = 234
obj.foo // should log: 'getting key "foo"::::: 234'


function convert (obj) {
  Object.keys(obj).forEach(key => {
    let internalValue = obj[key]
    Object.defineProperty(obj, key, {
      get () {
        console.log(`getting key "${key}": ${internalValue}`
        return internalValue
      },
      set (newValue) {
        internalValue = newValue
      }
    }
  }
}

```

### Dependency Tracking
