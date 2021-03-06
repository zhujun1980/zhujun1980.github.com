---
title: 概率论
layout: post
tags: math machine-learning
category: math
comments: false

---
{% include JB/setup %}

数学讨论的是 __确定的__ 事情，而随机事件是结局 __不确定__ 的事情。概率论就是用数学对这些现象进行的研究，用数学的确定性来描述不确定的随机过程。

#古典概率

###样本空间
对于随机事件，虽然准确预测它的结局很困难，但是我们可以准确的描述这个事件，例如：

> 掷一个正常的质量均匀的骰子，存在着六个互不相关的可能结局，而且每种结局应该是等可能的。

从上面的例子可以看出来，在对随机事件进行研究的过程中，我们不是仅考虑实际发生的__结局__，而是考虑 __所有可能的结局__ ，我们把所有这些结局组成一个`样本空间`，记为$$\Omega$$。

> 掷骰子的样本空间：\\[\Omega = \left\\{ 1, 2, 3, 4, 5, 6 \right\\} \\]

###概率
对于样本空间中的每个点称为基本事件，每个基本事件我们都可以指定一个`概率`，它是随机事件发生次数非常大时样本空间中每个基本事件发生次数在其中所占的分数。随机事件所得的某一个结局记为$$\omega$$，当然$$\omega \in\Omega$$。

> 在骰子均匀的情况下，每个结局都是等可能的，即 $$P(1) = P(2) = \dots = P(6) = {1\over6}$$.

###事件
我们对样本空间内的一些特定结果感兴趣，把它们称为事件。事件是这个随机过程的结果，例如：

> 关于掷骰子的一个事件是：掷出一个奇数点。

同样我们对于事件发生的概率也感兴趣。事件的概率是使事件可以等可能发生的基本事件的数目与总数目的分数。

> 掷出一个奇数点的概率是：事件的发生的基本事件数目3($$\left\{ 1, 3, 5 \right\}$$)与总数目 6的比值，$${3 \over 6} = {1 \over 2} $$

#概率基本法则

###概率的特性

* 概率总是满足：\\[0 \le P(A) \le 1, P(\Omega) = 1, P(\Phi) = 0 \\]
* 对于两个互斥的事件 A 和 B (即$$A \cap B = \Phi$$)，我们有：其中任何一个事件发生的概率等于它们概率的和，即：\\[P(A \cup B) = P(A) + P(B)\\]
* 对于任意两个事件 A 和 B，则：\\[P(A \cup B) = P(A) + P(B) - P(AB)\\]
* $$P(\overline A)$$ 是 __A 不发生__ 的事件，则：\\[P(\overline A) = 1 - P(A)\\]
* 若$$A \subset B$$，则：\\[P(B - A) = P(B) - P(A), P(A) \le P(B)\\]

###条件概率

* 条件概率，A 在 B 已经发生的条件下的概率：\\[P(A\|B) = {P(A \cap B) \over P(B)}\\]$$P(A \cap B)$$是 A 和 B 同时发生的概率，现在 B 已经发生，则 A 发生的概率就是 $$P(A \cap B)$$ 在 $$P(B)$$ 中所占的比值。
* 相互独立的事件 A 和 B，则：\\[P(A) = P(A\|B), P(B) = P(B\|A)\\]
* 如果事件 A 的发生和事件 B 的发生与否没有关系，那么这两个事件被称为是 __相互独立__ 的，因此它们同时发生的概率等于它们概率的积，即：\\[P(A \cap B) = P(A)P(B)\\]

###全概率定律和贝叶斯公式
把样本空间划分为若干不相交的部分$$B_i$$，即： \\[B_1 \cup \dots \cup B_i = \Omega\\] \\[B_i \cap B_j = \Phi, i \ne j \\]，B称为对样本空间的一个 __划分__ 。

某个事件的概率可以通过与构成这个划分的各个部分的重叠程度来确定，给出 __全概率定律__ ，即：\\[P(A) = \sum\_\{i=1\}^nP(A \| B_i)P(B_i)\\]

