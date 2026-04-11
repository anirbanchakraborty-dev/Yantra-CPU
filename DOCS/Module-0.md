# THE FUNDAMENTALS

## Part 1: What Is a Number System?

You already use one number system every day — **decimal** (base 10). It uses ten symbols: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9. When you write the number **347**, you're implicitly doing this:

```text
347 = 3×10² + 4×10¹ + 7×10⁰
    = 3×100 + 4×10  + 7×1
    = 300   + 40    + 7
```

Each digit's **position** determines its **weight** — the rightmost digit is the ones place (10⁰ = 1), the next is tens (10¹ = 10), then hundreds (10² = 100), and so on. The base (10) tells you how much each position is worth relative to its neighbor.

There's nothing magical about the number 10 here. We use it because humans have 10 fingers. Computers, on the other hand, are built from billions of switches called transistors that are either **on** or **off** — two states. So computers use **base 2**, which needs only two symbols: **0** and **1**. Each symbol is called a **bit** (binary digit).

---

## Part 2: Binary (Base 2)

The exact same positional logic applies. In binary, the positions are powers of **2** instead of powers of 10:

```text
Position:    7     6     5     4     3     2     1     0
Weight:     128    64    32    16     8     4     2     1
             2⁷    2⁶    2⁵    2⁴    2³    2²    2¹    2⁰
```

So the binary number `10110011` means:

```text
1×128 + 0×64 + 1×32 + 1×16 + 0×8 + 0×4 + 1×2 + 1×1
= 128 + 0 + 32 + 16 + 0 + 0 + 2 + 1
= 179
```

That's it. Binary is not a different kind of math — it's the same positional system, just with fewer symbols.

**Key terminology:**

- A **bit** is a single binary digit (0 or 1)
- A **nibble** is 4 bits (can represent 0–15, which is one hexadecimal digit)
- A **byte** is 8 bits (can represent 0–255)
- A **word** is 32 bits (can represent 0 to 4,294,967,295 as unsigned)

---

## Part 3: Converting Between Decimal and Binary

**Decimal → Binary (the "divide by 2" method):**

Let's convert decimal **43** to binary. You repeatedly divide by 2, recording the remainder each time, and then read the remainders bottom-to-top:

```text
43 ÷ 2 = 21  remainder 1  ← least significant bit (rightmost)
21 ÷ 2 = 10  remainder 1
10 ÷ 2 = 5   remainder 0
 5 ÷ 2 = 2   remainder 1
 2 ÷ 2 = 1   remainder 0
 1 ÷ 2 = 0   remainder 1  ← most significant bit (leftmost)
```

Read remainders bottom-to-top: **101011**

Verify: 32 + 0 + 8 + 0 + 2 + 1 = 43 ✓

**Binary → Decimal:** Just multiply each bit by its positional weight and sum, as we did with `10110011` above.

---

## Part 4: Hexadecimal (Base 16) — The Shorthand

Writing 32-bit numbers in binary is tedious and error-prone. Imagine trying to read `00000000110000010010000000000000` at a glance. Hexadecimal (base 16) solves this by grouping every 4 bits into a single symbol:

```text
Decimal:  0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15
Hex:      0  1  2  3  4  5  6  7  8  9   A   B   C   D   E   F
Binary: 0000 0001 0010 0011 0100 0101 0110 0111 1000 1001 1010 1011 1100 1101 1110 1111
```

So to convert the 32-bit binary `00000000110000010010000000000000` to hex, group into nibbles from the right:

```text
0000  0000  1100  0001  0010  0000  0000  0000
 0     0     C     1     2     0     0     0
```

Result: `0x00C12000` — much more readable. The `0x` prefix is the convention for "this is a hexadecimal number."

**This is why you'll see hex everywhere in hardware.** Waveform viewers, memory dumps, register contents, addresses — all displayed in hex because it maps perfectly to groups of 4 bits, and 32 bits always become exactly 8 hex digits.

In SystemVerilog, you'll write constants in different bases like this:

```text
8'b10110011     // 8-bit binary literal
8'hB3           // 8-bit hex literal (same value: 179)
8'd179          // 8-bit decimal literal (same value)
32'h00C12000    // 32-bit hex literal
```

