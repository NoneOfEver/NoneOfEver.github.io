---
title: "常见矩阵分解"
description: "Notes on LU, QR, Cholesky, eigendecomposition, SVD, and Schur decomposition."
lang: "zh-CN"
permalink: "/blog/matrix-decompositions/"
math: true
---
## 1. LU （Lower-Upper）三角分解



$$ A = LU$$


将矩阵分解为下三角矩阵 $$L$$ 和上三角矩阵 $$U$$，实际计算中通常包含行置换，即 $$PA=LU$$。

主要用于求解线性方程组 $$Ax=b$$，以及计算行列式。

## 2.QR（Orthogonal-Triangular）正交分解


$$A=QR$$


$$Q$$ 的列正交归一，$$R$$ 是上三角矩阵。

主要用于最小二乘问题、数值稳定的线性方程求解和特征值计算。

## 3.Cholesky 平方根分解


$$A=L L^T$$


对实对称正定矩阵，可以分解成下三角矩阵与其转置的乘积。

在凸二次优化、Kalman 滤波、MPC 和机器人动力学中非常常见，通常比通用 LU 分解更高效。

## 4.EVD特征值分解


$$A=V\Lambda V^{-1}$$


将可对角化方阵表示为特征向量矩阵、特征值对角矩阵及其逆矩阵的乘积。

主要用于系统稳定性分析、模态分析、矩阵指数计算和 PCA。

假设 \(A\) 有 \(n\) 个线性无关的特征向量：



$$Av_i=\lambda_i v_i$$



把这些特征向量排列起来：



$$ V=\begin{bmatrix}v_1&v_2&\cdots&v_n\end{bmatrix}$$



把特征值放到对角矩阵：



$$ \Lambda= \begin{bmatrix} \lambda_1&0&\cdots&0\\ 0&\lambda_2&\cdots&0\\ \vdots&\vdots&\ddots&\vdots\\ 0&0&\cdots&\lambda_n \end{bmatrix}$$


那么：



$$ AV=V\Lambda $$



由于 $$V$$ 可逆：



$$ A=V\Lambda V^{-1}$$



这就是特征值分解。

它的核心价值在于：通过坐标变换，将一个耦合的线性系统转换为若干个相互独立的标量系统。

注意，并不是所有矩阵都能对角化。只有具有足够多线性无关特征向量的矩阵，才能写成上述形式。

## 5.SVD奇异值分解


$$A=U\Sigma V^T$$


$$U,V$$ 为正交矩阵，$$\Sigma$$ 包含非负奇异值。适用于任意实矩阵，包括非方阵和秩亏矩阵。

主要用于伪逆、矩阵降维、病态问题分析以及机器人雅可比矩阵的奇异性分析。

## 6.Schur分解


$$A=QTQ^T$$


对实矩阵，$$Q$$ 为正交矩阵，$$T$$ 为实拟上三角矩阵（允许 $$2\times2$$对角块）。

主要用于数值特征值计算、矩阵函数和代数 Riccati 方程的数值求解。