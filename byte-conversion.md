#  Memory Allocation 

This guide explains **how data is stored in memory**, step by step, using simple examples. Read it in order if you are new.

---

## Step 1: Bit and Byte

* **Bit** → smallest unit of data → `0` or `1`
* **Byte** → group of **8 bits**

```
1 byte = 8 bits
```

Everything in a computer (numbers, text, images) is stored using bits.

---

## Step 2: Binary to Decimal (How a Byte Becomes a Number)

* **Binary** → base‑2 (only `0` and `1`)
* **Decimal** → base‑10 (`0–9`)

Each binary position represents a **power of 2**, from **right to left**.

### Example 1: `00001111`

| Bit | Power | Value |
| --- | ----- | ----- |
| 1   | 2⁰    | 1     |
| 1   | 2¹    | 2     |
| 1   | 2²    | 4     |
| 1   | 2³    | 8     |

```
1 + 2 + 4 + 8 = 15
```

**Result:** `00001111` → **15 (decimal)**

---

### Example 2: `1011`

| Bit | Power | Value |
| --- | ----- | ----- |
| 1   | 2³    | 8     |
| 0   | 2²    | 0     |
| 1   | 2¹    | 2     |
| 1   | 2⁰    | 1     |

```
8 + 0 + 2 + 1 = 11
```

**Result:** `1011` → **11 (decimal)**

---

## Step 3: Signed vs Unsigned Numbers

### Unsigned

* Stores **only positive values**
* All bits are used for value

```
Unsigned 1 byte → 0 to 255
```

### Signed

* Stores **positive and negative values**
* Uses **1 bit for sign** (Two’s Complement)

```
Signed 1 byte → -128 to 127
```

> ⚠️ Unsigned integers **do NOT** use a sign bit.

---

## Step 4: Binary and Hexadecimal

* **Hexadecimal** → base‑16
* Digits: `0–9` and `A–F`
* **Rule:** `1 hex digit = 4 bits`

### Binary → Hex Example

```
11010110
```

Group into 4 bits:

```
1101 0110
```

| Binary | Hex |
| ------ | --- |
| 1101   | D   |
| 0110   | 6   |

**Result:**

```
11010110 (binary) = 0xD6 (hex)
```

---

### Hex → Binary Example

```
3F
```

| Hex | Binary |
| --- | ------ |
| 3   | 0011   |
| F   | 1111   |

```
3F → 0011 1111 → 63 (decimal)
```

---

## Step 5: ASCII (Characters as Numbers)

* ASCII uses **1 byte per character**
* Values range from `0–127`

### ASCII Examples

| Character | ASCII | Binary   |
| --------- | ----- | -------- |
| 'A'       | 65    | 01000001 |
| 'a'       | 97    | 01100001 |
| '0'       | 48    | 00110000 |

```c
char c = 'A';
```

Memory stores:

```
'A' → 65 → 01000001
```

---

## Step 6: Variable Sizes and Ranges

> Sizes may vary by language or compiler, but these are common.

| Type   | Size    | Bits | Typical Use            | Signed Range        | Unsigned Range |
| ------ | ------- | ---- | ---------------------- | ------------------- | -------------- |
| char   | 1 byte  | 8    | characters             | -128 to 127         | 0 to 255       |
| short  | 2 bytes | 16   | small numbers          | -32,768 to 32,767   | 0 to 65,535    |
| int    | 4 bytes | 32   | common integers        | -2.1B to 2.1B       | 0 to 4.29B     |
| long   | 8 bytes | 64   | large numbers          | -9.22e18 to 9.22e18 | 0 to 1.84e19   |
| float  | 4 bytes | 32   | decimal (approx)       | IEEE‑754            | IEEE‑754       |
| double | 8 bytes | 64   | high precision decimal | IEEE‑754            | IEEE‑754       |

---

## Step 7: Strings = Multiple Bytes

A **string** is an **array of characters**.

### Example: "DOG"

| Character | ASCII |
| --------- | ----- |
| 'D'       | 68    |
| 'O'       | 79    |
| 'G'       | 71    |

```c
char s[] = "DOG";
```

Memory used:

```
D  O  G  \0
68 79 71  0
```

➡ **4 bytes total** (3 characters + null terminator)

---

## Step 8: Example Memory Layout

Store string **"Hi"** in memory:

| Character | ASCII | Binary   |
| --------- | ----- | -------- |
| 'H'       | 72    | 01001000 |
| 'i'       | 105   | 01101001 |
| '\0'      | 0     | 00000000 |

**Total memory used:** `3 bytes`

---

## Final Summary

* Everything is stored as **bits and bytes**
* Numbers use **binary**
* Characters use **ASCII**
* Hex is just a **readable shortcut for binary**
* Strings are **arrays of bytes**

