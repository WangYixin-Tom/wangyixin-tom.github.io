---
title: flask
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-22 20:23:24
password:
summary:
tags:
- python
- interview
categories:
- python
- interview
---

# Flask框架

## WEB框架

它们接收 HTTP 请求，然后分发任务，并生成 HTML，然后返回包含 HTML 的 HTTP 应答。

## 应用启动过程

- `run` 方法启动了 Flask 应用

- `run` 方法调用werkzeug 的 `run_simple` 方法，启动了服务器 `BaseWSGIServer`。在调用 run_simple 时，Flask 对象把自己 `self` 作为参数传进去了，在收到请求的时候，就知道调用谁的 `__call__` 方法。

## 请求处理过程

- 默认使用 `WSGIRequestHandler `类来作为 request handler，在接收到请求时这个类在被 `BaseWSGIServer` 调用时，会执行`execute`函数

- `execute`函数中，把 `environ` 和 `start_response` 传入，调用 `app`的`__call__` 。
- `app`的`__call__`中，调用了 `wsgi_app`方法（为了中间件）。

> 该方法最终返回的`response(environ, start_response)`中的response 是`werkzueg.response` 类的一个实例，是可调用的对象，负责生成最终的可遍历的响应体，并调用 `start_response` 形成响应头

-  `wsgi_app`方法中 调用 `request_context(environ)`函数建立了一个 `RequestContext` **请求上下文对象**
   
   -   `RequestContext`初始化根据传入的 `environ` 创建一个 `werkzeug.Request` 的实例
-  把请求上下文 `RequestContext`，调用`push`，压入`_request_ctx_stack` 栈。（这些操作是为了 flask 在处理多个请求的时候不会混淆）。
   
   -  `_request_ctx_stack` 是一个 `LocalStack` 类的实例，通过**Local实现线程隔离**，隔离是使用了`get_ident`,属性被保存到每个线程id对应的字典中了。
- `wsgi_app`方法中 调用  `full_dispatch_request`方法**请求分发**，开始实际的请求处理过程，这个过程中会生成 `response`对象来返回给服务器。
- 调用 `try_trigger_before_first_request_functions` 方法尝试调用 `before_first_request` 列表中的函数，只会执行一次
  	- 调用 `preprocess_request` 方法，调用 `before_request_funcs` 列表中所有的方法。

  > 可以检测用户是否登录，未登录使用`abort`返回错误，则后续不会分发

+ - 调用 `dispatch_request` 方法进行业务请求分发。
    - `_request_ctx_stack.top.request`获取请求上下文
    - 获取请求上下文在的`rule`
    - 调用 `view_functions` 中相应的**视图函数**（`rule.endpoint` 作为键值）并把参数值传入（`**req.view_args`），视图函数就是开发人员写的API接口了。视图函数的返回值或者错误处理视图函数的返回值会作为`rv`返回给`full_dispatch_request`。
  - 调用`finalize_request`根据 `rv` 生成响应
    - 调用`make_response` 方法会查看 rv 是否是要求的返回值类型，否则生成正确的返回类型。
    -  调用`process_response` 方法，实现`after_request`方法的调用
    - 返回`response`


- 如果当中出错，就生成相应的错误信息。

- Flask 都会把**请求上下文**出栈。

## 视图函数注册

在程序加载业务代码时，用修饰器 `route `注册视图函数，并实现 URL 到视图函数的映射。在` route` 方法中，调用了` add_url_rule `方法。主要流程如下：

- 准备好一个视图函数所支持的 HTTP 方法
- 通过 `url_rule_class` 创建一个 `rule` 对象，并把这个对象添加到自己的 `url_map`

> `rule` 对象是一个保存合法的（Flask 应用所支持的） URL、方法、`endpoint`（在 `**options` 中） 及它们的对应关系的数据结构
>
>  `url_map` 是保存`rule` 对象的集合

