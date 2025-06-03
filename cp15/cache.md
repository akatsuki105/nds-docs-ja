# キャッシュ制御レジスタ

Cache is enabled/controlled by Bit 2,3,12,14 in Control Register.

Cache regions are controlled via Protection Unit (PU).

Cache type can be detected via Cache Type Register.

## C7,Cx,y - Cache Commands (x=0..15, y=0..7) (W)

Write-only Cache Command Register. Cm,Op2 operands used to select a specific command, with parameter value in Rd.

```
  Cn,Cm,Op2 Rd   ARM9 Command
  C7,C0,4   0    Yes  Wait For Interrupt (Halt)
  C7,C5,0   0    Yes  Invalidate Entire Instruction Cache
  C7,C5,1   VA   Yes  Invalidate Instruction Cache Line
  C7,C5,2   S/I  -    Invalidate Instruction Cache Line
  C7,C5,4   0    -    Flush Prefetch Buffer
  C7,C5,6   0    -    Flush Entire Branch Target Cache
  C7,C5,7   IMP? -    Flush Branch Target Cache Entry
  C7,C6,0   0    Yes  Invalidate Entire Data Cache
  C7,C6,1   VA   Yes  Invalidate Data Cache Line
  C7,C6,2   S/I  -    Invalidate Data Cache Line
  C7,C7,0   0    -    Invalidate Entire Unified Cache
  C7,C7,1   VA   -    Invalidate Unified Cache Line
  C7,C7,2   S/I  -    Invalidate Unified Cache Line
  C7,C8,2   0    Yes  Wait For Interrupt (Halt), alternately to C7,C0,4
  C7,C10,1  VA   Yes  Clean Data Cache Line
  C7,C10,2  S/I  Yes  Clean Data Cache Line
  C7,C10,4  0    -    Drain Write Buffer
  C7,C11,1  VA   -    Clean Unified Cache Line
  C7,C11,2  S/I  -    Clean Unified Cache Line
  C7,C13,1  VA   Yes  Prefetch Instruction Cache Line
  C7,C14,1  VA   Yes  Clean and Invalidate Data Cache Line
  C7,C14,2  S/I  Yes  Clean and Invalidate Data Cache Line
  C7,C15,1  VA   -    Clean and Invalidate Unified Cache Line
  C7,C15,2  S/I  -    Clean and Invalidate Unified Cache Line
```

Parameter values (Rd) formats:

```
  0    Not used, should be zero
  VA   Virtual Address
  S/I  Set/index; Bit 31..(32-A) = Index, Bit (L+S-1)..L = Set ?
```

Note:

```
  Invalidate means to forget all data
  Clean means to write-back dirty cache lines to underlaying memory
  (Clean is important when having "Cache Write-Bufferability" enabled in PU)
```

## C9,C0,0 - Data Cache Lockdown

## C9,C0,1 - Instruction Cache Lockdown

(Width (W) of index field depends on cache ASSOCIATIVETY.)

```
Format A:
  0     ..(31-W)  Reserved/zero
  (32-W)..31      Lockdown Block Index

Format B:
  0..(W-1)   Lockdown Block Index
  W..30      Reserved/zero
  31         L
```

Cache/Write-buffer should not be enabled for the whole 4GB memory area, high-speed TCM memory doesn’t require caching, and caching would have fatal results on I/O ports. So, cache can be used only in combination with the Protection Unit, which allows to enable/disable caching in specified regions.


## Note

ARMv5 instruction set supports a Cache Prepare for Load opcode (PLD)
