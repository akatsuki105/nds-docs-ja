# メモリ保護ユニット(Memory Protection Unit)

メモリ保護ユニット(MPU) は 制御レジスタ`C1,C0,0`の bit0 をセットすることで有効になります。

MPUは、メモリの保護を行うための機能で、特定のメモリ領域に対してアクセス権限を設定できます。これにより、特定のコードやデータが不正にアクセスされるのを防ぐことができます。

> [!IMPORTANT]
> CP15では、本来(データと命令で8個ずつの)16個の保護領域を定義できますが、NDSでは保護領域はデータと命令で共有しています。
> ただし保護領域に対するアクセスパーミッションは、データアクセスと命令アクセスで別々のパーミッションを設定できます。

## C6,Cn,0 - データアクセス保護領域n (R/W)
## C6,Cn,1 - 命令アクセス保護領域n (R/W)

> [!IMPORTANT]
> NDSでは保護領域の範囲だけはデータと命令で共有のため`C6,Cn,1`は`C6,Cn,0`のミラーとなっています。
> また、どの保護領域にも属さないメモリ領域は、[Background regions](https://developer.arm.com/documentation/ddi0201/d/protection-unit/overlapping-regions/background-regions)として扱われ、アクセスすることができません。

保護領域`n`の有効化とサイズ、ベースアドレスを設定します。

```
  Bit     Expl.
  0       保護領域の有効化 (0=Disable, 1=Enable)
  1-5     保護領域のサイズ   (2 SHL X) ;min=(X=11)=4KB, max=(X=31)=4GB
  6-11    予約 (0)
  12-31   保護領域のベースアドレス (Addr = Y*4K; must be SIZE-aligned)
```

保護領域の範囲が被った場合は、最も優先度の高い保護領域として扱われます。(0が最も優先度が低く、7が最も優先度が高い)

## C2,C0,0 - Cachability Bits for Data/Unified Protection Region (R/W)
## C2,C0,1 - Cachability Bits for Instruction Protection Region (if any) (R/W)

```
  Bit   Expl.
  0-7   Cachable (C) bits for region 0-7
  8-31  予約 (0)
```

## C3,C0,0 - Cache Write-Bufferability Bits for Data Protection Regions (R/W)

Allows to select what to do when writing to a cached memory snippet:

Write-Through stores the data in the cache line (so subsequent cache reads return correct data), and additionally writes the data to underlaying memory.

Write-Back stores the data in the cache line only, and marks the line as dirty, but doesn’t update the underlaying memory (underlaying memory is updated only when the CPU decides to use the cache line for other purposes, or when the user is manually “Cleaning” the cache line).

```
  Bit   Expl.
  0-7   Bufferable (B) bits for region 0-7  (0=Write-Through, 1=Write-Back)
  8-31  Reserved (0)
```

Instruction fetches are, obviously, always read-operations. So, there are no write-bufferability bits for Instruction Protection Regions.

Note: Unrelated to the “Cache Write-Bufferability”, the ARM does also have a “Write Buffer” (a small FIFO that can queue only a few writes).

## C5,C0,0 - Access Permission Data/Unified Protection Region (R/W)
## C5,C0,1 - Access Permission Instruction Protection Region (if any) (R/W)
## C5,C0,2 - Extended Access Permission Data/Unified Protection Region (R/W)
## C5,C0,3 - Extended Access Permission Instruction Protection Region (if any) (R/W)

For C5,C0,0 and C5,C0,1:

```
  Bit   Expl.
  0-15  Access Permission (AP) bits for region 0-7 (Bits 0-1=AP0, 2-3=AP1, etc)
  8-31  Reserved (0)
```

For C5,C0,2 and C5,C0,3 (Extended):

```
  Bit   Expl.
  0-31  Access Permission (AP) bits for region 0-7 (Bits 0-3=AP0, 4-7=AP1, etc)
```

The possible AP settings (0-3 for C5,C0,0..1, or 0-15 for C5,C0,2..3) are:

```
  AP  Privileged User
  0   -          -
  1   R/W        -
  2   R/W        R
  3   R/W        R/W
  5   R          -
  6   R          R
```

Settings 5,6 only for Extended Registers, settings 4,7..15 are Reserved.


