# メモリアクセス時間

## システムクロック

```
  Bus clock  = 33MHz (33.513982 MHz)
  NDS7 clock = 33MHz (Busと同じ)
  NDS9 clock = 66MHz (Busの2倍)
```

このドキュメントでは、33MHzのクロックを1サイクルとして扱います。

つまり、NDS9のクロックサイクルはここでは0.5サイクルになります。

## メモリアクセスにかかる時間

以下の表は、ARM7/ARM9 CPUにおけるコードフェッチ・データフェッチのアクセス時間の違いを、シーケンシャル/ノンシーケンシャルの32bit/16bitアクセスで測定したものです。

注：8bitデータのアクセスにかかる時間は、16bitデータと同じ時間です。

注: GBAスロットのアクセス時間は`EXMEMCNT`レジスタで設定できます。

### コードフェッチ

### NDS7

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
メインメモリ (Read) (キャッシュなし) | 9 | 2 | 8 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 2 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 16 | 12 | 10 | 6 | 16 

### NDS9

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
メインメモリ (Read) (キャッシュなし) | 9 | 9 | 4.5 | 4.5 | 16 
WRAM,BIOS,I/O,OAM | 4 | 4 | 2 | 2 | 32 
VRAM,Palette RAM | 5 | 5 | 2.5 | 2.5 | 16 
GBA ROM (example 10,6 access) | 19 | 19 | 9.5 | 9.5 | 16 
ITCM, キャッシュ | 0.5 | 0.5 | 0.5 | 0.5 | 32 

NDS9のコードフェッチには、キャッシュ、ITCM、メインメモリ以外のすべてのノンシーケンシャルアクセスに3サイクルのペナルティがあります。(=3サイクル余分にかかります)

さらに、NDSはコード領域のシーケンシャルアクセスをサポートしていないため、すべてのコードフェッチは強制的に32bitのノンシーケンシャルアクセスにされます。これはTHUMBコードにも当てはまります。つまり2つの16bitオペコードを1回のノンシーケンシャルな32bitアクセスでフェッチします。

つまり、NDS9のコードフェッチは基本的に `N32 + 3`サイクルかかります。

### データフェッチ

**NDS7**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
メインメモリ (read) (キャッシュなし) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 1 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 15 | 12 | 9 | 6 | 16 
GBA ROM (example 10 access) | 9 | 10 | 9 | 10 | 8 

NDS7のコードフェッチと全く同じです。

ただし、コードフェッチと比べると、なぜかメインメモリのノンシーケンシャルアクセスは1サイクル遅く、GBAスロットのノンシーケンシャルアクセスは1サイクル速いです。

**NDS9**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
メインメモリ (read) (キャッシュなし) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 4 | 1 | 4 | 1 | 32 
VRAM,Palette RAM | 5 | 2 | 4 | 1 | 16 
GBA ROM (example 10,6 access) | 19 | 12 | 13 | 6 | 16 
GBA ROM (example 10 access) | 13 | 10 | 13 | 10 | 8 
TCM, キャッシュ | 0.5 | 0.5 | 0.5 | -- | 32 
Cache_Miss (BIOS) | 11 | 11 | 11 | -- | 32 
Cache_Miss (メインメモリ) | 23 | 23 | 23 | -- | 16 

シーケンシャルとノンシーケンシャル、16bitと32bitの両方のアクセスが可能なので、NDS9のコードフェッチよりも高速になっています。

ただし、NDS9のコードフェッチ同様、ノンシーケンシャルなアクセスには3サイクルのペナルティがあります。

またメインメモリへのノンシーケンシャルアクセスには、コードフェッチと比べると1サイクル余計に時間がかかります。

## Actual CPU Performance

バスのクロックと等しいクロックで動作するNDS7とは違い、66MHzで動作するNDS9は、メモリアクセスの待ち時間が非常に多く、実質的なバスのクロックは8～16MHz程度になっています。