__贝叶斯公式__ ，将条件概率公式和全概率公式结合起来，即：\\[P(B_i \| A) = {P(A \| B_i)P(B_i) \over \sum\_\{j=1\}^nP(A \| B_j)P(B_j)}\\]

#随机变量

###定义
__随机变量__ $$X$$ 是一个实值函数，它把样本空间$$\Omega$$中的元素映射到实数集合上，$$X : \Omega \to \mathbb R $$。

###概率质量函数(probability mass function或PMF)
离散型随机变量 $$X$$ 的一个特定输出 $$x$$ 的概率被称为 $$p(x)$$，则：\\[p(x) \equiv P(X = x) = P(A)\\]注意 $$x$$ 是个实数值，$$p(x)$$ 是它的概率，$$A$$ 是样本空间 $$\Omega$$ 中满足随机变量 $$X$$ 输出为 $$x$$ 的元素的集合，即样本空间中可能有多个元素被映射到同一个实数上，例如：

> 掷出一个奇数点，点数 $$\left\{1, 3, 5\right\}$$ 都被映射到 $$1$$，点数 $$\left\{2, 4, 6\right\}$$ 都被映射到 $$0$$。

函数$$p(x)$$被称为 __概率质量函数__ ，因为它为随机变量的每一个输出指定了一个概率的重量。一个随机变量在所有可能的输出$$x$$上的概率之和必定恰好为 1。

###概率密度函数(probability density function或PDF)
离散型随机变量至多有着与 $$\mathbb N$$ 同样多的输出，即虽然数量可能是无穷多个，但还是 __可数__ 的，可数的意思是通过一种方法可以把集合中的元素一个一个的列出来，而不会有遗漏；而连续型随机变量有着与 $$\mathbb R$$ 同样多的取值，而实数集合 $$\mathbb R$$ 是不可数的，就是我们没有办法列出所有的实数而不遗漏，因为任意两个实数之间都存在着无穷多个实数。因此在连续型随机变量中，追求一个特定值的概率是没有意义的，因为 1 被分配给不可数无穷多个点，这些点实际出现的概率必须是 $$0$$ ！对于连续型随机变量就要借助于微积分：我们不是去寻求单单一个输出的出现概率，而是寻求试验所产生的结局落在某个范围内（介于两个实数值之间）的概率，这是通过对 __概率密度函数$$\rho(x)$$__ 进行积分来完成的：\\[P(a \le X \le b) = \int\_\{a\}^b\rho(u)du\\]

概率密度函数的性质：

* 随机变量在整个实数轴上取值，那么：\\[\int\_\{-\infty\}^\infty \rho(u)du = 1\\]
* $$\rho(x)$$为非负函数。


#常见的概率分布

###二项分布

考虑只有两种结局的一项试验，在进行了很多次试验后，我们关心的是其中一个结局 __发生了多少次__ ，而不是 __什么时候发生__ ，这种随机变量的概率就符合 __二项分布__ 。例如：

> 掷硬币，正面的概率 $$p$$，反面的概率是 $$1 - p$$，在进行了 $$n$$ 次试验后，正面朝上出现的次数 $$X$$ 就是一个二项随机变量，记为： $$B(n, p)$$

$$X$$的取值范围是从 $$0$$ 到 $$n$$ 的整数集合。那么出现 $$r$$ 次的概率就是：\\[p(r) = {n \choose r} p^r(1-p)^{n-r}, {n \choose r} \equiv C_n^r = \{n! \over r! (n -r)!\}\\]其中$$C_n^r$$ 是从 $$n$$ 中选取 $$r$$ 个的方式数。它的解释是如果正面出现$$r$$次，那么反面就出现了$$n-r$$次，而每一次都是独立的，那么每一种情况的概率就是$$p^r(1-p)^{n-r}$$，一共有多少种情况呢，就是$${n \choose r}$$种，因为每种情况都是互斥的，所以出现$$r$$次的概率就是它们的和。

