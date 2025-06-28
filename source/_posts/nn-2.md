---
title: 神经网络学习笔记（2）—— 前向传播与反向传播的数学推导
date: 2025-06-29 00:26:13
categories:
- 学习
tags: 
- 神经网络
- MLP
---

# 向量求导定义

我们对向量求导作出如下定义：

$$
\begin{gathered}
\frac{\partial f(\mathbf{x})}{\partial\ \mathbf{x}_{n\times1}}:=\begin{bmatrix}
\frac{\partial f}{\partial x_1}&\frac{\partial f}{\partial x_2}&\cdots&\frac{\partial f}{\partial x_n}
\end{bmatrix}_{1\times n}
,\quad \frac{\partial f(\mathbf{x})}{\partial\ \mathbf{x}_{1\times n}}:=\begin{bmatrix}
\frac{\partial f}{\partial x_1}\\
\frac{\partial f}{\partial x_2}\\
\vdots\\
\frac{\partial f}{\partial x_n}
\end{bmatrix}_{n\times1}\\
\frac{\partial\ \mathbf{f}_{m\times1}(\mathbf{x})}{\partial\ \mathbf{x}_{n\times1}}:=\begin{bmatrix}
\frac{\partial f_1}{\partial x_1}&\frac{\partial f_1}{\partial x_2}&\cdots&\frac{\partial f_1}{\partial x_n}\\
\frac{\partial f_2}{\partial x_1}&\frac{\partial f_2}{\partial x_2}&\cdots&\frac{\partial f_2}{\partial x_n}\\
\vdots&\vdots&\ddots&\vdots\\
\frac{\partial f_m}{\partial x_1}&\frac{\partial f_m}{\partial x_2}&\cdots&\frac{\partial f_m}{\partial x_n}\\
\end{bmatrix}_{m\times n},\quad
\frac{\partial\ \mathbf{f}_{1\times m}(\mathbf{x})}{\partial\ \mathbf{x}_{1\times n}}:=\begin{bmatrix}
\frac{\partial f_1}{\partial x_1}&\frac{\partial f_2}{\partial x_1}&\cdots&\frac{\partial f_m}{\partial x_1}\\
\frac{\partial f_1}{\partial x_2}&\frac{\partial f_2}{\partial x_2}&\cdots&\frac{\partial f_m}{\partial x_2}\\
\vdots&\vdots&\ddots&\vdots\\
\frac{\partial f_1}{\partial x_n}&\frac{\partial f_2}{\partial x_n}&\cdots&\frac{\partial f_m}{\partial x_n}\\
\end{bmatrix}_{n\times m}
\end{gathered} 
$$

则有如下链式法则成立：

$$
\frac{\partial z}{\partial x}=\frac{\partial z}{\partial\mathbf{y}}\cdot\frac{\partial\mathbf{y}}{\partial x},\quad x,z\in\R,\ \mathbf{y}\in\R_{m\times1}
$$

$$
\frac{\partial z}{\partial\mathbf{x}}=\frac{\partial z}{\partial\mathbf{y}}\cdot\frac{\partial\mathbf{y}}{\partial\mathbf{x}},\quad z\in\R,\ \mathbf{y}\in\R_{m\times1},\ \mathbf{x}\in\R_{n\times1}
$$

$$
\frac{\partial z}{\partial x}=\frac{\partial\mathbf{y}}{\partial x}\cdot\frac{\partial z}{\partial\mathbf{y}},\quad x,z\in\R,\ \mathbf{y}\in\R_{1\times m}
$$

$$
\frac{\partial z}{\partial\mathbf{x}}=\frac{\partial\mathbf{y}}{\partial\mathbf{x}}\cdot\frac{\partial z}{\partial\mathbf{y}},\quad z\in\R,\ \mathbf{y}\in\R_{1\times m},\ \mathbf{x}\in\R_{1\times n}
$$

# 矩阵求导定义

我们对矩阵求导作出如下定义：

$$
\frac{\partial f(\mathbf{X})}{\partial\mathbf{X}_{m\times n}}:=\begin{bmatrix}
\frac{\partial f}{\partial X_{1,1}}&\frac{\partial f}{\partial X_{2,1}}&\cdots&\frac{\partial f}{\partial X_{m,1}}\\
\frac{\partial f}{\partial X_{1,2}}&\frac{\partial f}{\partial X_{2,2}}&\cdots&\frac{\partial f}{\partial X_{m,2}}\\
\vdots&\vdots&\ddots&\vdots\\
\frac{\partial f}{\partial X_{1,n}}&\frac{\partial f}{\partial X_{2,n}}&\cdots&\frac{\partial f}{\partial X_{m,n}}\\
\end{bmatrix}_{n\times m}
$$

