# 第一个Tornado Web程序

本小节介绍Tornado框架如何安装和如何编写地一个简单的Web程序

## 安装

Tornado在PyPI的列表中，无论你在哪个平台上，只要安装了python，安装了PyPI，就可以使用下面的命令安装
```
pip install tornado
```
可能每个平台中的Tornado版本都不相同，如果您是想使用最新版本的Tornado，就可以直接到[github](https://github.com/tornadoweb/tornado)下载，该项目包含Demo程序。执行下面命令快速clone

```
git clone https://github.com/tornadoweb/tornado.git
```

安装

```
cd tornado
python setup.py build
sudo python setup.py install
```

注意：Tornado可以运行在Python 2.6,2.7,3.2,3.3和3.4下。每个版本下都需要 `certifi`(Python版本的Ca绑定)，同时在3.0以下，您还必须安装 `backports.ssl_match_hostname`。如果您使用的 `pip` 或者 `easy_install` 来安装的Tornado，上面这些依赖都会自动安装好。

某些Tornado的高级特性依赖以下第三方库。

* unittest2 - 在Python 2.6版本中，运行Tornado的测试用例必需（Python的其他版本中不需要
* concurrent.futures - 是一个推荐的线程池，在Tornado中，如果要使用 `ThreadedResolver` 则必须要依赖它。Python3已经内置，只有当您的Python版本为2时需要安装。
* pycurl - libcurl 封装，tornado.curl_httpclient中依赖它。至少安装libcurl 7.18.2以上，7.21.1或更高是官方推荐的版本。
* Twisted - 另一个经典的Python异步框架，tornado.platform.twisted中依赖它。
* pycares - 是一个可供替代选择的非阻塞DNS解析器。
* Monotime - 提供 monotonic clock。同样只在Python2中需要

Tornado在各个平台下所使用不同的异步实现，在*nix下使用 `epool` ，BSD下使用 `kqueue` ，Window下使用 `select`。但受制于性能影响，BSD（如MAC OS X）或Window下仅用于开发，不适用于生产环境。

## Hello,World

下面给出使用Tornado的一个简单 "Hello,world"例子：

```
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

application = tornado.web.Application([
    (r"/", MainHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```
这个例子还并没有使用任何Tornado的异步特性。我们将在后续章节中逐渐解析并使用Tornado来实现高性能Web运用。