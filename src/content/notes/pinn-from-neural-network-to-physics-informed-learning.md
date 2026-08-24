---
title: "PINN 学习笔记（一）：从神经网络基础到物理约束损失"
description: "整理我学习 Physics-Informed Neural Networks 时，从 MLP、损失函数、反向传播一路理解到 PDE、边界条件以及 PINN 总损失的过程。"
date: 2026-08-24
category: "AI & Scientific Computing"
tags: ["PINN", "Neural Network", "PDE", "PyTorch", "Scientific Computing"]
draft: false
---

最近开始系统学习 PINN（Physics-Informed Neural Networks，物理信息神经网络）。这篇先整理理解 PINN 所需要的神经网络和 PDE 基础，以及 PINN 最核心的一件事：**如何把物理方程变成神经网络训练时的约束。**

## 1. 从 MLP 开始

最基础的多层感知机可以理解为三类结构：输入层、一个或多个隐藏层、输出层。

每一层首先进行线性变换：

$$
\mathbf{z}=W\mathbf{x}+\mathbf{b}
$$

然后经过激活函数：

$$
\mathbf{a}=\sigma(\mathbf{z})
$$

因此一个神经网络最基本的计算过程就是：输入经过权重和偏置得到预激活值，再经过激活函数传给下一层，多层重复后得到最终输出。训练阶段则根据输出计算损失，通过反向传播得到梯度并更新参数。

### 常见激活函数

学习过程中整理了几类常见激活函数：Sigmoid、Tanh、ReLU、Leaky ReLU 和 Softmax。

其中 ReLU 在多层网络中非常常见，但当输入小于 0 时输出和梯度都为 0，因此可能出现 Dead ReLU。Leaky ReLU 则通过在负半轴保留一个较小斜率，缓解这一问题。

Softmax 更多用于单标签多分类任务，把 logits 转换成类别概率分布。

## 2. Loss：网络到底在优化什么？

对于回归问题，我主要整理了 MSE、MAE、Huber Loss 和分位数损失。

### MSE

$$
MSE=\frac{1}{N}\sum_i(\hat y_i-y_i)^2
$$

MSE 连续可导、优化方便，但平方项会放大较大误差，因此对离群点比较敏感。

### MAE

$$
MAE=\frac{1}{N}\sum_i|\hat y_i-y_i|
$$

相比 MSE，MAE 对离群点更加鲁棒，但在零点不可导。

Huber Loss 则试图结合二者：误差较小时采用类似 MSE 的二次形式，误差较大时转为类似 MAE 的线性形式。

理解这些 Loss 很重要，因为 PINN 的核心同样是构造 Loss，只不过它优化的不只是“预测值和标签之间的误差”。

## 3. 训练、验证和测试

一个神经网络训练过程可以拆成三个阶段：

- 训练阶段：寻找较好的网络参数 $\theta$；
- 验证阶段：选择网络超参数；
- 测试阶段：评估模型在未见数据上的泛化表现。

我目前笔记里采用的一个简单划分是训练集 60%、验证集 20%、测试集 20%。具体比例并不是重点，重要的是不要使用测试集反复调整模型，否则测试结果就无法继续代表真正的未见数据表现。

## 4. 梯度下降与 Adam

训练神经网络，本质上需要根据损失对参数的梯度不断更新参数。

我学习了 SGD、Momentum SGD、Adagrad、RMSProp 和 Adam。其中 Adam 同时利用了一阶动量与二阶梯度平方的移动平均。

常见默认超参数包括：

- $\beta_1=0.9$
- $\beta_2=0.999$
- 学习率 $\eta=10^{-3}$
- $\epsilon=10^{-8}$

在 PyTorch 中可以通过 `torch.optim.Adam` 使用。

## 5. 反向传播与链式法则

反向传播的核心是链式法则。

前向传播时，网络逐层计算并缓存激活值和预激活值；反向传播时，则从输出端开始逐层向前计算梯度。

对于隐藏层，误差可以写成：

$$
\delta^{(l)}=(W^{(l+1)})^T\delta^{(l+1)}\odot\sigma'(A^{(l)})
$$