# 无 Batch 情况（一次训练一组数据）

$$
\mathbf{X}^k:=\begin{bmatrix}
X_1^k\\
X_2^k\\
\vdots\\
X_{D_k}^k\\
\end{bmatrix}_{D_k\times1},\ 
\mathbf{W}^k:=\begin{bmatrix}
W_{1,1}^k&W_{1,2}^k&\cdots&W_{1,D_{k-1}}^k\\
W_{2,1}^k&W_{2,2}^k&\cdots&W_{2,D_{k-1}}^k\\
\vdots&\vdots&\ddots&\vdots\\
W_{D_k,1}^k&W_{D_k,2}^k&\cdots&W_{D_k,D_{k-1}}^k\\
\end{bmatrix}_{D_k\times D_{k-1}},\ 
\mathbf{b}^k:=\begin{bmatrix}
b_1^k\\
b_2^k\\
\vdots\\
b_{D_k}^k\\
\end{bmatrix}_{D_k\times1}
$$

$$
\mathbf{P}^k=\mathbf{W}^k\mathbf{X}^{k-1}+\mathbf{b}^k=
\begin{bmatrix}
\sum_{j=1}^{D_{k-1}}W_{1,j}^kX_j^{k-1}+b_1^k\\
\sum_{j=1}^{D_{k-1}}W_{2,j}^kX_j^{k-1}+b_2^k\\
\vdots\\
\sum_{j=1}^{D_{k-1}}W_{i,j}^kX_j^{k-1}+b_i^k\\
\vdots\\
\sum_{j=1}^{D_{k-1}}W_{D_k,j}^kX_j^{k-1}+b_{D_k}^k\\
\end{bmatrix}_{D_k\times1} 
=\begin{bmatrix}
P_1^k\\
P_2^k\\
\vdots\\
P_i^k\\
\vdots\\
P_{D_k}^k\\
\end{bmatrix}_{D_k\times1} 
$$

$$
\mathbf{X}^k=f(\mathbf{P}^k)=\begin{bmatrix}
f(P_1^k)\\
f(P_2^k)\\
\vdots\\
f(P_{D_k}^k)\\
\end{bmatrix}_{D_k\times1} 
$$

那么，

