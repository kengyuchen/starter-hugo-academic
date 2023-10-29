---
title: Singular Value Decomposition (SVD)
date: 2022-11-15
updated: 2022-11-15
categories: "Note"
tags: 
   - "Math"
---

Singular Value Decomposition (SVD) is a way to decompose *any* real matrix $A \in \mathbb{R}^{m\times n}$ into a simple form.

{{< math >}}
$$
A = U\Sigma V^T
$$

where $U \in \mathbb{R}^{m \times m}$ and $V \in \mathbb{R}^{n \times n}$ and $\Sigma \in \mathbb{R}^{m \times n}$ is a matrix with only diagonal values. The following proof is referenced from the [note](https://gregorygundersen.com/blog/2018/12/20/svd-proof/).

<!--more-->

## Proof of Existence

### Case $n=m$

Since $A^TA$ is a $n \times n$ positive semi-definite (PSD) matrix, we can diagonalize it as

$$
A^TA = V \Lambda V^T = \sum_{i=1}^n \lambda_i \mathbf{v}_i \mathbf{v}_i^T = \sum_{i=1}^n \sigma_i^2 \mathbf{v}_i \mathbf{v}_i^T
$$

where $V$ is an orthonormal matrix with eigenvectors of $A^TA$ as its column vectors, and $\Lambda$ is a diagonal matrix with non-negative eigenvalues. 

$$
V = [\mathbf{v}_1\, \mathbf{v}_2\, \cdots\, \mathbf{v}_n],\quad
\Lambda = 
\begin{bmatrix}
\lambda_1 &0 &\cdots &0 \\
0 &\lambda_2 & &\vdots \\
\vdots & &\ddots \\
0 &\cdots & &\lambda_n
\end{bmatrix}
,\quad \lambda_i = \sigma_i^2 \geq 0
$$

For convention we permute the eigenvalues in descending order; that is $\lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_n$.

Now, consider the following vectors with $m$-dimension

$$
\mathbf{u}_i = \frac{A \mathbf{v}_i}{\sigma_i}, \quad 1\leq i \leq m
$$

One can check each $\mathbf{u}_i$ is a unit vector

$$
\|\mathbf{u}_i\|^2 = \frac{\mathbf{v}_i^TA^TA\mathbf{v}_i}{\sigma_i^2} = \frac{\mathbf{v}_i^T (\sigma_i^2 \mathbf{v}_i)}{\sigma_i^2} = 1
$$

If we define the following matrix

$$
U = [\mathbf{u}_1\, \mathbf{u}_2\, \cdots\, \mathbf{u}_m], \quad
\Sigma = 
\begin{bmatrix}
\sigma_1 &0 &\cdots &0 \\
0 &\sigma_2 & &\vdots \\
\vdots & &\ddots \\
0 &\cdots & &\sigma_n
\end{bmatrix}
$$

We see

$$
U = AV\Sigma^{-1} \implies A = U\Sigma V^{-1} = U\Sigma V^T
$$

### Case $n \neq m$

If $n \geq m$, since $\sigma_i = 0$ for all $i >m$,  we may simply remove the last $n-m$ columns in $\Sigma$. The equation above becomes

$$
U = AV
\begin{bmatrix}
\frac{1}{\sigma_1} &0 &\cdots &0 \\
0 &\frac{1}{\sigma_2} & & \vdots \\
\vdots & &\ddots \\
0 & & &\frac{1}{\sigma_m} \\
\vdots & & & \vdots\\
0 &\cdots & &0
\end{bmatrix}
\implies A = U\begin{bmatrix}
\sigma_1 &0 &\cdots & & &0 \\
0 &\sigma_2 & & & &\vdots \\
\vdots & &\ddots \\
0 &\cdots & &\sigma_m &\cdots &0
\end{bmatrix}
 V^T
$$

If $n < m$, we can remove the last $m-n$ rows in $\Sigma$, leading to

$$
U = AV
\begin{bmatrix}
\frac{1}{\sigma_1} &0 &\cdots & & &0 \\
0 &\frac{1}{\sigma_2} & & & &\vdots \\
\vdots & &\ddots \\
0 &\cdots & &\frac{1}{\sigma_n} &\cdots &0
\end{bmatrix}
\implies A = U
\begin{bmatrix}
\sigma_1 &0 &\cdots &0 \\
0 &\sigma_2 & & \vdots \\
\vdots & &\ddots \\
0 & & &\sigma_n \\
\vdots & & & \vdots\\
0 &\cdots & &0
\end{bmatrix}
 V^T
$$

Note that the vectors $\{\mathbf{u}_i\}_{i > m}$ are not defined. We replace those undefined vectors by any $n-m$ independent vectors which makes $\{\mathbf{u}_i\}$ span the whole $\mathbb{R}^m$.

{{< /math >}}