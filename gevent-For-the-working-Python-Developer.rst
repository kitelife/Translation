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

Greenlets
^^^^^^^^^^

gevent中使用的主要模式是 **Greenlet** ，一个轻量的协程，作为Python的C扩展模块提供使用。所有Greenlet都作为主程序运行在操作系统进程内，但协同调度。

::

    任意给定时刻只有一个greenlet正在运行。

这区别于任意多进程或者线程库所提供的实际并行概念，它们都是产生由操作系统调度的进程和POSIX线程，并且确实是并行的。

同步与异步执行
^^^^^^^^^^^^^^^^

并发的核心概念是一个大的任务可以被分解为一组子任务，这些子任务可以同时或者 *异步* 地调度执行，而不是一次一个或者 *同步* 地执行。两个子任务之间的切换就是我们所了解的上下文切换。

gevent中上下文切换是通过 *yielding* 完成的。举例来说，我们有两个上下文，通过调用gevent.sleep(0)相互切换。

::

    import gevent

    def foo():
        print('Running in foo')
        gevent.sleep(0)
        print('Explicit context switch to foo again')

    def bar():
        print('Explicit context to bar')
        gevent.sleep(0)
        print('Implicit context switch back to bar')

    gevent.joinall([
        gevent.spawn(foo),
        gevent.spawn(bar),
    ])

::

    Running in foo
    Explicit context to bar
    Explicit context switch to foo again
    Implicit context switch back to bar

将这个程序的控制流程可视化或者使用调试器走查一遍程序来观察上下文切换是有助于理解的。

.. image:: https://lh5.googleusercontent.com/-vWikJrvrsOc/UBagHZ0FKUI/AAAAAAAABFU/NNsvk2d50XA/s284/flow.gif

当我们在网络和输入输出相关功能中使用gevent，以便协同调度时，它的真正力量才会得以体现。

