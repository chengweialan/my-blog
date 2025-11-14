---
title: Score Matching中一个简单的数学推导
published: 2025-11-14
description: '论文中一个有趣的数学推导'
image: './images/GenerativeModeling/GenerativeModeling.png'
tags: ["人工智能", "Diffusion", "数学"]
category: '人工智能'
draft: false
lang: ''
series: '科研'
---


> 近期重新学习了一下Diffusion相关的基础知识，读到[Generative Modeling by Estimating Gradients of the Data Distribution](https://arxiv.org/pdf/1907.05600)这篇论文时注意到之前忽略的一个证明细节，具体问题可以见封面的图片。网上存在的一些证明，譬如在各个维度分别运算再求和，形式上不太统一、不太简洁。闲来无事当调度一下自己的数学思维，基于个人习惯写了一份证明。之前学过一段时间力学，对于向量尤其是导数运算习惯用爱因斯坦求和记号，我认为形式比较美观，且不容易犯错。

Score Matching的原始目标是最小化模型分数 $s_\theta(x)$ 与真实分数 $s(x) = \nabla_x \log p(x)$ 之间的均方误差（MSE）：

$$
 \frac{1}{2} \mathbb{E}_{p(x)} \left[ \| s_\theta(x) - s(x) \|^2_2 \right]
$$

其中，$p(x) = p_{data}(x)$。之前学过一些力学，我习惯使用爱因斯坦求和约定，姑且不在意上下标位置（方便运算），展开上述公式如下：

$$
 \frac{1}{2} \mathbb{E}_p \left[ s_{\theta,i} s_{\theta,i} - 2 s_{\theta,i} s_i + s_i s_i \right] = \frac{1}{2} \mathbb{E}_p [ \| s_\theta \|^2_2 ] - \mathbb{E}_p [ s_{\theta,i} s_i ] + \frac{1}{2} \mathbb{E}_p [ \| s \|^2_2 ]
$$

最后一项是关于 $\theta$ 的常数，可忽略。

交叉项为

$$
\mathbb{E}_p [ s_{\theta,i} s_i ] = \int p(x) \, s_{\theta,i}(x) \, (\partial_i \log p(x)) \, dx = \int s_{\theta,i} \,\partial_i p \, dx,
$$




其中 $s_i = \partial_i \log p = (\partial_i p)/p$，故 $p \cdot s_i = \partial_i p$。




构建一个向量场 $\mathbf{F}(x) = p(x) s_\theta(x)$，其散度为 $\nabla \cdot \mathbf{F} = \partial_i F_i = \partial_i (p s_{\theta,i})$。

根据乘积法则（product rule），
$$
\partial_i (p s_{\theta,i}) = p \partial_i s_{\theta,i} + s_{\theta,i} \partial_i p.
$$
因此，整个空间的积分形式为
$$
\int_{\mathbb{R}^d} \nabla \cdot \mathbf{F}(x) \, dx = \int_{\mathbb{R}^d} p \partial_i s_{\theta,i} \, dx + \int_{\mathbb{R}^d} s_{\theta,i} \partial_i p \, dx.
$$
高斯公式告诉我们，$\int \nabla \cdot \mathbf{F} \, dx$ 等于边界积分 $\int_{\partial \Omega} \mathbf{F} \cdot \mathbf{n} \, dS$。

概率密度函数积分为1，可以认为在无穷远衰减足够快。虽然单纯从数学角度讲，无穷远处可以存在有理数个点取值不为0，L2积分仍然是0；但在我们的问题中，我们有理由认为样本的分布不存在这种情况。$s_\theta(x)$ 的增长不超过线性，边界项为零：

$$
\int_{\mathbb{R}^d} \nabla \cdot \mathbf{F}(x) \, dx = 0.
$$
于是，
$$
\int s_{\theta,i} \partial_i p \, dx = - \int p \partial_i s_{\theta,i} \, dx = - \int p \, \operatorname{tr}(\nabla_x s_\theta(x)) \, dx,
$$


其中 $\operatorname{tr}(\nabla_x s_\theta) = \operatorname{tr}(\partial_i s_{\theta,j}) = \partial_i s_{\theta,i}$。因此，交叉项为


$$
\mathbb{E}_p [ s_{\theta,i} s_i ] = - \mathbb{E}_p [ \operatorname{tr}(\nabla_x s_\theta) ].
$$

代回损失函数（忽略常数），得优化目标，
$$
 \mathbb{E}_p \left[ \frac{1}{2} \| s_\theta(x) \|^2_2 + \operatorname{tr}(\nabla_x s_\theta(x)) \right].
$$
这个等价形式无需知道真实的 $s(x)$。
