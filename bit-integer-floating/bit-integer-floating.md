## Bits, Bytes, Integers, & Endianness

### Hexadecimal

- `1A2B` in hex
    - Decimal : $1*16^3 + A*16^2 + 2*16^1 + B*16^0 \\ = 1*16^3
     + 10*16^2
     + 2*16^1
     + 11*16^0$ =  6699 (decimal)
- `0100 0111 0010 0101` in binary:
    - Hex: 4 7 2 5

### Boolean Algebra

| AND | OR | NOT | XOR |
| --- | --- | --- | --- |
| & | \| | ~ | ^ |
- Exclusive-Or (XOR)
    - Either A=1 or B=1, but not both

### Shift Algorithm

- Left Shift: `x << y`
    - Fill with 0’s on right.
- Right Shift: `x >> y`
    - Fill with 0’s on left.
    - Replicate most significant bit on left.

### Negative Numbers
- **2’s Complement**
    - `-2` (decimal):
        - `~010` + `1`  →  `101` + `1`   →  `110`
- Range
    - Unsigned value:
        - Max:  $2^w – 1$
        - Min: 0
    - Signed value:
        - Max: $2^{w-1}-1$
        - Min: $-2^{w-1}$

## Floating Point

![IEEE754](/computer_system/images/IEEE754.png)

### Float Point Representation

$$
(-1)^S * M * 2^E
$$

- Sign bit `s` : Determine negative (1) or positive (0).
- Significand `M` (Mantissa): Normally fraction value [1.0, 2.0).
- Exponent `E` : Weight value by power of 2.

### Calculation

e.g. $-5/32$

- $-5/32 = -0.15625$

```bash
0.15625 * 2 = 0.3125
0.3125 * 2 = 0.625
0.625 * 2 = 1.25
0.25 * 2 = 0.5
0.5 * 2 = 1
```

- So we got $-0.00101$
- We move the point: $-1 * 1.01 * 2^3$
- $s=1$, $E=-3$, $M=101$
- $exp = -3 +3 = 0$
- So answer is `1 000 1010`

### 3 kinds of floating point numbers

1. **Normalized** (`exp`  ≠ 0 && `exp`  ≠ 11…11)
    - `E`  = `exp`  - Bias
2. **Denormalized** (`exp` = 0)
    - `E`  = 1 - Bias
    - `frac` without leading 1
3. **Special** (`exp` = 11…11)
    - frac = `00..00` : infinity
        - s = 1: -infinity
        - s = 0: +infinity
    - frac ≠ `00..00` : NaN

### Rounding

Cases:

- **Even**: least significant number is 0
- **Half way**: when bits to right of rounding position = $100_2$
    - To round when a value is exactly half way between digits, always round to the nearest even.
    - e.g.  Let’s say you’re rounding to 2 bits of fraction (mantissa):
        1. Closest representable values: `0.10` (0.5) and `0.11` (0.75)
        2. The halfway point is: `0.101` (0.625)
        3. So `0.101` (0.625) is halfway between `0.10` (0.5) and `0.11` (0.75) in binary.
        4. For `0.101` , it can be `0.10` or `0.11` . we round to the nearest even, so is `0.10` (0.5)