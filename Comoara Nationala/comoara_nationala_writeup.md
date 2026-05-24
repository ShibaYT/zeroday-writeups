# Comoara Nationala — CTF Writeup

**Author:** AndreiCat  
**Category:** Crypto  
**Points:** 500  
**Solves:** 6  
**Flag:** `ZDTM{bravo_tinere_sunt_mandru_de_tine}`

---

## Description

> Te-ai pregatit pentru bac, tinere?  
> Daca da vei obtine fl     agul mai repede decat ai zice peste.  
> Inainte sa pui intrebari, tinere, respecta regulile gramaticii limbii romane.

The description hints at Romanian high school literature ("bac" = Baccalaureate exam), pointing us toward classic Romanian literary works studied in school.

---

## Files

- `flag.zip` — 253 B, password-protected ZIP containing the flag
- `opere.txt` — 281 B, a list of encoded references

---

## Analysis

The contents of `opere.txt` are:

```
B:14:50:8
B:3:17:2
PLHA:1:20:3
MCN:8:115:7
LTI:1:1:5
MCN:15:36:1
B:6:42:4
PLHA:1:109:1
MCN:1:6:5
LTI:1:3:2
PLHA:1:67:1
LTI:1:3:1
LTI:1:2:3
LTI:1:2:1
MCN:3:4:3
B:10:77:6
LTI:1:1:3
B:11:70:4
PLHA:1:73:5
MCN:13:92:2
PLHA:1:345:5
B:1:6:7
PLHA:1:211:3
B:12:24:3
```

Each line follows the format: `BOOK:X:Y:Z`

The challenge title ("Comoara Nationala" = National Treasure) and the hint about Romanian grammar strongly suggest this is a **book cipher** using classic Romanian literary works required for the Baccalaureate exam.

The abbreviations map to:

| Code | Work |
|------|------|
| `B` | **Baltagul** by Mihail Sadoveanu |
| `PLHA` | **Povestea lui Harap-Alb** by Ion Creangă |
| `MCN` | **Moara cu Noroc** by Ioan Slavici |
| `LTI` | **Leoaică tânără, iubirea** by Nichita Stănescu |

These are all canonical texts on the Romanian Baccalaureate syllabus — exactly what the description was hinting at.

Since the challenge explicitly mentioned Romanian grammar rules and the bac, I immediately suspected the abbreviations referred to works from the Bac syllabus. After identifying them, I found all the texts on [cartipdf.io](https://cartipdf.io/), which hosts free PDFs of Romanian literature.

---

## Solving the Book Cipher

The format `BOOK:X:Y:Z` decodes as: **the Z-th letter of the Y-th word in chapter X** of the given book.

Decoding each line:

| Reference | Decoded Word |
|-----------|-------------|
| `B:14:50:8` | e |
| `B:3:17:2` | u |
| `PLHA:1:20:3` | a |
| `MCN:8:115:7` | m |
| `LTI:1:1:5` | i |
| `MCN:15:36:1` | v |
| `B:6:42:4` | i |
| `PLHA:1:109:1` | t |
| `MCN:1:6:5` | c |
| `LTI:1:3:2` | u |
| `PLHA:1:67:1` | v |
| `LTI:1:3:1` | i |
| `LTI:1:2:3` | n |
| `LTI:1:2:1` | t |
| `MCN:3:4:3` | e |
| `B:10:77:6` | p |
| `LTI:1:1:3` | o |
| `B:11:70:4` | t |
| `PLHA:1:73:5` | r |
| `MCN:13:92:2` | i |
| `PLHA:1:345:5` | v |
| `B:1:6:7` | i |
| `PLHA:1:211:3` | t |
| `B:12:24:3` | e |

Concatenating all decoded words gives:

```
euamivitcuvintepotrivite
```

Which reads: **"eu am ivit cuvinte potrivite"** — meaning *"I have conjured fitting words"*.

---

## Getting the Flag

The decoded string `euamivitcuvintepotrivite` is the password for `flag.zip`.

```bash
unzip -P euamivitcuvintepotrivite flag.zip
```

This extracts the flag file, revealing:

```
ZDTM{bravo_tinere_sunt_mandru_de_tine}
```

*("Bravo, young one, I am proud of you")*

---

## Summary

1. Recognize the book cipher format `BOOK:chapter:line:word_index`
2. Identify the four Romanian classic works from their abbreviations (Bac knowledge required!)
3. Look up each reference in the corresponding text to extract a letter/word
4. Concatenate to get the ZIP password: `euamivitcuvintepotrivite`
5. Unzip to get the flag

**Flag:** `ZDTM{bravo_tinere_sunt_mandru_de_tine}`
