---
title: Principal Component Analysis (PCA)
date: 2022-11-15
updated: 2022-11-15
categories: "Note"
tags: 
   - "Math"
---

Principal Component Analysis (PCA) can be viewed as a linear transform which maps a $n$-dimensional random vector to a smaller dimensional vector, but preserving most of its information.

<!--more-->

{{< math >}}
In the following we consider a random vector $x=(x_1,\; x_2,\; \cdots,\; x_n)^T$, where each $x_i$ is a random variable, representing one attribute or property of $x$. Inner product of two vectors $x, y$ is written as $x^Ty$.  All vectors are column vectors.

## Covariance Matrix

The variance of a random variable $x_i$ is defined like

$$
Var(x_i) = \mathbb{E}(x_i^2) - \mathbb{E}(x_i)^2
$$

where $\mathbb{E}(\cdot)$ is the expectation function. To define the variance of a random vector (a series of random variable), we use the covariance matrix.

The covariance (matrix) of a random vector $x$ can be defined as

$$
[Cov(x)]_{ij} = Cov(x_i, x_j) =  \mathbb{E}[x_ix_j] - \mathbb{E}[x_i]\mathbb{E}[x_j]
$$
{{< /math >}}

## Theorem 1

Let $v$ be any constant $m$-dimension vector, the statement holds

{{< math >}}
$$
Var(v^Tx)=v^TCov(x)v
$$
{{< /math >}}

### Proof

Consider

{{< math >}}
$$
\begin{aligned}
Var(v^Tx) 
&= Cov(v^Tx, v^Tx) \\
&= Cov(v_1x_1+\cdots+v_nx_n, v^Tx) \\
&= v^T\left( Cov(x_i, v^Tx) \right)_{i\leq n} \\
&= v^T\left( Cov(x_i, v_1x_1+\cdots+v_nx_n) \right)_{i\leq n} \\
&= v^T Cov(x) v
\end{aligned}
$$

$\square$
{{< /math >}}

## Theorem 2

Let $\{\alpha_k\}_n$ be a series of vectors such that

{{< math >}}
$$
\alpha_k = \text{argmax}_v 
\left \{Var(v^Tx) \mid\|v\|=1, \ v^Tx \perp \{\alpha_i^Tx\}_{i< k}
\right\}
$$
{{< /math >}}

Then $\alpha_k$ is the eigenvector corresponding to the $k$-th largest eigenvalues for matrix $Cov(x)$.

### Proof

Let $\Sigma=Cov(x)$. First consider case $k=1$. We use Lagrange multiplier

$$
L=Var(v^Tx)-\beta(v^Tv - 1)=v^T\Sigma v - \beta(v^Tv - 1)
$$

By differentiation,

$$
\frac{\partial}{\partial v}L = 2 \Sigma v - 2 \beta v = 0 \Longleftrightarrow
\Sigma v = \beta v
$$

We see when the maximum is attained, the vector is the eigenvector of $\Sigma$ that corresponds to the largest eigenvalue. We derive the first vector $\alpha_1$.

For $k=2$, we want $\alpha_2^Tx \perp \alpha_1^Tx$, and the condition is equivalent to

$$
0 = Cov(\alpha_2^Tx, \alpha_1^Tx) = \alpha_2^T \Sigma \alpha_1 = \beta \alpha_2^T \alpha_1
$$

So we just require that $\alpha_2^T \alpha_1 = 0$. Again, by Lagrange multiplier,

$$
L=Var(v^Tx)-\beta_1(v^Tv - 1) - \beta_2(v^T\alpha_1) = v^T\Sigma v - \beta(v^Tv - 1) - \beta_2(v^T\alpha_1)
$$

By differentiation,

$$
\frac{\partial}{\partial v}L = 2 \Sigma v - 2 \beta_1 v - \beta_2 \alpha_1 = 0
$$

Since $\alpha_2 \perp \alpha_1$, the value for $\beta_2 = 0$ naturally, and we obtain 

$$
\Sigma v = \beta_1 v
$$

When the maximum is attained, the vector is the eigenvector of $\Sigma$ that corresponds to the second largest eigenvalue. This gives us $\alpha_2$.

The remaining vectors can be derived in a similar way.  $\square$

## Principal Component

So far we get a series of independent vectors $\alpha_k$ which, by theorem 2, maximize the variance of $v^Tx$.  The first component of $x$ can be defined as $\alpha_1^Tx$, and the second component is defined as $\alpha_2^Tx$, and so on. We may think it as a series of linear transformations with corresponding matrices $A_i$ such that 

$$
A_1 = [\alpha_1], \; A_2 = [\alpha_1 \; \alpha_2], \; \cdots, \; A_n = [\alpha_1 \; \alpha_2 \; \cdots \alpha_n]
$$

If we want to reduce the vector $x$ to a smaller dimension $r$ by a linear transformation, which is what PCA does, we simply do the corresponding matrix multiplication as

$$
y = A_r^T x  = (\alpha_1^T x,\; \alpha_2^T x,\; \cdots,\; \alpha_r^T x)
$$

Notice that the variance is maximized in the scope of linear transformation.
