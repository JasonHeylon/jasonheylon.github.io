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

```javascript

const dep = new Dep()

autorun(() => {
  dep.depend()
  console.log('updated')
})

// should log: "updated"

dep.notify()
// should log: "updated"

// --------

window.Dep = class Dep {
  constructor () {
    this.subscribers = new Set()
  }

  depend () {
    if (activeUpdate) {
      this.subscribers.add(activeUpdate)
    }
  }

  notify () {
    this.subscribers.forEach(sub => sub())
  }
}

// JS one thread execution
let activeUpdate

function autorun (update) {
  function wrappedUpdate () {
    activeUpdate = wrappedUpdate
    update()
    activeUpdate = null
  }

  wrappedUpdate()
}

```


### mini Observer

```javascript
const state = {
  count: 0
}

observe(state)

autorun(() => {
  console.log(state.count)
})
// should log "count is: 0"

state.count++
// should log "count is: 1"

```

## Plugins

```javascript
Vue.use(plugin)

function (Vue, options) {
  // ... plugin code
}


Vue.mixin(options)
// apply to all Vue component after mixin

$options // merged options


```


### simple plugin

```javascript
  const vm = new Vue({
    data: { foo: 10 },
    rules: {
      foo: {
        value: value => value > 1,
        message: 'foo must be greater than one'
      }
    }
  })

  vm.foo = 0 // should log "foo must be greater than one"

  // ------

  const RulesPlugin = {
    install (Vue) {
      Vue.mixin({
        created () {
          if (this.$options.rules){
            Object.keys(this.$options.rules).forEach(key => {
              const rule = this.$options.rules[key]

              this.$watch(key, (newValue) => {
                const result = rule.validate(newValue)
                if (!result) {
                  console.log(rule.message)
                }
              })
            })
          }
        }
      })
    }
  }

  Vue.use(RulesPlugin)

```


## Render Function

Initial Render

Template
-> (compiled into) Render Function
-> (returns) Virtual DOM
-> (generates) Actual DOM


Subsequent Updates

Render Function
-> (returns) New Virtual DOM
-> (diffed against Old Virtual DOM) DOM Updates
-> (applied to) Actual DOM


* Actual DOM

```javascript
document.createELement('Div')
// "[object HTMLDivElement]"
```
> Browser Native Object (expensive)

* Virtual DOM

```javascript
vm.$createElement('div')
// { tag: 'div', data: {.attrs: {}, ... }, children: [] }

```

> Plain JavaScript Object (cheap)


Render Function API

```javascript

import MyComponent from '...'

export default {
  render (h) {
    return h('div', {}, [...])
    // h (myComponent, { props: {...} })
  }
}

```

### Dynamically Render Tags

```html
<div id="app">
  <example :tags=['h1', 'h2', 'h3']></example>
</div>
```

```javascript
Vue.component('example', {
  props: ['tags'],
  render (h) {
    const children = this.tags.map((tag, idx) => h(tag, idx))
    return h('div', { attrs: { id: 'hello' } }, children)
  }
})

new Vue({el: '#app'})

```

### About the Functional Components

* In Vue implementation, functional components are expanded eagerly
* Functional components's `render` function will be invoked inside parent components's render function.
* It's kind of expended in line which makes it extremely cheap, because there's no overhead of creating an instance.
* Perfect use case is performance optimizations when have a log of leaf nodes(eg: a huge list).

```javascript
// transform to functional component

Vue.component('example', {
  props: ['tags'],
  functional: true,
  render (h, context) {
    // context.props: this.props in normal vue components
    // context.slots(): this.$slots in normal vue components
    // context.children: raw children
    // context.parent: root component, it's the div[id=app] component in this current codes.

    const children = context.props.tags.map((tag, idx) => h(tag, idx))
    return h('div', { attrs: { id: 'hello' } }, children)
  }
})

new Vue({el: '#app'})

```


### Dynamically Render Components

``` html
<div id="app">
  <example :ok="ok"></example>
  <button @click="ok = !ok">toggle</button>
</div>
```

```javascript
const Foo = {
  render: h => h('div', 'foo')
}
const Bar = {
  render: h => h('div', 'bar')
}

Vue.component('example', {
  props: ['ok'],
  render (h) {
    // return h(Foo)
    return this.ok ? h(Foo) : h(Bar)
  }
})

new Vue({
  el: '#app',
  data: { ok: true }
})

```

### Higher-Order Components

```html

<div id="app">
  <smart-avatar username='vuejs' id='hello'></smart-avatar>
</div>

```

```javascript

// mock API
function fetchURL (username, cb) {
  setTimeout(() => {
    cb(`https://avatars.some-site.com/avatars/${username}.jpg`)
  }, 500)
}

const Avatar = {
  props: ['src'],
  template: '<img :src="src">'
}

