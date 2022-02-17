# Encryption

## Encryption

Encryption is the process of converting data into a format in which the original content is not accessible. Encryption is reversible, i.e. it is possible to decrypt the ciphertext (encrypted data) and obtain the original content. Some classic examples of encryption are the [Caesar cipher](https://en.wikipedia.org/wiki/Caesar\_cipher), the [Bacon cipher](https://en.wikipedia.org/wiki/Bacon's\_cipher) and the [substitution cipher](https://en.wikipedia.org/wiki/Substitution\_cipher). Encryption algorithms are of two types: Symmetric and Asymmetric.

### Symmetric Encryption

Symmetric algorithms use one key or secret to encrypt the data and use the same key to decrypt it. A basic example of symmetric encryption is XOR.

```python
from pwn import xor
print(xor("Hello World", "secret"))
b';\x00\x0f\x1e\nT$\n\x11\x1e\x01'
```

In the above code, the plaintext is `Hello World`, and the key is `secret`. Anyone who has the key can decrypt the ciphertext and get the plaintext.

```python
from pwn import xor
print(xor(';\x00\x0f\x1e\nT$\n\x11\x1e\x01', "secret"))
b'Hello World'
```

The `b` in the above outputs denotes a byte string. This distinction was not made before `Python3`.

Other examples of symmetric algorithms are [AES](https://en.wikipedia.org/wiki/Advanced\_Encryption\_Standard), [DES](https://en.wikipedia.org/wiki/Data\_Encryption\_Standard), [3DES](https://en.wikipedia.org/wiki/Triple\_DES) and [Blowfish](https://en.wikipedia.org/wiki/Blowfish\_\(cipher\)#The\_algorithm). These algorithms may be vulnerable to attacks such as brute-force keying, [frequency analysis](https://en.wikipedia.org/wiki/Frequency\_analysis), [fill-in oracle attack](https://en.wikipedia.org/wiki/Padding\_oracle\_attack), etc.

### Asymmetric Encryption

On the other hand, asymmetric algorithms divide the key into two parts (i.e., public and private). The public key can be given to anyone who wishes to encrypt information and transmit it securely to the owner. The owner then uses his private key to decrypt the content. Examples of asymmetric algorithms include [RSA](https://en.wikipedia.org/wiki/RSA\_\(cryptosystem\)), [ECDSA](https://en.wikipedia.org/wiki/Elliptic\_Curve\_Digital\_Signature\_Algorithm) and [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman\_key\_exchange).

One of the most prominent uses of asymmetric encryption is the `secure hypertext transfer protocol` (`HTTPS`) in the form of **Secure Sockets Layer** (`SSL`). When a client connects to a server hosting an `HTTPS` website, a public key exchange takes place. The client browser uses this public key to encrypt any data sent to the server. The server decrypts the incoming traffic before passing it to the processing service.