The prefix number (8, 32) is the width in bits. The letter after the apostrophe is the base (`b` = binary, `h` = hex, `d` = decimal).

---

## Part 5: Unsigned vs. Signed — The Core Problem

Everything above works perfectly for **unsigned** (non-negative) integers. An 8-bit unsigned number can represent 0 to 255. A 32-bit unsigned number can represent 0 to 4,294,967,295.

But real programs need **negative numbers**. You want to compute `5 - 8 = -3`. So the hardware must have a way to represent `-3` as a binary pattern.

The question is: **how do you encode a negative number using only 0s and 1s?**

There's no minus sign in hardware. There are only bits. Several schemes have been tried throughout history. Let me walk through them so you understand _why_ we use the one we do.

---

## Part 6: Sign-Magnitude (the intuitive but flawed approach)

The simplest idea: use the leftmost bit as a "sign bit." 0 means positive, 1 means negative. The remaining bits hold the magnitude (absolute value).

For an 8-bit sign-magnitude system:

```text
+5  = 0 0000101      (sign=0, magnitude=5)
-5  = 1 0000101      (sign=1, magnitude=5)
+0  = 0 0000000
-0  = 1 0000000      ← problem!
```

**Two fatal problems for hardware:**

1. **Two representations of zero** (+0 and -0). This complicates every comparison circuit — to check if a value is zero, you'd need to check both patterns.

2. **Addition hardware becomes complicated.** To add two sign-magnitude numbers, you can't just feed them into a simple adder. You have to check the signs, compare the magnitudes, decide whether to add or subtract, then set the correct sign on the result. That's a lot of extra logic.

Early computers (like the IBM 7090 in the 1950s) used sign-magnitude. They abandoned it for good reason.

---

## Part 7: Two's Complement (the elegant solution)

Two's complement is the scheme used by virtually every modern CPU. It's brilliant because **addition of signed numbers uses the exact same hardware as addition of unsigned numbers.** You don't need separate add and subtract circuits for signed vs. unsigned — the same adder works for both.

**How it works for N bits:**

- **Positive numbers** (and zero) are represented exactly as in unsigned binary. The most significant bit (MSB) happens to be 0.
- **Negative numbers** are represented such that adding a number and its negative gives 2ᴺ (which overflows to zero in N bits).

**How to negate a number (find the two's complement):**

1. Flip all the bits (change every 0 to 1 and every 1 to 0) — this is called the **one's complement**
2. Add 1

Example: negate +5 in 8 bits:

```text
+5          = 00000101
Flip bits   = 11111010  (one's complement)
Add 1       = 11111011  (two's complement = -5)
```

Let's verify: add +5 and -5:

```text
  00000101   (+5)
+ 11111011   (-5)
----------
 100000000
```

The result is 9 bits, but we're working with 8 bits, so the leading 1 is discarded (it "falls off" the end). The remaining 8 bits are `00000000` — zero. It works!

**The 8-bit two's complement range:**

```text
01111111 = +127   (largest positive)
01111110 = +126
...
00000001 = +1
00000000 =  0
11111111 = -1
11111110 = -2
...
10000001 = -127
10000000 = -128   (most negative — note there's one extra negative value!)
```

The range is **-128 to +127** for 8 bits. In general, for N bits: **-2^(N-1) to +2^(N-1) - 1**. For 32 bits, that's **-2,147,483,648 to +2,147,483,647**.

**Key insight — the MSB as a "sign indicator":** In two's complement, if the MSB is 0, the number is non-negative. If the MSB is 1, the number is negative. This looks like sign-magnitude, but the encoding is fundamentally different. The MSB has a weight of **-2^(N-1)** rather than acting as a flag.

For 8 bits, the bit weights are:

```text
Bit position:    7      6    5    4   3   2   1   0
Weight:        -128    64   32   16   8   4   2   1
```

So `11111011` = **-128** + 64 + 32 + 16 + 8 + 0 + 2 + 1 = **-128 + 123 = -5** ✓

This is the proper mathematical way to think about two's complement — it's not a "flip and add 1 trick," it's a genuine positional number system where the MSB has a negative weight.

---

## Part 8: Why Two's Complement Wins — The Hardware Argument

This is the deep reason Yantra-CPU (and every modern CPU) uses two's complement. Watch what happens when we subtract 3 from 5 using nothing but an adder:

**Step 1:** Represent both numbers:

```text
 5 = 00000101
 3 = 00000011
```

**Step 2:** To compute 5 - 3, we compute 5 + (-3). Negate 3:

```text
 3 = 00000011
-3 = 11111101  (flip + add 1)
```

**Step 3:** Add:

```text
  00000101   (5)
+ 11111101   (-3)
----------
 100000010
```

Drop the carry-out: `00000010` = **2** ✓

The ALU we will build for Yantra-CPU will do exactly this. Looking at the SUB operation — it flips the bits of operand B (bitwise NOT), feeds the result into the adder, and sets the carry-in to 1. That's "flip and add 1" — two's complement negation — done entirely with the adder that we will use for ADD. The same adder circuit handles both addition and subtraction. One piece of hardware, two operations. This is the entire reason two's complement exists.

---

## Part 9: Sign Extension — Critical for Yantra-CPU

This concept will come up repeatedly when we build the decoder and immediate extender.

Yantra-CPU's I-type instructions have an 18-bit immediate field. But the ALU operates on 32-bit values. So before the immediate reaches the ALU, it must be **extended** from 18 bits to 32 bits. How you extend it depends on whether you're treating the value as signed or unsigned.

**Sign extension (for signed values):** Copy the MSB (the sign bit) into all the new upper positions.

```text
18-bit immediate: 11_1111_1111_1111_1011  (this is -5 in 18-bit two's complement)

Sign-extend to 32 bits:
11111111111111_11_1111_1111_1111_1011

The top 14 bits are all 1s (copies of the original MSB).
```

This preserves the value: -5 in 18 bits becomes -5 in 32 bits.

**Zero extension (for unsigned/logical values):** Fill the new upper positions with zeros.

```text
18-bit value: 11_1111_1111_1111_1011

Zero-extend to 32 bits:
00000000000000_11_1111_1111_1111_1011
```

Now it's a large positive number (262,139) — completely different meaning!

In Yantra-CPU's ISA, ADDI, LW, SW, and branches use **sign extension** (because offsets and addresses can be negative). ANDI, ORI, XORI use **zero extension** (because bitwise operations treat the immediate as a bit pattern, not a signed number).

---

## Part 10: Overflow — When Arithmetic Goes Wrong

With a fixed number of bits, some results simply don't fit. This is **overflow**.

**Unsigned overflow:** Adding two 8-bit unsigned numbers: 200 + 100 = 300. But 300 > 255 (the max for 8 bits). The result wraps around: 300 - 256 = 44. The carry-out flag signals this happened.

**Signed overflow:** This is trickier and more dangerous. It occurs when the result of an operation on two signed numbers is too large (positive or negative) to represent.

The rule: **signed overflow occurs when two numbers of the same sign are added and the result has the opposite sign.**

```text
+100 + +50 = +150   → but 8-bit signed max is +127 → overflow!

In binary:
  01100100  (+100)
+ 00110010  (+50)
----------
  10010110  → this is -106 in two's complement! Wrong!
```

Both inputs had MSB = 0 (positive), but the result has MSB = 1 (negative). That's the overflow flag.

The ALU we built for Yantra-CPU has exactly this: an **overflow flag** that checks whether the carry into the MSB differs from the carry out of the MSB. This flag is critical for the SLT (set-less-than) instruction to work correctly with signed numbers.

---

## Part 11: Fixed-Point Representation (Brief Awareness)

Yantra-CPU works only with integers — no floating point. But you should be aware that it's possible to represent fractional numbers in binary using **fixed-point** notation.

The idea: you mentally place a "binary point" at a fixed position within the word. Bits to the left are the integer part, bits to the right are the fractional part.

```text
8-bit fixed-point with 4 integer bits and 4 fractional bits:

0110.1100 = 4 + 2 + 0.5 + 0.25 = 6.75
```

The fractional bit weights are: 2⁻¹ = 0.5, 2⁻² = 0.25, 2⁻³ = 0.125, 2⁻⁴ = 0.0625.

The hardware doesn't know or care about the binary point — it's a **convention** agreed upon by the programmer. The same adder and multiplier circuits work regardless. You just interpret the result differently. This is an elegant trick, but Yantra-CPU won't need it. We'll work exclusively with 32-bit two's complement integers.