function withAvatarURL (InnerComponent) {
  // we also can do some optimizations here, eg: create an object to cache avatar urls.
  return {
    props: ['username'],
    data () {
      return {
        url: 'https://placeholder.avatar.com/avatar.jpg'
      }
    },
    created () {
      fetchURL(this.username, url => {
        this.url = url
      })
    },
    render (h) {
      return h(InnerComponent,  {
        props: {
          src: this.url,
          attrs: this.$attrs // pass other attributes to inner component
        }
      }
    }
  }
}

const SmartAvatar = withAvatarURL(Avatar)

new Vue({
  el: '#app',
  components: { SmartAvatar }
})

```


## State management
### Passing Props
```html
<div id=“app”>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <button @click=“count++”>increment</button>
</div>
```

```javascript
new Vue({
  el: “#app”,
  data: { counter: 0 }
  components: {
    Counter: {
      props: [‘count’],
      template: ‘<div>{{ count }}</div>’
    }
  }
})
```

### Shared Object

> Why data need to be a function?
> Most of the time, we want each component instance to have its own separate unique piece of data, instead of all of these components sharing the same piece of data.

```html
<div id=“app”>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <button @click=“inc”>increment</button>
</div>
```


```javascript
const state = {
  count: 0
}

const Counter = {
  // render: h => h(‘div’, state.count)
  data () {
    return state
  },
  render (h) {
    return h(‘div’, this.count)
  }
}

new Vue({
  el: “#app”.
  components: { Counter },
  methods: {
    inc () {
      state.count++
    }
  }
})
```

### Shared Instance

```html
<div id=“app”>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <button @click=“inc”>increment</button>
</div>
```

```javascript
const state = new Vue({
  data: {
    count: 0
  },
  methods: {
    inc () {
      this.count++
    }
  }
})

const Counter = {
  render: h => h(‘div’, state.count)
}

new Vue({
  el: “#app”.
  components: { Counter },
  methods: {
    inc () {
      state.inc()
    }
  }
})
```

In this situation, we can make the state and alteration together in one object. In real world, the logic here maybe more complex than that only a increment of a count variable.

### Mutation
```html
<div id=“app”>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <counter :count=“count”></counter>
  <button @click=“inc”>increment</button>
</div>
```

```javascript
function createStore ({ state, mutations }) {
  return new Vue({
    data: { state },
    methods: {
      commit (mutationType) {
        mutations[mutationType](this.state)
      }
    }
  })
}

const store = createStore ({
  state: { count: 0 },
  mutations: {
    inc (state) {
      state.count++
    }
  }
})

const Counter = {
  render (h) {
    return h(‘div’, store.state.count)
  }
}

new Vue({
  el: ‘#app’,
  components: { Counter },
  methods: {
    inc () {
      store.commit(‘inc’)
    }
  }
})

```


### Functional
```html
<div id=“app”></div>
```


```javascript
function app ({ el, model, view, actions }) {
  const wrappedActions = {}

  Object.keys(actions).forEach(key => {
    const originalAction = actions[key]
    wrappedActions[key] = () => {
      const nextModel = originalAction(vm.model)
      vm.model = nextModel
    }
  })

  const vm = new Vue({
    el,
    data: {
      model
    },
    render (h) {
      return view(h, this.model, wrappedActions)
    }
  })
}

app({
  el: ‘#app’,
  model: {
    count: 0
  },
  actions: {
    inc: ({ count }) => ({ count: count + 1 }),
    dec: ({ count }) => ({ count: count - 1 })
  },
  view: (h, model, actions) => h(‘div’, { attrs: { id: ‘app’}}, [
    model.count, ‘ ’,
    h(‘button’, { on: { click: actions.inc }}, ‘+’),
    h(‘button’, { on: { click: actions.dec }}, ‘-’),
  ])
})

```


### Basic Hash Router
```html
<div>
  <component :is=“url”></component>
  <a href=“#foo”>foo</a>
  <a href=“#bar”>bar</a>
</div>
```


```javascript
// window.location.hash
//window.addEventListener(‘hashchange’, () => {
//})


window.addEventListener(‘hashchange’, () => {
  app.url = window.location.hash.slice(1)
})

const app = new Vue({
  el: ‘app’,
  data: {
    url: window.location.hash.slick(1)
  },
  components: {
    foo: { template: ‘<div>foo</div>’ },
    bar: { template: ‘<div>bar</div>’ }
  }
})

```


### Route Table
```html
<div>
  <a href=“#foo”>foo</a>
  <a href=“#bar”>bar</a>
</div>
```

```javascript
const Foo = { template: ‘<div>foo</div>’ }
const Bar = { template: ‘<div>bar</div>’ }
const NotFound = { template: ‘<div>not found!</div>’ }

const routeTable = {
  ‘foo’: Foo,
  ‘bar’: Bar
}

window.addEventListener(‘hashchange’, () => {
  app.url = window.location.hash.slice(1)
})

const app = new Vue({
  el: ‘#app’,
  data: {
    url: window.location.hash.slice(1)
  },
  render (h) {
    return h(‘div’, [
      h(routeTable[this.url] || NotFound),
      h(‘a’, { attrs: { href: ‘#foo’ }}, ‘foo’),
      ‘ | ’,
      h(‘a’, { attrs: { href: ‘#bar’ }}, ‘bar’)
    ])
  }
})

```


### Path to Regular Expressions And Dynamic Route

route:  `user/:username`
url: `user/someone`
params: `{ username: ‘someone’ }`

ref: [Path-To-Regexp](https://github.com/pillarjs/path-to-regexp)

```html

<div id=“app”></div>
```

```javascript
// some thing we need here
const keys = []
const regex = pathToRegexp(‘foo/:bar’, keys)

const result = regex.exec(‘/foo/123‘)
// result [‘/foo/123’, ‘123’]
// [{ name: ‘bar’ }]
// to { bar: ‘123’ }


// ——————
const Foo = {
  props: [‘id’],
  template: ‘<div>foo with id: {{ id }}</div>’
}
const Bar = { template: ‘<div>bar</div>’ }
const NotFound = { template: ‘<div>not found!</div>’ }

const routeTable = {
  ‘/foo/:id’: Foo,
  ‘/bar’: Bar
}

const compiledRoutes = []

Object.keys(routeTable).forEach(path => {
  const dynamicSegments = []
  const regex = pathToRegexp(path, dynamicSegments)
  const component = routeTable[path]

  compiledRoutes.push({
    component,
    regex,
    dynamicSegments
  })
})

window.addEventListener(‘hashchange’, () => {
  app.url = window.location.hash.slice(1)
})

const app = new Vue({
  el: ‘#app’,
  data: {
    url: window.location.hash.slice(1)
  },
  render (h) {
    const path = ‘/’ + this.url
    let componentToRender = NotFoud
    let props = {}

    compiledRoutes.some(route => {
      const match = route.regex.exec(path)
      if (match) {
        componentToRender = route.component

        dynamicSegments.forEach((segment, index) => {
          props[segment.name] = match[index + 1]
        })
      }
    })

    return h(‘div’, [
      h(componentToRender, { props }),
      h(‘a’, { attrs: { href: ‘#foo/123’ }}, ‘foo 123’),
      ‘ | ’,
      h(‘a’, { attrs: { href: ‘#foo/234’ }}, ‘foo 234’),
      ‘ | ’,
      h(‘a’, { attrs: { href: ‘#bar’ }}, ‘bar’)
    ])
  }
})

```


## Form Validation
- Markup-based ([vee-validate](http://vee-validate.logaretm.com))
- Model-based ([vuelidate](https://github.com/vuelidate/vuelidate))

### Create a Validation Library

```html
<div id=“app”>
  <form @submit=“validagte”>
    <input v-model=“text”>
  <br>
  <input v-model=“email”>

  <ul v-if=“!$v.valid” style=“color: red”>
    <li v-for=“error in $v.errors”>
      {{ error }}
    </li>
  </ul>

  <input type=“submit” :disabled=“!$v.valid”>
  </form >
</div>
```

```javascript
const validationPlugin = {
  install (Vue) {
    Vue.mixin({
      computed () {
        $v () {
          const rules = this.$options.validations
          let valid = true
          const errors = []

          Object.keys(rules).forEach(key => {
            const rule = rules[key]
            const value = this[key]
            const result = rule.validate(value)
            if (!result) {
              valid = false
              errors.push(rule.message(key, value))
            }
          }
        }
      }
    })
  }
}

Vue.use(validationPlugin)

const emailRE = //

new Vue({
  el: ‘#app’,
  data: {
    text: ‘foo’,
    email: ‘’
  },
  validations: {
    text: {
      validate: value => value.length >= 5,
      message: (key, value) => `${key} should have a min length of 5`
    },
    email: {
      validate: value => emailRE.test(value),
      message: key => `${key} must be a valid email`
    }
  },
  methods: {
    validate (e) {
      if (!this.$v.valid) {
        e.preventDefault()
        alert(‘not valid!’)
      }
    }
  }
})

```


## I18n
### Internationalization Approaches


```html
<div id=“app”>
  <h1>{{ $t(‘welcome-message’) }}</h1>

  <button>English @click=“changeLang(‘en’)”</button>
  <button>中文 @click=“changeLang(‘zh’)”</button>
  <button>Dutch @click=“changeLang(‘nl’)”</button>
</div>
```

```javascript
const i18nPlugin = {
  install (Vue, locaels) {
    Vue.proptype.$t = function (id) {
      return locales[this.$root.lang]
    }
  }
}

Vue.use(i18nPlugin, {
  en: { ‘welcome-message’: ’hello’ },
  zh: { ‘welcome-message’: ’你好’ }
  nl: { ‘welcome-message’: ’Hallo’ }
})

new Vue({
  el: ‘#app’,
  data: {
    lang: ‘en’
  },
  methods: {
    changeLang (lang) {
      this.lang = lang
    }
  }

})

```