$$
\begin{gathered}
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{X}^{k}}\cdot\frac{\partial\mathbf{X}^k}{\partial\mathbf{P}^k},\quad
\frac{\partial\mathcal{L}}{\partial\mathbf{X}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\frac{\partial\mathbf{P}^{k+1}}{\partial\mathbf{X}^k}\\
\Rightarrow\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\frac{\partial\mathbf{P}^{k+1}}{\partial\mathbf{X}^k}\cdot\frac{\partial\mathbf{X}^k}{\partial\mathbf{P}^k}
\end{gathered} 
$$

其中，

$$
\frac{\partial\mathbf{X}^k}{\partial\mathbf{P}^k}=\frac{\partial f(\mathbf{P}^k)}{\partial\mathbf{P}^k}=\begin{bmatrix}
f'(P_1^k)&0&\cdots&0\\
0&f'(P_2^k)&\cdots&0\\
\vdots&\vdots&\ddots&\vdots\\
0&0&\cdots&f'(P_{D_k}^k)\\
\end{bmatrix}_{D_k\times D_k}
$$

$$
\begin{aligned}
\frac{\partial\mathbf{P}^{k+1}}{\partial\mathbf{X}^k}&=\frac{\partial(\mathbf{W}^{k+1}\mathbf{X}^k+\mathbf{b}^{k+1})}{\partial\mathbf{X}^k}\\
\Rightarrow\left(\frac{\partial\mathbf{P}^{k+1}}{\partial\mathbf{X}^k}\right)_{i,j}&=\frac{\partial(\sum_{m=1}^{D_k}W_{i,m}^{k+1}X_m^k+b_i^{k+1})}{\partial X_j^k}=W_{i,j}^{k+1}
\end{aligned} 
$$

$$
\frac{\partial\mathbf{P}^{k+1}}{\partial\mathbf{X}^k}=\mathbf{W}^{k+1}
$$

代入得到：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\mathbf{W}^{k+1}\cdot\begin{bmatrix}
f'(P_1^k)&0&\cdots&0\\
0&f'(P_2^k)&\cdots&0\\
\vdots&\vdots&\ddots&\vdots\\
0&0&\cdots&f'(P_{D_k}^k)\\
\end{bmatrix}_{D_k\times D_k}
$$

为了化简上式，我们令

$$
f'(\mathbf{P}^k):=\begin{bmatrix}
f'(P_1^k)\\
f'(P_2^k)\\
\vdots\\
f'(P_{D_k}^k)\\
\end{bmatrix}_{D_k\times 1}
$$

并记 Element-wise 乘

$$
\mathbf{A}=\begin{bmatrix}
A_1\\
A_2\\
\vdots\\
A_n
\end{bmatrix}_{1\times n},\quad
\mathbf{B}=\begin{bmatrix}
B_1\\
B_2\\
\vdots\\
B_n
\end{bmatrix}_{1\times n},\quad
\mathbf{A}\odot\mathbf{B}:=\begin{bmatrix}
A_1B_1\\
A_2B_2\\
\vdots\\
A_nB_n
\end{bmatrix}_{1\times n}
$$

则原式可化简为

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\left(\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\mathbf{W}^{k+1}\right)\odot f'(\mathbf{P}^k)^T\tag{1}
$$

可见需要递推求解，即反向传播。考虑输出层（第 $n$ 层）：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\cdot\frac{\partial\mathbf{y}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\cdot\frac{\partial f_{\text{output}}(\mathbf{P}^n)}{\partial\mathbf{P}^n}
$$

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\odot f'_{\text{output}}(\mathbf{P}^n)^T\tag{2}
$$

接下来考虑权重梯度：

$$
\frac{\partial\mathcal{L}}{\partial W_{i,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\cdot\frac{\partial\mathbf{P}^k}{\partial W_{i,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\cdot\begin{bmatrix}
0\\
\vdots\\
0\\
X_j^{k-1},\ _{\text{row}=i}\\
0\\
\vdots\\
0
\end{bmatrix}_{D_k\times1}=\frac{\partial\mathcal{L}}{\partial P_i^k}\cdot X_j^{k-1} 
$$

根据矩阵求导的定义：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k}=\mathbf{X}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\tag{3}
$$

考虑偏置梯度：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\cdot\frac{\partial\mathbf{P}^k}{\partial\mathbf{b}^k},\quad\frac{\partial\mathbf{P}^k}{\partial\mathbf{b}^k}=\frac{\partial(\mathbf{W}^k\mathbf{X}^{k-1}+\mathbf{b}^k)}{\partial\mathbf{b}^k}=\frac{\partial\mathbf{b}^k}{\partial\mathbf{b}^k}=\mathbf{I}_{D_k\times D_k}
$$

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\tag{4}
$$

梯度下降，更新权重与偏置（维度对齐）：

$$
\mathbf{W}^k_{(s+1)}=\mathbf{W}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k_{(s)}}^T\tag{5}
$$

$$
\mathbf{b}^k_{(s+1)}=\mathbf{b}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k_{(s)}}^T\tag{6}
$$

联立 $(1),(2),(3),(4),(5),(6)$：

$$
\begin{dcases}
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\odot f'_{\text{output}}(\mathbf{P}^n)^T\\
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\left(\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\mathbf{W}^{k+1}\right)\odot f'(\mathbf{P}^k)^T\\
\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k}=\mathbf{X}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\\
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\\
\mathbf{W}^k_{(s+1)}=\mathbf{W}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k_{(s)}}^T\\
\mathbf{b}^k_{(s+1)}=\mathbf{b}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k_{(s)}}^T
\end{dcases} 
$$

# 有 Batch 情况（一次训练多组数据）

$$
\mathbf{X}^k:=\begin{bmatrix}
X_{1b_1}^k&X_{1b_2}^k&\cdots&X_{1b_{D_b}}^k\\
X_{2b_1}^k&X_{2b_2}^k&\cdots&X_{2b_{D_b}}^k\\
\vdots&\vdots&\ddots&\vdots\\
X_{D_kb_1}^k&X_{D_kb_2}^k&\cdots&X_{D_kb_{D_b}}^k\\
\end{bmatrix}_{D_k\times D_b},\ 
\mathbf{b}^k:=\begin{bmatrix}
b_1^k\\
b_2^k\\
\vdots\\
b_{D_k}^k\\
\end{bmatrix}_{D_k\times1}
$$

