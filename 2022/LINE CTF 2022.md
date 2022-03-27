---
author: "rand0m"
title: "LINE CTF 2022 Writeup"
date: 2022-03-27T09:02:37+09:00
description: "Write up for LINE CTF 2022 / crypto / ss-puzzle"
tags: ["crypto", "CTF", "LINE CTF", "Writeup"]
categories: ["CTF", "Writeup", "Crypto"]
draft: false
---

This article offers a writeup for the LINE CTF 2022's crypto challenge, "ss-puzzle."

<!--more-->

## Crypto
### ss-puzzle

description:
> I had stored this FLAG securely in five separate locations. However, three of the shares were lost and one was partially broken. Can you restore flag?`

#### files:
1. `ss_puzzle.py`
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# 64 bytes
FLAG = b'LINECTF{...}'

def xor(a:bytes, b:bytes) -> bytes:
    return bytes(i^j for i, j in zip(a, b))


S = [None]*4
R = [None]*4
Share = [None]*5

S[0] = FLAG[0:8]
S[1] = FLAG[8:16]
S[2] = FLAG[16:24]
S[3] = FLAG[24:32]

# Ideally, R should be random stream. (Not hint)
R[0] = FLAG[32:40]
R[1] = FLAG[40:48]
R[2] = FLAG[48:56]
R[3] = FLAG[56:64]

Share[0] = R[0]            + xor(R[1], S[3]) + xor(R[2], S[2]) + xor(R[3],S[1])
Share[1] = xor(R[0], S[0]) + R[1]            + xor(R[2], S[3]) + xor(R[3],S[2])
Share[2] = xor(R[0], S[1]) + xor(R[1], S[0]) + R[2]            + xor(R[3],S[3])
Share[3] = xor(R[0], S[2]) + xor(R[1], S[1]) + xor(R[2], S[0]) + R[3]
Share[4] = xor(R[0], S[3]) + xor(R[1], S[2]) + xor(R[2], S[1]) + xor(R[3],S[0])


# This share is partially broken.
Share[1] = Share[1][0:8]   + b'\x00'*8       + Share[1][16:24] + Share[1][24:32]

with open('./Share1', 'wb') as f:
    f.write(Share[1])
    f.close()

with open('./Share4', 'wb') as f:
    f.write(Share[4])
    f.close()
```
2. Share1
```
b"%$>*1 '\x15\x00\x00\x00\x00\x00\x00\x00\x00+\x07\x19\x07:\r,/\x02\x14%\x1c\t@H\x13"
```
3. Share4
```
b'\x1d\x08\x08\x1b-\x1d\x121\x0316\x1e3\x19\x06\x1c\x06\x07\x00\x1b:\x0f1\x1f934)&ug\x06'
```

---

#### What we know
- the values of 
  - $R[0] \oplus S[0]$ ,  $R[2] \oplus S[3]$ , $R[3] \oplus S[2]$,  $R[0] \oplus S[3]$ , $R[1] \oplus S[2]$ , $R[2] \oplus S[1]$ , $R[3] \oplus S[0]$
  - `S[0] = FLAG[0:8] = b"LINECTF{"`

Since we know that the value of $S[0]$ is `b"LINECTF{"`, we get the value of $R[0]$ and $R[3]$[^1].
[^1]: $\because{} (P \oplus K) \oplus K = P$

$$R[0] = (R[0] \oplus S[0]) \oplus S[0]$$
$$R[3] = (R[3] \oplus S[0]) \oplus S[0]$$
And by repeating the above steps, we get the full flag.
