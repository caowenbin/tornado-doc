# Web HTTP协议实现

web模块里面最常用的RequestHandler类，通常我们自己的Handler都是继承它。现在我们先来看看经常接触到的方法。

## methods HTTP常用方法

Tornado里面默认支持 GET,HEAD,POST,DELETE,PATCH,PUT,OPTIONS方法，如果您还需要其他方法，可以子类中重写 `SUPPORTED_METHODS`。如果要使用某个方法，直接在子类中重写该方法。比如：

代码清单2-1 Tornado支持方法 methods.py

```
class MethodHandler(tornado.web.RequestHandler):
    def head(self):
    	# do_something()
        self.set_status(200)
    def get(self):
    	# do_something()
        self.write("you call get method.")
    def post(self):
    	# do_something()
        self.write("you call post method.")
    def put(self):
    	# do_something()
        self.write("you call put method.")
    def delete(self):
    	# do_something()
        self.write("you call delete method.")
    def patch(self):
    	# do_something()
        self.write("you call patch method.")
    def options(self):
    	# do_something()
        self.write("you call options method.")
```

我们可以使用 `curl -v -X {method} http://localhost:8080/book` 来检查所有方法，后面会讲到如何使用但愿测试来测试我们的代码。

## 方法拦截

我们可以通过重写initialize方法来为Handler子类额外的初始化操作，比如初始化数据库对象。我们也可以重写prepare方法来为get/post等添加统一拦截。

代码清单2-2 方法拦截init-pre.py

```
class InitPreHandler(tornado.web.RequestHandler):
    def initialize(self, database):
        self.database = database
    def prepare(self):
        print "do prepare."
    def get(self):
        print "do get."
        # do_something()
    def post(self):
        print "do post."
        # do_something()
```

## argument相关

我们一般使用get_argument和get_arguments方法来获取客户端提交的参数，它们的区别是后者返回list。这两个方法不区分参数是在url还是在body。我们有时间也可能从url路径中获取参数，这在RESTful风格架构中就很常见了，Tornado的每个方法都支持无限的参数匹配，比如/book/1/comment/2 我们可能使用 `/book/(\w+)/comment/(\w+)` 来匹配该url，只用在对应的方法添加参数列表即可，例如 `get(self,book_id,comment_id)` 。

代码清单2-3 获取参数 argument.py

```
class CommentHandler(tornado.web.RequestHandler):
    def get(self,book_id,comment_id):
        print "book_id is:",book_id,"comment_id is:",comment_id
        self.write({"content":"hello tornado."})

class CommentsHandler(tornado.web.RequestHandler):
    def post(self,book_id):
        content = self.get_argument("content",None)
        if not content:
            self.set_status(400)
            self.write("content is null.")
            self.finish()
            return

        print "book_id is:",book_id,"content:",content
        new_comment_id = 3 # maybe you would insert comment to db
        self.write({"content":content,"comment_id":new_comment_id})
application = tornado.web.Application([
    (r"/book/(\w+)/comment/(\w+)", CommentHandler),
    (r"/book/(\w+)/comments", CommentsHandler),
])
```

## headers相关

在某些运用中，特别是api运用，可能会设置统一的Response Header，此时我们可以在子类中重写set_default_headers。自定义header可以使用set_header(name, value)和add_header(name,value)。下面是一个使用的例子

代码清单2-4 headers相关 headers.py

```
class MainHandler(tornado.web.RequestHandler):
    def set_default_headers(self):
        self.set_header("Server","MyServer")
        self.set_header("Server","MyServer1")
    def get(self):
        self.add_header("My-Header","Hello")
        self.add_header("My-Header","world")
        self.write("Hello, world")
```

可以看到set_header会覆盖以前的值，add_header不会覆盖，如果我们查看Resonse Header会看到类似如下输出

```
Etag:"e02aa1b106d5c7c6a98def2b13005d5b84fd8dc8"
My-Header:Hello
My-Header:world
Server:MyServer1
```