$$
\mathbf{W}^k:=\begin{bmatrix}
W_{1,1}^k&W_{1,2}^k&\cdots&W_{1,D_{k-1}}^k\\
W_{2,1}^k&W_{2,2}^k&\cdots&W_{2,D_{k-1}}^k\\
\vdots&\vdots&\ddots&\vdots\\
W_{D_k,1}^k&W_{D_k,2}^k&\cdots&W_{D_k,D_{k-1}}^k\\
\end{bmatrix}_{D_k\times D_{k-1}}
$$

$$
\begin{aligned}
\mathbf{P}^k=\mathbf{W}^k\mathbf{X}^{k-1}+\mathbf{b}^k&=
\begin{bmatrix}
\sum_{j=1}^{D_{k-1}}W_{1,j}^kX_{jb_1}^{k-1}+b_1^k&\cdots&\sum_{j=1}^{D_{k-1}}W_{1,j}^kX_{jb_{D_b}}^{k-1}+b_1^k\\
\sum_{j=1}^{D_{k-1}}W_{2,j}^kX_{jb_1}^{k-1}+b_2^k&\cdots&\sum_{j=1}^{D_{k-1}}W_{2,j}^kX_{jb_{D_b}}^{k-1}+b_2^k\\
\vdots&\vdots&\vdots\\
\sum_{j=1}^{D_{k-1}}W_{i,j}^kX_{jb_1}^{k-1}+b_i^k&\cdots&\sum_{j=1}^{D_{k-1}}W_{i,j}^kX_{jb_{D_b}}^{k-1}+b_i^k\\
\vdots&\vdots&\vdots\\
\sum_{j=1}^{D_{k-1}}W_{D_k,j}^kX_{jb_1}^{k-1}+b_{D_k}^k&\cdots&\sum_{j=1}^{D_{k-1}}W_{D_k,j}^kX_{jb_{D_b}}^{k-1}+b_{D_k}^k\\
\end{bmatrix}_{D_k\times D_b}\\ 
&=\begin{bmatrix}
P_{1b_1}^k&\cdots&P_{1b_{D_b}}^k\\
P_{2b_1}^k&\cdots&P_{2b_{D_b}}^k\\
\vdots&\vdots&\vdots\\
P_{ib_1}^k&\cdots&P_{ib_{D_b}}^k\\
\vdots&\vdots&\vdots\\
P_{D_kb_1}^k&\cdots&P_{D_kb_{D_b}}^k\\
\end{bmatrix}_{D_k\times D_b}
\end{aligned} 
$$

$$
\mathbf{X}^k=f(\mathbf{P}^k)=\begin{bmatrix}
f(P_{1b_1}^k)&f(P_{1b_2}^k)&\cdots&f(P_{1b_{D_b}}^k)\\
f(P_{2b_1}^k)&f(P_{2b_2}^k)&\cdots&f(P_{2b_{D_b}}^k)\\
\vdots&\vdots&\ddots&\vdots\\
f(P_{D_kb_1}^k)&f(P_{D_kb_2}^k)&\cdots&f(P_{D_kb_{D_b}}^k)\\
\end{bmatrix}_{D_k\times D_b} 
$$

那么（取合适的向量代入链式法则），

$$
\begin{gathered}
\frac{\partial\mathcal{L}}{\partial P_{i,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{X}_{:,j}^{k}}\cdot\frac{\partial\mathbf{X}_{:,j}^k}{\partial P_{i,j}^k},\quad
\frac{\partial\mathcal{L}}{\partial\mathbf{X}_{:,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^{k+1}}\cdot\frac{\partial\mathbf{P}_{:,j}^{k+1}}{\partial\mathbf{X}_{:,j}^k}\\
\Rightarrow\frac{\partial\mathcal{L}}{\partial P_{i,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^{k+1}}\cdot\frac{\partial\mathbf{P}_{:,j}^{k+1}}{\partial\mathbf{X}_{:,j}^k}\cdot\frac{\partial\mathbf{X}_{:,j}^k}{\partial P_{i,j}^k}
\end{gathered} 
$$

其中，

$$
\frac{\partial X_{i,j}^k}{\partial P_{s,t}^k}\neq0\ \Leftrightarrow\ i=s,\ j=t\quad\Rightarrow\quad\frac{\partial\mathbf{X}_{:,j}^k}{\partial P_{i,j}^k}=\begin{bmatrix}
0\\
\vdots\\
0\\
f'(P_{ib_j}^k),\ _{\text{row}=i}\\
0\\
\vdots\\
0
\end{bmatrix}_{D_k\times1}
$$

$$
\frac{\partial P_{i,j}^{k+1}}{\partial X_{sb_t}^k}=\frac{\partial(\sum_{m=1}^{D_k}W_{i,m}^{k+1}X_{mb_j}^k+b_i^{k+1})}{\partial X_{sb_t}^k}=W_{i,s}^{k+1}\ \Leftrightarrow\ j=t
$$

$$
\frac{\partial\mathbf{P}_{:,j}^{k+1}}{\partial\mathbf{X}_{:,j}^k}=\mathbf{W}^{k+1}
$$

代入得到：

$$
\frac{\partial\mathcal{L}}{\partial P_{i,j}^k}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^{k+1}}\cdot\mathbf{W}^{k+1}\cdot\begin{bmatrix}
0\\
\vdots\\
0\\
f'(P_{ib_j}^k),\ _{\text{row}=i}\\
0\\
\vdots\\
0
\end{bmatrix}_{D_k\times1}=\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^{k+1}}\cdot\mathbf{W}_{:,i}^{k+1}\cdot f'(P_{ib_j}^k)
$$

行扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^k}=\left(\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^{k+1}}\cdot\mathbf{W}^{k+1}\right)\odot f'(\mathbf{P}_{:,j}^k)^T
$$

列扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\left(\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\mathbf{W}^{k+1}\right)\odot f'(\mathbf{P}^k)^T\tag{1}
$$

可见需要递推求解，即反向传播。考虑输出层（第 $n$ 层）：

$$
\frac{\partial\mathcal{L}}{\partial P_{i,j}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}_{:,j}}\cdot\frac{\partial\mathbf{y}_{:,j}}{\partial P_{i,j}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}_{:,j}}\cdot\frac{\partial f_{\text{output}}(\mathbf{P}_{:,j}^n)}{\partial P_{i,j}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}_{:,j}}\cdot f'_{\text{output}}(P_{i,j}^n)
$$

行扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{:,j}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}_{:,j}}\odot f'_{\text{output}}(\mathbf{P}_{:,j}^n)^T
$$

列扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\odot f'_{\text{output}}(\mathbf{P}^n)^T\tag{2}
$$

接下来考虑权重梯度：

$$
\frac{\partial\mathcal{L}}{\partial W_{i,j}^k}=\frac{\partial\mathbf{P}_{i,:}^k}{\partial W_{i,j}^k}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}=\frac{\partial(\sum_{m=1}^{D_{k-1}}W_{i,m}^{k}\mathbf{X}_{m,:}^{k-1}+b_i^{k})}{\partial X_{sb_t}^k}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}=\mathbf{X}_{j,:}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}
$$

行扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{W}_{i,:}^k}=\mathbf{X}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}
$$

列扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k}=\mathbf{X}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\tag{3}
$$

考虑偏置梯度：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\frac{\partial\mathbf{P}_{i,:}^k}{\partial\mathbf{b}^k}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}=\frac{\partial(\sum_{m=1}^{D_{k-1}}W_{i,m}^{k}\mathbf{X}_{m,:}^{k-1}+b_i^{k})}{\partial X_{sb_t}^k}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}=\mathbf{J}_{1\times D_b}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}_{i,:}^k}
$$

列扩展：

$$
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\mathbf{J}_{1\times D_b}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\tag{4}
$$

梯度下降，更新权重与偏置（维度对齐）：

$$
\mathbf{W}^k_{(s+1)}=\mathbf{W}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k_{(s)}}^T\tag{5}
$$

$$
\mathbf{b}^k_{(s+1)}=\mathbf{b}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k_{(s)}}^T\tag{6}
$$

联立 $(1),(2),(3),(4),(5),(6)$：

$$
\begin{dcases}
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^n}=\frac{\partial\mathcal{L}}{\partial\mathbf{y}}\odot f'_{\text{output}}(\mathbf{P}^n)^T\\
\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}=\left(\frac{\partial\mathcal{L}}{\partial\mathbf{P}^{k+1}}\cdot\mathbf{W}^{k+1}\right)\odot f'(\mathbf{P}^k)^T\\
\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k}=\mathbf{X}^{k-1}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\\
\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k}=\mathbf{J}_{1\times D_b}\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{P}^k}\\
\mathbf{W}^k_{(s+1)}=\mathbf{W}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{W}^k_{(s)}}^T\\
\mathbf{b}^k_{(s+1)}=\mathbf{b}^k_{(s)}-\alpha\cdot\frac{\partial\mathcal{L}}{\partial\mathbf{b}^k_{(s)}}^T
\end{dcases} 
$$