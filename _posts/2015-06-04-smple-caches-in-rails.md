---
layout: post
title:  "Rails中的常用cache"
date:   2015-06-04 12:00:00
categories: ruby rails cache
comments: true
---

## ETag

ETag是浏览器和服务器验证页面是否修改的工具。

一般流程是： 浏览器发起请求，rails渲染界面，生成Etag，并放入headers['ETag']，浏览器接到header并把ETag值存下，下次请求相同页面url时会在请求header中把之前存下来的值放入头部headers['If-None-Match']中，Rails渲染页面，生成ETag比对浏览器传入的headers['If-None-Match']是否与新生成的ETag相同，如果相同直接返回状态码 304 Not Modified。

1. 默认的ETag
```ruby
headers['ETag'] = Digest::MD5.hexdigest(body)
```

2. 自定义刷新ETag
```ruby
class ItemsController < ApplicationController
  def show
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
end
```
以上fresh_when方法相当于设置ETag如下
```ruby
headers['ETag'] = Digest::MD5.hexdigest(@item.cache_key)
# => item/2-2014224150000
# cach_key 为 <model name>/<id>-<updated_at>
```

3. 当ETag依赖多个因素时
```ruby
class ItemsController < ApplicationController
  def show
    @item = Item.find(params[:id])
    fresh_when([@item, current_user.id, current_user.age])
  end
  def edit
    @item = Item.find(params[:id])
    fresh_when([@item, current_user.id, current_user.age])
  end
  def most_recent
    @item = Item.find(params[:id])
    fresh_when([@item, current_user.id, current_user.age])
  end
end
```
可以写成如下
```ruby
class ItemsController < ApplicationController
  etag { current_user.id }
  etag { current_user.age }
  def show
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
  def edit
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
  def most_recent
    @item = Item.find(params[:id])
     fresh_when(@item)
  end
end
```

## Memcached && Dalli
使用memecached作为rails缓存一般使用dalli为client。
```ruby
# Gemfile
gem 'dalli'

# config/environments/production.rb
config.action_controller.perform_caching = true
config.cache_store = :dalli_store, 'mem-server:11211', 'mem-server:11211', {:expires_in => 30.day, :compress => true }
```

```ruby
Rails.cache.read('key')
Rails.cache.write('key', value)
Rails.cache.fetch('key') { value }
```

## Digest Cache
片段缓存用于页面级的缓存很简单方便。
```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :document
end
```

```html
<!-- app/views/projects/show.html.erb -->
<%= render @project.documents %>

<!-- app/views/documents/_document.html.erb -->
<article>
  <h3><%= document.title %></h3>
  <ul>
    <%= render document.comments %>
  </ul>
  <%= link_to 'View details', document %>
</article>

<!-- app/views/comments/_comment.html.erb -->
<li><%= comment %></li>
```

为comment添加cache
```html
<!-- app/views/comments/_comment.html.erb -->
<% cache comment do %>
    <li><%= comment %></li>
<% end %>
```
缓存的key默认是comment.cache_key
```ruby
comment.cache_key
# => comment/5-2014224150000
# <model name>/<id>-<updated_at>
```

为document添加cache
```html
<!-- app/views/documents/_document.html.erb -->
<% cache document do %>
  <article>
    <h3><%= document.title %></h3>
    <ul>
      <%= render document.comments %>
    </ul>
    <%= link_to 'View details', document %>
  </article>

  <!-- app/views/comments/_comment.html.erb -->
  <li><%= comment %></li>
<% end %>
```
这样在document下新增或者删除comment，时需要刷新document的cache_key
```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :document, touch: true
end
```

手动添加版本号
```html
<!-- app/views/documents/_document.html.erb -->
<% cache [document, 'v2'] do %>
  <article>
    <h3><%= document.title %></h3>
    <ul>
      <%= render document.comments %>
    </ul>
    <%= link_to 'View details', document %>
  </article>

  <!-- app/views/comments/_comment.html.erb -->
  <li><%= comment %></li>
<% end %>
```

## Rails.cache
缓存购物车总价
```ruby
class ShoppingCart < ApplicationRecord
  has_many :items

  def total_price
    Rails.cache.fetch "#{cache_key}", expires_in: 6.hours do
      # loop item sum price
    end
  end
end

class Item < ApplicationRecord
  belongs_to :shopping_cart, touch: true
  def price
    # cal price and return
  end
end
```

