# 制御レジスタ

## C1,C0,0 - 制御レジスタ (R/W, 特定のビットはRのみ)

```
  Bit  Expl.
  0    MMU/PU Enable         (0=無効, 1=有効) (Fixed 0 if none)
  1    Alignment Fault Check (0=無効, 1=有効) (Fixed 0/1 if none/always on)
  2    Data/Unified Cache    (0=無効, 1=有効) (Fixed 0/1 if none/always on)
  3    Write Buffer          (0=無効, 1=有効) (Fixed 0/1 if none/always on)
  4    Exception Handling    (0=26bit, 1=32bit)    (Fixed 1 if always 32bit)
  5    26bit-address faults  (0=有効, 1=無効) (Fixed 1 if always 32bit)
  6    Abort Model (pre v4)  (0=Early, 1=Late Abort) (Fixed 1 if ARMv4 and up)
  7    エンディアン          (0=Little, 1=Big, この値は変更できない)
  8    System Protection bit (MMU-only)
  9    ROM Protection bit    (MMU-only)
  10   Implementation defined
  11   Branch Prediction     (0=無効, 1=有効)
  12   Instruction Cache     (0=無効, 1=有効) (ignored if Unified cache)
  13   例外ベクタの開始アドレス  (0=0x0000_0000, 1=0xFFFF_0000)
  14   Cache Replacement     (0=Normal/PseudoRandom, 1=Predictable/RoundRobin)
  15   Pre-ARMv5 Mode        (0=Normal, 1=Pre ARMv5; LDM/LDR/POP_PC.Bit0/Thumb)
  16   DTCM有効化            (0=無効, 1=有効)
  17   DTCM Load Mode        (0=R/W, 1=DTCM Write-only)
  18   ITCM有効化            (0=無効, 1=有効)
  19   ITCM Load Mode        (0=R/W, 1=ITCM Write-only)
  20   Reserved              (0)
  21   Reserved              (0)
  22   Unaligned Access      (?=Enable unaligned access and mixed endian)
  23   Extended Page Table   (0=Subpage AP Bits Enabled, 1=Disabled)
  24   Reserved              (0)
  25   CPSR E on exceptions  (0=Clear E bit, 1=Set E bit)
  26   Reserved              (0)
  27   FIQ Behaviour         (0=Normal FIQ behaviour, 1=FIQs behave as NMFI)
  28   TEX Remap bit         (0=No remapping, 1=Remap registers used)
  29   Force AP              (0=Access Bit not used, 1=AP[0] used as Access bit)
  30   Reserved              (0)
  31   Reserved              (0)
```

Various bits in this register may be read-only (fixed 0 if unsupported, or fixed 1 if always activated).

On NDS ARM9, bit0,2,7,12..19 are R/W, bit3..6 are always set, all other bits are always zero.

On 3DS ARM11, bit0..2,8..9,11..13,15,22..23,25,28..29 are R/W, bit3..6,14,16,18 are always set, all other bits are always zero.
