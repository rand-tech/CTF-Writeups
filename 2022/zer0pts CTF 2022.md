---
author: "rand0m"
title: "zer0pts CTF 2022 Writeup"
date: "2022-03-22"
description: "Write up for zer0pts CTF 2022 / crypto / Anti-Fermat"
tags: ["crypto", "CTF", "zer0pts CTF", "Writeup"]
categories: ["CTF", "Writeup","Crypto"]
draft: false
math: true
---

This article offers a writeup for the zer0pts CTF 2022's crypto challenge, "Anti-Fermat."

<!--more-->


## crypto
### Anti-Fermat
description: `I invented Anti-Fermat Key Generation for RSA cipher since I'm scared of the Fermat's Factorization Method.`

#### files: 
1. task.py
```python
from Crypto.Util.number import isPrime, getStrongPrime
from gmpy import next_prime
from secret import flag

# Anti-Fermat Key Generation
p = getStrongPrime(1024)
q = next_prime(p ^ ((1<<1024)-1))
n = p * q
e = 65537

# Encryption
m = int.from_bytes(flag, 'big')
assert m < n
c = pow(m, e, n)

print('n = {}'.format(hex(n)))
print('c = {}'.format(hex(c)))
```

2. output.txt
_(the output of the task.py)_

---

#### What we know
- $n$ ($=p\times{}q$)
- $p$ is a strong prime
- $n = \left(\dfrac{p+q}{2}\right)^2 - \left(\dfrac{p-q}{2}\right)^2$ ([Fermat's factorization method](https://en.wikipedia.org/wiki/Fermat's_factorization_method))
- $m < n$

#### 
By experimenting, I realized that the value of `p - (q^((1<<1024)-1)) - 1`
and `p + q - (1<<1024)` are the same and both were small. 
- $p+q \approx 1\ll{}1024$

Thus, using the Fermat's factorization method ($n = \left((p+q)/2\right)^2 - \left((p-q)/2\right)^2$), we get the following approximation.
$$p \approx \dfrac{(1\ll{}1024) + \sqrt{(1\ll{}1024)^2 - 4n}}{2}$$

After checking prime numbers near the approximation using the `next_prime` function, we get the actual `p` and `q`.

