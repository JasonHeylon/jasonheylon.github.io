---
layout: post
title:  "使用ActiveModel::Errors#details"
date:   2015-02-05 12:00:00
categories: ruby
comments: true
---
最近Rails新增了一个新特性，我们可以使用它获得一个validator类型的对象。
```ruby
class User < ActiveRecord::Base
  validates :name, presence: true
end

user = User.new
user.valid?
user.errors.details
# => {name: [{error: :blank}]}
```

在写api服务时，当你不需要返回一个翻译过的错误消息，而在客户端希望通过判断symbols来给用户以合适的提示时，这个特性会很方便。

你也可以传递额外的参数来添加自定义的错误类型。

```ruby
class User < ActiveRecord::Base
  validate :adulthood

  def adulthood
    errors.add(:age, :too_young, years_limit: 18) if age < 18
  end
end

user = User.new(age: 15)
user.valid?
user.errors.details
# => {age: [{error: :too_young, years_limit: 18}]}
```

所有的内建validators默认都是以hash返回。


这个特性将在Rails 5.0中提供，不过你可以在Rails4.x项目中使用这个gem：[active_model-errors_details](https://github.com/cowbell/active_model-errors_details)。


>*英文原文：[https://cowbell-labs.com/2015-01-22-active-model-errors-details.html](https://cowbell-labs.com/2015-01-22-active-model-errors-details.html)*
