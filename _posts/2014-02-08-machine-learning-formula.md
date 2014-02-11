---
title: Machine Learning中的数学公式
layout: post
tags: machine-learning
category: machine-learning
comments: true
---
{% include JB/setup %}

这些公式主要来自Coursera上的[Machine Learning](https://www.coursera.org/course/ml)课程，汇总在这里，以备查找。

#Linear Regression

###Cost function

\\[J(\theta) = \{1 \over 2m\}\sum\_\{i=1\}^m(h_\theta(x^\{(i)\}) - y ^\{(i)\}) ^ 2 \\]

$$h_\theta(x) = \theta^Tx = \theta_0x_0 + \theta_1 x_1 + \cdots + \theta_nx_n$$

###Regularized cost function
\\[J(\theta) = \{1 \over 2m\}\left(\sum\_\{i=1\}^m(h_\theta(x^\{(i)\}) - y ^\{(i)\}) ^ 2 \right) + \{\lambda \over 2m\}\sum\_\{j=1\}^n\theta_j^2\\]

###Gradient with regularization
\\[\begin{aligned}
{\partial J(\theta) \over \partial \theta_0} = {1 \over m}\sum\_\{i=1\}^m(h\_\theta(x^\{(i)\}) - y^\{(i)\})x_j^\{(i)\} \quad\quad \text{for $j = 0$}.\\\
{\partial J(\theta) \over \partial \theta_j} = \left( {1 \over m}\sum\_\{i=1\}^m(h\_\theta(x^\{(i)\}) - y^\{(i)\})x_j^\{(i)\} \right) + \{\lambda \over m\} \theta_j \quad\text{for $j \geq 1$}.
\end{aligned}\\]

###Normal equation

\\[\theta = (X^TX)^{-1}X^Ty\\]

`Normal equation`是计算回归参数 $$\theta$$ 的一种简便方法，它不像`Gradient Descent`那样需要不断的迭代来训练$$\theta$$，套用上面的公式就可以直接计算出$$\theta$$，比较容易实现。但是它在Feature很多的情况下（即`n`很大时）性能会非常差，不如`Gradient Descent`。

#Logistic Regression

###Cost function

\\[J(\theta) = -\{1 \over m\}\left[\sum\_\{i=1\}^my^\{(i)\}\log(h\_\theta(x^\{(i)\})) + (1 - y^\{(i)\})\log(1 - h\_\theta(x^\{(i)\}))\right] \\]

$$h_\theta(x) = g(\theta^Tx) = {1 \over 1 + e ^ {-\theta^Tx}}$$

###Regularized cost function

\\[J(\theta) = -\{1 \over m\}\left[\sum\_\{i=1\}^my^\{(i)\}\log(h\_\theta(x^\{(i)\})) + (1 - y^\{(i)\})\log\left(1 - h\_\theta(x^\{(i)\})\right)\right] + \{\lambda \over 2m\} \sum\_\{j=1\}^n\theta_j^2\\]

###Gradient

\\[ {\partial J(\theta) \over \partial \theta_j} = {1 \over m}\sum\_\{i=1\}^m(h\_\theta(x^\{(i)\}) - y^\{(i)\})x_j^\{(i)\}\\]

###Gradient with regularization

\\[\begin{aligned} 
{\partial J(\theta) \over \partial \theta_0} = {1 \over m}\sum\_\{i=1\}^m(h\_\theta(x^\{(i)\}) - y^\{(i)\})x_j^\{(i)\} \quad\quad \text{for $j = 0$}. \\\
{\partial J(\theta) \over \partial \theta_j} = \left( {1 \over m}\sum\_\{i=1\}^m(h\_\theta(x^\{(i)\}) - y^\{(i)\})x_j^\{(i)\} \right) + \{\lambda \over m\} \theta_j \quad\text{for $j \geq 1$}.
\end{aligned} \\]

#Neural network

###Cost function

\\[J(\theta) = \{1 \over m\}\sum\_\{i=1\}^m\sum\_\{k=1\}^K\left[-y_k^\{(i)\}\log((h\_\theta(x^\{(i)\}))_k) - (1 - y_k^\{(i)\})\log(1 - (h\_\theta(x^\{(i)\}))\_k)\right] \\]

###Regularized cost function

\\[J(\theta) = \{1 \over m\}\sum\_\{i=1\}^m\sum\_\{k=1\}^K\left[-y_k^\{(i)\}\log((h\_\theta(x^\{(i)\}))_k) - (1 - y_k^\{(i)\})\log(1 - (h\_\theta(x^\{(i)\}))\_k)\right] + {\lambda \over 2m}\sum\_\{l=1\}^\{L-1\}\sum\_\{i=1\}^\{sl\}\sum\_\{j=1\}^\{sl+1\}\left(\theta\_\{ji\}^\{(l)\} \right)^ 2 \\]

###Gradient with regularization

\\[\begin{aligned} 
\{\partial \over \partial \Theta\_\{ij\}^\{(l)\} \} J(\Theta) = \{1 \over m\}\Delta\_\{ij\}^\{(l)\} \quad\quad \text{for $j = 0$}. \\\
\{\partial \over \partial \Theta\_\{ij\}^\{(l)\} \} J(\Theta) = \{1 \over m\}\Delta\_\{ij\}^\{(l)\} + {\lambda \over m} \Theta\_\{ij\}^\{(l)\} \quad\quad \text{for $j \geq 1$}.
\end{aligned}\\]

\\[\begin{aligned} 
\Delta\_\{ij\}^\{(l)\} = \Delta\_\{ij\}^\{(l)\} + a_j^\{(l)\} \delta_j^\{(l+1)\}\\\
\delta^\{(L)\} = a^\{(L)\} - y^\{(i)\}\\\
a^\{(1)\} = x\\\
z^\{(l)\} = \Theta^\{(l-1)\}a^\{(l-1)\} \\\
a^\{(l)\} = g\left(z^\{(l)\}\right) \quad \left(\text\{add $a_0^\{(l)\}$\}\right)
\end{aligned}\\]

#Softmax Regression
###Hypothesis function

\\[\begin{aligned} 
h\_\theta(x) = \begin{bmatrix} \\\
P(y = 1 | x; \theta) \\\ P(y = 2 | x; \theta) \\\ \vdots \\\ P(y = K | x; \theta) 
\end{bmatrix} = \frac{1}{ \sum_{j=1}^{K}{\exp(\theta^{(j)\top} x) }} \begin{bmatrix} \exp(\theta^{(1)\top} x ) \\\ \exp(\theta^{(2)\top} x ) \\\ \vdots \\\ \exp(\theta^{(K)\top} x ) \\\ \end{bmatrix} 
\end{aligned}\\]

###Cost function

\\[\begin{aligned}
J(\theta) = \- \left[ \sum\_\{i=1\}^\{m\} \sum\_\{k=1\}^{K} 1\left\\{y^\{(i)\} = k\right\\}\log\frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})} \right] \end{aligned}\\]

###Gradient

\\[\begin{align} 
\nabla\_{\theta^{(k)}} J(\theta) = - \sum_{i=1}^{m}{ \left[ x^{(i)} \left( 1\\{ y^{(i)} = k\\} - P(y^{(i)} = k | x^{(i)}; \theta) \right) \right] } 
 \end{align} \\]

---
