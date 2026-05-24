# MacroPlex — CTF Writeup

**Author:** AndreiCat  
**Category:** Malware  
**Points:** 50  
**Solves:** —  
**Flag:** `ZDTM{Mara.M@macroplex.com_docm_m4lw4re}`

---

## Description

The challenge provided a phishing-training email in `.eml` format. The goal was to parse the email headers, identify the attached macro-enabled Word document, and statically inspect the VBA project for the training answer.

The flag format was `ZDTM{Q1_Q2_Q3}`, hinting at three parts to find.

---

## Analysis

### Step 1 — Parse the email

The original file was `Training.eml`. Parsing its headers revealed the recipient address:

```
To: "Mara.M@macroplex.com" <Mara.M@macroplex.com>
```

This gives us **Q1**: `Mara.M@macroplex.com`

The MIME structure also showed one attachment: `Q2_report.docm` — a macro-enabled Word document.

The file extension gives us **Q2**: `docm`

### Step 2 — Inspect the attachment

`.docm` files are ZIP-based Office documents. Listing the archive contents revealed a VBA project binary:

```
word/vbaProject.bin
```

Statically reading the binary (decoded as `latin1`) and searching for readable strings revealed the following training message embedded in the VBA code:

```
Normally this contains malicious code. For educational purposes, the answer you seek is m4lw4re
```

This gives us **Q3**: `m4lw4re`

### Step 3 — Extract the flag with Python

```python
from email import policy
from email.parser import BytesParser
from pathlib import Path
from zipfile import ZipFile
import io, re

eml_path = Path("Training.eml")
msg = BytesParser(policy=policy.default).parsebytes(eml_path.read_bytes())

recipient = msg["to"].addresses[0].addr_spec

attachment_name = None
attachment_data = None
for part in msg.walk():
    if part.get_content_disposition() == "attachment":
        attachment_name = part.get_filename()
        attachment_data = part.get_payload(decode=True)
        break

extension = attachment_name.rsplit(".", 1)[1]

with ZipFile(io.BytesIO(attachment_data)) as docm:
    vba = docm.read("word/vbaProject.bin")

text = vba.decode("latin1", errors="ignore")
answer = re.search(r"answer you seek is ([A-Za-z0-9_{}@.-]+)", text).group(1)

print(f"ZDTM{{{recipient}_{extension}_{answer}}}")
```

Output:

```
ZDTM{Mara.M@macroplex.com_docm_m4lw4re}
```

---

## Steps Summary

1. Parse `Training.eml` — extract recipient address (`Mara.M@macroplex.com`) and attachment name (`Q2_report.docm`)
2. Unzip the `.docm` and read `word/vbaProject.bin`
3. Search the VBA binary for the hidden training string to get `m4lw4re`
4. Assemble the three parts into the flag

**Flag:** `ZDTM{Mara.M@macroplex.com_docm_m4lw4re}`
