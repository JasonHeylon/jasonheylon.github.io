---
layout: post
title:  "学习ruby世界的web程序框架（二） -- 实现最简单的一个web框架"
date:   2017-11-15 00:00:00
categories: build_a_web_application_framework
tags: ruby rack framework sinatra
comments: true
---

上一篇文章讲了Rack的基本使用方法，这次就基于Rack写一个web应用框架吧。
这个框架包含了最简单基础的一些功能：router, 自定义的filter, 自定义的helper, 支持erb模板，session，serve静态文件等。

## 响应一个get请求
首先给这个框架起个名字 -- Panama，最近这首歌还是挺流行的。Panama主要模仿Sinatra的DSL来实现的。

首先我们先写出程序逻辑吧，当请求首页时，页面返回 Hello Panama。 非常适合第一个完成的任务。

```ruby
# app.rb
require 'panama'
class App < Panama::Base
  get '/' do
    "Hello Panama"
  end
end

# config.ru
require_relative 'app'
run App.new

```

config.ru是启动rackup的默认文件名，在目录下直接使用rackup 不需要传参数时会默认读取这个文件的。

从App中看到 我们需要实现Panama::Base#get的类方法，方法接受一个路径和一个block，并且把block和路径对应的存起来，再等到call方法被触发时运行这个block，把返回的字符串放到body中返回。

```ruby
# panama.rb

require 'rack'

module Panama
  class Base
    # 首先要有一个实例方法call方法来返回
    def call(env)
      # 创建Rack::Request 的实例，方便之后获取请求相关数据
      request = Rack::Request.new(env)

      # 找到访问路径所对应的block
      route_block = block_by_verb_path(request.request_method, request.path_info)

      # 创建Rack::Response 的实例，用于方法返回
      response = Rack::Response.new

      if route_block
        # 执行block并写入response
        response.write(route_block.call)
      else
        # 如果没有找到，说明是没有在路由定义的
        response.status = 404
        response.write('Not found')
      end

      # finish方法实际的返回其实可以认为是[status_code, header_hash, body_array ]
      response.finish
    end

    class << self
      # routers是类变量，
      attr_reader :routers

      def get(path, &block)
        # routers 是结构是 routers[method][path] = block
        @routers ||= {}
        @routers['GET'] ||= {}
        @routers['GET'][path] ||= block
      end

    end

    private

    def block_by_verb_path(verb, path)
      # 获取之前存入的block
      self.class.routers.fetch(verb, {})[path]
    end

  end
end
```
shell中运行rackup后就可以在浏览器看到结果了，一切都很好。
get方法接受一个路径字符串和一个block，并存入类变量routers，并为routers创建了读方法，这样当有请求进来的时候，call方法被调用，通过private方法block_by_verb_path传入请求动词和路径就可以找到对应的block，并且运行它；如果通过请求没有找到block就返回404。

我们再把其他的请求方法都加上

```ruby
# panama.rb

module Panama

  class Base
    # ...
    class << self
      # ...

      %w{get post put patch delete head option}.each do |action|
        define_method action.to_sym do |path, &block|
          add_route(action.upcase, path, &block)
        end
      end

      def add_route(verb, path, &block)
        @routers ||= {}
        @routers[verb] ||= {}
        @routers[verb][path] ||= block
      end

    end

  end

end

```
这样就可以用过 get post等方法传入block和path进行简单的路由操作啦。

## 获得请求参数
其实请求参数的获取可以通过Rack::Resonse#params方法获得，为了方便我们需要实现像下面的用法，
```ruby
# app.rb
# ...

get '/hello' do
  "Hello #{params['name'] || 'Panama'}"
  # /hello?name=Jason  ->  Hello Jason
end

# ...
```

为了实现这个需求，我们添加实例方法params

```ruby
panama.rb

require 'rack'

module Panama

  class Base

    attr_reader :request

    def call(env)
      @request = Rack::Request.new(env)

      route_block = block_by_verb_path(request.request_method, request.path_info)

      response = Rack::Response.new

      if route_block
        route_block.call
      else
        response.status = 404
        response.write('Not found')
      end

      response.finish
    end

    def params
      request.params
    end

    # ...
  end
end

```

好像很简单就实现了，但是运行起来的时候会发现params方法找不到。原因在于block是一个闭包，route_block隔绝了当前的作用域。解决这个问题就要把这个block带入当前作用域，这里使用BasicObject#instance_eval方法让block运行在Panama::Base的实例中。