####贝努利分布

想象二项分布中，$$n = 1$$，那么随机变量$$X \in {0, 1}$$，我们说$$X$$服从 __Bernoulli__ 分布，Bernoulli可用于二元分类问题：\\[Ber(x \| \theta) = \left\\{\begin{array}{ll} \theta & \textrm\{if $x=1$\} \\\\\\
 1-\theta & \textrm\{if $x=0$\} \\\\\\ \end{array} \right.\\]

####斯特林公式

斯特林公式是快速计算阶乘的一种方法，它可以让二项分布计算更快一些：\\[n! \approx \sqrt{2\pi n}n^n e^{-n}\\]

####二项分布的泊松近似

二项分布中我们关心的是成功次数的概率，但有的时候成功发生的非常的稀少，比如一大批货物中的次品率、数据传输中的错误率等等。这时可以找到一种对于二项分布的近似，它的计算量要小的多：\\[B(r; n, p) \approx e^{-\lambda}{\lambda ^r \over r!} \equiv P(r; \lambda), \lambda = np\\]它的证明使用了一些数学技巧，利用了 n 很大，而 p 很小这个事实，忽略一些高阶项，但它对二项分布近似的非常好，而且对于给定的 $$r$$ 泊松近似计算起来比二项分布容易多了。

###多项式分布

想象扔一个$$K$$面的骰子，共掷了 $$n$$ 次，令$$x$$为一个随机向量，$$x = \left \{ x_1, \dots x_K\right \}$$，其中$$x_j$$是第 $$j$$ 面出现的次数，显然n为随机向量各项的和，那么：\\[Mu(x \| n, \theta) = \triangleq {n \choose x_1 \dots x_K} \prod\_{j=1}^K \theta\_j^{x\_j} \\] $$\theta_j$$是$$j$$面的概率，为多项式分布，而\\[ {n \choose x\_1 \dots x\_K} \triangleq {n! \over x\_1! x\_2! \dots x\_K!}\\]为把 $$n$$ 划分到随机向量 $$x_1 \dots x_K$$ 的不同方式数。

> 比如某一面出现了 $$m$$次，那么它可能出现 $$n$$ 次试验的任何位置，最后只要加起来等于 $$m$$ 就是符合要求的，当然其他的面加起来要满足了 $$n - m$$ 次。

同样n为1时，随机向量 x 服从 multinoulli distribution ，x由0和1，其中只有一位为 1，其余都为 0 ，1 就是正面朝上的那个面。可用于多元分类。

> K = 3，随机变量编码为 {1, 0, 0}, {0, 1, 0}, {0, 0, 1}

###经验分布(empirical distribution)
存在一个数据集合 $$D = \left\{x_1 \dots x_N \right\}$$，我们定义经验分布：\\[p\_{emp}(A) \triangleq {1 \over N} \sum\_{i=1}^N \delta_{x_i}(A)\\] $$ \delta\_{x_i}(A)$$是 __Dirac measure__，\\[\delta\_{x_i}(A) = \left \\{\begin{array}{ll} 0 & \{if x \notin A\} \\\\\\
1 & \{if x \in A\} \end{array}\right.\\]这个定义是比较直白的，数据集合中的各项数据在集合A中数量与总数据项的比值。有时为每一个数据定义一个权重$$w_j$$，$$0 \le w_j \le 1, \sum\_{i=1}^N w_j = 1$$，\\[p(x) = \sum\_{i = 1}^N w_j \delta\_{x_i}(x)\\]对于没有出现在数据集合的任何点，经验概率认为是0，没有相关经验，可能这也是经验概率得名的原因吧。

###泊松分布

###正态分布

#期望与方差

###平均数与期望

###离散程度与方差

#概率论三大定理

###切比雪夫不等式

####标准差

#####标准化随机变量

###大数定律

####蒙特卡罗积分

###中心极限定理

