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

当我们在网络和输入输出相关功能中使用gevent，以便协同调度时，它的真正力量才会得以体现。Gevent兼顾了所有细节以保证你的网络代码库能随时隐式地切换它们的greenlet上下文。这一强大能力是再怎么强调都是不过分的。但也许使用示例更有助于理解。

这一例子中，select()函数通常是一个阻塞调用，在几个文件描述符之间轮询。

::

    import time
    import gevent
    from gevent import select

    start = time.time()
    tic = lamda: 'at %1.1f seconds' %(time.time() - start)

    def gr1():
        # Busy waits for a second, but we don't want to stick around...
        print('Started Polling: ', tic())
        select.select([],[],[],2)
        print('Ended Polling: ', tic())

    def gr2():
        # Busy waits for a second, but we don't want to stick around...
        print('Started Polling: ', tic())
        select.select([], [], [], 2)
        print('Ended Polling: ', tic())

    def gr3():
        print("Hey lets do some stuff while the greenlets poll, at ", tic())
        gevent.sleep(1)

    gevent.joinall([
        gevent.spawn(gr1),
        gevent.spawn(gr2),
        gevent.spawn(gr3),
    ])

::

    Started Polling: at 0.0 seconds
    Started Polling: at 0.0 seconds
    Hey lets do some stuff while the greenlets poll, at at 0.0 seconds
    Ended Polling: at 2.0 seconds
    Ended Polling: at 2.0 seconds

另一个显得有些编造的例子中定义了一个task函数，其结果是非确定性的(比如，对于同样的输入，不能保证它的输入是相同的)。这个函数执行的副作用是其执行过程会暂停一个随机秒数。

::

    import gevent
    import random

    def task(pid):
        """
        Some non-deterministic task
        """
        gevent.sleep(random.randint(0,2)*0.001)
        print('Task', pid, 'done')

    def synchronous():
        for i in range(1, 10):
            task(i)

    def asynchronous():
        threads = [gevent.spawn(task, i) for i in xrange(10)]
        gevent.joinall(threads)

    print('Synchronous: ')
    synchronous()

    print('Asynchronous: ')
    asynchronous()

::
    
    Synchronous:
    ('Task', 1, 'done')
    ('Task', 2, 'done')
    ('Task', 3, 'done')
    ('Task', 4, 'done')
    ('Task', 5, 'done')
    ('Task', 6, 'done')
    ('Task', 7, 'done')
    ('Task', 8, 'done')
    ('Task', 9, 'done')
    Asynchronous:
    ('Task', 0, 'done')
    ('Task', 2, 'done')
    ('Task', 5, 'done')
    ('Task', 3, 'done')
    ('Task', 9, 'done')
    ('Task', 4, 'done')
    ('Task', 8, 'done')
    ('Task', 1, 'done')
    ('Task', 6, 'done')
    ('Task', 7, 'done')

在同步的情况下，所有的任务都是顺序执行的，这样每个任务的执行都会导致主程序的阻塞(比如，暂停主程序的执行)。

这个程序的重要部分是gevent.spawn会将给定的函数包装进一个Greenlet线程。经过初始化的greenlets列表存储在数组threads中，然后传给函数gevent.joinall，它会阻塞当前程序以执行所有给定greenlets。只有当所有greenlet执行终止，当前程序才会继续向前执行。

需要注意的重要事实是，异步情况下，执行的次序本质上是随机的，并且异步情况总的执行时间远远少于同步情况。事实上，同步情况下最大完成时间是20秒，因为每个任务都要暂停2秒。异步情况的最大执行时间大约是2秒，因为任务不会阻塞其他任务的执行。

一个更加常见的使用案例是，异步地从一个服务器抓取数据，鉴于远程服务器的负载，访问请求的fetch()执行时间会有所不同。

::

    import gevent.monkey
    gevent.monkey.patch_socket()

    import gevent
    import urllib2
    import simplejson as json

    def fetch(pid):
        response = urllib2.urlopen('http://json-time.appspot.com/time.json')
        result = response.read()
        json_result = json.loads(result)
        datetime = json_result['datetime']

        print 'Process ', pid, datetime
        return json_result['datetime']

    def synchronous():
        for i in range(1, 10):
            fetch(i)

    def asynchronous():
        threads = []
        for i in range(1, 10):
            threads.append(gevent.spawn(fetch, i))
        gevent.joinall(threads)

    print 'Synchronous:'
    synchronous()

    print 'Asynchronous:'
    asynchronous()
