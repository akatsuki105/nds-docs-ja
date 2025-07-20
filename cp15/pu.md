# 保護ユニット(Protection Unit)

保護ユニット は 制御レジスタ`C1,C0,0`の bit0 をセットすることで有効になります。

## C2,C0,0 - Cachability Bits for Data/Unified Protection Region (R/W)
## C2,C0,1 - Cachability Bits for Instruction Protection Region (if any) (R/W)

```
  Bit   Expl.
  0-7   Cachable (C) bits for region 0-7
  8-31  Reserved (0)
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

## C6,C0..C7,0 - Protection Unit Data/Unified Region 0..7 (R/W)
## C6,C0..C7,1 - Protection Unit Instruction Region 0..7 (if any) (R/W)

```
  Bit     Expl.
  0       Protection Region Enable (0=Disable, 1=Enable)
  1-5     Protection Region Size   (2 SHL X) ;min=(X=11)=4KB, max=(X=31)=4GB
  6-11    Reserved (0)
  12-31   Protection Region Base address (Addr = Y*4K; must be SIZE-aligned)
```

Overlapping Regions are allowed, Region 7 is having highest priority, region 0 lowest priority.

## Background Region

Additionally, any memory areas outside of the eight Protection Regions are handled as Background Region, this region has neither Read nor Write access.

## Unified Region Note

On the NDS, the Region registers are unified (C6,C0..C7,1 are read/write-able mirrors of C6,C0..C7,0). Nethertheless, the Cachabilty and Permission registers are NOT unified (separate registers exists for code and data settings).
