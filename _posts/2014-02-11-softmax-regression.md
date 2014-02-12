---
title: Softmax Regression
layout: post
tags: machine-learning mlalgo
category: machine-learning
comments: true
---
{% include JB/setup %}

###二元分类

逻辑回归是一种二元分类算法，它的输出是
$$y^{(i)}\in\{0,1\}$$，也就是计算样本等于`1`的概率（$$ P(y^{(i)} = 1 | x^{(i)} ; \theta) $$），而样本等于`0`的概率就是$$1 - P(y^{(i)} = 1 | x^{(i)} ; \theta)$$。类似的应用形式包括：垃圾邮件检测（spam or non-spam），肿瘤检测（良性 or 恶性）

###多元分类
那如果存在多个分类，将如何应用逻辑回归呢？比如存在$$K$$个分类，$$y^{(i)}\in\{1, 2, \ldots, K\}$$。这时有两个办法：

* one-vs-all，即训练 $$K$$ 个二元分类器，分别计算样本是 $$k$$ 的概率，最后取其中最大的那个值作为结果。
* softmax regression，Softmax regression是逻辑回归(Logistic regression)的一般化形式。它也是计算样本等于k的概率，但是这K个概率总和等于1，这点不同于one-vs-all的方法。

softmax regression的Cost function是：

\\[\begin{aligned} 
h\_\theta(x) = \begin{bmatrix} \\\
P(y = 1 | x; \theta) \\\ P(y = 2 | x; \theta) \\\ \vdots \\\ P(y = K | x; \theta) 
\end{bmatrix} = \frac{1}{ \sum_{j=1}^{K}{\exp(\theta^{(j)\top} x) }} \begin{bmatrix} \exp(\theta^{(1)\top} x ) \\\ \exp(\theta^{(2)\top} x ) \\\ \vdots \\\ \exp(\theta^{(K)\top} x ) \\\ \end{bmatrix} 
\end{aligned}\\]

\\[\begin{aligned}
J(\theta) = \- \left[ \sum\_\{i=1\}^\{m\} \sum\_\{k=1\}^{K} 1\left\\{y^\{(i)\} = k\right\\}\log\frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})} \right] \end{aligned}\\]

Softmax regression最后得到$$K$$个参数列表，即$$\Theta$$是个$$N * K$$的矩阵，$$N$$是Feature的数量。

###Matlab实现
    function [f,g] = softmax_regression(theta, X,y)
      %
      % Arguments:
      %   theta - A vector containing the parameter values to optimize.
      %       In minFunc, theta is reshaped to a long vector.  So we need to
      %       resize it to an n-by-(num_classes-1) matrix.
      %       Recall that we assume theta(:,num_classes) = 0.
      %
      %   X - The examples stored in a matrix.  
      %       X(i,j) is the i'th coordinate of the j'th example.
      %   y - The label for each example.  y(j) is the j'th example's label.
      %
      m=size(X,2);
      n=size(X,1);

      % theta is a vector;  need to reshape to n x num_classes.
      theta=reshape(theta, n, []);
      num_classes=size(theta,2)+1;

      % initialize objective value and gradient.
      f = 0;
      g = zeros(size(theta));

      %
      % TODO:  Compute the softmax objective function and gradient using vectorized code.
      %        Store the objective function value in 'f', and the gradient in 'g'.
      %        Before returning g, make sure you form it back into a vector with g=g(:);
      %
      %%% YOUR CODE HERE %%%

      %ny = full(sparse(y, 1:m, 1))

      theta = [theta, zeros(n, 1)];

      %M = bsxfun(@minus, theta' * X, max(theta' * X, [], 1));
      M = theta' * X;
      h = bsxfun(@rdivide, exp(M), sum(exp(M)));
      h1 = h';
      I = sub2ind(size(h1), 1:size(h1, 1), y);
      f = (-1) * sum(log(h1(I)));

      p = eye(m, 10);
      ny = p(y',:);
      g = (-1) * (X * (ny - h1));

      g = g(:, 1:num_classes-1);
      g=g(:); % make gradient a vector for minFunc
    end

在MINST这个手写数字数据集中，Softmax的Test accuracy大约是92.1%，属于underfitting（欠拟合）.

###One-vs-all和softmax regression
在构建一个多元分类器时，如何选择算法，究竟是使用one-vs-all，还是使用softmax regression呢？这取决于你的分类是否互斥。比如进行手写数字识别，那么就应该使用softmax regression，因为一个手写数字只能是10个数字中一个确定的值，不可能同时是多个值。但如果进行图片分类，那么就可以使用one-vs-all，一张图片可以是包含人物的照片，也可以是室内照片等等。

更多关于Softmax regression的介绍

* [http://ufldl.stanford.edu/tutorial/index.php/Softmax_Regression](http://ufldl.stanford.edu/tutorial/index.php/Softmax_Regression)
