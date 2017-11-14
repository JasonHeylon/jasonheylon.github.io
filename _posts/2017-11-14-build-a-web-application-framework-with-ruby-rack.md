---
layout: post
title:  "用ruby写一个web程序框架 一起开始于rack"
date:   2017-11-14 00:00:00
categories: ruby
comments: true
---

最近几年都在写使用ruby来进行web开发，得力于ruby界的强壮web框架和丰盛的gem包，大部分时间都在专注于业务代码，一直没有更深入的探寻业务逻辑之下的web框架逻辑，究竟这些框架都做了什么事呢？所以在空闲时期开始了解一些稍微low level的东西，过程中觉得或许我们自己来实际写一个web框架更有利于自己的理解，然后一切开始了。

首选选择一个模仿对象吧，虽然我工作中用到最多的应该是Rails了，但是看着Rails这么庞大的身体，觉得一下子亚历山大，或许小巧的Sinatra是个很好的开始，就从模仿Sinatra开始吧。很快，第一个遇到的东西就是Rack。

## Rack, a modular Ruby webserver interface
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

一个请求在我们的程序中的过程：
客户端请求 => ResponseWrapper => SimpleLogger(打印日志) => App(返回文本) => SimpleLogger => ResponseWrapper(包装html) => 客户端

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


