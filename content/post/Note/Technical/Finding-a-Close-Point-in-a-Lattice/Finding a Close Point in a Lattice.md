---
title: Finding a Close Point in a Lattice
author: admin
date: 2023-07-10
updated: 2023-11-06
math: true
toc: true
toc_depth: 4
categories: "Note"
tags: 
   - "Math"
   - "Algorithm"
   - "Cryptography"
---

- Reference
    1. [Bab85] [On Lovász’ lattice reduction and the nearest lattice point problem](https://link.springer.com/article/10.1007/bf02579403)
    2. [Kle00] [Finding the closest lattice vector when it's unusually close](https://dl.acm.org/doi/10.5555/338219.338661)
    3. [GPV08] [Trapdoors for Hard Lattices and New Cryptographic Constructions](https://eprint.iacr.org/2007/432.pdf)
    4. [DP16] [Fast Fourier Orthogonalization](https://eprint.iacr.org/2015/1014.pdf)
    5. [GMRR22] [Power analysis attacks on falcon.](https://doi.org/10.46586/tches.v2022.i3.141-164)

{{< toc >}}


## Problem Setting
{{< math >}}
Lattice $\mathcal{L}(\mathbf{B})$ of some matrix $\mathbf{B} = \{\mathbf{b}_1,\cdots,\mathbf{b}_n\}$ where $\mathbf{b}_i$’s are column vectors is

$$
\mathcal{L}(\mathbf{B}) = \left\{ \mathbf{B}x =\sum_{i=1}^n x_i\mathbf{b}_i : x_i \in \mathbb{Z}\right\}
$$

The goal is that, given some vector $\mathbf{c}$, find vector $\mathbf{z} \in \mathcal{L}(\mathbf{B})$ which is close to $\mathbf{c}$, with the closeness be measured by Euclidean distance. 
{{< /math >}}

## Definition
{{< math >}}

The Gram-Schmidt Orthogonalization (GSO) of a full-rank matrix $\mathbf{B}$ is $\mathbf{\tilde B} = \{\mathbf{\tilde b}_1 ,\cdots,\mathbf{\tilde b}_n\}$  where

$$
\quad \mathbf{\tilde b}_i = \mathbf{b}_i - \sum_{j=1}^{i-1} \frac{\langle \mathbf{ b}_i,\mathbf{\tilde b}_j \rangle}{\langle \mathbf{\tilde b}_j,\mathbf{\tilde b}_j \rangle} \mathbf{\tilde b}_j
$$

Note that we can write as

$$
\mathbf{B} = \mathbf{\tilde BU} \quad \text{or} \quad \mathbf{B}^t = \mathbf{L\tilde B}^t
$$

for some upper-triangular matrix $\mathbf{U}$ or lower-triangular matrix $\mathbf{L}$ with $1$ on their diagonals.

$$
\mathbf{U} =
\begin{bmatrix}
1 & \frac{\langle \mathbf{ b_2},\mathbf{\tilde b_1} \rangle}{\langle \mathbf{\tilde b_1},\mathbf{\tilde b_1} \rangle}  & \frac{\langle \mathbf{ b_3},\mathbf{\tilde b_1} \rangle}{\langle \mathbf{\tilde b_1},\mathbf{\tilde b_1} \rangle}  & \cdots 
&  \frac{\langle \mathbf{ b_n},\mathbf{\tilde b_1} \rangle}{\langle \mathbf{\tilde b_1},\mathbf{\tilde b_1} \rangle}   
\\
0 & 1 & \frac{\langle \mathbf{ b_3},\mathbf{\tilde b_2} \rangle}{\langle \mathbf{\tilde b_2},\mathbf{\tilde b_2} \rangle}  & \cdots & \frac{\langle \mathbf{ b_n},\mathbf{\tilde b_2} \rangle}{\langle \mathbf{\tilde b_2},\mathbf{\tilde b_2} \rangle} \\
0 & 0 & 1 & \cdots & \vdots \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & 0 & 1
\end{bmatrix}
$$

Gaussian function for $n$-dimension:

$$
\rho_{s,\mathbf{c}}(\mathbf{x}) = \exp \left( -\pi \frac{ \|\mathbf{x} - \mathbf{c}\|^2}{s^2}\right)
$$

We may omit $\mathbf{c}$ if $\mathbf{c} = \mathbf{0}$ . The discrete Gaussian distribution can be defined as:

$$
D_{\Lambda,s,\mathbf{c}}(\mathbf{x}) = \frac{\exp \left( -\pi \frac{ \|\mathbf{x} - \mathbf{c}\|^2}{s^2}\right)}{\sum _{\mathbf{v} \in \Lambda}\exp \left( -\pi \frac{ \|\mathbf{v} - \mathbf{c}\|^2}{s^2}\right)} = \frac{\rho_{s,\mathbf{c}}(\mathbf{x})}{\rho_{s,\mathbf{c}}(\mathbf{\Lambda})}
$$
{{< /math >}}


## Babai’s Round-off Algorithm [Bab05]

Given vector $\mathbf{c}$. Find vector $\mathbf{z} \in \mathcal{L}(\mathbf{B})$ which is close to $\mathbf{c}$.

First find $\mathbf{B}^{-1}$. Then consider $\mathbf{v} \gets  \mathbf{B} \cdot \lfloor \mathbf{B^{-1} c} \rceil$. 

Note that
{{< math >}}
$$
\mathbf{c} - \mathbf{v} = \mathbf{B} \cdot \mathbf{B^{-1}c} -\mathbf{B} \cdot \lfloor \mathbf{B^{-1} c} \rceil  \in \mathbf{B} \cdot [-1,1]^n 
$$
{{< /math >}}

## Nearest-Plane Algorithm [Kle00]
Given vector $\mathbf{c}$. Find vector $\mathbf{z} \in \mathcal{L}(\mathbf{B})$ which is close to $\mathbf{c}$.

Let  $\mathbf{c}_n = \mathbf{c}$ and $\mathbf{v}_n = \mathcal{0}$ .

For $i = n,\cdots,1$:

1. $c_i' \gets \frac{\langle \mathbf{c}_i, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle}$
2. Let $z_i \gets \lfloor c_i' \rceil$ 
3. {{< math >}} $\mathbf{c}_{i-1} \gets \mathbf{c}_i - z_i \mathbf{b}_i$ and $\mathbf{v}_{i-1} \gets \mathbf{v}_i + z_i \mathbf{b}_i$ {{< /math >}}

Output $\mathbf{v}_0$.

### Theorem 1

{{< math >}}
$$
\mathbf{v}_0 - \mathbf{c} = \sum_{i=1}^n (z_i - c_i')\mathbf{\tilde b}_i \in \mathbf{\tilde B} \cdot[-\frac{1}{2},\frac{1}{2}]^n 
$$

### *Proof:*

We claim that for $j=1,2,\cdots,n$

$$
\mathbf{v}_0 - \mathbf{v}_j - \pi_j(\mathbf{c}_j) = \sum_{i=1}^j (z_i - c_i')\mathbf{\tilde b}_i
$$

where $\pi_j(\cdot)$ is the orthogonal projection onto $\text{span}(\mathbf{b}_1,\cdots,\mathbf{b}_j)$

$$
\pi_j(\mathbf{t}) =  \sum_{k=1}^{j} \frac{\langle \mathbf{t},\mathbf{\tilde b}_k \rangle}{\langle \mathbf{\tilde b}_k ,\mathbf{\tilde b}_k \rangle} \mathbf{\tilde b}_k \quad \text{and} \quad \mathbf{\tilde b}_j = \mathbf{b}_{j} -\pi_{j-1}(\mathbf{b}_{j})
$$

Observe that

$$
\pi_j(\mathbf{c}_j) = c_j' \mathbf{\tilde b}_j +\pi_{j-1}(\mathbf{c}_j)
\quad \text{and} \quad
\mathbf{\tilde b}_j = \mathbf{b}_{j} -\pi_{j-1}(\mathbf{b}_{j})
$$

Note that when $j=n$, the above equation becomes what we desire since $\mathbf{v}_n = \mathbf{0}$ and $\pi_n(\mathbf{c}_n) = \pi_n(\mathbf{c}) = \mathbf{c}$.

We prove by induction. For $j=1$,

$$
\mathbf{v}_0 - \mathbf{v}_1 - \pi_1(\mathbf{c}_1) = z_1\mathbf{b}_1 - \pi_1(\mathbf{c}_1) = z_1\mathbf{\tilde b}_1 - c_1'\mathbf{\tilde b}_1
$$

Now suppose the equation is true for all $j < k$. When $j=k$,

$$
\begin{aligned}
\mathbf{v}_0 - \mathbf{v}_j - \pi_j(\mathbf{c}_j) &= \mathbf{v}_0 - (\mathbf{v}_{k-1} - z_k \mathbf{b}_k) - \pi_{k-1}(\mathbf{c}_{k}) + c_k'\mathbf{\tilde b}_k \\
&= \mathbf{v}_0 - \mathbf{v}_{k-1} +z_k\mathbf{b}_k - \pi_{k-1}(\mathbf{c}_{k-1} + z_k\mathbf{b}_k) - c_k'\mathbf{\tilde b}_k \\
&= \mathbf{v}_0 - \mathbf{v}_{k-1}  - \pi_{k-1}(\mathbf{c}_{k-1}) + z_k(\mathbf{b}_k - \pi_{k-1}(\mathbf{b}_k)) - c_k'\mathbf{\tilde b}_k \\
&= \sum_{i=1}^{k-1} (z_i - c_i')\mathbf{\tilde b}_i + z_k\mathbf{\tilde b}_k - c_k'\mathbf{\tilde b}_k
\end{aligned}
$$

{{< /math >}}

### Another Version by $\mathbf{U}$

First find $\mathbf{t} = \mathbf{B}^{-1} \mathbf{c}$ such that $\mathbf{Bt} = \mathbf{c}$.

For $i = n,\cdots,1$:

1. $t_i' \gets t_i + \sum_{j>i} \mathbf{U}_{ij} (t_j - z_j)$
2. Let $z_i \gets \lfloor t_i' \rceil$ 

Output $\mathbf{v} = \sum_{i=1}^n z_i\mathbf{b}_i$ .

#### Proof of Equivalence

{{< math >}}

These two are actually computing the same thing!

We show that $t_i' = c_i'$. Note that

$$
c_i' = \frac{\langle \mathbf{c} - \sum_{j>i} z_j \mathbf{b}_j, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} = \frac{\langle \mathbf{c}, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} 
- \frac{\langle \sum_{j>i} z_j \mathbf{b}_j, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle}
$$

Also note that $\mathbf{U}_{ij} = \frac{\langle \mathbf{ b}_j,\mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i,\mathbf{\tilde b}_i \rangle}$ . Therefore,

$$
\sum_{j>i} \mathbf{U}_{ij} z_j = \sum_{j>i} z_j  \frac{\langle \mathbf{b}_j, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} = \frac{\langle \sum_{j>i} z_j \mathbf{b}_j, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} 
$$

Now we need to show that $t_i + \sum_{j>i} \mathbf{U}_{ij}t_j = \frac{\langle \mathbf{c}, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle}$ . Since $\mathbf{\tilde B}$ is full-rank and orthogonal,

$$
\mathbf{c} = \sum_{i=1}^n \frac{\langle \mathbf{c}, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle}  \mathbf{\tilde b}_i = \mathbf{\tilde B} 
\left[ \frac{\langle \mathbf{c}, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} 
\right]_i
$$

Therefore,

$$
\left[ \frac{\langle \mathbf{c}, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle} 
\right]_i = \mathbf{\tilde B} ^{-1}\mathbf{c} = \mathbf{\tilde B} ^{-1}\mathbf{Bt} = \mathbf{Ut} = 
\left[ \sum_{j=1}^n \mathbf{U}_{ij} t_j \right]_i 
= 
\left[ t_i + \sum_{j>n} \mathbf{U}_{ij} t_j \right]_i
$$
{{< /math >}}



### Illustration from [DP16]

![Illustration](post/Finding-a-Close-Point-in-a-Lattice/illustration.png)

## Randomized Version [GPV08]
Given vector $\mathbf{c}$. Find vector $\mathbf{z} \in \mathcal{L}(\mathbf{B})$ which follows the distribution $D_{\mathcal{L}(\mathbf{B}),\sigma,\mathbf{c}}$

Let  $\mathbf{c}_n = \mathbf{c}$ and $\mathbf{v}_n = \mathcal{0}$ .

For $i = n,\cdots,1$:

1. $c_i' \gets \frac{\langle \mathbf{c}_i, \mathbf{\tilde b}_i \rangle}{\langle \mathbf{\tilde b}_i, \mathbf{\tilde b}_i \rangle}$, $\sigma_i=\frac{\sigma}{\| \mathbf{\tilde b}_i \|}$

2. Let $z_i \gets  D_{\mathbb{Z},\sigma_i',c_i'}$ 
3. {{< math >}}$\mathbf{c}_{i-1} \gets \mathbf{c}_i - z_i \mathbf{b}_i$ and $\mathbf{v}_{i-1} \gets \mathbf{v}_i + z_i \mathbf{b}_i$ {{< /math >}}

Output $\mathbf{v}_0$.

Note that Theorem 1 also holds.

### Theorem 2

{{< math >}}

For $\sigma \geq \|\mathbf{B}\| \cdot \omega(\sqrt{\log(n)}) = \max_{i} \|\mathbf{b}_i\| \cdot \omega(\sqrt{\log(n)})$,

$$
\mathbf{v}_0 \stackrel \Delta\sim D_{\mathcal{L}(\mathbf{B}),\sigma,\mathbf{c}}
$$

{{< /math >}}

#### Proof

{{< math >}}

Let $\mathbf{v}_0 = \sum_{i=1}^n z_i\mathbf{b}_i$ where $z_i \gets  D_{\mathbb{Z},\sigma_i',c_i'}$. Then the probability of outputing $\mathbf{v}_0$ is

$$
\prod_{i=1}^n D_{\mathbb{Z},\sigma_i',c_i'}(z_i) = \frac{\prod \rho_{\sigma_i',c_i'}(z_i)}{\prod \rho_{\sigma_i',c_i'}(\mathbb{Z})}
$$

Note that $\rho_{\sigma_i',c_i'}(x) = \rho_{\sigma_i'}(x-c_i') = \rho_\sigma(x \cdot \|\mathbf{\tilde b}_i\|)$. The numerator is equal to

$$
\begin{aligned}
\prod_{i=1}^n \rho_{\sigma_i', c_i'}(z_i) = \prod_{i=1}^n \rho_{\sigma}((z_i-c_i')\cdot\|\mathbf{\tilde b}_i\|) 
&= \rho_{\sigma} \left (\sum_{i=1}^n (z_i-c_i')\cdot \mathbf{\tilde b}_i\right)\\
&=\rho_\sigma(\mathbf{v}_0 - \mathbf{c}) \\
&= \rho_{\sigma,\mathbf{c}}(\mathbf{v}_0)
\end{aligned}
$$

{{< /math >}}

Now we need to prove that the denominator is close to $\rho_{\sigma,\mathbf{c}}(\mathcal{L}(\mathbf{B}))$. Recall the lemma from [smoothing parameter](https://www.notion.so/Smoothing-Parameter-for-Gaussian-Function-f8c424cb8f504dec8064fab8ca57023a?pvs=21), since $\sigma_i \geq \omega(\sqrt{\log(n)})$, $\sigma_i \geq \eta_\epsilon(\mathbb{Z})$ for negligible $\epsilon$ and 

{{< math >}}
$$
\rho_{\sigma_i',c_i'}(\mathbb{Z}) \in \left[ \frac{1-\epsilon}{1+\epsilon},1 \right]\cdot \rho_{\sigma_i'}(\mathbb{Z})
$$

Therefore,

$$
\prod \rho_{\sigma_i',c_i'}(\mathbb{Z}) \in \left[\left(\frac{1-\epsilon}{1+\epsilon} \right)^n, 1 \right] \cdot \prod \rho_{\sigma_i'}(\mathbb{Z})
$$

Notice that the range is now independent of $c_i'$ or $\mathbf{c}$. In particular, this shows the probability of outputting $\mathbf{v}_0$ is equal to

$$
\Pr[\mathbf{v}_0]=\frac{\rho_{\sigma,\mathbf{c}}(\mathbf{v}_0)}{\prod \rho_{\sigma_i',c_i'}(\mathbb{Z})} \in \rho_{\sigma,\mathbf{c}}(\mathbf{v}_0) \cdot \left[1,\left(\frac{1+\epsilon}{1-\epsilon} \right)^n\right] \cdot \frac{1}{\prod \rho_{\sigma_i'}(\mathbb{Z})}
$$

Since summing over all possible $\mathbf{v}_0$ is $1$, we know that

$$
\prod \rho_{\sigma_i'}(\mathbb{Z}) \in \left[1,\left(\frac{1+\epsilon}{1-\epsilon} \right)^n\right] \cdot \rho_{\sigma,\mathbf{c}}(\mathcal{L}(\mathbf{B}))
$$

Now the probability of outputting $\mathbf{v}_0$ is within $\left[\left(\frac{1-\epsilon}{1+\epsilon} \right)^n,\left(\frac{1+\epsilon}{1-\epsilon} \right)^n\right]$ of $D_{\mathcal{L}(\mathbf{B}),\sigma,\mathbf{c}}(\mathbf{v}_0)$. For negligible $\epsilon$, $\left(\frac{1+\epsilon}{1-\epsilon} \right)^n$ is also negligible, and we can show that the statistical distance is negligible.
{{< /math >}}


> Remark: This also implies that by calling this algorithm multiple times, one can get the “closest” point with high probability.