```ruby

# response.write(route_block.call)

response.write(instance_eval(&route_block))

```
这样route_block在运行的时候就可以访问到Panama::Base的实例方法了。

## filters

filters的实现和router差不多。先写调用代码，在before filter 中打印请求路径和请求参数

```ruby
# app.rb
require 'panama'
class App < Panama::Base

  before do
    puts "request at #{request.path}, params: #{params.inspect}"
  end

  # ...
end

```

首先在Panama::Base添加before和after类方法，传入block，然后再call方法中取出filter，并分别在route_block执行前后运行，同样要使用instance_eval，因为在filter中也会调用像params这样的方法。以下panama到目前为止的完整代码

```ruby
equire 'rack'

module Panama

  class Base

    attr_reader :request

    def call(env)
      @request = Rack::Request.new(env)

      route_block = block_by_verb_path(request.request_method, request.path_info)

      response = Rack::Response.new

      if route_block

        # before_filters
        before_filters = filters_by_type(:before)
        before_filters.each { |filter| instance_eval(&filter) }

        response.write(instance_eval(&route_block))

        # after_filters
        after_filters = filters_by_type(:after)
        after_filters.each { |filter| instance_eval(&filter) }

      else
        response.status = 404
        response.write('Not found')
      end

      response.finish
    end


    def params
      request.params
    end


    class << self
      attr_reader :routers, :filters

      %w{get post put patch delete head option}.each do |action|
        define_method action.to_sym do |path, &block|
          add_route(action.upcase, path, &block)
        end
      end

      def before(&block)
        add_filter(:before, block)
      end

      def after(&block)
        add_filter(:after, block)
      end

      private

      def add_filter(type, block)
        @filters ||= {}
        @filters[type] ||= []
        @filters[type] << block
      end

      def add_route(verb, path, &block)
        @routers ||= {}
        @routers[verb] ||= {}
        @routers[verb][path] ||= block
      end
    end

    private

    def block_by_verb_path(verb, path)
      self.class.routers.fetch(verb, {})[path]
    end

    def filters_by_type(type)
      self.class.filters.fetch(type, {})
    end

  end

end
```
这样每次有请求时，程序都会输出请求信息到shell中了。


## render json

当在block中调用content_type方法，传入symbol :json来设置返回数据类型为json。

```ruby
# app.rb

get '/json' do
  content_type :json

  {
    name: 'Jason',
    age: 18
  }.to_json
end

```

实现这个功能，先在Panama::Base创建实例方法content_type。
这里需要使用Rack中Rack::Mime模块的MIME_TYPES常量，它是一个hash，以文件后缀名为key，value是该后缀名对应的HTTP Headers中Content-Type对应的内容。

```ruby
# 文件在rack代码中： lib/rack/mime.rb

MIME_TYPES = {
  ".123"       => "application/vnd.lotus-1-2-3",
  ".3dml"      => "text/vnd.in3d.3dml",
  ".3g2"       => "video/3gpp2",
  ".3gp"       => "video/3gpp",
  ".a"         => "application/octet-stream",
  ".acc"       => "application/vnd.americandynamics.acc",
  ".ace"       => "application/x-ace-compressed",
  ".acu"       => "application/vnd.acucobol",
  ".aep"       => "application/vnd.audiograph",
  ".afp"       => "application/vnd.ibm.modcap",
  ".ai"        => "application/postscript",
  ".aif"       => "audio/x-aiff",
  # ...
}
```

我们设置默认返回的是html： Rack::Mime::MIME_TYPES[".html"]，并把response改为实例变量@response

```ruby
# panama.rb
require 'rack'

module Panama

  class Base
    attr_reader :request
    attr_accessor :response

    def call(env)
      @request = Rack::Request.new(env)

      route_block = block_by_verb_path(request.request_method, request.path_info)

      @response = Rack::Response.new
      @response['Content-Type'] = Rack::Mime::MIME_TYPES[".html"]
      # ...
    end

  end

end

```
content_type方法中设置正确的response['Content-Type']就可以了。
```ruby
# panama.rb
def content_type(type)
  self.response['Content-Type'] = Rack::Mime::MIME_TYPES[".#{type}"] if Rack::Mime::MIME_TYPES[".#{type}"]
end

```
这样在访问/json时就可以看到正确的header返回了： Content-Type:application/json


## redirect_to

跳转方法也是经常用到的 当使用redirect_to(path)方法时跳转到目标path。

```ruby
# app.rb

get '/redirect' do
  redirect_to '/'
end
```
实现实例方法Panama::Base#redirect_to
这里用到Rack::Response#redirect方法，它接受两个参数，路径和状态码。

