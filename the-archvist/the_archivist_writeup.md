# the archivist — CTF Writeup

**Author:** exuletz  
**Category:** Crypto  
**Points:** 186 (55 + 10 + 18 + 11 + 92)  
**Solves:** 10  
**Flags:** 5/5

---

## Description

> this guy is clever, trust

A series of 5 progressive crypto challenges, each one layering a new encryption scheme on top of the last. The archivist thinks he's clever — let's prove otherwise.

---

## Challenge 1 — XOR (55 pts)

> The archivist claimed his encryption was impossible to reverse.

**Ciphertext:**
```
392b322b1e3d2c3d392a2a2a283c39352624312623343a312b2e28392c313c26351b
```

The hex-encoded ciphertext is 34 bytes long — suspiciously close to the length of a `ZDTM{...}` flag. This screams **XOR cipher**.

Since we know the flag starts with `ZDTM{`, we XOR the first 5 bytes of the ciphertext against `ZDTM{` to recover the first 5 bytes of the key:

```python
ct = bytes.fromhex('392b322b1e3d2c3d392a2a2a283c39352624312623343a312b2e28392c313c26351b')
known = b'ZDTM{'
key_start = bytes([ct[i] ^ known[i] for i in range(5)])
# => b'coffe'
```

The key starts with `coffe` — almost certainly `coffee`. Testing it:

```python
key = b'coffee'
flag = bytes([ct[i] ^ key[i % len(key)] for i in range(len(ct))])
# => ZDTM{XOR_LOOKS_SCARIER_THAN_IT_IS}
```

**Flag 1:** `ZDTM{XOR_LOOKS_SCARIER_THAN_IT_IS}`

---

## Challenge 2 — Hex + Base64 (10 pts)

> The archivist thought he learned from his previous mistake. Did hex?

**Ciphertext:**
```
576b525554587446546b4e5052456c4f5231394a5531394f5431526652553544556c6c5156456c50546e303d
```

The description hints at hex. Decoding the hex first:

```python
step1 = bytes.fromhex('576b525554587446546b4e5052456c4f...').decode()
# => WkRUTXtFTkNPRElOR19JU19OT1RfRU5DUllQVElPTn0=
```

That trailing `=` is a dead giveaway — Base64. Decoding:

```python
import base64
flag = base64.b64decode(step1).decode()
# => ZDTM{ENCODING_IS_NOT_ENCRYPTION}
```

The flag itself is the lesson: encoding is not encryption.

**Flag 2:** `ZDTM{ENCODING_IS_NOT_ENCRYPTION}`

---

## Challenge 3 — Vigenère (18 pts)

> A handwritten poem was found beside this one.

**Ciphertext:** `KXVQ{RJTVNL_AYPXRZ_CZICMU_CVDJLVHPLMEE}`

The poem is the first stanza of **Luceafărul** by Mihai Eminescu — a canonical Romanian poem. The key is the title: `LUCEAFARUL`.

To confirm, we recover the key from the known flag prefix `ZDTM`:

```python
cipher = 'KXVQ'
plain  = 'ZDTM'
key = [chr((ord(c) - ord(p)) % 26 + ord('A')) for c, p in zip(cipher, plain)]
# => ['L', 'U', 'C', 'E'] — start of LUCEAFARUL ✓
```

Decrypting with Vigenère:

```python
def vigenere_decrypt(ct, key):
    result = []
    ki = 0
    for c in ct:
        if c.isalpha():
            p = chr((ord(c) - ord('A') - (ord(key[ki % len(key)]) - ord('A'))) % 26 + ord('A'))
            result.append(p)
            ki += 1
        else:
            result.append(c)
    return ''.join(result)

flag = vigenere_decrypt('KXVQ{RJTVNL_AYPXRZ_CZICMU_CVDJLVHPLMEE}', 'LUCEAFARUL')
# => ZDTM{RETETA_PENTRU_CIORBA_ARDELENEASCA}
```

**Flag 3:** `ZDTM{RETETA_PENTRU_CIORBA_ARDELENEASCA}`

---

## Challenge 4 — RSA (11 pts)

> The archivist finally embraced modern cryptography.  
> "Classical ciphers are toys, small ones." he wrote.

**Given:**
```
n = 9408759790383105133...
c = 5678860086285826141...
```

The hint "small ones" refers to a **small public exponent**. No `e` is given, which strongly suggests `e = 3`. If `e = 3` and the message is short enough that `m^3 < n`, then `c = m^3` exactly — no modular reduction occurs, so we can simply take the integer cube root of `c`:

```python
import gmpy2
m, exact = gmpy2.iroot(c, 3)
# exact = True
flag = bytes.fromhex(hex(m)[2:]).decode()
# => ZDTM{FAST_ENCRYPTION_CAN_BE_A_VERY_SLOW_IDEA}
```

The cube root was exact, confirming `e = 3` and no padding — a classic RSA textbook mistake.

**Flag 4:** `ZDTM{FAST_ENCRYPTION_CAN_BE_A_VERY_SLOW_IDEA}`

---

## Challenge 5 — Columnar Transposition (92 pts)

> "I am too old for this.  
> I'll give you the flag, just help me encrypt this message using the 4 column thingy.  
> Forgot it's name. Btw, X"  
> 
> `JUSTMOVELETTERSNOTTHEMEANINGNOW`

The "4 column thingy" is a **columnar transposition cipher** with 4 columns. The `X` is the null padding character.

Write the plaintext into a grid of 4 columns (padding with `X` to fill):

```
J U S T
M O V E
L E T T
E R S N
O T T H
E M E A
N I N G
N O W X
```

Then read off column by column:

```
Col 1: J M L E O E N N
Col 2: U O E R T M I O
Col 3: S V T S T E N W
Col 4: T E T N H A G X
```

```python
msg = "JUSTMOVELETTERSNOTTHEMEANINGNOWX"  # padded to 32 chars
rows = [msg[i:i+4] for i in range(0, 32, 4)]
ct = ''.join(rows[r][c] for c in range(4) for r in range(len(rows)))
# => JMLEOENNUOERTMIOSVTSTENWTETNHAGX
```

**Flag 5:** `ZDTM{JMLEOENNUOERTMIOSVTSTENWTETNHAGX}`

---

## Summary

| # | Cipher | Key/Trick | Flag |
|---|--------|-----------|------|
| 1 | XOR (repeating key) | Key = `coffee` | `ZDTM{XOR_LOOKS_SCARIER_THAN_IT_IS}` |
| 2 | Hex → Base64 | Two layers of encoding | `ZDTM{ENCODING_IS_NOT_ENCRYPTION}` |
| 3 | Vigenère | Key = `LUCEAFARUL` (Eminescu poem) | `ZDTM{RETETA_PENTRU_CIORBA_ARDELENEASCA}` |
| 4 | RSA (e=3, no padding) | Cube root attack | `ZDTM{FAST_ENCRYPTION_CAN_BE_A_VERY_SLOW_IDEA}` |
| 5 | Columnar transposition (4 cols) | Read column by column, pad with X | `ZDTM{JMLEOENNUOERTMIOSVTSTENWTETNHAGX}` |
