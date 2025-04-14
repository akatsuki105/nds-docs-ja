# メモリアクセス時間

```
  Bus clock  = 33MHz (33.513982 MHz)
  NDS7 clock = 33MHz (Busと同じ)
  NDS9 clock = 66MHz (Busの2倍)
```

このドキュメントでは、バスクロック(33MHz)を1サイクルとして扱います。

つまり、NDS9のクロックサイクルはここでは0.5サイクルになります。

## メモリアクセスにかかる時間

以下の表は、ARM7/ARM9 CPUにおけるコードフェッチ・データフェッチのアクセス時間の違いを、シーケンシャル/ノンシーケンシャルの32bit/16bitアクセスで測定したものです。

注：8bitデータのアクセスにかかる時間は、16bitデータと同じ時間です。

注: GBAスロットのアクセス時間は`EXMEMCNT`レジスタで設定できます。

### コードフェッチ

```
  NDS7/CODE             NDS9/CODE
  N32 S32 N16 S16 Bus   N32 S32 N16 S16 Bus
  9   2   8   1   16    9   9   4.5 4.5 16  Main RAM (read) (cache off)
  1   1   1   1   32    4   4   2   2   32  WRAM,BIOS,I/O,OAM
  2   2   1   1   16    5   5   2.5 2.5 16  VRAM,Palette RAM
  16  12  10  6   16    19  19  9.5 9.5 16  GBA ROM (example 10,6 access)
  -   -   -   -   -     0.5 0.5 0.5 0.5 32  TCM, Cache_Hit
  -   -   -   -   -     (--Load 8 words--)  Cache_Miss
```

NDS9のコードフェッチでは、すべてのノンシーケンシャルアクセス（キャッシュ、TCM、メインメモリを除く）に3サイクルのペナルティが追加されます。

さらに、NDSはコード領域のシーケンシャルアクセスをサポートしていないため、すべてのコードフェッチは強制的に32bitのノンシーケンシャルアクセスにされます。これはTHUMBコードにも当てはまります。つまり2つの16bitオペコードを1回のノンシーケンシャルな32bitアクセスでフェッチします。

つまり、NDS9のコードフェッチは基本的に `N32 + 3`サイクルかかります。

### データフェッチ

```
  NDS7/DATA             NDS9/DATA
  N32 S32 N16 S16 Bus   N32 S32 N16 S16 Bus
  10  2   9   1   16    10  2   9   1   16  Main RAM (read) (cache off)
  1   1   1   1   32    4   1   4   1   32  WRAM,BIOS,I/O,OAM
  1?  2   1   1   16    5   2   4   1   16  VRAM,Palette RAM
  15  12  9   6   16    19  12  13  6   16  GBA ROM (example 10,6 access)
  9   10  9   10  8     13  10  13  10  8   GBA RAM (example 10 access)
  -   -   -   -   -     0.5 0.5 0.5 -   32  TCM, Cache_Hit
  -   -   -   -   -     (--Load 8 words--)  Cache_Miss
  -   -   -   -   -     11  11  11  -   32  Cache_Miss (BIOS)
  -   -   -   -   -     23  23  23  -   16  Cache_Miss (Main RAM)
```

NDS7のデータフェッチはコードフェッチと全く同じです。 ただし、コードフェッチと比べると、なぜかメインメモリのノンシーケンシャルアクセスは1サイクル遅く、GBAスロットのノンシーケンシャルアクセスは1サイクル速いです。

NDS9のデータフェッチは、シーケンシャルとノンシーケンシャル、16bitと32bitの両方のアクセスが可能なので、コードフェッチよりも高速になっています。 ただし、コードフェッチ同様、ノンシーケンシャルなアクセスには3サイクルのペナルティがあります。またメインメモリへのノンシーケンシャルアクセスには、コードフェッチと比べると1サイクル余計に時間がかかります。

## Actual CPU Performance

