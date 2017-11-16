---
layout: post
title:  "学习ruby世界的web程序框架 1 -- 开始于rack"
date:   2017-11-14 00:00:00
categories: learn_web_application_framework_with_ruby
tags: ruby rack framework
comments: true
---

本篇文章是介绍rack很基础性文章。

最近几年都在写使用ruby来进行web开发，得力于ruby界的强壮web框架和丰盛的gem包，大部分时间都在专注于业务代码，一直没有更深入的探寻业务逻辑之下的web框架逻辑，究竟这些框架都做了什么事呢？所以在空闲时期开始了解一些稍微low level的东西，很快，第一个遇到的东西就是Rack。

## Rack
> a modular Ruby webserver interface

写过Ruby代码更准确的是写过Rails，Sinatra程序的人一定都听过rack这个东西。（挺多刚刚开始接触Rails的人常常会把rack和rake弄混，除了长得像应该没有更多的关系了吧）

通俗的说，Rack就是把Http请求引入到Ruby程序，然后接受Ruby程序返回的数据，再把数据返回给客户端的一个接口。

## 第一个Rack程序
Rack在接收http请求后会调用Ruby Object的call方法，并传入唯一参数，一般叫env，其中包含客户端的请求信息等，call方法的返回数据就会展现到客户端。那么 写一个基于Rack的web程序就是写这个Ruby Object的过程。

下面来写一个最简单的基于Rack的web程序。

```ruby
# app.rb
# 首先要安装Rack gem， 然后require进来
require 'rack'

class App
# 程序入口 call
  def call(env)
    return [
      '200',
      {
        'Content-Type' => 'text/html'
      },
      [
        'Hello Rack'
      ]
    ]
  end
end
```

这就是最简单的Hello Rack了，首先http请求进入call方法，方法返回一个数组： Http Code, Header信息, 和body数组。

然后我们需要一个web server来启动我们的Rack程序，WEBrick是Ruby默认安装的一个web server。在程序的最后加入一行来使程序运行起来。

WEBrick 默认绑定的端口是8080，也可以通过Port参数传入指定的端口号。

```ruby
# app.rb

Rack::Handler::WEBrick.run App.new, port: 8888
```

然后再在shell里运行app.rb
```shell
ruby app.rb
```

这样打开浏览器http://localhost:8888 就可以看到我们的第一个Rack程序啦。

## Rack Middleware

Rack中间件也是我们常常能看到的一个名词，我们通过Rack中间件可以处理request和response、处理用户验证、打印日志、缓存、监控....

其实Rack中间件和上面的Hello Rack程序有一些相识，我们来写一个最简单的打印日志的Rack中间件并且应用到Hello Rack程序上。

这个Rack中间件的功能就是在程序接到请求后打印一句话到终端

```ruby
# simple_logger.rb
class SimpleLogger
  def initialize(app)
    @app = app
  end

  def call(env)
    puts "[#{Time.now}]received a http request."
    @app.call(env)
  end
end
```

和上面的Hello Rack程序最大的区别就是多了一个名为app的实例变量，在初始化时由外部传入，在call方法调用app的call方法并返回，这个要被传入的app就可以使我们之前的Hello Rack程序。

```ruby
# app.rb
# 首先引入这个中间件
require_relative 'simple_logger'
...

# 最后在使用这个中间件
Rack::Handler::WEBrick.run SimpleLogger.new(App.new), Port: 8888
```

再一次在终端运行服务
```shell
ruby app.rb
```
当使用浏览器访问localhost:8888时，在控制台就可以看到我们写入的输出。
```shell
[2017-11-14 17:02:23 +0800]received a http request.
```

我们再写一个中间件，来处理返回数据，把我们的Hello Rack包在html body中。
```ruby
# response_wrapper.rb
class ResponseWrapper
  def initialize(app)
    @app = app
  end

  def call(env)
    http_code, headers, body = @app.call(env)
    html_body = "<html> \
    <head><title>Hello Rack</title></head> \
    <body>#{body[0]}</body> \
    </html>"

    [
      http_code,
      headers,
      [html_body]
    ]
  end
end
```

在修改app.rb
```ruby
# app.rb

require_relative 'response_wrapper'
...
Rack::Handler::WEBrick.run ResponseWrapper.new(SimpleLogger.new(App.new)), Port: 8888
```
这样我们就使用了两个中间件了。

简单描述一下一个请求在我们的程序中的过程：


