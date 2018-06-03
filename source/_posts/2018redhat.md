---
title: 红帽杯2018部分writeup
date: 2018-05-11 01:05:15
tags:
- ctf
- redhat
- pwn
- reverse
- crypto
categories: ctf
---
熬了一天，感觉pwn和re的水平退步不少。

### PWN

这次比赛中的pwn的题目难度一般，3个pwn分别涉及到
- 栈溢出 ——game server
- null byte offset-by-one —— shellcode manager
- 格式化串漏洞 —— Starcraft RPG

中间在漏洞利用过程中也踩到了一些坑（可能是长期没打没有手感，有时间会整理一下）。
<!-- more -->
### RE

re只看了最简单的icm，感觉自己re方面真是非常菜，赛后重新整理了getFlag的脚本，进行了简化：

``` python
#!/usr/bin/env python
'''
    pip install cryptography
'''
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend


def getCipherAlgorithm(key, mode):
    backend = default_backend()
    cipher = Cipher(algorithms.IDEA(key), mode, backend=backend)
    return cipher

def encrypt(plainMsg, key, mode):
    cipher = getCipherAlgorithm(key, mode)
    encryptor = cipher.encryptor()
    res = encryptor.update(plainMsg) + encryptor.finalize()
    return res

def decrypt(cipherMsg, key, mode):
    cipher = getCipherAlgorithm(key, mode)
    decryptor = cipher.decryptor()
    res = decryptor.update(cipherMsg) + decryptor.finalize()
    return res

def getKey():
    from ctypes import *
    res = []
    buf = ""
    libc = cdll.LoadLibrary("libc.so.6")
    libc.srand(0x78C819C3)
    for i in range(16):
        buf += "{:02x}".format(libc.rand() % 256).decode('hex')
    return buf

def getCipherText():
    secret = [
        0xd0,  0xe0,  0xab,  0x9c,  0xcd,  0x78,  0x5b,  0x54,
        0x3d,  0xe4,  0xea,  0x33,  0x51,  0x44,  0x6d,  0x3c,
        0x4e,  0xce,  0xdf,  0xb5,  0x41,  0x0,  0x1c,  0xec,
        0xe3,  0x1b,  0xc3,  0x8c,  0x91,  0x25,  0x7f,  0x1b,
        0x60,  0xfe,  0x35,  0x9c,  0xea,  0x4,  0x4c,  0x87,
        0x8d,  0x97,  0x93,  0x5c,  0xb8,  0x9a,  0x70,  0x75,
    ]
    buf = ""
    for i in range(len(secret)):
        secret[i] = (119-i) ^ secret[i]
    for i in range(len(secret)):
        secret[i] = secret[i] ^ (8 - i%8)
        buf += "{:02x}".format(secret[i]).decode('hex')

    return buf

def getFlag():
    cipherText = getCipherText()
    key = getKey()
    mode = modes.ECB()
    msg = decrypt(cipherText, key, mode)
    print msg

if __name__ == '__main__':
    getFlag()
```

wcm解密脚本：

```python
'''
SM4 from https://github.com/yixiangzhike/AlgorithmSM
'''
from SM4 import *
from ctypes import *


def getKey():
    key = ""
    libc = cdll.msvcrt
    libc.srand(0x2872DD1B)
    for i in range(16):
        key += "{:02x}".format(libc.rand() % 256)
    return key.decode('hex')


def getCipherText():
    secret = [
        0xf4,  0x88,  0x91,  0xc2,  0x9b,  0x20,  0x5b,  0x3,
        0xf1,  0xed,  0xf6,  0x13,  0x46,  0x3c,  0x55,  0x81,
        0x61,  0xf,  0xff,  0x14,  0x6e,  0x1c,  0x48,  0x28,
        0x79,  0x9f,  0x85,  0xaf,  0xc5,  0x58,  0xd,  0xd6,
        0xa5,  0xd9,  0x64,  0xfd,  0x46,  0x9,  0x8c,  0xdf,
        0x3b,  0xa5,  0x37,  0x62,  0x5a,  0xa6,  0xd2,  0x4b,
    ]
    v9 = 51
    cipherText = ""
    while v9 - 51 < 48:
        cipherText += "{:02x}".format(secret[v9-51] ^ v9)
        v9 += 1
    return cipherText.decode('hex')


key = getKey().encode('hex')
print "key is {}".format(key)
cipherText = getCipherText().encode('hex')
print "cipherText is {}".format(cipherText)
sm4 = SM4(key=key)
msg = sm4.sm4_decrypt(cipherText, SM4_ECB)
print msg.decode('hex')
```

### Crypto