- `view_functions`中加入`endpoint`、视图函数的映射关系

在 Flask 应用收到请求时，这些被绑定到 url_map 上的 Rule 会被查看，来找到它们对应的视图函数。在 `dispatch_request` 方法中，从 `_request_ctx_stack.top.request` 得到 `rule` 并从这个 `rule` 找到 `endpoint`，最终找到用来处理该请求的正确的视图函数的。

## 请求的过程总结

- 在请求发出之前，Flask 注册好了所有的视图函数和 URL映射，服务器在自己身上注册了 Flask 应用。

- 请求到达服务器，服务器准备好 environ 和 make_response 函数，然后调用了自己身上注册的 Flask 应用。

- 通过 `__call__ `中转到 wsgi_app 的方法。它首先通过 environ 创建了请求上下文，并将它推入栈，使得 flask 在处理当前请求的过程中都可以访问到这个请求上下文。

-  `full_dispatch_request`中开始处理这个请求，依次调用 `before_first_request_funcs` `before_request_funcs view_functions `中的函数，并最终通过 `finalize_request` 生成一个 `response `对象，调用`after_request_funcs `进行 response 生成后的后处理。

- Flask 调用这个 response 对象，最终调用了 make_response 函数，并返回了一个可遍历的响应内容。

- 服务器发送响应。

## Flask 和 werkzeug关系

Flask 和 werkzeug 是强耦合的，一些非常细节的工作，其实都是 werkzeug 库完成的：

- 封装 Response 和 Request 类型供 Flask 使用，在实际开发中，我们在请求和响应对象上的操作，调用的其实是 werkzeug 的方法。

- 实现 URL到视图函数的映射，并且能把 URL中的参数传给该视图函数。我们看到了 Flask 的 url_map 属性并且看到了它如何绑定视图函数和错误处理函数，但是具体的映射规则的实现，和在响应过程中的 UR- 解析，都是由 werkzeug 完成的。

- 通过 _request_ctx_stack 对 Flask 实现线程保护。

## 默认session处理机制?

flask的session是基于cookie的会话保持。**简单的原理**即：

当客户端进行第一次请求时，客户端的HTTP request（cookie为空）到服务端，服务端创建session，视图函数中填写session，请求结束时，session内容填写入response的cookie中并返回给客户端，客户端的cookie中便保存了用户的数据。

当同一客户端再次请求时， 客户端的HTTP request中cookie已经携带数据，视图函数根据cookie中值做相应操作（如已经携带用户名和密码就可以直接登陆）。

**请求第一次来时，session是什么时候生成的？存放在哪里？**

- 客户端的请求进来时，会生成RequestContext对象。其中定义了session，且初值为None。

- 在ctx.push()函数中，所有和 session 有关的调用，都转发到 `session_interface` 的方法调用上，而默认的 `session_inerface`为`SecureCookieSessionInterface()`
  - 执行`SecureCookieSessionInterface.open_session()`来生成默认session对象
    - 获取session签名的算法
    - 获取*request.cookies*，请求第一次来时，**request.cookies为空**，即返回`SecureCookieSession`,session就是一个特殊的字典

**当请求第二次来时，session生成的是什么？**

**request.cookies不为空**，, 获取cookie的有效时长，如果cookie依然有效，通过与写入时同样的签名算法将cookie中的值解密出来并写入字典并返回中，若cookie已经失效，则仍然返回'空字典'。

**特殊的SecureCookieSession字典有那些功能？如何实现的？**

`permanent`（flask 插件会用到这个变量）、`modified`（表明实例是否被更新过，如果更新过就要重新计算并设置 cookie，因为计算过程比较贵，所以如果对象没有被修改，就直接跳过）

`SecureCookieSession` 是基于 `CallbackDict` 实现的，这个类可以指定一个函数作为 on_update 参数，每次有字典操作的时候（`__setitem__`、`__delitem__`、clear、popitem、update、pop、setdefault）会调用这个函数。
**session什么时候写入cookie中？session的生命周期？**

