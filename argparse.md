原文：[argparse – Command line option and argument parsing](http://pymotw.com/2/argparse/)

译者：[youngsterxyf](https://github.com/youngsterxyf)

argparse模块作为optparse的一个替代被添加到Python2.7。argparse的实现支持一些不易于添加到optparse以及要求向后不兼容API变化的特性，因此以一个新模块添加到标准库。


### 与optparse相比较

argparse的API类似于optparse，甚至在很多情况下通过更新所使用的类名和方法名，使用argparse作为一个简单的替代。然而，有些地方在添加新特性时不能保持直接兼容性。

你必须视情况决定是否升级已有的程序。如果你已编写了额外的代码以弥补optparse的局限，也许你想升级程序以减少你需要维护的代码量。若argparse在所有部署平台上都可用，那么新的程序应尽可能使用argparse。

### 设置一个解析器

使用argparse的第一步就是创建一个解析器对象，并告诉它将会有些什么参数。那么当你的程序运行时，该解析器就可以用于处理命令行参数。

解析器类是 **ArgumentParser** 。构造方法接收几个参数来设置用于程序帮助文本的描述信息以及其他全局的行为或设置。

{% highlight python %}
import argparse
parser = argparse.ArgumentParser(description='This is a PyMOTW sample program')
{% endhighlight %}

### 定义参数 

argparse是一个全面的参数处理库。参数可以触发不同的动作，动作由 **add_argument()** 方法的 *action* 参数指定。
支持的动作包括保存参数（逐个地，或者作为列表的一部分），当解析到某参数时保存一个常量值（包括对布尔开关真/假值的特殊处理），统计某个参数出现的次数，以及调用一个回调函数。

默认的动作是保存参数值。在这种情况下，如果提供一个类型，那么在存储之前会先把该参数值转换成该类型。如果提供 *dest* 参数，参数值就保存为命令行参数解析时返回的命名空间对象中名为该 *dest* 参数值的一个属性。

### 解析一个命令行

定义了所有参数之后，你就可以给 **parse_args()** 传递一组参数字符串来解析命令行。默认情况下，参数是从 `sys.argv[1:]` 中获取，但你也可以传递自己的参数列表。选项是使用GNU/POSIX语法来处理的，所以在序列中选项和参数值可以混合。

**parse_args()** 的返回值是一个**命名空间**，包含传递给命令的参数。该对象将参数保存其属性，因此如果你的参数 `dest` 是 `"myoption"`，那么你就可以`args.myoption` 来访问该值。

### 简单示例

以下简单示例带有3个不同的选项：一个布尔选项（`-a`），一个简单的字符串选项（`-b`），以及一个整数选项（`-c`）。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(description='Short sample app')

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

print parser.parse_args(['-a', '-bval', '-c', '3'])
{% endhighlight %}

有几种方式传递值给单字符选项。以上例子使用了两种不同的形式，`-bval`和`-c val`。

{% highlight bash %}
$ python argparse_short.py
Namespace(a=True, b='val', c=3)
{% endhighlight %}

在输出中与`'c'`关联的值是一个整数，因为程序告诉**ArgumentParser**在保存之前先转换该参数。

“长”选项名字，即选项的名字多于一个字符，以相同的方式进行处理。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(description='Example with long option names')

parser.add_argument('--noarg', action="store_true", default=False)
parser.add_argument('--witharg', action="store", dest="witharg")
parser.add_argument('--witharg2', action="store", dest="witharg2", type=int)

print parser.parse_args(['--noarg', '--witharg', 'val', '--withargs=3'])
{% endhighlight %}

结果也类似：

{% highlight bash %}
$ python argparse_long.py
Namespace(noarg=True, witharg='val', witharg2=3)
{% endhighlight %}

argparse区别于optparse的一个地方是对非选项参数值的处理。optparse只进行选项解析，而argparse是一个全面的命令行参数解析工具，也处理非选项参数。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(description='Example with non-optional arguments')

parser.add_argument('count', action="store", type=int)
parser.add_argument('units', action="store")

print parser.parse_args()
{% endhighlight %}

在这个例子中，“count”参数是一个整数，“units”参数存储为一个字符串。其中任意一个参数若没有在命令行中提供，或给定的值不能被转换为正确的类型，就会报告一个错误。

{% highlight bash %}
$ python argparse_arguments.py 3 inches

Namespace(count=3, units='inches')

$ python argparse_arguments.py some inches

usage: argparse_arguments.py [-h] count units
argparse_arguments.py: error: argument count: invalid int value: 'some'

$ python argparse_arguments.py

usage: argparse_arguments.py [-h] count units
argparse_arguments.py: error: too few arguments
{% endhighlight %}

### 参数动作

argparse内置6种动作可以在解析到一个参数时进行触发：

`store`
    保存参数值，可能会先将参数值转换成另一个数据类型。若没有显式指定动作，则默认为该动作。

`store_const`
    保存一个被定义为参数规格一部分的值，而不是一个来自参数解析而来的值。这通常用于实现非布尔值的命令行标记。

`store_ture`/`store_false`
    保存相应的布尔值。这两个动作被用于实现布尔开关。

`append`
    将值保存到一个列表中。若参数重复出现，则保存多个值。

`append_const`
    将一个定义在参数规格中的值保存到一个列表中。

`version`
    打印关于程序的版本信息，然后退出

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('-s', action='store', dest='simple_value',
        help='Store a simple value')

parser.add_argument('-c', action='store_const', dest='constant_value',
        const='value-to-store',
        help='Store a constant value')

parser.add_argument('-t', action='store_true', default=False,
        dest='boolean_switch',
        help='Set a switch to true')
parser.add_argument('-f', action='store_false', default=False,
        dest='boolean_switch',
        help='Set a switch to false')

parser.add_argument('-a', action='append', dest='collection',
        default=[],
        help='Add repeated values to a list')

parser.add_argument('-A', action='append_const', dest='const_collection',
        const='value-1-to-append',
        default=[],
        help='Add different values to list')
parser.add_argument('-B', action='append_const', dest='const_collection',
        const='value-2-to-append',
        help='Add different values to list')

parser.add_argument('--version', action='version', version='%(prog)s 1.0')

results = parser.parse_args()
print 'simple_value     =', results.simple_value
print 'constant_value   =', results.constant_value
print 'boolean_switch   =', results.boolean_switch
print 'collection       =', results.collection
print 'const_collection =', results.const_collection
{% endhighlight %}

{% highlight bash%}
$ python argparse_action.py -h

usage: argparse_action.py [-h] [-s SIMPLE_VALUE] [-c] [-t] [-f]
                          [-a COLLECTION] [-A] [-B] [--version]

optional arguments:
  -h, --help       show this help message and exit
  -s SIMPLE_VALUE  Store a simple value
  -c               Store a constant value
  -t               Set a switch to true
  -f               Set a switch to false
  -a COLLECTION    Add repeated values to a list
  -A               Add different values to list
  -B               Add different values to list
  --version        show program's version number and exit

$ python argparse_action.py -s value

simple_value     = value
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -c

simple_value     = None
constant_value   = value-to-store
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -t

simple_value     = None
constant_value   = None
boolean_switch   = True
collection       = []
const_collection = []

$ python argparse_action.py -f

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -a one -a two -a three

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = ['one', 'two', 'three']
const_collection = []

$ python argparse_action.py -B -A

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = ['value-2-to-append', 'value-1-to-append']

$ python argparse_action.py --version

argparse_action.py 1.0
{% endhighlight %}

### 选项前缀

argparse选项的默认语法是基于Unix约定的，使用一个“-”前缀来表示命令行开关。argparse支持其他前缀，因此你可以使得你的程序遵照本地平台的默认语法（例如，在Window上使用“/”）或者遵循不同的约定。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(description='Change the option prefix charaters', 
        prefix_chars='-+/')

parser.add_argument('-a', action="store_false", default=None,
        help='Turn A off')

parser.add_argument('+a', action="store_true", default=None,
        help='Turn A on')

parser.add_argument('//noarg', '++noarg', action="store_true", default=False)

print parser.parse_args()
{% endhighlight %}

将**ArgumentParser** 方法的*prefix_chars* 参数设置为一个字符串，该字符串包含所有允许用来表示选项的字符。需要理解的是虽然*prefix_chars*包含允许用于开关的字符，但单个参数定义只能使用一种给定的开关语法。这让你可以对使用不同前缀的选项是否是别名（比如独立于平台的命令行语法的情况）或替代选择（例如，使用“+”表明打开一个开发，“-”则为关闭一个开关）进行显式地控制。在上述例子中，`+a`和`-a`是不同的参数，`//noarg` 也可以 `++noarg` 提供，但不是 `--noarg`。

{% highlight python %}
$ python argparse_prefix_chars.py -h

usage: argparse_prefix_chars.py [-h] [-a] [+a] [//noarg]

Change the option prefix characters

optional arguments
    -h, --help  show this help message and exit
    -a  Turn A off
    +a  Turn A on
    //noarg,++noarg

$ python argparse_prefix_chars.py +a

Namespace(a=True, noarg=False)

$ python argparse_prefix_chars.py -a

Namespace(a=False, noarg=False)

$ python argparse_prefix_chars.py //noarg

Namespace(a=None, noarg=True)

$ python argparse_prefix_chars.py ++noarg

Namespace(a=None, noarg=True)

$ python argparse_prefix_chars.py --noarg

usage: argparse_prefix_chars.py [-h] [-a] [+a] [//noarg]
argparse_prefix_chars.py: error: unrecognized arguments: --noarg
{% endhighlight %}

### 参数来源

目前为止所见的例子中，提供给解析器的参数列表来自于显式传递的一个列表，或隐式地从sys.argv获取的。显式传递列表在你使用argparse来处理类命令行但并不是来自命令行（比如来自一个配置文件）的指令之时比较有用。

{% highlight python %}
import argparse
from ConfigParser import ConfigParser
import shlex

parser = argparse.ArgumentParser(description='Short sample app')

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

config = ConfigParser()
config.read('argparse_witH_shlex.ini')
config_value = config.get('cli', 'options')
print 'Config: ', config_value

argument_list = shlex.split(config_value)
print 'Arg List:', argument_list

print 'Results:', parser.parse_args(argument_list)
{% endhighlight %}

shlex使得切分存储在配置文件中的字符串非常容易。

{% highlight bash %}
$ python argparse_with_shlex.py

Config: -a -b 2
Arg List: ['-a', '-b', '2']
Results: Namespace(a=True, b='2', c=None)
{% endhighlight %}

另一种自己处理配置文件的方法是使用*fromfile_prefix_chars*指定一个包含一组要待处理参数的输入文件来告诉argparse怎样识别参数。

{% highlight python %}
import argparse
from ConfigParser import ConfigParser
import shlex

parser = argparse.ArgumentParser(description='Short sample app',
        fromfile_prefix_chars='@'
        )

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

print parser.parse_args(['@argparse_fromfile_prefix_chars.txt'])
{% endhighlight %}

该示例代码在找到一个以*@*为前缀的参数时即停止往下读取，然后从以该参数命名的文件中查找更多的参数。例如，输入文件`argparse_fromfile_prefix_chars.txt`包含一系列参数，一行一个：

    -a
    -b
    2

那么处理该文件产生的输出为：

    $ python argparse_fromfile_prefix_chars.py

    Namespace(a=True, b='2', c=None)


### 自动生成选项

经过配置argparse会自动添加选项用来生成帮助信息以及为你的应用程序显示版本信息。

**ArgumentParser**的参数*add_help* 控制帮助信息相关的选项。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(add_help=True)

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

print parser.parse_args()
{% endhighlight %}

帮助选项（-h和--help）默认是添加的，但可以通过将*add_help*设置为false来禁用。

{% highlight python%}
import argparse

parser = argparse.ArgumentParser(add_help=False)

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

print parser.parse_args()
{% endhighlight %}

虽然`-h`和`--help`是事实上的请求帮助的标准选项名称，但一些应用或argparse的使用要么不需要提供帮助要么需要将这两个选项名称用于其他目标。

    $ python argparse_with_help.py -h

    usage: argparse_with_help.py [-h] [-a] [-b B] [-c C]

    optional arguments:
        -h, --help  show this help message and exit
        -a
        -b B
        -c C

    $ python argparse_without_help.py -h

    usage: argparse_without_help.py [-a] [-b B] [-c C]
    argparse_without_help.py: error: unrecognized arguments: -h

当在**ArgumentParser**构造方法设置*版本*后，就会添加版本选项（`-v`和`--version`）。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(version='1.0')

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)

print parser.parse_args()

print 'This is not printed'
{% endhighlight %}

两种形式的选项爱那个都会打印程序的版本字符串，然后立即退出程序。

    $ python argparse_with_version.py -h

    usage: argparse_with_version.py [-h] [-v] [-a] [-b B] [-c C]

    optional arguments:
        -h, --help  show this help message and exit
        -v, --version   show program's version number and exit
        -a
        -b B
        -c C

    $ python argparse_with_version.py -v

    1.0


### 解析器组

argparse包含若干特性用于组织你的参数解析器，使得实现更为简单，也能提高输出帮助信息的可用性。

**共享解析器规则**

我们常常需要实现一套命令行程序，这些程序都带一组参数，只是在某些方面有特殊化。例如，如果所有程序都需要在用户进行任何实际的操作之前对用户进行认证，那么它们就都需要支持`--user`和`--password`选项。你可以共享的选项来定义一个“父母”解析器，然后令单个程序的解析器从该“父母”解析器继承共享选项，这样就不必显式为每个**ArgumentParser**添加共享选项。

第一步是以共享的参数定义建立“父母”解析器。由于“父母”解析器的后代使用者会添加相同的帮助选项，从而会引发一个异常，所以在基础解析器中我们关闭自动帮助选项生成。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(add_help=False)

parser.add_argument('--user', action="store")
parser.add_argument('--password', action="store")
{% endhighlight %}

接下来，以*父母解析器*集创建另一个解析器：

{% highlight python %}
import argparse
import argparse_parent_base

parser = argparse.ArgumentParser(parents=[argparse_parent_base.parser])

parser.add_argument('--local-arg', action="store_true", default=False)

print parser.parse_args()
{% endhighlight %}

得到的程序带有三个选项：

    $ python argparse_uses_parent.py -h

    usage: argparse_uses_parent.py [-h] [--user USER] [--password PASSWORD]
                               [--local-arg]

    optional arguments:
        -h, --help           show this help message and exit
        --user USER
        --password PASSWORD
        --local-arg

**冲突的选项**

前一个例子指出以相同的参数名字为一个解析器添加两个参数处理器会引发一个异常。可以通过传递一个*conflict_handler*来改变冲突消除行为。argparse有两个内置的冲突处理器`error`（默认）和`resolve`，`resolve`会基于冲突选项的添加顺序来选择一个参数处理器。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(conflict_handler='resolve')

parser.add_argument('-a', action="store")
parser.add_argument('-b', action="store", help="Short alone")
parser.add_argument('--long-b', '-b', action="store", help="Long and short together")

print parser.parse_args(['-h'])
{% endhighlight %}

由于最后一个处理器所给定的参数名已被使用，那么本例中独立选项`-b`将被`--long-b`的别名所覆盖。

    $ python argparse_conflict_handler_resolve.py

    usage: argparse_conflict_handler_resolve.py [-h] [-a A] [--long-b LONG_B]

    optional arguments:
        -h, --help  show this help message and exit
        -a A
        --long-b LONG_B, -b LONG_B
                Long and short together

切换**add_argument()**的调用顺序就可以启用独立的选项：

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(conflict_handler='resolve')

parser.add_argument('-a', action="store")
parser.add_argument('--long-b', '-b', action="store", help='Long and short together')
parser.add_argument('-b', action="store", help='Short alone')

print parser.parse_args([-h])
{% endhighlight %}

现在两个选项可以一起使用了。

    $ python argparse_conflict_handler_resolve2.py

    usage: argparse_conflict_handler_resolve2.py [-h] [-a A] [--long-b LONG_B] [-b B]

    optional arguments:
        -h, --help  show this help message and exit
        -a A
        --long-b LONG_B Long and short together
        -b B    Short alone

**参数群组**

argparse能将参数定义组合成“群组”。默认情况下是使用两个群组，一个是选项的群组，另一个是必须的与位置相关的参数群组。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser(description='Short sample app')

parser.add_argument('--optional', action="store_true", default=False)
parser.add_argument('positional', action="store")

print parser.parse_args()
{% endhighlight %}

群组在输出的帮助信息中显示为分开的“与位置相关的参数”和“可选参数”两个部分：

    $ python argparse_default_grouping.py

    usage: argparse_default_grouping.py [-h] [--optional] positional

    Short sample app

    positional arguments:
        positional

    optional arguments:
        -h, --help  show this help message and exit
        --optional


你可以调整群组来提高帮助信息中群组的逻辑性，这样相关选项或值能记录在一起。可以使用自定义群组来重写之前的共享选项的示例，如此在帮助信息中身份认证的选项就可以显示在一起。

在基础解析器中使用**add_argument_group()**来创建一个“身份认证”群组，然后逐个添加身份认证相关的选项到该群组。

{% highlight python %}
import argparse

parser = argparser.ArgumentParser(add_help=False)

group = parser.add_argument_group('authentication')

group.add_argument('--user', action="store")
group.add_argument('--password', action="store")
{% endhighlight %}

与之前一样，程序使用基于群组的父母解析器列表作为*parents*的值。

{% highlight python %}
import argparse
import argparse_parent_with_group

parser = argparse.ArgumentParser(parents=[argparse_parent_with_group.parser])

parser.add_argument('--local-arg', action="store_true", default=False)

print parser.parse_args()
{% endhighlight %}

现在输出的帮助信息一起显示身份认证选项。

    $ python argparse_uses_parent_with_group.py -h

    usage: argparse_uses_parent_with_group.py [-h] [--user USER] [--password PASSWORD] [--local-arg]

    optional arguments:
        -h, --help  show this message and exit
        --local-arg

    authentication:
        --user USER
        --password PASSWORD

**互斥选项**

定义互斥的选项是选项分组特性的一个特例，使用**add_mutually_exclusive_group()**而不是**add_argument_group()**。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

group = parser.add_mutually_exclusive_group()
group.add_argument('-a', action='store_true')
group.add_argument('-b', action="store_true")

print parser.parse_args()
{% endhighlight %}

argparse会为你强制执行互斥性，因此一次使用仅能给出该群组的选项中的一个。

    $ python argparse_mutually_exclusive.py -h

    usage: argparse_mutually_exclusive.py [-h] [-a | -b]

    optional arguments:
        -h, --help  show this help message and exit
        -a
        -b

    $ python argparse_mutually_exclusive.py -a
    
    Namespace(a=True, b=False)

    $ python argparse_mutually_exclusive.py -b
    
    Namespace(a=False, b=True)

    $ python argparse_mutually_exclusive.py -a -b

    usage: argparse_mutually_exclusive.py [-h] [-a | -b]
    argparse_mutually_exclusive.py: error: argument -b: not allowed with argument -a


**嵌套解析器**

上述的父母解析器方式是在相关命令之间共享选项的方式之一。另一种方式是将多个命令组合进一个程序中，使用子解析器来处理命令行的每个部分。就像`svn`，`hg`，以及其他带有多个命令行行为或子命令的程序那样。

一个用于处理文件系统目录的程序可能会像这样定义命令用于创建、删除、以及列出一个目录的内容：

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

subparsers = parser.add_subparsers(help='commands')

# A list command
list_parser = subparsers.add_parser('list', help='List contents')
list_parser.add_argument('dirname', action='store', help='Directory to list')

# A create command
create_parser = subparsers.add_parser('create', help='Create a directory')
create_parser.add_argument('dirname', action='store', help='New directory to create')
create_parser.add_argument('--read-only', default=False, action='store_true',
        help='Set permissions to prevent writing to the directory')

# A delete command
delete_parser = subparsers.add_parser('delete', help='Remove a directory')
delete_parser.add_argument('dirname', action='store', help='The directory to remove')
delete_parser.add_argument('--recursive', '-r', default=False, action='store_true',
        help='Remove the contents of the directory, too')

print parser.parse_args()
{% endhighlight %}

输出的帮助信息显示作为“命令”的命名子解析器能够在命令行中作为位置参数进行指定。

    $ python argparse_subparsers.py -h

    usage: argparse_subparsers.py [-h] {list, create, delete} ...

    positional arguments:
        {list, create, delete} commands
            list    List contents
            create  Create a directory
            delete  Remove a directory

    optional arguments:
        -h, --help  show this help message and exit


每个子解析器也有自己的帮助信息，描述那个命令的参数和选项。

    $ python argparse_subparsers.py create -h

    usage: argparse_subparsers.py create [-h] [--read-only] dirname

    positional arguments:
        dirname New directory to create

    optional arguments:
        -h, --help  show this help message and exit
        --read-only Set permissions to prevent writing to the directory

参数被解析后，**parse_args()**返回的**Namespace**对象仅包含与指定的命令相关的值。

    $ python argparse_subparsers.py delete -r foo

    Namespace(dirname='foo', recursive=True)


### 高级参数处理

至今为止的示例展示了简单的布尔标识、字符串或数字参数选项、以及位置参数。对于变长参数列表、枚举类型数据、以及常量，argparse支持复杂的参数规格。

**可变形参列表**

你可以配置单个参数的定义使其能够匹配所解析的命令行的多个参数。根据需要或期望的参数个数，设置*nargs*为这些标识值之一：

    值  含义
    N   参数的绝对个数（例如：3）
    ?   0或1个参数
    *   0或所有参数
    +   所有，并且至少一个参数

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('--three', nargs=3)
parser.add_argument('--optional', nargs='?')
parser.add_argument('--all', nargs='*', dest='all')
parser.add_argument('--one-or-more', nargs='+')

print parser.parse_args()
{% endhighlight %}

解析器强制执行参数计数指令，生成一个精确的语法图作为命令帮助文本的一部分。

    $ python argparse_nargs.py -h

    usage: argparse_nargs.py [-h] [--three THREE THREE THREE]
                         [--optional [OPTIONAL]] [--all [ALL [ALL ...]]]
                         [--one-or-more ONE_OR_MORE [ONE_OR_MORE ...]]

    optional arguments:
    -h, --help            show this help message and exit
    --three THREE THREE THREE
    --optional [OPTIONAL]
    --all [ALL [ALL ...]]
    --one-or-more ONE_OR_MORE [ONE_OR_MORE ...]

    $ python argparse_nargs.py

    Namespace(all=None, one_or_more=None, optional=None, three=None)

    $ python argparse_nargs.py --three

    usage: argparse_nargs.py [-h] [--three THREE THREE THREE]
                         [--optional [OPTIONAL]] [--all [ALL [ALL ...]]]
                         [--one-or-more ONE_OR_MORE [ONE_OR_MORE ...]]
    argparse_nargs.py: error: argument --three: expected 3 argument(s)

    $ python argparse_nargs.py --three a b c

    Namespace(all=None, one_or_more=None, optional=None, three=['a', 'b', 'c'])

    $ python argparse_nargs.py --optional

    Namespace(all=None, one_or_more=None, optional=None, three=None)

    $ python argparse_nargs.py --optional with_value

    Namespace(all=None, one_or_more=None, optional='with_value', three=None)

    $ python argparse_nargs.py --all with multiple values

    Namespace(all=['with', 'multiple', 'values'], one_or_more=None, optional=None, three=None)

    $ python argparse_nargs.py --one-or-more with_value

    Namespace(all=None, one_or_more=['with_value'], optional=None, three=None)

    $ python argparse_nargs.py --one-or-more with multiple values

    Namespace(all=None, one_or_more=['with', 'multiple', 'values'], optional=None, three=None)

    $ python argparse_nargs.py --one-or-more

    usage: argparse_nargs.py [-h] [--three THREE THREE THREE]
                         [--optional [OPTIONAL]] [--all [ALL [ALL ...]]]
                         [--one-or-more ONE_OR_MORE [ONE_OR_MORE ...]]
    argparse_nargs.py: error: argument --one-or-more: expected at least one argument


**参数类型**

argparse将所有参数值都看作是字符串，除非你告诉它将字符串转换成另一种数据类型。**add_argument()**的*type*参数以一个转换函数作为值，被**ArgumentParser**用来将参数值从一个字符串转换成另一种数据类型。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('-i', type=int)
parser.add_argument('-f', type=float)
parser.add_argument('--file', type=file)

try:
    print parser.parse_args()
except IOError, msg:
    parser.error(str(msg))
{% endhighlight %}

任何需要单个字符串参数的可调用对象都可以传递给*type*，包含内置类型如**int()**, **float()**, 以及**file()**。

    $ python argparse_type.py -i 1

    Namespace(f=None, file=None, i=1)

    $ python argparse_type.py -f 3.14

    Namespace(f=3.14, file=None, i=None)

    $ python argparse_type.py --file argparse_type.py

    Namespace(f=None, file=<open file 'argparse_type.py', mode 'r' at 0x1004de270>, i=None)


如果类型转换失败，argparse会引发一个异常。*TypeError*和*ValueError*会被自动捕获，并为用户转换为一个简单的错误消息。其他异常，如下面一个例子中输入文件不存在，则其*IOError*必须由调用者来处理。

    $ python argparse_type.py -i a

    usage: argparse_type.py [-h] [-i I] [-f F] [--file FILE]
    argparse_type.py: error: argument -i: invalid int value: 'a'

    $ python argparse_type.py -f 3.14.15

    usage: argparse_type.py [-h] [-i I] [-f F] [--file FILE]
    argparse_type.py: error: argument -f: invalid float value: '3.14.15'

    $ python argparse_type.py --file does_not_exist.txt

    usage: argparse_type.py [-h] [-i I] [-f F] [--file FILE]
    argparse_type.py: error: [Errno 2] No such file or directory: 'does_not_exist.txt'

要想将一个输入参数限制为一个预定义集中的某个值，则使用*choices*参数。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('--mode', choices=('read-only', 'read-write'))

print parser.parse_args()
{% endhighlight %}

如果`--mode`的参数值不是所允许的值中的一个，就会产生一个错误并停止执行。

    $ python argparse_choices.py -h

    usage: argparse_choices.py [-h] [--mode {read-only,read-write}]

    optional arguments:
    -h, --help            show this help message and exit
    --mode {read-only,read-write}

    $ python argparse_choices.py --mode read-only

    Namespace(mode='read-only')

    $ python argparse_choices.py --mode invalid

    usage: argparse_choices.py [-h] [--mode {read-only,read-write}]
    argparse_choices.py: error: argument --mode: invalid choice: 'invalid'
    (choose from 'read-only', 'read-write')


**文件参数**

虽然**文件**对象可以单个字符串参数值来实例化，但并不允许你指定访问模式。**FileType**让你能够更加灵活地指定某个参数应该是个文件，包括其访问模式和缓冲区大小。

{% highlight python %}
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('-i', metavar='in-file', type=argparse.FileType('rt'))
parser.add_argument('-o', metavar='out-file', type=argparse.FileType('wt'))

try:
    results = parser.parse_args()
    print 'Input file:', results.i
    print 'Output file:', results.o
except IOError, msg:
    parser.error(str(msg))
{% endhighlight %}

上例中与参数名关联的值是一个打开文件句柄。在使用完该文件后应自己负责关闭该文件。

    $ python argparse_FileType.py -h

    usage: argparse_FileType.py [-h] [-i in-file] [-o out-file]

    optional arguments:
    -h, --help   show this help message and exit
    -i in-file
    -o out-file

    $ python argparse_FileType.py -i argparse_FileType.py -o temporary_file.\
    txt

    Input file: <open file 'argparse_FileType.py', mode 'rt' at 0x1004de270>
    Output file: <open file 'temporary_file.txt', mode 'wt' at 0x1004de300>

    $ python argparse_FileType.py -i no_such_file.txt

    usage: argparse_FileType.py [-h] [-i in-file] [-o out-file]
    argparse_FileType.py: error: argument -i: can't open 'no_such_file.txt': [Errno 2] No such file or directory: 'no_such_file.txt'


**自定义动作**

除了前面描述的内置动作之外，你也可以提供一个实现了Action API的对象来自定义动作。作为*action*传递给**add_argument()**的对象应接受描述所定义形参的实参，并返回一个可调用对象，作为*parser*的实参来处理形参，*namespace*存放解析的结果、参数值，以及触发动作的*option_string*。

argparse提供了一个**Action**类作为要定义的新动作的基类。构造方法是处理参数定义的，所以你只要在子类中覆盖**__call__()**。

{% highlight python %}
import argparse

class CustomAction(argparse.Action):
    def __init__(self,
            option_strings,
            dest,
            nargs=None,
            const=None,
            default=None,
            type=None,
            choices=None,
            required=False,
            help=None,
            metavar=None):
        argparse.Action.__init__(self,
                option_strings=option_strings,
                dest=dest,
                nargs=nargs,
                const=const,
                default=default,
                type=type,
                choices=choices,
                required=required,
                help=help,
                metavar=metavar)
        print
        print 'Initializing CustomAction'
        for name,value in sorted(locals().items()):
            if name == 'self' or value is None:
                continue
            print '  %s = %r' % (name, value)
        return

    def __call__(self, parser, namespace, values, option_string=None):
        print
        print 'Processing CustomAction for "%s"' % self.dest
        print '  parser = %s' % id(parser)
        print '  values = %r' % values
        print '  option_string = %r' % option_string
        
        # Do some arbitrary processing of the input values
        if isinstance(values, list):
            values = [ v.upper() for v in values ]
        else:
            values = values.upper()
        # Save the results in the namespace using the destination
        # variable given to our constructor.
        setattr(namespace, self.dest, values)

parser = argparse.ArgumentParser()

parser.add_argument('-a', action=CustomAction)
parser.add_argument('-m', nargs='*', action=CustomAction)
parser.add_argument('positional', action=CustomAction)

results = parser.parse_args(['-a', 'value', '-m' 'multi-value', 'positional-value'])
print
print results
{% endhighlight %}

*values*的类型取决于*nargs*的值。如果该参数允许多个值，则*values*会是一个列表，即使其仅包含一个列表项。

*option_string*的值也取决于原有的参数规范。对于位置相关的、必需的参数，*option_string*始终为**None**。

    $ python argparse_custom_action.py

    Initializing CustomAction
        dest = 'a'
        option_strings = ['-a']
        required = False

    Initializing CustomAction
        dest = 'm'
        nargs = '*'
        option_strings = ['-m']
        required = False

    Initializing CustomAction
        dest = 'positional'
        option_strings = []
        required = True

    Processing CustomAction for "a"
        parser = 4299616464
        values = 'value'
        option_string = '-a'

    Processing CustomAction for "m"
        parser = 4299616464
        values = ['multi-value']
        option_string = '-m'

    Processing CustomAction for "positional"
        parser = 4299616464
        values = 'positional-value'
        option_string = None

    Namespace(a='VALUE', m=['MULTI-VALUE'], positional='POSITIONAL-VALUE')