3dlight，当时就考虑用解多元一次方程组的方法来求解，当时搞不动了，后来抽空写了下生成系数矩阵的脚本，迎刃而解

``` python
import numpy as np
import os


def str2arr(str):
    return [[[(ord(str[i * 8 + j]) >> k & 1) for k in xrange(8)] for j in xrange(8)] for i in xrange(8)]


def str2arr_rev(arr):
    ret = ""
    for i in xrange(8):
        for j in xrange(8):
            ret += chr(int(''.join(map(str, arr[i][j][::-1])), 2))
    return ret


def arr2str(arr):
    ret = ''
    for i in xrange(8):
        for j in xrange(8):
            for k in xrange(8):
                ret += chr(arr[i][j][k])
    return ret


def arr2str_rev(str2):
    ret = [[[0 for k in xrange(8)] for j in xrange(8)] for i in xrange(8)]
    for i in xrange(8):
        for j in xrange(8):
            for k in xrange(8):
                ret[i][j][k] = ord(str2[i*64+j*8+k])
    return ret


def check(x, y, z):
    if x < 0 or x > 7 or y < 0 or y > 7 or z < 0 or z > 7:
        return False
    return True


def light(arr, i, j, k, x, y, z, power): # square
    if check(i + x, j + y, k + z):
        arr[i + x][j + y][k + z] += power  # top right
    if x != 0 and check(i - x, j + y, k + z):
        arr[i - x][j + y][k + z] += power #
    if y != 0 and check(i + x, j - y, k + z):
        arr[i + x][j - y][k + z] += power
    if z != 0 and check(i + x, j + y, k - z):
        arr[i + x][j + y][k - z] += power
    if x != 0 and y != 0 and check(i - x, j - y, k + z):
        arr[i - x][j - y][k + z] += power
    if x != 0 and z != 0 and check(i - x, j + y, k - z):
        arr[i - x][j + y][k - z] += power
    if y != 0 and z != 0 and check(i + x, j - y, k - z):
        arr[i + x][j - y][k - z] += power
    if x != 0 and y != 0 and z != 0 and check(i - x, j - y, k - z):
        arr[i - x][j - y][k - z] += power


def encrypt(flag, power):
    ret = [[[0 for _ in xrange(8)] for _ in xrange(8)] for _ in xrange(8)]
    lights = str2arr(flag)
    for i in range(8):
        for j in range(8):
            for k in range(8):
                if lights[i][j][k] == 1: # bit is 1
                    for x in range(power):
                        for y in range(power - x):
                            for z in range(power - x - y):
                                light(ret, i, j, k, x, y, z, power - x - y - z)
    return arr2str(ret)


def getMatrix():
    A = [[ 0 for i in range(8**3)] for j in range(8**3)]
    for row in range(len(A)):
        ord_x = row / 8**2
        ord_y = (row - ord_x*(8**2))/8
        ord_z = row % 8
        A[row][ord_x * 8**2 + ord_y * 8 + ord_z] = 2
        if ord_x > 0:
            A[row][(ord_x-1)* 8**2 + ord_y*8 + ord_z] = 1
        if ord_x < 7:
            A[row][(ord_x+1)* 8**2 + ord_y*8 + ord_z] = 1
        if ord_y > 0:
            A[row][ord_x* 8**2 + (ord_y-1) *8 + ord_z] = 1
        if ord_y < 7:
            A[row][ord_x* 8**2 + (ord_y+1) *8 + ord_z] = 1
        if ord_z > 0:
            A[row][ord_x * 8**2 + ord_y * 8 + ord_z -1 ] = 1
        if ord_z < 7:
            A[row][ord_x * 8**2 + ord_y * 8 + ord_z + 1] = 1
    return np.array(A)


def getCipher():
    flag = "flag{abcdefg_hij_klm_nop_qrst_uvwxyz_0123456789_1234567890__xyz}"
    shuffle_flag = ''.join(flag[0::2][i] + flag[-1::-2][i] for i in xrange(32))
    cipher = encrypt(shuffle_flag, 2)
    CipherArr =arr2str_rev(cipher)
    return np.array(CipherArr).flatten()


def decrypt():
    solvea = np.linalg.solve(getMatrix(), getCipher())
    t = []
    for a in solvea:
        if abs(a) < 0.0001:
            t.append(0)
        else:
            t.append(1)
    flag = ""
    for i in range(0, len(t), 8):
        flag += chr(int(''.join(map(str, t[i:i+8]))[::-1], 2))
    return ''.join(flag[0::2][i] + flag[-1::-2][i] for i in xrange(32))


print decrypt()
```
