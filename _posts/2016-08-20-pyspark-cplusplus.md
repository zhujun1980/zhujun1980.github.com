---
title: 如何在PySpark中调用C/C++代码
layout: post
tags: Spark C/C++
category: Spark
comments: true
---
{% include JB/setup %}

<img src="/assets/cplus_spark.jpg" alt="spark c++" style="width:600;height:300">

这几年Spark以其高性能和使用方便等诸多优点获得越来越多人的关注。从使用的角度来看，Spark的学习曲线要比Hadoop更加平缓，一个普通工程师用一到两周时间把官方的programming-guide过一遍就能干活，这其中主要归功于Spark对于Python的良好支持，毕竟相较于Scala、Java和R来说Python更容易上手。

大部分时间Python都能满足需求，但是有些时候我们需要Python和一些C/C++代码进行通信，比如：对Spark离线训练的模型进行在线预估，而在线系统出于性能的考虑，一般使用C++等静态语言开发，而我们又希望对于某些核心功能（比如Feature extraction），离线和在线使用同一套代码，最好用同一种语言编写，这样维护成本更低，从而也避免了因使用不同语言（比如Python和C++）而引入的bug。

关于Python和C++相互调用，我们有两种选择：

* 在线系统中的用C++调用Python；
* 在离线训练时Python调用C++。

因为在线系统对于性能比较敏感，所以第一种并不十分可行。这里我选择了第二种方法，在Python中调用C++，主要工作如下：

* 首先要准备好C++代码
* 把第一步的代码编译成so
* 在pyspark中引入这个so，调用其中的方法，获取输出结果

####第一步

就不多讲了，这是基础。。

####第二步

编译so有两种选择：

* 编译成普通的so，导出必须的函数。普通so可以用python的`ctype`包引入，ctype可以把python的数据结构转成C语言对应的内存格式，从而实现通讯。假如你的数据类型比较简单，或者直接使用一些开源软件编译成的so，可以试试这种方法，具体可参见python [ctype文档](https://docs.python.org/2/library/ctypes.html)。

* 把C++代码包装成标准的Python extension，这个的好处是只要导入了so文件，剩下的工作就像调用普通Python函数一样简单，而且可以直接传输Python数据机构(比如list/dict/tuple等)；它的缺点就是需要额外工作来开发扩展，关于如何编写Python扩展，网上教程很多，比如Python官网的[教程](https://docs.python.org/2/extending/extending.html)。另外还可以使用 libboost_python生成。

我在这里选择的就是第二种方法，编译成Python扩展。

__Tips__：因为so会依赖比较多，而运行环境可能又复杂不一致，所以我建议尽量做静态链接，比如libpython2.7.a，甚至libstdc++都可以做static linking，这样就避免了一些奇怪的版本依赖问题，当然代价是生成的文件也比较大，不过在当今普遍的大内存服务器里，每台机器占用十几MB也不算什么。

####第三步
第三步就比较tricky。

首先在spark-submit时把so文件作为附加文件提交到spark：

~~~
spark-submit \
--files /path/to/pyfeature.so \
other arguments... \
your_jobs.py
~~~

假如你的so还需要一些配置，也一同提交，只要用`,`分隔就行了

~~~
--files /path/to/pyfeature.so,pyconf.conf
~~~

下面我们使用python的`imp`包导入so，imp有个`load_dynamic`方法可以动态加载so

~~~
import imp
def loadLib(so_path):
    if 'pyfeature' not in sys.modules:
        featureLib = imp.load_dynamic("pyfeature", so_path)
        return featureLib
    return sys.modules["pyfeature"]
~~~

__注意__：这里使用sys.modules检查是否已经引入so，避免重复导入

最后就是如何获取这个so在各个node server的绝对路径，这也不难，只要使用pyspark的SparkFiles类：

~~~
from pyspark import SparkFiles
pyLib = loadLib(SparkFiles.get("pyfeature.so"))
pyLib.call_so_func()
~~~

OK，到这里，就基本上完成了。

__Tips 1__：起先我把so和其他需要的.py文件压缩为一个zip文件，然后用spark-submit的`--py-files`提交，但是出现`ImportError: No module named pyfeature`，无法找到so对应的package，而压缩包里的其他py却是可以import的，就so却无法导入。

__Tips 2__：如果把so放到python系统的`ld_dynload`目录，so应该会被python解释器自动加载，不过每次so变更时都要重新部署一次，当集群里服务器很多，或代码变更较频繁时，这种方法就不太适合了。

总结
本文介绍一些实用技巧，方便在pyspark中调用已有的C/C++代码。