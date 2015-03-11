# tornado模块

本章主要介绍tornado的主要模块，底层模块和一些其他功能性模块，包含以下内容（小结排列）。

* web - HTTP协议实现
* gen - 异步封装
* testing - 单元测试
* template - 模板实现
* locate - 国际化支持
* options - 配置工具
* httpserver - HTTP服务器实现
* iostream - 非阻塞式socket
* ioloop - 核心I/O循环
* httpclient - 非阻塞式HTTP客户端
* auth - 第三方认证
* autoreload - 自动重载实现
* log - 日志
* wsgi - Web Service Gateway Interface

在内容安排上，并不是每小结都是详细介绍，而是某些重要的章节会详细讲解，某些次要的章节则会简单介绍。比如我们使用Tornado最常打交道的 Web模块，Tornado核心gen模块，iostream和ioloop模块。本章并不会过多讲解原理，而是就使用方式上进行讲解，后续章节会有专门的篇幅进行详细讲解，如果您已了解了基本用法，可以直接跳过本章，或者选取其中几个小结进行查看。