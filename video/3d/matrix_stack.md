# 行列スタック

NDSは3つの行列スタックと2つの行列スタックポインタを持っています。(Coordinate/Directional はスタックを共有しています)

```
  Matrix Stack________Valid Stack Area____Stack Pointer___________________
  Projection Stack    0..0  (1 entry)     0..1  (1bit) (GXSTAT: 1bit)
  Coordinate Stack    0..30 (31 entries)  0..63 (6bit) (GXSTAT: 5bit only; 後述)
  Directional Stack   0..30 (31 entries)  (uses Coordinate Stack Pointer)
  Texture Stack       One..None?          0..1  (1bit) (GXSTAT: N/A)
```

Which of the stacks/matrices depends on the current Matrix Mode (as usually,

but with one exception; stack operations MTX_PUSH/POP/STORE/RESTORE in Mode 1 are acting same as in Mode 2):

```
  MTX_MODE = 0      --> Projection Stack
  MTX_MODE = 1 or 2 --> BOTH Coordinate AND Directional Stack
  MTX_MODE = 3      --> Texture Stack
```

スタックポインタの初期値はゼロであり、現在の値はGXSTATから読み取ることができます。

このレジスタはスタックのオーバーフローも示してくれます。

すべてのスタックについて、上半分（つまり 1 または 32..63）は下半分（つまり 0 または 0..31）のミラーとなっています。

## 4000444h - Cmd 11h - MTX_PUSH - Push Current Matrix on Stack (W)

Parameters: None. Sets [S]=C, and then S=S+1.

## 4000448h - Cmd 12h - MTX_POP - Pop Current Matrix from Stack (W)

Sets S=S-N, and then C=[S].

```
  Parameter Bit0-5:  Stack Offset (signed value, -30..+31) (usually +1)
  Parameter Bit6-31: Not used
```

Offset N=(+1) pops the most recently pushed value, larger offsets of N>1 will “deallocate” N values (and load the Nth value into C). Zero or negative values can be used to pop previously “deallocated” values.

The stack has only one level (at address 0) in projection mode, in that mode, the parameter value is ignored, the offset is always +1 in that mode.

## 400044Ch - Cmd 13h - MTX_STORE - Store Current Matrix on Stack (W)

Sets [N]=C. The stack pointer S is not used, and is left unchanged.

```
  Parameter Bit0-4:  Stack Address (0..30) (31 causes overflow in GXSTAT.15)
  Parameter Bit5-31: Not used
```

The stack has only one level (at address 0) in projection mode, in that mode, the parameter value is ignored.

## 4000450h - Cmd 14h - MTX_RESTORE - Restore Current Matrix from Stack (W)

Sets C=[N]. The stack pointer S is not used, and is left unchanged.

```
  Parameter Bit0-4:  Stack Address (0..30) (31 causes overflow in GXSTAT.15)
  Parameter Bit5-31: Not used
```

The stack has only one level (at address 0) in projection mode, in that mode, the parameter value is ignored.

In Projection mode, the parameter for POP, STORE, and RESTORE is unused - not sure if the parameter (ie. a dummy value) is - or is not - to be written to the command FIFO?

There appear to be actually 32 entries in Coordinate & Directional Stacks, entry 31 appears to exist, and appears to be read/write-able (although the stack overflow flag gets set when accessing it).
