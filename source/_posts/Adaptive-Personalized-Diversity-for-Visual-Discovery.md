---
title: 'Adaptive, Personalized Diversity for Visual Discovery'
date: 2018-02-05 13:25:09
tags:
- 机器学习
categories:
- 机器学习
---
本文是一篇论文笔记，Amzon 关于多样性的论文[Adaptive, Personalized Diversity for Visual Discovery](http://dl.acm.org/citation.cfm?id=2959171)，分为： 基于贝叶斯回归模型的评分模型 ，基于主题的多样性模型子模块，基于用户的个性化多样性模型. 未完待续
<!-- more -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>


这篇文章来自 [洪亮勋博客分享](http://weibo.com/ttarticle/p/show?id=2309404022330610299674),[Adaptive, Personalized Diversity for Visual Discovery](http://dl.acm.org/citation.cfm?id=2959171)文章为RecSys 2016的文章。主要解决的问题：

- 基于贝叶斯回归模型的评分模型
- 基于主题的多样性模型子模块
- 基于用户的个性化多样性模型

![](https://ww1.sinaimg.cn/large/be5dc787gw1f81r06spqrj20px0lkjvp.jpg)

## 基于贝叶斯回归模型的评分模型
本文使用了Bayesian Linear Probit Regression来做最基础的Click Modeling，具体公式：

$$ P(click|item is viewed) = f(\sum_{x_i is active }{x_i}) $$


本文利用Thompson sampling [an empirical evaluation of thompson sampling](http://www.research.rutgers.edu/~lihong/pub/Chapelle12Empirical.pdf)，摘录了具体的实现

### Regularized logistic regression with batch updates

Require Regualization parameter $\lambda>0$

 - $m_i=0,q_i=\lambda$,{Each wi has an independent prior $N(m_i,q^{-1}_i)$}
 - for t = 1,...T do
   - get new batch of training data ($x_j,y_j$),j=1,..n
   - find w as the minimizer of :`$\frac{1}{2} \sum^d_{i=1}{q_i(w_i-m_i)^2} +  \sum^n_{j=1}{log(1+exp(-y_jw^Tx_j))}$
   - $m_i=w_i$
   - $q_i=q_i + \sum^n_{j=1}{x^2_{ij}p_j(1-p_j)},p_j=(1+exp(-w^Tx_j))^{-1}${Laplace approximation}
 - end for
 - 
 
## 基于主题的多样性模型子模块
submodelar 资料[satnford cs](http://theory.stanford.edu/~jvondrak/data/submod-tutorial-1.pdf),详细的sumbmodular原理没有看懂，有时间再了解。

本文的方法，定义$A:={a_1,..,a_n}$为推荐n个item的一些属性（主题）集合。其中属性集合为one-hot编码$a_i\in\{0,1\}^d$.假设我们从n个集合中选取k个子集。
$$
p(A_k,w)=<w,log(1+ \sum_{a_i\in{A_k}})> + \sum_{a_i\in{A_k}}{s(a_i)}
$$
其中，$s(a_i)$是item a的点击率预测（表征属性的质量或点击的概率），

有了以上的定义，选取对应最大的集合：
$$
A^*_k := argmax_{A_k\in{A},|A_k|=k}{p(A_k,w)}
$$

$$
A_0:=0 and A_{i+1} := A_i \cup\{argmax_{a\in{A/A_i}}{p((A_i \cap {a},w))} \}
$$

## 基于用户的个性化多样性模型 
### learning adaptive global weights
文章对比了logistic regression、clicks over expected clicks,click-thru-rate with additive smoothing三种方法，实验效果差不多。所以试验中使用的基于平滑的CTR。

$$
w_j = \frac{c_j+\alpha}{v_j+\alpha+\beta}
$$
其中，$c_j,v_j$分别为类别j的点击和曝光量。

### user specific weights
#### user modeling
$$
w_u = (c_u + \alpha_0)||c_u+\alpha_0||^{-1}_1
$$
其中，$c_u=\{c_{u1},..,c_{ud}\}$为用户u在不同主题上的点击数。$alpha_0$的设置（can be chosen to math pre-specified business rules or to highlight certain category preferences）,不是特别懂。
#### user click singnal diffusion
这里主要是利用先验知识，如喜欢看游戏的主题，也喜欢看军事主题。所以加入主题的相似度矩阵（类似cf的 item-cf）。
$$
w_u = M w_u ||Mw_u||^{-1}_1
$$
其中，$M_{ij} = \frac{cout(i,j)}{count(j)}$,cout(i,j)为同时看主题i,j的用户数 ，cout(j)为看主题j的用户数。M可以看做利用共现主题对w的平滑。

## 结论
本文有用的几个结论：
- 用户对于主题的多样性比较敏感。