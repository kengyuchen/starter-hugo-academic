---
title: Barrett Reduction and Montgomery Reduction
date: 2022-10-31
updated: 2023-04-20
categories: "Note"
tags: 
   - "Algorithm"
   - "Cryptography"
---

Given a variable $a$ and a prime number $q$, we want to securely compute $a \text{ mod } q$.

The Barrett reduction is usually used in addition.

The Montgomery reduction is usually used in multiplication and reduction.

<!--more-->

## Barrett Reduction

For unsigned variable $a$, we use the formula:

$$
a \text{ mod }q = a - \lfloor a/q \rfloor \cdot q
$$

We approximate the value $\lfloor a/q \rfloor$ by multiplication and shift, which is much cheaper.

$$
\lfloor a/q \rfloor \approx \lfloor a \cdot \lfloor 2^k/q \rfloor / 2^k \rfloor
$$

One can pre-compute $\lfloor 2^k / q \rfloor$ , and the operation $\lfloor \cdot /2^k \rfloor$ is just a $k$ bit shift. That is,

$$
a \text{ mod }q \approx a - (( a \cdot \lfloor 2^k/q \rfloor ) \gg k) \cdot q
$$

For signed variable, we use

$$
a \text{ mod }q = a - \lfloor a/q \rceil \cdot q \approx a - \lfloor a \cdot \lfloor 2^k/q \rceil / 2^k \rceil \cdot q
$$

Note that the operation $\lfloor \cdot /2^k \rceil$ is equal to $(\cdot + 2^{k-1}) \gg k$, so we derive

$$
a \text{ mod }q \approx a - (( a \cdot \lfloor 2^k/q \rceil + 2^{k-1} ) \gg k) \cdot q
$$

The Barrett reduction formula is just an approximation, so there is a limited range for the input variable.

### Algorithm (Unsigned)

Given $a, q$, output approximation of $n,r$ such that $a = nq + r$.

- Pre-compute $c = \lfloor 2^k/q \rfloor$
- $n\gets (a \cdot \lfloor 2^k/q \rfloor) \gg k$
- $r \gets a - n\cdot q$

### Arm Cortex-M Assembly

- Unsigned 32-bit Barrett reduction (pre-compute number $c = \lfloor 2^k / q \rfloor$
    
    ```wasm
    mul t, a, c
    lsr t, t, #k
    mls a, t, q, a ; a = a - t*q
    ```
    
- Unsigned 64-bit Barrett reduction (pre-compute number $c = \lfloor 2^k / q \rfloor$
    
    ```wasm
    umull tl, th, a, c
    (lsr th, th, #(k-32))
    mls a, th, q, a ; a = a - th*q
    ```
    
- Signed 32-bit Barrett reduction (pre-compute number $c = \lfloor 2^{32} / q \rceil$
    
    ```wasm
    smmulr t, a, c ; t = (a*c + 2^31) >> 32
    mls a, t, q, a ; a = a - t*q
    ```
    

## Montgomery Reduction

Instead of output $a \text{ mod }q$, we output $aR^{-1} \text{ mod } q$, and later multiply by $R$. For efficiency we choose $R = 2^k > q$.

Given $a$, we first compute a number $t$ such that $R$ divide $a + t\cdot q$. The value $t$ is derived by

$$
t = -q^{-1}a \text{ mod } R
$$

The Montgomery reduction does the following operation:

$$
a \xmapsto[]{} (a + t\cdot q) \gg k
$$

Note that this output is equal to $aR^{-1} \text{ mod }q$.

### Algorithm

Given $a, q$, output $r$ such that $aR^{-1} \text{ mod }q = r$

- Pre-compute $q^{-1} \text{ mod }R$
- $t\gets -q^{-1} \cdot a \text{ mod } R$
- $r \gets (a + t\cdot q) \gg k =  \frac{a + t\cdot q}{R}$

### Montgomery Multiplication

Recall that ${\sf MR}(a) = aR^{-1} \text{ mod }q$. We do not need to correct the value at once.

Let the Montgomery domain for a variable $a$ be the value $aR \text{ mod }q$. To turn to Montgomery domain, one just multiply by $R$ ; to turn a Montgomery domain number back, one just do a Montgomery reduction.

Now, suppose we want to compute $a \cdot b \text{ mod }q$, we first turn them into Montgomery domain to get $aR$ and $bR$, do the multiplication, and reduce the product by Montgomery reduction. The result becomes

$$
{\sf MR}(aR \cdot bR) = abR
$$

which is the product in Montgomery domain!

In short, to do a series of multiplication, one can

1. Convert all variables into Montgomery domain by.
    - $a \mapsto aR \text{ mod } q$
2. Perform multiplications in Montgomery domain; i.e. a normal multiplication followed by a Montgomery reduction.
    - ${\sf MR}(aR \cdot bR) = abR \text{ mod }q$
3. Convert back to normal domain by simply doing one Montgomery reduction (or multiplication with 1)
    - ${\sf MR}(aR) = {\sf MR}(aR \cdot 1) = a \text{ mod }q$

### Ranges

For unsigned input, 

- $a < qR$
- $t=-q^{-1}a \text{ mod } R \in [0, R-1]$
- The output of Montgomery reduction is in $[0, 2q]$

For signed input,

- $a \in [-(R-1)q/2, (R-1)q/2]$
- $t=-q^{-1}a \; \text{mod}^\pm \; R \in [-R/2+1, R/2-1]$
- The output of Montgomery reduction is in $[-q, +q]$.

### Arm Cortex-M Assembly

Assume the value $a, b$ are already in Montgomery domain, and we pre-compute the value $q'=-q^{-1} \text{ mod }R$.

- Signed 32-bit Montgomery multiplication
    
    ```wasm
    smull tl, th, a, b
    mul t2, tl, q'
    smlal tl, th, t2, q ; the result is in th
    ```
    
- Signed 16-bit Montgomery reduction
    
    ```wasm
    smulbb t, a, b
    smulbb t2, t, q'
    smlabb t2, t2, q, t
    asr t2, t2, #16
    ```