# ID Codesレジスタ

## C0,C0,0 - MainIDレジスタ (R)

MainIDレジスタはARMプロセッサ(≠コプロセッサ)の情報を提供するレジスタです。

 bit  |  内容
---- | ---- 
12-15 | ARMプロセッサの世代 (0=Pre-ARM7, 7=ARM7, other=Post-ARM7)

他のbitの内容はARMの世代によって変わってきます。

### Post-ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-15  | Primary Part Number (Bit12-15 must be other than 0 or 7) (eg. 946h for ARM946)
16-19 | Architecture (1=v4, 2=v4T, 3=v5, 4=v5T, 5=v5TE, 6=v6, ?=v7)
20-23 | Variant Number
24-31 | Implementor (41h=ARM, 44h=Digital Equipment Corp, 69h=Intel)

### ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-15  | Primary Part Number (Bit12-15 must be 7)
16-22 | Variant Number
23    | Architecture        (0=v3, 1=v4T)
24-31 | Implementor         (41h=ARM, 44h=Digital Equipment Corp, 69h=Intel)

### Pre-ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-11  | Processor ID LSBs (30h=ARM3/v2, 60h,61h,62=ARM600,610,620/v3)
12-31 | Processor ID MSBs (fixed, 41560h)

Note: On the NDS9, this register is 41059461h (ARMv5TE, ARM946, rev1). NDS7 and GBA don't have CP15s.

## C0,C0,1 - Cache Type Register (R)

```
Bit:
  0-11  Instruction Cache (bits 0-1=len, 2=m, 3-5=assoc, 6-8=size, 9-11=zero)
  12-23 Data Cache        (bits 0-1=len, 2=m, 3-5=assoc, 6-8=size, 9-11=zero)
  24    Separate Cache Flag (0=Unified, 1=Separate Data/Instruction Caches)
  25-28 Cache Type (0,1,2,6,7=see below, other=reserved)
         Type Method         Cache cleaning         Cache lock-down
         0    Write-through  Not needed             Not supported
         1    Write-back     Read data block        Not supported
         2    Write-back     Register 7 operations  Not supported
         6    Write-back     Register 7 operations  Format A
         7    Write-back     Register 7 operations  Format B      ;<-- NDS9
  29-31 Reserved (zero)
```

The 12bit Instruction/Data values are decoded as shown below,

```
  Cache Absent  = (ASSOC=0 and M=1)       ;in that case overriding below
  Cache Size    = 200h+(100h*M) shl SIZE  ;min 0.5Kbytes, max 96Kbytes
  Associativity = (1+(0.5*M)) shl ASSOC   ;min 1-way,     max 192-way
  Line Length   = 8 shl LEN               ;min 8 bytes,   max 64 bytes
```

For Unified cache (Bit 24=0), Instruction and Data values are identical.

Note: On the NDS9, this register is 0F0D2112h (Code=2000h bytes, Data=1000h bytes, assoc=whatever, and line size 32 bytes each). NDS7 and GBA don’t have CP15s (nor any code/data cache).

## C0,C0,2 - Tightly Coupled Memory (TCM) Size Register (R)

```
  0-1   Reserved    (0)
  2     ITCM Absent (0=Present, 1=Absent)
  3-5   Reserved    (0)
  6-9   ITCM Size   (Size = 512 SHL N) (or 0=None)
  10-13 Reserved    (0)
  14    DTCM Absent (0=Present, 1=Absent)
  15-17 Reserved    (0)
  18-21 DTCM Size   (Size = 512 SHL N) (or 0=None)
  22-31 Reserved    (0)
```

Note: On the NDS9, this register is 00140180h (ITCM=8000h bytes, DTCM=4000h bytes). NDS7 and GBA don’t have CP15s (nor any ITCM/DTCM).

## C0,C0,3..7 - Reserved (R)

使われないレジスタです。内容は`C0,C0,0`のミラーです。