`process_response`判断session是否为空，如果不为空，则执行`save_session()`，其中通过`response.set_cookie`将session写入。这样便完成session的写入response工作，并由response返回至客户端。

## 上下文

### **flask上下文种类**

current_app、g 是应用上下文。 request、session 是请求上下文。

### 为什么要用上下文

flask从客户端获取到请求时，要让视图函数能访问一些对象，这样才能处理请求。例如请求对象就是一个很好的例子。要让视图函数访问请求对象，一个显而易见的方法就是将其作为参数传入视图函数，不过这回导致程序中的每个视图函数都增加一个参数，为了避免大量可有可无才参数把视图函数弄得一团糟，flask使用上下文临时把某些对象变为全局可访问（只是当前线程的全局可访问）。

### **请求上下文和应用上下文两者区别**

请求上下文:保存了客户端和服务器交互的数据。request处理http请求，session  处理用户信息。LocalStack用来存储请求上下文

应用上下文:flask 应用程序运行过程中，保存一些配置信息，比如程序名、数据库连接、应用信息等。

g 用来存储开发者自定义的一些数据，不用通过传参的方式获取参数了。current_app 当前激活程序的程序实例。LocalStack用来存储应用上下文。

### **生命周期** 

- current_app的生命周期最长，只要当前程序实例还在运行，都不会失效。
- Request和g的生命周期为一次请求期间，当请求处理完成后，生命周期也就完结了
- Session就是传统意义上的session了。只要它还未失效（用户未关闭浏览器、没有超过设定的失效时间），那么不同的请求会共用同样的session。

### **为什么上下文需要放在栈中？**

1.应用上下文：

Flask底层是基于werkzeug，werkzeug是可以包含多个app的，所以这时候用一个栈来保存，如果你在使用app1，那么app1应该是要在栈的顶部，如果用完了app1那么app应该从栈中删除，方便其他代码使用下面的app。

2.请求上下文：

如果在写测试代码，或者离线脚本的时候，我们有时候可能需要创建多个请求上下文，这时候就需要存放到一个栈中了。使用哪个请求上下文的时候，就把对应的请求上下文放到栈的顶部，用完了就要把这个请求上下文从栈中移除掉。

### 上下文管理流程?

每次有请求过来的时候，flask 会先创建当前线程或者进程需要处理的两个重要上下文对象，把它们保存到隔离的栈里面，这样视图函数进行处理的时候就能直接从栈上获取这些信息。

1.请求到来时，将session和request封装到ctx对象中；

2.对session作补充；

3.将包含了request和session的ctx对象放到一个容器中（每一个请求都会根据线程/协程加一个惟一标识）；

4.视图函数使用的时候须要根据当前线程或协程的惟一标识，获取ctx对象，再取ctx对象中取request和session（视图函数使用的时候，须要根据当前线程获取数据。）

5.请求结束时，根据当前线程/协程的惟一标记，将这个容器上的数据移除。

###  为什么要把 request context 和 application context 分开？每个请求不是都同时拥有这两个上下文信息吗？

虽然在实际运行中，每个请求对应一个 request context 和一个 application context，但是在测试或者 python shell 中运行的时候，用户可以单独创建 request context 或者 application context，这种灵活度方便用户的不同的使用场景

### 为什么Local对象中的stack 维护成一个列表？