バスクロックと同じクロックで動作するNDS7とは違い、66MHzで動作するNDS9は、メモリアクセスの待ち時間が非常に多く、実質的なバスクロックは8～16MHz程度になっています。唯一の例外は、キャッシュまたはTCM内のコードアクセスとデータアクセスで、0.5サイクル(NDS9の1クロックサイクル)でアクセスできます。<sup>[1](#bus)</sup> つまり、NDS9では、すべての外部アクセス（およびI/O）はバスクロックに遅延されるため、66MHzのクロックを活かせるのはキャッシュとTCMでのみです。

ARM9コードフェッチ は 常に `N32 + 3` サイクルです。

```
  S16/N16アクセスは存在しない (thumb-double-fetchingのため) (後述)
  S32 は N32 になる (ARM9はシーケンシャルアクセスの高速化をサポートしていないため)
```

NDS7のN32アクセスと同じタイミングで、さらに3サイクル待つことになります。

```
Eg. an ARM9 N32 or S32 to 16bit bus will take: N16 + S16 + 3 waits.
Eg. an ARM9 N32 or S32 to 32bit bus will take: N32 + 3 waits.
```

メインメモリは、常にノンシーケンシャルアクセスに対して3サイクルのペナルティがかかります。ARM7であっても同様です。

ARM9のデータフェッチは、シーケンシャルなアクセスでは高速にアクセスすることができ、16bitアクセスも可能です。 ただし、ノンシーケンシャルなアクセスに対しては3サイクルのペナルティが追加されます。

```
Only exceptions are cache and tcm which do not have that penalty.
 Eg. LDRH on 16bit-data-bus is N16+3waits.
 Eg. LDR  on 16bit-data-bus is N16+S16+3waits.
 Eg. LDM  on 16bit-data-bus is N16+(n*2-1)*S16+3waits.

Eventually, data fetches can take place parallel with opcode fetches.
 That is NOT true for LDM (works only for LDR/LDRB/LDRH).
 That is NOT true for DATA in SAME memory region than CODE.
 That is NOT true for DATA in ITCM (no matter if CODE is in ITCM).
```

## NDS9バス

ARM7と違い、ARM9はコードバスとデータバスが独立しているため、(両者が異なるメモリ領域にある場合)コードフェッチとデータフェッチを同時に実行できます。

普通、オペコードの実行時間は `(codetime+datatime)` として計算されますが、2つのバスを使えば、`max(codetime,datatime) `となるため、(コードアクセスより速い)データアクセスにかかる時間は無視できます。

In practice, DTCM and Data Cache access can take NULL cycles (however, data access to ITCM can't).

When executing code in cache/itcm, data access to non-cache/tcm won't be any faster than with only one bus (as it's best, it could subtract 0.5 cycles from datatime, but, the access must be "aligned" to the bus-clock, so the "datatime-0.5" will be rounded back to the original "datatime").

キャッシュされていないメインメモリのコードを実行し、（メインメモリやキャッシュ/TCM以外の）データにアクセスする場合、実行時間は通常 `(codetime+datatime)-2` サイクルになります。

## NDS9の内部サイクル

オペコードの実行時間は`(codetime+datatime)`ですが、それに加えて`MUL`のようないくつかのオペコードには1つまたは複数の内部サイクル(Iサイクル)が含まれます。

ARM7と比較すると、ARM9ではその内部サイクルの動作が若干異なります。 まず、NDS9では、内部サイクルは当然「半サイクル」です（つまり、33MHz単位ではなく、66MHz単位でカウントされます）。ただし、TCM/キャッシュ外の次のメモリアクセス時に「フルサイクル」に丸められる場合があります）。

And, the ARM9 is in some cases "skipping" the internal cycles, that often depending on whether or not the next opcode is using the result of the current opcode.

Another big difference is that the ARM9 has lost the fast-multiply feature for small numbers; in some cases that may result in faster execution, but may also result in slower execution (one workaround would be to manually replace MUL opcodes by the new ARM9 halfword multiply opcodes)

最も実行速度が遅くなるのは、フラグを更新するMULオペコードです。これは、ARMモードの `MULS, MLAS, SMULLS` やTHUMBモードの乗算命令すべてが該当します。

## NDS9 THUMBコードフェッチ

NDS9のTHUMBモードでは1回の32bitリードで2つの16bitオペコードをフェッチします。

もちろん、これは2つのオペコードがワードアラインメントされた領域内にある場合にのみ機能します（例えば、ワードアラインされたアドレスのループは、アラインメントされていないループよりも高速になります）。

しかし、このダブルオペコードフェッチは、分岐命令後のオペコードのような不要なフェッチも含め、16bitバス幅のアクセスでも行われるため、大幅な速度低下が発生する可能性があります。

## DMA

一部のケースでは、DMAによるメインメモリの読み込みサイクルが、他のメモリへのDMAによる書き込みサイクルと同時に行われることが報告されています。

## VRAMウェイトステート

NDS9では、（ビデオコントローラがCPUと同時にアクセスする場合）VRAMアクセスに1サイクルのウェイトステートが追加されることがあります。(`DISPCNT.7`でF-Blank中の場合は、起きません) 

また、キャプチャ機能を使用すると、VRAMに追加のウェイトステートが発生するようです。

NDS7にマッピングされたVRAMは、常に追加のウェイトステートなしで利用できます。

<sup id="bus">1: これは、キャッシュがヒットすることを前提としており、キャッシュがミスした場合、キャッシュメモリのタイミングは1.4MHz程度まで下がるかもしれません。</sup>
