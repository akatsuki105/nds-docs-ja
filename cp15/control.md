# 制御レジスタ

## C1,C0,0 - 制御レジスタ (R/W)

いくつかのビットは、サポートされていない場合は常に`0`、または常に有効な場合は常に`1`として常に読み取り専用です。

- NDSのARM9では、 bit0,2,7,12..19 が R/W、bit3..6 は常に`1`、残りは常に`0`です。
- 3DSのARM11では、bit0..2,8..9,11..13,15,22..23,25,28..29 が R/W、bit3..6,14,16,18 は常に`1`、残りは常に`0`です。

```
  Bit     Expl.
  0       MMU/PU有効化               (0=無効, 1=有効)
  1       Alignment Fault Check      (0=無効, 1=有効)
  2       データキャッシュ or ユニファイドキャッシュ (0=無効, 1=有効, データかユニファイド かは C0,C0,1 で指定 (NDS9は常にデータキャッシュ))
  3-6     予約 (1)
  7       エンディアン               (0=Little, 1=Big)
  8       System Protection bit      (MMU-only)
  9       ROM Protection bit         (MMU-only)
  10      Implementation defined
  11      分岐予測                   (0=無効, 1=有効)
  12      命令キャッシュ             (0=無効, 1=有効) (ユニファイドキャッシュの場合はこのbitは無視)
  13      例外ベクタの開始アドレス   (0=0x0000_0000, 1=0xFFFF_0000)
  14      Cache Replacement          (0=Normal/PseudoRandom, 1=Predictable/RoundRobin)
  15      Pre-ARMv5 Mode             (0=Normal, 1=Pre ARMv5; LDM/LDR/POP_PC.Bit0/Thumb)
  16      DTCM有効化                 (0=無効, 1=有効)
  17      DTCM Load Mode             (0=R/W, 1=DTCM Write-only)
  18      ITCM有効化                 (0=無効, 1=有効)
  19      ITCM Load Mode             (0=R/W, 1=ITCM Write-only)
  20-21   予約(0)
  22      Unaligned Access           (?=Enable unaligned access and mixed endian)
  23      Extended Page Table        (0=Subpage AP Bits Enabled, 1=Disabled)
  24      予約(0)
  25      CPSR E on exceptions       (0=Clear E bit, 1=Set E bit)
  26      予約(0)
  27      FIQ Behaviour              (0=Normal FIQ behaviour, 1=FIQs behave as NMFI)
  28      TEX Remap bit              (0=No remapping, 1=Remap registers used)
  29      Force AP                   (0=Access Bit not used, 1=AP[0] used as Access bit)
  30-31   予約(0)
```

