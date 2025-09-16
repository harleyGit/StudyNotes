[**Curve25519椭圆加密算法**](#Curve25519椭圆加密算法)



<br/><br/><br/>

***
<br/>

> <h1 id="Curve25519椭圆加密算法">Curve25519椭圆加密算法</h1>

**Curve25519** 是一种椭圆曲线 Diffie–Hellman（ECDH）算法的实现，主要用来**安全地在双方之间协商出一个共享密钥**，然后这个密钥可以作为对称加密算法（如 AES）的密钥来加密/解密数据。


<br/>


**核心思想：**
1. 双方各自生成一个 32 字节的私钥（随机数）。
2. 各自根据私钥计算出对应的公钥（椭圆曲线上一点）。
3. 双方交换公钥。
4. 每一方用自己的私钥和对方的公钥计算出相同的共享密钥。

***
<br/>

**具体流程（Python 示例）**

> 这里使用 `cryptography` 库来演示 Curve25519 的密钥交换。

```bash
pip install cryptography
```

```python
from cryptography.hazmat.primitives.asymmetric import x25519
from cryptography.hazmat.primitives.ciphers.aead import ChaCha20Poly1305
import os

# 1️⃣ 生成双方的私钥
alice_private_key = x25519.X25519PrivateKey.generate()
bob_private_key   = x25519.X25519PrivateKey.generate()

# 2️⃣ 计算公钥
alice_public_key = alice_private_key.public_key()
bob_public_key   = bob_private_key.public_key()

# 3️⃣ 交换公钥并计算共享密钥
alice_shared_key = alice_private_key.exchange(bob_public_key)
bob_shared_key   = bob_private_key.exchange(alice_public_key)

assert alice_shared_key == bob_shared_key
print("共享密钥:", alice_shared_key.hex())

# 4️⃣ 用共享密钥进行对称加密（例如 ChaCha20-Poly1305）
key = alice_shared_key  # 32 字节，可直接作为密钥
nonce = os.urandom(12)  # ChaCha20 的随机 12 字节 nonce

aead = ChaCha20Poly1305(key)
ciphertext = aead.encrypt(nonce, b"Hello Curve25519!", None)
print("密文:", ciphertext.hex())

# 解密
plaintext = aead.decrypt(nonce, ciphertext, None)
print("解密后的明文:", plaintext.decode())
```

<br/>

运行后输出示例：

```
共享密钥: 0a1f3e... (32字节)
密文: 48eaf5...
解密后的明文: Hello Curve25519!
```

<br/>

**总结**

* Curve25519 只负责**安全生成共享密钥**，并不直接做数据加密。
* 生成的共享密钥通常用来派生对称密钥（AES、ChaCha20）。
* 它的优势是：高性能、抗侧信道攻击、安全参数成熟，被广泛用于 TLS、Signal、SSH 等协议。

有用GO和Swift写成的后端和移动端的代码，请看Go的`‌hg_safe_v0.go`和**Swift**的`‌HGCurveUtil.swift`文件。