---
title: "线性与非线性、控制仿射系统"
description: "Standard forms for linear, nonlinear, and control-affine dynamical systems."
lang: "zh-CN"
permalink: "/blog/linear-nonlinear-control-affine-systems/"
math: true
---
## 1. 线性系统的标准形式

连续时间线性系统通常写成：


$$
 \dot x=Ax+Bu
$$


其中：
- $$x\in\mathbb R^n$$：状态向量
- $$u\in\mathbb R^m$$：控制输入
- $$A$$：状态矩阵
- $$B$$：输入矩阵

判断线性系统的关键是：$$A$$ 和 $$B$$ 不能依赖于 $$x$$ 或 $$u$$，但它们可以随时间变化。 

例如：


$$\dot x=3x+2u$$



这是线性系统，因为它对 $$x$$ 和 $$u$$ 都是线性的。
而：


$$\dot x=x^2+2u$$


就不是线性系统，因为状态 $$x$$ 出现了平方项。

# 2.控制仿射系统的标准形式
一般的非线性动力学系统可以写成：


$$\dot{\mathbf x}=F(\mathbf x,\mathbf u)$$


其中：
- $$\mathbf x\in\mathbb R^n$$：状态向量
- $$\mathbf u\in\mathbb R^m$$：控制输入
- $$F$$：描述系统动力学的非线性函数

如果能够写成：


$$ \dot{\mathbf x}=f(\mathbf x)+g(\mathbf x)\mathbf u$$


那么它就是控制仿射系统。

其中：
- $$f(\mathbf x)$$：漂移项（Drift），描述没有控制输入时系统如何演化。
- $$g(\mathbf x)$$：输入矩阵，描述控制输入如何影响系统状态。
- $$\mathbf u$$：控制输入。

注意：$$f(\mathbf x)$$ 和 $$g(\mathbf x)$$ 都可以是非线性函数。
例如：


$$\begin{cases} \dot x_1=x_2\\ \dot x_2=-\sin x_1+(1+x_1^2)u \end{cases}$$


可以写成：


$$\dot{\mathbf x}= \underbrace{\begin{bmatrix}x_2\\-\sin x_1\end{bmatrix}}_{f(\mathbf x)} + \underbrace{\begin{bmatrix}0\\1+x_1^2\end{bmatrix}}_{g(\mathbf x)}u$$


虽然系统对状态是非线性的，但对输入 $$u$$是仿射的。

# 3.仿射系统
 

$${\quad \dot x=Ax+Bu+c}$$


 