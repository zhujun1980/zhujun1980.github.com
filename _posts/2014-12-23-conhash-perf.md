---
title: 一致性哈希的碰撞问题
layout: post
tags: 算法
category: 算法
comments: true
---
{% include JB/setup %}

最近在做一项线上服务的优化：一组使用一致性哈希进行散列的资源，请求分配的非常不均匀，不同资源之间的负载差别达到一倍以上。对此我写了个测试程序来检测一致性哈希的散列情况，[代码在这里](https://gist.github.com/zhujun1980/7e907705ab61a3e71a7b)，一致性哈希算法使用[库libconhash](http://www.codeproject.com/Articles/56138/Consistent-hashing)，它使用red-black tree进行存储。

进行1亿次随机数查询，一致性哈希节点node数量由size变量进行控制，虚拟节点vnode数量由conhash_set_node的第三个参数replicas控制，大致的测试结果如下

##size = 5，replicas = 16

~~~
0: 28633779 28.633779
1: 18736762 18.736762
2: 21437812 21.437812
3: 22081047 22.081047
4: 9110600 9.110600

real	0m56.475s
user	0m56.423s
sys	0m0.043s
~~~
第一列是每个key所归属的node（加起来等于1亿），第二列是所占百分比。这个配置和我们线上的设置一样，可见分配的很不均匀，部分node相差3倍多。

##size = 5，replicas = 32
~~~
0: 20259653 20.259653
1: 25453902 25.453902
2: 22836106 22.836106
3: 17200515 17.200515
4: 14249824 14.249824

real	0m57.140s
user	0m57.123s
sys	0m0.008s
~~~
这会稍微好点，也还是不太均匀。

##size = 5，replicas = 64
~~~
0: 27168351 27.168351
1: 17449142 17.449142
2: 18628693 18.628693
3: 18401042 18.401042
4: 18352772 18.352772

real	0m58.419s
user	0m58.401s
sys	0m0.008s
~~~
除了第一个，其他4个基本平衡了

##size = 5，replicas = 128
~~~
0: 21857353 21.857353
1: 15865343 15.865343
2: 21036951 21.036951
3: 20663761 20.663761
4: 20576592 20.576592

real	0m59.991s
user	0m59.973s
sys	0m0.008s
~~~
除了第二个，其他基本接近均值

##size = 5，replicas = 1024
~~~
0: 20320144 20.320144
1: 20437531 20.437531
2: 20285847 20.285847
3: 19820752 19.820752
4: 19135726 19.135726

real	1m6.901s
user	1m6.872s
sys	0m0.018s
~~~
基本平衡了，另外查询时间也有小幅提高，但考虑到进行了1亿次循环，其实对于单次查询基本上没有变化，这主要得益于red-black tree的高性能。另外使用的内存也有所提高（文中没有列出，但是通过pmap可以监测到），毕竟vnode数量增加了64倍（相比16个），不过在可以接受的范围。

##size的调整
size的调整也具有同样的效果，而且在应用中，资源往往是固定的和稀缺的，所以就不在列出了。

##hash算法的影响

libconhash默认使用md5进行hash，实际上它仅使用了md5值的前4个字节，同时libconhash也支持用户自定义hash函数，我使用了[murmur_hash3](https://code.google.com/p/smhasher/)：

##size = 5，replicas = 16，murmur_hash3
~~~
0: 18363319 18.363319
1: 19885604 19.885604
2: 19289442 19.289442
3: 20613835 20.613835
4: 21847800 21.847800

real	0m32.383s
user	0m32.349s
sys	0m0.029s
~~~
__*在仅有5个node和16个vnode的情况下，分配情况比md5的128个vnode还要好，而且查询时间缩短了一半！*__

不过这个算法也不是没有缺点，它的性能取决于seed的选择，我测试了一些数字（基本上都是素数），发现`0x7293AF`这个值比较好，而其他的数或者当前时间戳效果还不如md5好。

#结论

影响一致性哈希散列效果的因素主要有：

* 节点node的数量
* 虚拟节点vnode的数量
* 哈希算法的质量

和查询的key关系不大（前提是key足够的随机）