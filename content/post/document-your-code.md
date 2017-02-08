+++
draft = true
title = "document your code"
tags = []
categories = []
description = ""
menu = ""
banner = ""
date = "2017-02-08T20:43:30+08:00"
images = []
+++

# 一个典型python项目的目录结构

很多开源python项目因为遵循了约定俗成的惯例或使用了相同的工具而具有相似的目录结构

一个典型的python项目的目录结构长啥样？

[requests](https://github.com/kennethreitz/requests)的一级目录

  * AUTHORS.rst  (贡献者名单)
  * CONTRIBUTING.md  (提交MR或PR之前需要看的文档)
  * **docs**  (文档目录)
  * **ext**
  * HISTORY.rst  (版本变更日志)
  * LICENSE  (授权声明)
  * Makefile  (自动化构建软件[make](https://zh.wikipedia.org/wiki/Make)的配置文件)
  * MANIFEST.in  (python packaging额外文件清单)
  * NOTICE
  * README.rst  (项目说明书)
  * **requests**  (项目代码目录，名字和项目名一致)
  * requirements-to-freeze.txt
  * requirements.txt  (项目依赖的其它python package列表)
  * setup.py  (python packaging 脚本)
  * **tests**  (测试代码目录)

[flask](https://github.com/pallets/flask)的一级目录

  * **artwork**
  * AUTHORS
  * CHANGES  (同requests的HISTORY.rst)
  * CONTRIBUTING.rst
  * **docs**
  * **examples**  (示例代码目录)
  * **flask**  (源代码目录，和项目名相同)
  * LICENSE
  * Makefile
  * MANIFEST.in
  * README
  * **scripts**
  * setup.cfg  (python packaging 默认配置)
  * setup.py
  * test-requirements.txt
  * **tests**
  * tox.ini  (测试工具[tox](https://github.com/tox-dev/tox)的配置文件)

两个项目的共同点：

1. 使用[sphinx](http://www.sphinx-doc.org/)生成文档
2. 使用[pytest](http://pytest.org/)测试框架
3. 和项目名命名一致的python源码包
4. 根目录有readme, contributing, authors, license和版本变更文档

ps. [python packaging]基本结构

## 书写项目文档
### 一行代码，两行注释
写代码的同时注释说明函数的输入输出，模块的基本功能和项目框架结构或者在注释中插入示例代码，方便新
人（或几个月后的你重新）熟悉项目，修复问题，添加功能或重构代码．

### 上手sphinx
Hugo，Jekyll这些工具能将markdown文档转化成静态网页. Sphinx的主要功能与此类似，读取用户书写
的reST格式文档并输出html静态页面．reST和markdown极为相似，但是允许用户自定义指令扩展功能，正
基于此，sphinx的autodoc插件根据指令抽取代码中的注释插入到生成的静态页面中．

**如何开始**
```bash
$ pip install --user sphinx
$ cd /path/to/project
$ mkdir docs && cd docs  # 新建文档目录
$ sphinx-quickstart  # 交互式命令行工具自动生成一些必要文件，见下图
$ # 书写reST格式的项目文档
$ make html
```

实际项目**requests**的文档目录

![](/img/docs.png)

### 抽取代码注释插入文档的实例
[flask/helper.py](https://github.com/pallets/flask/blob/master/flask/helpers.py)
截取的一段代码，注意函数的docstring
```python
def flash(message, category='message'):
    """Flashes a message to the next request.  In order to remove the
    flashed message from the session and to display it to the user,
    the template has to call :func:`get_flashed_messages`.

    .. versionchanged:: 0.3
       `category` parameter added.

    :param message: the message to be flashed.
    :param category: the category for the message.  The following values
                     are recommended: ``'message'`` for any kind of message,
                     ``'error'`` for errors, ``'info'`` for information
                     messages and ``'warning'`` for warnings.  However any
                     kind of string can be used as category.
    """
    flashes = session.get('_flashes', [])
    flashes.append((category, message))
    session['_flashes'] = flashes
    message_flashed.send(current_app._get_current_object(),
                         message=message, category=category)

```

[flask/docs/api.rst](https://raw.githubusercontent.com/pallets/flask/master/docs/api.rst)文档中插入指令
```
.. autofunction:: flash
```

生成[html](http://flask.pocoo.org/docs/0.12/api/#message-flashing)
```bash
make html # 调用sphinx-build命令行工具
```

[reST基本语法]:http://www.sphinx-doc.org/en/1.5.1/rest.html
[python packaging]: https://packaging.python.org/
[sphinx]: http://www.sphinx-doc.org/