## 输出相关

Tornado提供了基本的函数来方便我们使用输出渲染，包含了输出文本、渲染模板、页面调整、重定向、自定义错误页面。

#### 输出

直接输出文本可以使用write方法。该方法会自动将python 中的dict输出问JSON，并设置 `Content-Type` 为 `application/json`。（如何要禁用该功能，可以在write之后调用set_header）

代码清单2-5 write方法 write.py

```
class WriteHandler(tornado.web.RequestHandler):
    def get(self):
        self.write({"content":"hello tornado."})

class WriteDoNotAutoSetHeaderHandler(tornado.web.RequestHandler):
    def get(self):
        self.write({"content":"hello tornado."})
        self.set_header("Content-Type","text/html")
```

#### 渲染模板

渲染模板会使用到的方法有：render(template_name,**kwargs)、render_string(template_name, **kwargs)、get_template_namespace()。下面给出使用例子。

代码清单2-6 render渲染模板 render.py

```
class RenderHandler(tornado.web.RequestHandler):
    def get(self):
        books = []
        for i in xrange(1,10):
            book = {
                "id":i,
                "title":"book title%d"%i
            }
            books.append(book)
        self.render("render.html",books=books)
    def get_template_namespace(self):
        namespace = {
            "title":"hello tornado"
        }
        return namespace

class RenderStringHandler(tornado.web.RequestHandler):
    def get(self):
        books = []
        for i in xrange(1,10):
            book = {
                "id":i,
                "title":"book title%d"%i
            }
            books.append(book)
        html = self.render_string('render.html',books=books)
        self.write(html)

## render.html
{{title}}
{%for book in books%}
<h1>{{book.get("id","")}}</h1>
<h1>{{book.get("title","")}}</h1>
{%end%}
```

#### 重定向

重定向分永久重定向和临时重定向，对应http状态码分别为301和302。一般使用临时重定向302，永久重定向为不可返回。

#### 自定义错误

要自定义错误，只需要重写write_error方法，该方法可以直接调用write，render，set_header方法，可用的参数都在 `kwargs[exc_info]`种。

#### Cookies

Cookie保存在客户端，由客户端自己维护，根据“不能相信任何来自客户端的数据”原则，我们不能保存明文，尽量少的信息是比较好的。在Tornado中，提供了两组方法来管理cookie，get_cookie，set_cookie和get_secure_cookie，set_secure_cookie，后者提供了好的安全性。当然path，domain以及expires和其他Web框架都是一样的。

代码清单 2-7 使用Cookies cookies.py

```
class CookiesHandler(tornado.web.RequestHandler):
    def get(self,book_id,comment_id):
        tornado_doc = self.get_cookie("tornado-doc",None)
        if not tornado_doc:
            print "has no tornado-doc cookie"
            self.set_cookie("tornado-doc","hello tornado doc.")
            return

        assert tornado_doc == "hello tornado doc."

class SecureCookiesHandler(tornado.web.RequestHandler):
    def get(self,book_id,comment_id):
        tornado_doc = self.get_secure_cookie("tornado-doc",None)
        if not tornado_doc:
            print "has no tornado-doc cookie"
            self.set_secure_cookie("tornado-doc","hello tornado doc.")
            return
            
        assert tornado_doc == "hello tornado doc."

if __name__ == "__main__":
    application = tornado.web.Application([
        (r"/cookies", CookiesHandler),
        (r"/secure_cookes", SecureCookiesHandler),
    ],cookie_secret="gjsAFuqQTJymMOGmGxwRu0vqTDQbYkqRqx5aE5Noz3Y=") ## 必须替换为您自己的cookie_secret
    application.listen(8080)
    tornado.ioloop.IOLoop.instance().start()
```

## Application类

在web模块中，除了RequestHandler类接触得最多之外，Application也是我们常使用的。细心的读者应改在前面的例子中注意到Application的使用，它主要处理Tornado的参数配置。下面给出常见的参数及其使用说明。

* 


