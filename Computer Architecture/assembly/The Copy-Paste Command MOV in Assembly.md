![[Introduction to assembly#The "Copy-Paste" Command, MOV in Assembly]]

### MOVSX (Move with Sign-Extend)

**Use this for:** **Signed integers** (negative or positive numbers).

MOVSX copies the source value to the destination and fills all the remaining upper bits with the **Sign Bit** (the most significant bit of the source). This preserves the numerical value of negative numbers.

- If the source is **positive** (top bit is 0), upper bits become **0**.
- If the source is **negative** (top bit is 1), upper bits become **1** (F in Hex).

#### Example

Moving the 8-bit value `0xFB` (which is **-5** in signed decimal) into a 16-bit register.

- **Instruction:** `MOVSX AX, BL` (assume `BL` holds `0xFB`)
- **Result in AX:** `0xFFFB`
- **Why:** It keeps the value as **-5**.

---
### MOVZX (Move with Zero-Extend)

**Use this for:** **Unsigned integers** (positive numbers only) or raw data/characters.

MOVZX copies the source value to the destination and simply fills all the remaining upper bits with **Zeros**. It treats the source as a purely positive number.

#### Example

Moving the same 8-bit value `0xFB` (which is **251** in unsigned decimal) into a 16-bit register.

- **Instruction:** `MOVZX AX, BL` (assume `BL` holds `0xFB`)
    
- **Result in AX:** `0x00FB`
    
- **Why:** It keeps the value as **251**.