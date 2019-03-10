---
layout: post
title: "[译]深入Rails中的CSRF Protection"
date: 2018-03-08 00:00:00
categories: security
comments: true
---

如果你现在在使用Rails进行开发，你很可能就正在使用CSRF protection。它几乎是在[一开始的时候就有的特性](https://github.com/rails/rails/commit/4e3ed5bc44f6cd20c9e353ab63fd24b92a7942be)，而且就像一些其他的Rails特性一样，它会使你的生活更加美好，基本上你都不用考虑它具体做了什么。

简单来说，跨站请求伪造 (Cross-site request forgery，通常缩写为CSRF)就是一个未认证用户伪造了一个数据请求并把它发送到服务器，让服务器认为这个请求是发自一个已认证用户。Rails会生成一个唯一的token，并且在每一次接到客户端请求时对请求进行身份验证（注：可以简单说验证这个请求是不是在我们的网站上正常发起的），以避免受到CSRF攻击。

最近，我在Unbounce工作中有一个功能需要我考虑如何解决客户端Javascript的请求。这个过程让我发现我当时对CSRF的了解仅仅停留在它的字面意思上。

我决定深入的看一下Rails的源码以了解Rails是如何对这种攻击进行防护的。接下来我会分享一下在这次对Rails中的CSRF Protection探索中所发现的信息。我们将看到token是如何在每次服务器相应中被生成的，以及在接下来的请求中如何使用token来验证客户端身份的。


# 基础知识

Rails中CSRF分别存储在两个地方。一个唯一的token会被插入到HTML中。同时会有一个相同的token会被储存在session cookie中。当用户发送一个POST请求时，会把CSRF token从HTML中拿出来放到请求中发送到服务器。Rails会比对session cookie中的token 和 请求中带有的从HTML获取的token 是否匹配。

![Rails对两个CSRF进行比对](/assets/posts/2018-03-08-a-deep-dive-into-csrf-protection-in-rails/rails_csrf_basic.png "Rails对两个CSRF进行比对")

# 如何使用

作为Rails开发者，你什么都不用做就可以使用CSRF protection. 在 `application_controller.rb` 中有以下一行代码，它开启了Rails的CSRF protection：

```ruby
protect_from_forgery with: :exception
```

然后，在`application.html.erb`文件中，你会发现下面这行代码：

```ruby
<%= csrf_meta_tags %>
```

就是这些，它已经在Rails中很久了，而且我们几乎不需要管它。但Rails究竟是如何实现的呢？

# 生成和加密

我们从 `#csrf_meta_tags`开始。 这是一个简单的helper方法，它会把CSRF token插入到HTML中。

```ruby
# actionview/lib/action_view/helpers/csrf_helper.rb

def csrf_meta_tags
  if protect_against_forgery?
    [
      tag("meta", name: "csrf-param", content: request_forgery_protection_token),
      tag("meta", name: "csrf-token", content: form_authenticity_token)
    ].join("\n").html_safe
  end
end
```

这个csrf-token标签就是我们需要关注的，它就是魔法发生的地方。其中`#form_authenticity_token`方法就是获取到真正的token地方。接下来我们就要进入到RequestForgeryProtection这个module内部了。激动的时刻开始了。

RequestForgeryProtection module处理了所有的与CSRF有关的事。其中包括之前我们在ApplicationController中看到的最著名的`#protect_from_forgery`方法，这个方法会设置一些hook来确保CSRF token的校验能在每次请求时都会被触发，以及当校验失败时该如何响应。而且它也会关系到CSRF token的生成，加密与解析。让我最喜欢的是这个module所涉及的范围很小，在这一个文件中，顺着一些view的helper方法往下看，你会看到CSRF protection的整个实现。

让我们继续深入CSRF token直到它被插入到HTML中。`#form_authenticity_token` 是一个简单的wrapper方法，它接受任意的hash参数，也包括session本身，继续往下会看到`#masked_authenticity_token`方法:

```ruby
# actionpack/lib/action_controller/metal/request_forgery_protection.rb

# Sets the token value for the current session.
def form_authenticity_token(form_options: {})
  masked_authenticity_token(session, form_options: form_options)
end

# Creates a masked version of the authenticity token that varies
# on each request. The masking is used to mitigate SSL attacks
# like BREACH.
def masked_authenticity_token(session, form_options: {}) # :doc:
  # ...
  raw_token = if per_form_csrf_tokens && action && method
    # ...
  else
    real_csrf_token(session)
  end

  one_time_pad = SecureRandom.random_bytes(AUTHENTICITY_TOKEN_LENGTH)
  encrypted_csrf_token = xor_byte_strings(one_time_pad, raw_token)
  masked_token = one_time_pad + encrypted_csrf_token
  Base64.strict_encode64(masked_token)
end
```

根据[per-form CSRF tokens in Rails5](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#per-form-csrf-tokens)中的介绍，`#masked_authenticity_token` 方法已经变得有点复杂了。因为这次我们的目的更关注原始的“每次请求一个CSRF token” 实现，也就是之前提到的meta标签中的那个，所以我们可以先只关注方法中`else`的部分，在这个部分中调用了`#real_csrf_token`，并把值返回给`raw_token`。

为什么我们需要把`session`传入`#real_csrf_token`方法中呢？ 这个方法做了两件事：首先生成一个未被加密处理的token，然后把这个token插入到session cookie中：

```ruby
# actionpack/lib/action_controller/metal/request_forgery_protection.rb

def real_csrf_token(session) # :doc:
  session[:_csrf_token] ||= SecureRandom.base64(AUTHENTICITY_TOKEN_LENGTH)
  Base64.strict_decode64(session[:_csrf_token])
end
```

还记得这个方法最初是我们在application模板中调用的`#csrf_meta_tags`吗，这是一个经典的Rails魔法 -- 它保证了session cookie中的token和页面上的token总是匹配的，因为每一次生成的token都是先储存在session cookie中，然后再把存在session cookie中的token插入到页面中的。

让我们看一下`#masked_authenticity_token`方法的最后部分：

```ruby
  one_time_pad = SecureRandom.random_bytes(AUTHENTICITY_TOKEN_LENGTH)
  encrypted_csrf_token = xor_byte_strings(one_time_pad, raw_token)
  masked_token = one_time_pad + encrypted_csrf_token
  Base64.strict_encode64(masked_token)
```

是时候进行一些加密工作了。现在已经把token插入到了session cookie，接下来方法就要解决到怎样把token返回到HTML的了，这里我们还提前做了些预防（主要是[降低SSL漏洞攻击的可能性](https://github.com/rails/rails/pull/16570)，现在我不会涉及到这些）。需要注意的是session cookie中的token是没有经过我们的加密的，因为从Rails4开始session cookie默认就会被加密了。

首先，我们生成了一次性密钥（one-time pad），我们将要使用它来加密token。[一次性密钥](https://en.wikipedia.org/wiki/One-time_pad)是一种 使用随机生成的密钥来对定长的纯文本进行加密的方法，并且需要这个密钥对已经加密过的消息进行解密操作。之所以它被称作“一次性”密钥，是因为每以个密钥只用于加密一个文本，然后就会被销毁。Rails就是为每一个新的CSRF token都重新生成一个一次性密钥，然后使用它来对token进行 XOR 按位异或操作。再将一次性密钥拼接到上一步得到的字符串前面进行混淆，然后使用Base64进行编码，这样就可以把加密好的token返回到页面中了。

![Rails CSRF 加密过程](/assets/posts/2018-03-08-a-deep-dive-into-csrf-protection-in-rails/rails_csrf_encrypt.png "Rails CSRF 加密过程"){:class="img-responsive"}

当这些操作完成后，加密好的认证token就会返回到堆栈中，回传到application模板中：
```html
<meta name="csrf-param" content="authenticity_token" />
<meta name="csrf-token" content="vtaJFQ38doX0b7wQpp0G3H7aUk9HZQni3jHET4yS8nSJRt85Tr6oH7nroQc01dM+C/dlDwt5xPff5LwyZcggeg==" />
```

# 解析和验证
我们已经看到了CSRF token的生成过程， 以及它是如何被存入到HTML和cookie中的，现在该让我们看看Rails在收到的请求时是怎样进行验证的。

当用户在你的站点提交一个表单时，CSRF token就会与表单数据一起提交(默认是名为`authenticity_token`的参数)，也可以将它放在HTTP头`X-CSRF-Token`中发送。

回忆一下在ApplicationController中的这行代码：
```ruby
protect_from_forgery with: :exception
```

其中，`#protect_from_forgery`方法会在每一个controller action的生命周期中添加before_action：

```ruby
before_action :verify_authenticity_token, options
```

这个before action会将params或header中传来的CSRF token与session cookie中的token进行比较。

```ruby
# actionpack/lib/action_controller/metal/request_forgery_protection.rb

def verify_authenticity_token # :doc:
  # ...
  if !verified_request?
    # handle errors ...
  end
end

# ...

def verified_request? # :doc:
  !protect_against_forgery? || request.get? || request.head? ||
    (valid_request_origin? && any_authenticity_token_valid?)
end
```
在一些判断之后（比如我们不需要验证HEAD和GET的请求），真正的验证发生`#any_authenticity_token_valid?`方法中：

```ruby
def any_authenticity_token_valid? # :doc:
  request_authenticity_tokens.any? do |token|
    valid_authenticity_token?(session, token)
  end
end
```
因为token可能在请求的params或者header中，Rails要求params和header的token中至少有一个与session cookie中的token相匹配才算通过验证。

`#valid_authenticity_token`是很长的一个方法，但其实它只是把`#masked_authenticity_token`方法中的操作反过来做一遍，先对token进行解析然后再做校验：

```ruby
def valid_authenticity_token?(session, encoded_masked_token) # :doc:
  # ...

  begin
    masked_token = Base64.strict_decode64(encoded_masked_token)
  rescue ArgumentError # encoded_masked_token is invalid Base64
    return false
  end

  if masked_token.length == AUTHENTICITY_TOKEN_LENGTH
    # ...

  elsif masked_token.length == AUTHENTICITY_TOKEN_LENGTH * 2
    csrf_token = unmask_token(masked_token)

    compare_with_real_token(csrf_token, session) ||
      valid_per_form_csrf_token?(csrf_token, session)
  else
    false # Token is malformed.
  end
end
```

首先，我们需要对已经被Base64编码过的字符串进行解码，解码后我们得到了被混淆过的token，然后去掉之前加入的混淆，再将得到的token与session中的token进行对比：

```ruby
def unmask_token(masked_token) # :doc:
  one_time_pad = masked_token[0...AUTHENTICITY_TOKEN_LENGTH]
  encrypted_csrf_token = masked_token[AUTHENTICITY_TOKEN_LENGTH..-1]
  xor_byte_strings(one_time_pad, encrypted_csrf_token)
end
```

在`#unmask_token`方法中，我们需要把混淆过的token拆分为原来的两部分：一次性密钥和加密后的token。然后再对这两个字符串进行XOR操作，最后得到了Rails在最开始生成的明文token。

最后，在`#compare_with_real_token`方法使用ActiveSupport::SecureUtils来对两个token进行比较：

```ruby
def compare_with_real_token(token, session) # :doc:
  ActiveSupport::SecurityUtils.secure_compare(token, real_csrf_token(session))
end
```

最后，我们的请求顺利的通过验证啦~

# 结论
我之前从来没有对CSRF有过这么多的思考，Rails其实已经帮我们做了很多事，很多时候我们总是会觉得 它“这样就可以工作了”。每一次去探索这些魔法背后的实现过程都是很美妙的。

我认为CSRF protection的实现是一个很好的对代码进行职责拆分的例子，通过创建一个module来暴露一个简单的而稳定的公共接口，底层的实现可以随意做一些小修改也不会影响到其余的源代码 -- 而且你可以看到在这些年Rails团队为CSRF protection添加的一些功能，就比如 per-form tokens。

在每一次深入Rails源码的过程中我都会学到很多。我希望这些也会激励着你，在遇到一些Rails魔法的时候，去看一看魔法下面的真实样子吧。


英文原文: [A Deep Dive into CSRF Protection in Rails](https://medium.com/rubyinside/a-deep-dive-into-csrf-protection-in-rails-19fa0a42c0ef)