得到本层误差后，可以继续得到权重和偏置的梯度：

$$
\frac{\partial \mathcal L}{\partial W^{(l)}}=\delta^{(l)}(x^{(l-1)})^T
$$

$$
\frac{\partial \mathcal L}{\partial b^{(l)}}=\delta^{(l)}
$$

这部分是理解 PINN 自动微分的基础。

## 6. 从神经网络走向 PDE

PINN 中的 Physics 指的就是物理规律，而这些规律通常通过偏微分方程 PDE 表达。

PDE 包含多个自变量以及未知函数对这些变量的偏导数。常见例子包括：

- 热传导方程：抛物型；
- 波动方程：双曲型；
- 拉普拉斯方程：椭圆型。

仅有 PDE 本身通常还不够，还需要边界条件和初始条件。

### Dirichlet 边界条件

直接给出边界处函数值：

$$
u(x,t)=q(x)
$$

### Neumann 边界条件

给出边界处函数的法向导数：

$$
\frac{\partial u}{\partial x}=g(x)
$$

### 初始条件

对于含时间变量的动态 PDE，还需要规定初始时刻的状态：

$$
u(x,0)=f(x)
$$

## 7. PINN 最核心的一步：把物理规律写进 Loss

普通监督学习通常依赖输入和标签之间的误差，而 PINN 会进一步要求神经网络输出满足 PDE、边界条件和初始条件。

因此正问题中的总损失可以写成：

$$
\mathcal L=\lambda_{PDE}\mathcal L_{PDE}+\lambda_{BC}\mathcal L_{BC}+\lambda_{IC}\mathcal L_{IC}
$$

### PDE 残差损失

$$
\mathcal L_{PDE}=\frac{1}{N_r}\sum_{i=1}^{N_r}\left|\mathcal N[u_\theta(x_i,t_i)]-s(x_i,t_i)\right|^2
$$

这里 $u_\theta(x,t)$ 是神经网络输出的物理场，$\mathcal N[\cdot]$ 是 PDE 微分算子。

它的直观含义是：**如果网络预测的函数确实符合物理规律，把它代回 PDE 后，残差就应该尽可能接近 0。**

### 边界条件损失

以 Dirichlet 条件为例：

$$
\mathcal L_{BC}=\frac{1}{N_{BC}}\sum_{i=1}^{N_{BC}}|u_\theta(x_i)-q(x_i)|^2
$$

它要求网络在边界处符合给定的物理约束。

### 初始条件损失

$$
\mathcal L_{IC}=\frac{1}{N_{IC}}\sum_{i=1}^{N_{IC}}|u_\theta(x_i,0)-f(x_i)|^2
$$

它要求网络在 $t=0$ 时符合系统给定的初始状态。

到这里，我对 PINN 的第一层理解就比较清楚了：**网络仍然是在最小化 Loss，但 Loss 中加入了来自 PDE 的物理约束。**

## 8. 正问题和反问题

对于反问题，在 PDE、BC、IC 三项之外，还可以加入真实观测数据：

$$
\mathcal L=\lambda_{PDE}\mathcal L_{PDE}+\lambda_{BC}\mathcal L_{BC}+\lambda_{IC}\mathcal L_{IC}+\lambda_{Data}\mathcal L_{Data}
$$

其中：

$$
\mathcal L_{Data}=\frac{1}{N_d}\sum_{i=1}^{N_d}|u_\theta(x_i,t_i)-u(x_i,t_i)|^2
$$

这一项衡量网络预测物理场与实验测量、传感器数据或仿真真值之间的偏差。

## 9. 下一篇：自动微分与完整训练流程

理解损失函数之后，下一个问题就是：PDE 里面需要 $\partial u/\partial t$、$\partial^2u/\partial x^2$ 等导数，这些导数怎么从神经网络里得到？

答案是自动微分。

下一篇准备继续整理：

`采样点 → 网络前向传播 → AutoDiff → PDE Residual → Loss → Backpropagation → Adam → 收敛`

这也是把 PINN 从公式真正落实到 PyTorch 代码时最关键的一条链路。