唯一の例外は、キャッシュまたはTCM内のコードアクセスとデータアクセスで、0.5サイクル(NDS9の1クロックサイクル)でアクセスできます。<sup>[1](#bus)</sup>

まとめると、NDS9では、すべての外部メモリアクセス（およびI/O）はバスクロックに遅延されるため、66MHzのクロックを活かせるのはキャッシュとTCMでのみです。

```
ARM9 コードフェッチ
  常に N32 + 3 サイクル
ARM9 opcode fetches are always N32 + 3 waits.
- S16 and N16 do not exist (because thumb-double-fetching) (see there).
- S32 becomes N32 (ie. the ARM9 does NOT support fast sequential timing).

That N32 is having same timing as normal N32 access on NDS7, plus 3 waits.
- Eg. an ARM9 N32 or S32 to 16bit bus will take: N16 + S16 + 3 waits.
- Eg. an ARM9 N32 or S32 to 32bit bus will take: N32 + 3 waits.
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

ARM7と違い、ARM9はコードバスとデータバスが独立しているため、コードフェッチとデータフェッチを同時に実行できます。(ただし、両者が異なるメモリ領域にあることが条件です。)

普通、オペコードの実行時間は`(codetime+datatime) `として計算されますが、2つのバスを使えば、`MAX(codetime,datatime) `となります。

In practice, DTCM and Data Cache access can take NULL cycles (however, data access to ITCM can't).

When executing code in cache/itcm, data access to non-cache/tcm won't be any faster than with only one bus (as it's best, it could subtract 0.5 cycles from datatime, but, the access must be "aligned" to the bus-clock, so the "datatime-0.5" will be rounded back to the original "datatime").

キャッシュされていないメインメモリでコードを実行し、（メインメモリやキャッシュ/TCM以外の）データにアクセスする場合、実行時間は通常`codetime+datatime-2`サイクルになります。

## NDS9内部サイクル

Additionally to codetime+datatime, some opcodes include one or more internal cycles. Compared with ARM7, the behaviour of that internal cycles is slightly different on ARM9. First of, on the NDS9, the internal cycles are of course "half" cycles (ie. counted in 66MHz units, not in 33MHz units) (although they may get rounded to "full" cycles upon next memory access outside tcm/cache). And, the ARM9 is in some cases "skipping" the internal cycles, that often depending on whether or not the next opcode is using the result of the current opcode.
Another big difference is that the ARM9 has lost the fast-multiply feature for small numbers; in some cases that may result in faster execution, but may also result in slower execution (one workaround would be to manually replace MUL opcodes by the new ARM9 halfword multiply opcodes); the slowest case are MUL opcodes that do update flags (eg. MULS, MLAS, SMULLS, etc. in ARM mode, and all ALL multiply opcodes in THUMB mode).

## NDS9 THUMBコードフェッチ

THUMBモードでは、NDS9は1回の32bitリードで2つの16bitオペコードをフェッチします。

もちろん、これは2つのオペコードがワードアラインメントされた領域内にある場合にのみ機能します（例えば、ワードアラインされたアドレスのループは、アラインメントされていないループよりも高速になります）。

しかし、このダブルオペコードフェッチは、分岐命令後のオペコードのような不要なフェッチも含め、16bitバス幅のアクセスでも行われるため、大幅な速度低下が発生する可能性があります。

## メインメモリ

Reportedly, the main memory access times would be 5 cycles (nonsequential read), 4 cycles (nonsequential write), and 1 cycle (sequential read or write). Plus whatever termination cycles. Plus 3 cycles on nonsequential access to the last 2-bytes of a 32-byte block.
That's of course all wrong. Reads are much slower than 5 cycles. Not yet tested if writes are faster. And, I haven't been able to reproduce the 3 cycles on last 2-bytes effect, actually, it looks more as if that 3 cycles are accidently added to ALL nonsequential accesses, at ALL main memory addresses, and even to most OTHER memory regions... which might be the source of the PENALTY which occurs on VRAM/WRAM/OAM/Palette and I/O accesses.

## DMA

In some cases DMA main memory read cycles are reportedly performed simultaneously with DMA write cycles to other memory.

## VRAM Waitstates

Additionally, on NDS9, a one cycle wait can be added to VRAM accesses (when the video controller simultaneously accesses it) (that can be disabled by Forced Blank, see DISPCNT.Bit7). Moreover, additional VRAM waitstates occur when using the video capture function.
Note: VRAM being mapped to NDS7 is always free of additional waits.

<sup id="bus">1: これは、キャッシュがヒットすることを前提としており、キャッシュがミスした場合、キャッシュメモリのタイミングは1.4MHz程度まで下がるかもしれません。</sup>
