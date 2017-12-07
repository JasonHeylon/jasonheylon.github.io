---
layout: post
title:  "使用Rails + Vue + Vuex + jwt的用户登录"
date:   2017-12-01 12:00:00
categories: rails vue js
comments: true
---

由于现代的web应用越来越重前端，大部分都会使用前后端分离来实现，基本的用户登录的功能一定会遇到，最近在搭架子的我也遇到了这个问题，以下记录一下。

## 简单的Rails后台
首先新建一个Rails项目

添加gem
```ruby
# Gemfile
gem 'devise'
gem 'jwt'
gem 'active_model_serializers'
```

使用devise创建用户User

使用jwt gem创建用户的jwt token
```ruby
# app/utilities/jwt_token.rb
require 'jwt'

module JwtToken
  ALGORITHM = 'HS256'

  def self.issue(payload)
    JWT.encode(
      payload,
      auth_secret,
      ALGORITHM)
  end
  def self.decode(token)
    JWT.decode(token,
      auth_secret,
      true,
      { algorithm: ALGORITHM }).first
  end
  def self.auth_secret
    ENV["AUTH_SECRET"]
  end
end
```
修改user model
```ruby
# app/models/user.rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  def token
    JwtToken.issue(user_id: self.id)
  end

end
```

这里使用了active_model_serializer来输出json
```ruby
# app/serializers/user_session_serializer.rb
class UserSessionSerializer < ActiveModel::Serializer
  attributes :token, :username
end
```

写routes和controller

```ruby
# app/config/routes.rb
namespace :api, format: 'json' do
  namespace :v1 do
    devise_scope :user do
      post '/signin', to: 'sessions#create'
    end
  end
end
```

```ruby
# app/controllers/api/v1/sessions_controller.rb
class API::V1::SessionsController < ActionController::API
  def create
    @user = User.where(email: params[:email].downcase).first

    if @user && @user.valid_password?(params[:password])
      render json: @user, serializer: UserSessionSerializer
    else
      render json: { status: 'failed', msg: I18n.t('users.login_failed') }, status: 403
    end
  end
end
```
后台工作完成，这样 post /signin 就可以完成用户的验证了

## 前端vue
我们使用vue-cli 脚手架
```bash
npm i vue-cli -g

vue init webpack login-sample
```
安装必要的包
```bash
npm i --save vuex axios
```
在src下创建目录结构和文件
```ruby
├── App.vue
├── api             # 存放api请求, 以模块划分，比如session是登录模块
│   └── session.js
├── assets
│   └── styles
│       └── base.scss
├── components      # 存放组件
│   ├── Footer.vue
│   └── Header.vue
├── main.js
├── router
│   └── index.js
├── store           # vuex目录
│   ├── index.js    # 入口导出store
│   ├── modules     # 模块store
│   │   └── session.js
│   └── mutation_types.js # mutation type的一个字典
└── views           # 存放页面组件
    ├── Home.vue    # 首页
    └── SignIn.vue  # 登录页面
```

首先完成api。
```javascript
// src/api/session.js

import axios from 'axios'

const baseUrl = 'http://localhost:3000/api/v1'

const URLS = {
  SIGN_IN_API_URL: `${baseUrl}/signin`
}

export default {
  signInUser (signInData) {
    return axios.post(URLS.SIGN_IN_API_URL, signInData).then((response) => {
      return Promise.resolve(response.data)
    }).catch((error) => {
      return Promise.reject(error)
    })
  }
}
```
定义mutation types
```javascript
// src/store/mutation_types.js
export const SET_SIGNED_IN = 'SET_SIGNED_IN'
```

然后写session module
```javascript
// src/store/modules/session.js
import * as types from '../mutation_types'
import sessionApi from '../../api/session'

// inital state
// current_user: { username, token }
const state = {
  currentUser: {}
}

// getters
const getters = {
  currentUser: state => state.currentUser
}

// actions
const actions = {
  signIn ({ commit, state }, signInData) {
    // api sign in success commit
    sessionApi.signInUser(signInData).then((userInfo) => {
      if (userInfo['user']) {
        // 这里简单判断返回数据是否有含有user
        commit(types.SET_SIGNED_IN, userInfo['user'])
      }
    }).catch((errorMessage) => {
      // 失败处理
    })
  }
}

// mutations
const mutations = {
  [types.SET_SIGNED_IN] (state, userInfo) {
    // 把服务器返回的json存入state
    state.currentUser = userInfo
  }
}

export default {
  state,
  getters,
  actions,
  mutations
}

```
Vuex 的入口文件
```javascript
// src/store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

import session from './modules/session'

Vue.use(Vuex)

const store = new Vuex.Store({
  modules: {
    session // 目前只有session一个module
  }
})

export default store

```

程序入口main.js

```javascript
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'

Vue.config.productionTip = false

new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: { App }
})

```


写页面
```html
<!-- src/views/Home.vue -->
<template>
  <div>
    <!-- 把用户信息打印到页面 -->
    {{currentUser}}
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'Home',
  computed: {
    ...mapGetters({
      currentUser: 'currentUser'
    })
  },
  components: {

  }
}
</script>

<style lang="sass" scoped>
</style>

```

```html
<!-- src/views/SignIn.vue -->
<template>
  <div class="signin-form">
    <form action="" @submit="submitSignin">
      <input type="text" v-model="email" required />
      <input type="password" v-model="password" required />
      <input type="submit" value="登陆" />
    </form>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'signin',
  data () {
    return {
      email: '',
      password: ''
    }
  },
  computed: {
    ...mapGetters({
      currentUser: 'currentUser'
    })
  },
  watch: {
    // currentUser已经有token时，登陆成功跳转到首页
    currentUser (userInfo) {
      if (userInfo['token']) {
        this.$router.push('/')
      }
    }
  },
  methods: {
    submitSignin (e) {
      e.preventDefault()
      this.$store.dispatch('signIn', { email: this.email, password: this.password })
    }
  }
}
</script>

<style lang="sass" scoped>
</style>

```
