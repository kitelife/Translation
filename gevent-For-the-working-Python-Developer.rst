Gevent教程
============

**Gevent社区编写**

原文： `gevent For the Woring Python Developer <http://sdiehl.github.com/gevent-tutorial/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

gevent是一个基于libev的并发库，为各种并发以及网络相关任务提供干净的API。

简介
------

本教程的结构安排假定读者具备中等的Python水平。不需要具备并发相关的知识。目标是为你提供使用gevent需要掌握的工具，帮助你解决存在的并发问题，并且从今天开始能编写异步应用程序。

**贡献者**

按贡献的时间先后顺序排列： `Stephen Diehl <http://www.stephendiehl.com/>`_ , `Jeremy Bethmont <https://github.com/jerem>`_ , `sww <https://github.com/sww>`_ , `Bruno Bigras <https://github.com/brunoqc>`_ , `David Ripton <https://github.com/dripton>`_ , `Travis Cline <https://github.com/traviscline>`_ , `Boris Feld <https://github.com/Lothiraldan>`_

本文是一篇根据MIT许可证发布的协作文档。需要添加东西？发现排印错误？从 `Github <https://github.com/sdiehl/gevent-tutorial>`_ 签出一个分支，修改之后请求合并就可以了。欢迎任何贡献！

核心
------
