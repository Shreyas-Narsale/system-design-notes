## Encoding and Decoding

### Encoding
Encoding means converting data into another format.

### Decoding
Decoding is the reverse process — converting encoded data back to its original form.

### Important
- Encoding ≠ Encryption
- Encoding is not for security, it’s for compatibility.

Examples: Base64, ASCII, Unicode.

---

## Base64
Base64 is an encoding scheme that uses 64 characters:
- A–Z → 26
- a–z → 26
- 0–9 → 10
- + / → 2
- = → padding

Total = 64 characters

---

## Base64 Encoding Works (Step-by-Step)
Let’s encode the text: `"Man"`

### Step 1: Convert characters to ASCII (binary)

| Character | ASCII | Binary (8-bit) |
|-----------|-------|----------------|
| M         | 77    | 01001101       |
| a         | 97    | 01100001       |
| n         | 110   | 01101110       |

Combined bits:
```
010011010110000101101110
```

### Step 2: Split into 6-bit chunks
Why 6-bit chunks? for 64 unique keys, needed 6 bits : 2^6 = 64

```
010011 010110 000101 101110
```

### Step 3: Convert each 6-bit chunk to decimal
- 010011 → 19
- 010110 → 22
- 000101 → 5
- 101110 → 46

### Step 4: Map to Base64 characters
- 19 → T
- 22 → W
- 5  → F
- 46 → u

Encoded result:
```
TWFu
```

## Base64 Decoding (Reverse Process)