![hello_rack_app_flow](http://ozf1ijnw5.bkt.clouddn.com/hello_rack_app_flow.png "hello_rack_app_flow")
其实很多复杂的Rack框架 都是使用很多层Rack中间件来组装起来的，请求数据从最外层向内传递进去，数据的返回从内再又一层层向上包装处理返回的。但是如果中间件多了的话像 ResponseWrapper.new(SimpleLogger.new(App.new)) 这种写法会很长很长，也很难看了。

## 使用rackup命令启动rack程序
Rack gem中带了rackup命令，它可以更方便的启动rack程序，提供了Rack::Builder的DSL来进行配置，更优雅的使用middleware，而且可以自动检测程序所运行的环境。

rackup 命令接受一个配置文件为参数，配置文件以.ru为后缀。

先把app.rb 改名为app.ru

在app.ru中修改最后一行代码

```ruby
# app.ru

...

# Rack::Handler::WEBrick.run SimpleLogger.new(App.new), Port: 8888

# 使用中间件使用 use方法
use ResponseWrapper
use SimpleLogger

run App.new
```

在shell中执行
```shell
// 参数-p 指定端口
rackup app.ru -p 8888
```
这样程序就启动起来了，并且rackup命令会自动选择合适的web server。

注意的是使用中间件的顺序，先use的中间件会在中间件的上层。

本文只是简单讲解了一下rack app/middleware的基本概念和基本使用，要是写一个基于rack的web框架了解这些基本知识还是很有必要的。

## env in call

对了，call方法的参数env到底是什么？ env中包含了很多请求信息，比如请求的动词，访问路径，cookie信息，服务信息等，在实际的web框架中会经常用到的。我们可以打印出来如下：
```ruby
{"GATEWAY_INTERFACE"=>"CGI/1.1", "PATH_INFO"=>"/favicon.ico", "QUERY_STRING"=>"", "REMOTE_ADDR"=>"127.0.0.1", "REMOTE_HOST"=>"127.0.0.1", "REQUEST_METHOD"=>"GET", "REQUEST_URI"=>"http://localhost:8888/favicon.ico", "SCRIPT_NAME"=>"", "SERVER_NAME"=>"localhost", "SERVER_PORT"=>"8888", "SERVER_PROTOCOL"=>"HTTP/1.1", "SERVER_SOFTWARE"=>"WEBrick/1.3.1 (Ruby/2.4.2/2017-09-14)", "HTTP_HOST"=>"localhost:8888", "HTTP_CONNECTION"=>"keep-alive", "HTTP_PRAGMA"=>"no-cache", "HTTP_CACHE_CONTROL"=>"no-cache", "HTTP_USER_AGENT"=>"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36", "HTTP_ACCEPT"=>"image/webp,image/apng,image/*,*/*;q=0.8", "HTTP_REFERER"=>"http://localhost:8888/", "HTTP_ACCEPT_ENCODING"=>"gzip, deflate, br", "HTTP_ACCEPT_LANGUAGE"=>"zh-CN,zh;q=0.8,en;q=0.6", "rack.version"=>[1, 3], "rack.input"=>#<Rack::Lint::InputWrapper:0x00007fb3da858aa0 @input=#<StringIO:0x00007fb3da85b2c8>>, "rack.errors"=>#<Rack::Lint::ErrorWrapper:0x00007fb3da858a78 @error=#<IO:<STDERR>>>, "rack.multithread"=>true, "rack.multiprocess"=>false, "rack.run_once"=>false, "rack.url_scheme"=>"http", "rack.hijack?"=>true, "rack.hijack"=>#<Proc:0x00007fb3da858d20@/Users/JasonHeylon/.rvm/gems/ruby-2.4.2/gems/rack-2.0.3/lib/rack/lint.rb:525>, "rack.hijack_io"=>nil, "HTTP_VERSION"=>"HTTP/1.1", "REQUEST_PATH"=>"/favicon.ico", "rack.tempfiles"=>[]}
```

直接获取env中的内容可能让人不太舒服，Rack也提供了Rack::Request类来更方便使用env信息，生成Rack::Request实例只需要把env传入Rack::Request#new方法。
```ruby
request = Rack::Request.new(env)

# url中的参数
request.params
# url的访问路径
request.path_info
# cookie
request.cookies
# 请求方式
request.request_method
# 判断是不是get请求
request.get?
# 是否ssl
request.ssl?
```

是的，有了Rack::Request当然就会有Rack::Response类了，它主要是为了返回相应数据所用。Rack::Response#new接受三个参数： def initialize(body=[], status=200, header={})
```ruby

response = Rack::Response.new
# 设置cookie
response.set_cookie(key, {})
# 设置http code
response.status = 404
# 设置etag
response.etag = "666"
# 跳转，第二个参数是http code默认302
response.redirect(destination_url, 302)
# 写body
response.write('This is a rack app')

# 当处理完要返回的信息时，调用finish方法，此方法返回同样也是[http_code, headers, body]
# 但返回数据的body是Rack::BodyProxy的一个实例，包含实例对象@body就是之前的body数组，并且实在Rack::BodyProxy上实现了实例方法each遍历@body的每个元素
response.finish


# 比如之前的Hello Rack可以改写使用Rack::Response的返回

resp = Rack::Response.new(env)

resp.status = 200
resp.set_header('Content-Type', 'text/html')
resp.write('Hello Rack')
resp.finish

```

待续。。。。:bowtie: