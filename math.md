# [演算ユニット](https://mgba-emu.github.io/gbatek/#dsmaths)

DSは 演算ユニット(除算器/平方根演算器) を持っています。

## 0x0400_0280 - NDS9 - DIVCNT - Division Control (R/W)

```
  0-1   Division Mode    (0-2=See below) (3=Reserved; same as Mode 1)
  2-13  不使用
  14    Division by zero (0=Okay, 1=Division by zero error; 64bit Denom=0)
  15    Busy             (0=Ready, 1=Busy) (Execution time see below)
  16-31 不使用
```

Division Modes and Busy Execution Times

```
  Mode  Numer / Denom = Result, Remainder ; Cycles
  0     32bit / 32bit = 32bit , 32bit     ; 18 clks
  1     64bit / 32bit = 64bit , 32bit     ; 34 clks
  2     64bit / 64bit = 64bit , 64bit     ; 34 clks
```

Division is started when writing to any of the DIVCNT/NUMER/DENOM registers.

## 0x0400_0290 - NDS9 - DIV_NUMER - 64bit Division Numerator (R/W)

## 0x0400_0298 - NDS9 - DIV_DENOM - 64bit Division Denominator (R/W)

Signed 64bit values (or signed 32bit values in 32bit modes, the upper 32bits are then unused, with one exception: the DIV0 flag in DIVCNT is set only if the full 64bit DIV_DENOM value is zero, even in 32bit mode).

## 0x0400_02A0 - NDS9 - DIV_RESULT - 64bit Division Quotient (=Numer/Denom) (R)

## 0x0400_02A8 - NDS9 - DIVREM_RESULT - 64bit Remainder (=Numer MOD Denom)

Signed 64bit values (in 32bit modes, the values are sign-expanded to 64bit).

## Division Overflows

Overflows occur on “DIV0” and “-MAX/-1” (eg. -80000000h/-1 in 32bit mode):

```
  DIV0     -->  REMAIN=NUMER, RESULT=+/-1 (with sign opposite of NUMER)
  -MAX/-1  -->  RESULT=-MAX               (instead +MAX)
```

On overflows in 32bit/32bit=32bit mode: the upper 32bit of the sign-expanded 32bit result are inverted. This feature produces a correct 64bit (+MAX) result in case of the incorrect 32bit (-MAX) result. The feature also applies on DIV0 errors (which makes the sign-expanded 64bit result even more messed-up than the normal 32bit result).

The DIV0 flag in DIVCNT.14 indicates DENOM=0 errors (it does not indicate “-MAX/-1” errors). The DENOM=0 check relies on the full 64bit value (so, in 32bit mode, the flag works only if the unused upper 32bit of DENOM are zero).

## 0x0400_02B0 - NDS9 - SQRTCNT - Square Root Control (R/W)

```
  0     Mode (0=32bit input, 1=64bit input)
  1-14  不使用
  15    Busy (0=Ready, 1=Busy) (Execution time is 13 clks, in either Mode)
  16-31 不使用
```

Calculation is started when writing to any of the SQRTCNT/PARAM registers.

## 0x0400_02B4 - NDS9 - SQRT_RESULT - 32bit - Square Root Result (R)

## 0x0400_02B8 - NDS9 - SQRT_PARAM - 64bit - Square Root Parameter Input (R/W)

Unsigned 64bit parameter, and unsigned 32bit result.

## IRQ Notes

Push all DIV/SQRT values (parameters and control registers) when using DIV/SQRT registers on interrupt level, and, after restoring them, be sure to wait until the busy flag goes off, before leaving the IRQ handler.

## BIOS Notes

The NDS9 and NDS7 BIOSes additionally contain software based division and square root functions, which are NOT using above hardware registers (even the NDS9 functions are raw software).

## Timing Notes

The Div/Sqrt timings are counted in 33.51MHz units. Although the calculations are quite fast, mind that reading/writing the result/parameter registers takes up additional clock cycles (especially due to the PENALTY cycle glitch for non-sequential accesses; parts of that problem can be eventually bypassed by using sequential STMIA/LDMIA opcodes) (nethertheless, in some cases, software may be actually faster than the hardware registers; eg. for small 8bit numbers; that of course NOT by using the BIOS software functions which are endless inefficient).
