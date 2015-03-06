用tornado开发高性能web运用
=======

Tornado 和现在的主流 Web 服务器框架（包括大多数 Python 的框架）有着明显的区别：它是非阻塞式服务器，而且速度相当快。得利于其 非阻塞的方式和对 epoll 的运用，Tornado 每秒可以处理数以千计的连接，这意味着对于实时 Web 服务来说，Tornado 是一个理想的 Web 框架。

阅读本书需要基本的python知识，本书由浅入深，为您揭开tornado异步框架的魅力。

本书源码在 Github 上维护，欢迎参与：[https://github.com/weilaihui/tornado-doc](https://github.com/weilaihui/tornado-doc)。贡献者 [名单](https://github.com/weilaihui/tornado-doc/graphs/contributors)。

## 参加步骤
* 在 GitHub 上 `fork` 到自己的仓库，如 `docker_user/tornado-doc`，然后 `clone` 到本地，并设置用户信息。
```
$ git clone git@github.com:docker_user/tornado-doc.git
$ cd docker_practice
$ git config user.name "yourname"
$ git config user.email "your email"
```
* 修改代码后提交，并推送到自己的仓库。
```
$ #do some change on the content
$ git commit -am "Fix issue #1: change helo to hello"
$ git push
```
* 在 GitHub 网站上提交 pull request。
* 定期使用项目仓库内容更新自己仓库内容。
```
$ git remote add upstream https://github.com/weilaihui/tornado-doc
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```
