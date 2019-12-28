# Intro

Loss function



Loss Function 是一个函数用来计算预测标签与真实标签的差距 $L: y \times y^{\prime} \rightarrow \mathbb{R}_{+}$ 。

常见的Loss function

1. $\{-1,+1\} \times\{-1,+1\}$ , $L\left(y, y^{\prime}\right)=1_{y^{\prime} \neq y}$ 
2. $L\left(y, y^{\prime}\right)=\left(y^{\prime}-y\right)^{2}$ 作用域是 $J\times J$ , $J$ 是 $\mathbb R$ 的一个子集。

假设：将特征(特征向量)映射到标签Y的机器学习第1页的基础集的一组函数。在我们的示例中，这些函数可以是将电子邮件特征映射到Y = {垃圾邮件，非垃圾邮件}的一组函数。更一般地，假设可以是将特征映射到不同集合Y’的函数。它们可以是将电子邮件特征向量映射到解释为分数(Y′= R)的实数的线性函数，较高的分数值比较低的分数值更能指示垃圾邮件。
$$
\begin{aligned}\left|R(h)-\widehat{R}_{S}(h)\right| &=|\underset{(x, y) \sim \mathcal{D}}{\mathbb{E}}[L(h(x), y)]-\underset{(x, y) \sim \mathcal{D}}{\mathbb{E}}[L(h(x), y)]| \\ &=\left|\int_{0}^{M}(\underset{(x, y) \sim \mathcal{D}}{\mathbb{P}}[L(h(x), y)>t]-\underset{(x, y) \sim \mathcal{D}}{\mathbb{P}}[L(h(x), y)>t]) d t\right| \\ & \leq\left.M \sup _{t \in[0, M]}\right|_{(x, y) \sim \mathcal{D}}[L(h(x), y)>t]-\underset{(x, y) \sim \mathcal{D}}{\mathbb{P}}[L(h(x), y)>t] | \\ &=M \sup _{t \in[0, M]}\left|R(c(h, t))-\widehat{R}_{S}(c(h, t))\right| \end{aligned}
$$