测试的时候可以添加多个上下文，另外一个原因是 flask 可以[多个 application 同时运行](http://flask.pocoo.org/docs/0.12/patterns/appdispatch/#combining-applications)

### Flask中多app应用是怎么完成？

请求进来时，可以根据URL的不同，交给不同的APP处理。

使用Flask类建立不一样的app对象，而后借助DispatcherMiddleware类来实现。

### Local对象和threading.local对象的区别

Thread Local 则是一种特殊的对象，它的“状态”对线程隔离 —— 也就是说每个线程对一个 Thread Local 对象的修改都不会影响其他线程。原理也非常简单，只要以线程的 ID 来保存多份状态字典即可。

werkzeug.local.Local和threading.local**区别**如下：

（1）werkzeug使用了自定义的`__storage__`保存不同线程下的状态

（2）werkzeug提供了释放本地线程的release_local方法

（3）werkzeug通过get_ident函数来获得线程标识符

#### **为什么造轮子**

WSGI不保证每个请求必须由一个线程来处理，如果WSGI服务器不是每个线程派发一个请求，而是每个协程派发一个请求，所以如果使用thread local变量可能会造成请求间数据相互干扰，因为一个线程中存在多个请求。所以werkzeug给出了自己的解决方案：werkzeug.local模块。

#### 除 了Local 

Werkzeug 还实现了两种数据结构：LocalStack 和 LocalProxy。

LocalStack 是用 Local 实现的栈结构，可以将对象推入、弹出，也可以快速拿到栈顶对象。当然，所有的修改都只在本线程可见。

LocalProxy用于代理Local对象和LocalStack对象，而所谓代理就是作为中间的代理人来处理所有针对被代理对象的操作。LocalStack无法再动态更新了，而使用Proxy实现了动态更新。重载了绝大多数操作符，以便在调用LocalProxy的相应操作时，通过`_get_current_object` method来获取真正代理的对象，然后再进行相应操作

## 蓝图

### 作用

- 将不同的功能模块化，构建大型应用

- 优化项目结构

- 增强可读性,易于维护

###  使用

- 新增了一个xxx.py的文件，实例化Blueprint应用
- xxx.py编写视图函数，使用Blueprint应用示例设置路由
- run.py中，注册蓝图示例

## 如何在Flask中访问会话?

用户第一次请求后，将产生的状态信息保存在session中，这时可以把session当做一个容器，它保存了正在使用的所有用户的状态信息；这段状态信息分配了一个唯一的标识符用来标识用户的身份，将其保存在响应对象的cookie中；当第二次请求时，解析cookie中的标识符，拿到标识符后去session找到对应的用户的信息。

在flask中，如果我们想要获取session信息，直接通过flask的session获取就可以了，这是因为session是一个代理对象，代理当前请求上下文的session属性。

## wsgi

Web服务器和Web应用程序或框架之间的一种简单而通用的接口。描述了web server如何与web application交互、web application如何处理请求。

**应用程序端**

WSGI 规定每个 python 程序（Application）必须是一个可调用的对象（实现了`__call__` 函数的方法或者类），接受两个参数 environ（WSGI 的环境信息） 和 start_response（开始响应请求的函数），并且返回 iterable。

## Werkzeug

werkzeug 的定位并不是一个 web 框架，而是 HTTP 和 WSGI 相关的工具集，可以用来编写 web 框架，也可以直接使用它提供的一些帮助函数。

werkzeug 提供了 python web WSGI 开发相关的功能：

- 路由处理：如何根据请求 URL 找到对应的视图函数

- request 和 response 封装: 提供更好的方式处理request和生成response对象

- 自带的 WSGI server: 测试环境运行WSGI应用

## RESTful

REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

（1）每一个URI代表一种资源；

（2）客户端和服务器之间，传递这种资源的某种表现层；

（3）客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

"资源"是一种信息实体，它可以有多种外在表现形式。**我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。**

所有的状态都保存在服务器端。因此，**如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。**

## 参考资料

https://juejin.im/post/6844903533238566925#heading-10

http://www.pythondoc.com/flask/appcontext.html

http://www.pythondoc.com/flask/reqcontext.html

https://cizixs.com/2017/01/12/flask-insight-routing/

https://www.cnblogs.com/kendrick/p/7649772.html

https://www.cnblogs.com/panlq/p/13266426.html