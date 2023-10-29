---
title: Note on Number Theoretic Transform
date: 2022-06-18
updated: 2022-06-18
categories: "Note"
tags: 
   - "Algorithm"
   - "Cryptography"
---


# NTT

Github Repo: [python-NTT](https://github.com/kengyuchen/python-NTT.git)

# FFT

Recall for a sequence of signal $x=(x_0, x_1,\cdots,x_{N-1})$. The Fast-Fourier Transform does

$$
\hat{x} =FFT(x)= \left ( \sum_{n=0}^{N-1} x_n e^{-2 \pi i\frac{ n m}{N}} \right)_{0\leq m\leq N-1}
$$

{{< math >}}
$$
\begin{aligned}
\hat{x}_m = \sum_{n=0}^{N-1} x_n e^{-2 \pi i\frac{ n m}{N}}
&= \sum_{n=0}^{N/2-1}x_{2n} e^{-i2\pi \frac{(2n)m}{N}} + \sum_{n=0}^{N/2-1}x_{2n+1} e^{-i2\pi \frac{(2n+1)m}{N}} \\
&= \sum_{n=0}^{N/2-1}x_{2n} e^{-i2\pi \frac{nm}{N/2}} + e^{-i2\pi \frac{m}{N}} \sum_{n=0}^{N/2-1}x_{2n+1} e^{-i2\pi \frac{nm}{N/2}} \\
&= FFT( (x_{2n})_{0 \leq n\leq \frac{N}{2}-1} )_m + e^{-i2\pi \frac{m}{N}} FFT((x_{2n+1})_{0 \leq n\leq \frac{N}{2}-1})_m
\end{aligned}
$$

This allows using two $N/2$-length FFT to get the $N$-length FFT. The twiddle factor is


$$
\omega_N^m = e^{-i2\pi \frac{m}{N}}
$$


Let $y_n = FFT(x_{2n} ), y_n' = FFT(x_{2n+1} )$. Note two facts:

$$
y_n = y_{n+\frac{N}{2}},\quad y_n' = y_{n+\frac{N}{2}}' \\
\omega_N^{m+\frac{N}{2}} = -\omega_N^{m}
$$

Therefore,


$$
\hat{x}_{m} =
\begin{cases}
y_m + \omega_N^m y_m' & m \leq \frac{N}{2} \\
y_{m-\frac{N}{2}} - \omega_N^{m-\frac{N}{2}}y_{m-\frac{N}{2}}' & \frac{N}{2} < m \leq N
\end{cases}
$$

For example, let $N=8$,

$$
\hat{x}_0 = y_0 + \omega_8^0 y_0' \qquad \hat{x}_4 = y_0 - \omega_8^0 y_0' \\
$$

$$
\hat{x}_1 = y_1 + \omega_8^1 y_1' \qquad \hat{x}_5 = y_1 - \omega_8^1 y_1' \\
$$

$$
\hat{x}_2 = y_2 + \omega_8^2 y_2' \qquad \hat{x}_6 = y_2 - \omega_8^2 y_2' \\
$$

$$
\hat{x}_3 = y_3 + \omega_8^3 y_3' \qquad \hat{x}_7 = y_3 - \omega_8^3 y_3' \\
$$


Let 

$$
CT(a, b, \omega) \to (a + \omega b, a - \omega b)
$$

WIth bit-reversing order in 2 bits:

$$
(\hat{x}_0, \hat{x}_4) \gets CT(y_0, y_0', \omega_8^0) \\
$$
$$
(\hat{x}_2, \hat{x}_6) \gets CT(y_2, y_2', \omega_8^2) \\
$$
$$
(\hat{x}_1, \hat{x}_5) \gets CT(y_1, y_1', \omega_8^1) \\
$$
$$
(\hat{x}_3, \hat{x}_7) \gets CT(y_3, y_3', \omega_8^3) \\
$$
Let $z_n = FFT(y_{2n} ), z_n' = FFT(y_{2n+1} )$,

$$
(y_0, y_2) \gets CT(z_0, z_0', \omega_4^0) \\

(y_1, y_3) \gets CT(z_1, z_1', \omega_4^1) \\
$$

Finally,

$$
(z_0, z_1) \gets CT(x_0, x_4, \omega_2^0) \\
(z_0', z_1') \gets CT(x_2, x_6, \omega_2^0)
$$

{{< /math >}}

## Code

```python
import numpt as np

def CT_Butterfly(x_in, twiddle_factor, length):
	x_out = np.zeros(len(x_in), dtype = np.cdouble)
	if length <= 1:
		x_out[0] = x_in[0]
		return x_out
	halflen = length >> 1
	for i in range(halflen):
		a = x_in[i]
		bc = x_in[i+halflen] * twiddle_factor
		x_out[i] = a + bc
		x_out[i + halflen] = a - bc
	return x_out

def bit_reverse(x, n):
	if type(x) == int:
		b = '{:0{width}b}'.format(x, width=n)
		return int(b[::-1], 2)
	else:
		x_out = np.zeros(len(x), dtype = int)
		for i in range(len(x)):
			x_out[i] = x[bit_reverse(i, n)]
		return x_out
```

```python
for i in range(logN):
	n = 1 << i # number of butterflies in current layer
	seqlen = N // n # length of butterfly
	for j in range(n):
		twiddle_factor = w ** bit_reverse(j, logN-1) # bit-reverse j in N/2 bits
		x[seqlen*j: seqlen*(j+1)] = CT_Butterfly(x[seqlen*j: seqlen*(j+1)], twiddle_factor, seqlen)

x = [x[bit_reverse(i, logN)] for i in range(N)]
```

For inverse, just do the same operations but with twiddle factor as its inverse. Remember to multiply by $\frac{1}{N}$ at the end.

```python
for i in range(logN):
	n = 1 << i # number of butterflies in current layer
	seqlen = N // n # length of butterfly
	for j in range(n):
		twiddle_factor = w ** (-bit_reverse(j, logN-1))
		x[seqlen*j: seqlen*(j+1)] = CT_Butterfly(x[seqlen*j: seqlen*(j+1)], twiddle_factor, seqlen)

x = [x[bit_reverse(i, logN)] / N for i in range(N)]
```