```ruby
# 默认状态码302
def redirect_to(path, status_code = 302)
  self.response.redirect(path, status_code)
end
```

这样当访问/redirect就会跳回到首页了


## erb
使用html模板引擎几乎是每个web程序框架必备的功能，我们也来简单实现以下支持erb显示吧。首先要设置erb模板的文件路径，然后调用erb方法传入erb文件。
```ruby
# app.rb
class App < Panama::Base
  # 03
  # 设置erb文件存在views目录下
  set :template_directory, File.join(File.expand_path('..', __FILE__), 'views')
  get '/about' do
    @name = 'Jason'

    erb :about
  end
  # ...
end
```

view/about.erb
```html
<!-- view/about.erb -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>About</title>
</head>
<body>
  <div class="about me">
    My name is <%= @name %>
  </div>
</body>
</html>
```

首先实现Panama::Base.set类方法，
```ruby
# panama.rb

module Panama

  class Base
    # ...

    class << self
      attr_accessor :settings
      # ...
      def set(key, value)
        @settings ||= {}
        @settings[key] = value
      end

    end
    # ...
  end
end

```
set方法将key和value存入类变量@settings中。

实现Panama::Base#erb实例方法，首先读取erb文件，然后生成ERB实例，调用实例方法result返回编译好的html。
但是怎么才能把block里定义的实例变量传入到erb文件中呢，好在ERB的result方法接受一个Binding类型的参数，这里我们调用Kernel#binding得到当前作用域的Binding并传入result方法就可以了。

```ruby
require 'rack'
require 'erb'
module Panama

  class Base
    def erb(template_name)
      template_path = File.join(settings[:template_directory], "#{template_name.to_s}.erb")

      template = File.read(template_path)
      # 传入当前作用域绑定
      ERB.new(template).result(binding)
    end

    # 获取类变量settings
    def settings
      self.class.settings
    end
  end
end

```

## 使用session存储登陆状态

完成一个简单的登陆功能，先把路由写出来

```ruby
# app.rb

helpers do
  def is_login?
    session[:current_user]
  end
end

before do
  if request.path == '/need_login'
    redirect_to '/login' unless is_login?
  end
end

get '/need_login' do
  "You can view this page only logged in"
end

get '/login' do
  redirect_to '/need_login' if is_login?

  erb :login
end

post '/login' do
  # session to store is login
  if params['username'] == 'jason' && params['password'] == '123456'
    session[:current_user] = true
    redirect_to '/need_login'
  else
    redirect_to '/login'
  end
end

```
```html
<!-- login.erb -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Login</title>
</head>
<body>
  <div class="">
    <form action="/login" method="post">
      <input type="text" name="username" value="">
      <input type="password" name="password" value="">
      <button type="submit" name="login">Login</button>
    </form>
  </div>
</body>
</html>
```

首先 need_login这个页面是需要登陆后才可以访问，需要在before filter里判断session是否存有登录状态，如果没有就跳转到login页面。
实现Panama::Base.helpers类方法和Panama::Base#session实例方法。
helpers类方法简单实现，只要把传入的block在class内eval就可以创建Panama::Base的实例方法了。因为路由中的block也在Panama::Base实例内运行，所以helpers中定义的实例方法也是可以被block中的代码访问到的了。

```ruby
# panama.rb
def helpers(&block)
  class_eval(&block)
end
```


实现session方法也很简单，因为Rack已经包含了Rack::Session模块，所以我们的session方法只要返回request.session就可以了，它会返回Rack::Session::Abstract::SessionHash实例，可以使用[]= 和[] 方法设置或读取session。

```ruby
# panama.rb
def session
  self.request.session
end
```
然而现在session还不能工作，我们需要个容器存储我们的session数据，我们选择存入cookie，这里使用了Rack::Session::Cookie中间件，修改config.ru代码。

```ruby
# config.ru
require_relative 'app'

use Rack::Session::Cookie,  :key => 'panama.session',
                            :path => '/',
                            :secret => 'this_is_secret'

run App.new
```
在打开/need_login时成功跳转到 /login, 然后填如用户名和密码点登陆后也顺利跳转到了/need_login。并且可以看到cookie已经存在了。

![panama_session_cookie](http://ozf1ijnw5.bkt.clouddn.com/panama_session_cookie.png "panama_session_cookie")

这样panama就可以完成用户登录的功能啦。

好了 计划中的Panama框架已经完成了，尽管一切都很简陋，而且还有很多需要优化的地方。