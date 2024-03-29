# キャッシュ と TCM

キャッシュとTCMはコプロセッサ(CP15)によって制御されています。

これらはNDS9のためのものです。NDS7は TCMやキャッシュ、CP15 を持ちません。

## TCM(密結合メモリ)

TCM = `Tightly Coupled Memory` = `密結合メモリ`

バスのクロックがARM9のCPUクロックより遅いため、メインメモリへのアクセスはパフォーマンスに制限がかかります。

TCMはARM9のコアに直接入ってるメモリで、CPUクロックと同じ速度(66MHz)で動作します。

キャッシュと違いプログラマが内容をコントロールできるのが特徴で、命令コード用TCM(ITCM)とデータ用TCM(DTCM)があります。

CPUがTCMにアクセスしている間はバスが空くので、その間にDMAコントローラを動作させることができます。

```
  ITCM 32KB, base=00000000h (固定)
  DTCM 16KB, base=moveable  (デフォルトでは base=27C0000h)
```

ITCMは移動できませんが、NDSのファームウェアではITCMのサイズを32MBに設定しているため、`0x0-0x1FF_FFFF`のITCMミラーが生成されます。

DTCMはCP15の`C9,C1,1`でどこにマッピングするかを設定できます。

さらに、PU(後述)を使ってその領域のメモリをロック/アンロックすることができます。このトリックにより、下位32MBのメモリ内のどこにでもITCMを移動させることができます。

## キャッシュ

- 4KBのデータキャッシュ と　8KBの命令キャッシュ
- 4wayセットアソシアティブ
- キャッシュラインは1本32バイト
- Read-allocate method (ie. writes are not allocating cache lines)
- Round-robin and Pseudo-random replacement algorithms selectable
- Cache Lockdown, Instruction Prefetch, Data Preload
- Data write-through and write-back modes selectable

## 保護ユニット(PU)

デフォルトでは次のように設定されています。

Region Name | Address | Size | Cache | WBuf | Code | Data 
-- | -- | -- | -- | -- | -- | -- 
Background     | 0x0000_0000 | 4GB   | -  | -  | -   | -
I/O and VRAM   | 0x0400_0000 | 64MB  | -  | -  | R/W | R/W
メインメモリ     | 0x0200_0000 | 4MB   | On | On | R/W | R/W
ARM7-dedicated | 0x027C_0000 | 256KB | -  | -  | -   | -
GBA Slot       | 0x0800_0000 | 128MB | -  | -  | -   | R/W
DTCM           | 0x027C_0000 | 16KB  | -  | -  | -   | R/W
ITCM           | 0x0100_0000 | 32KB  | -  | -  | R/W | R/W
BIOS           | 0xFFFF_0000 | 32KB  | On | -  | R   | R
Shared Work    | 0x027F_F000 | 4KB   | -  | -  | -   | R/W

Notes: In Nintendo's hardware-debugger, Main Memory is expanded to 8MB (for that reason, some addresses are at 27NN000h instead 23NN000h) (some of the extra memory is reserved for the debugger, some can be used for game development). 

Region 2 and 7 are not understood? GBA Slot should be max 32MB+64KB, rounded up to 64MB, no idea why it is 128MB? DTCM and ITCM do not use Cache and Write-Buffer because TCM is fast. 

Above settings do not allow to access Shared Memory at 37F8000h? Do not use cache/wbuf for I/O, doing so might suppress writes, and/or might read outdated values.

The main purpose of the Protection Unit is debugging, a major problem with GBA programs have been faulty accesses to memory address 00000000h and up (due to \[base+offset\] addressing with uninitialized (zero) base values). 

This problem has been fixed in the NDS, for the ARM9 processor at least, still there are various leaks: For example, the 64MB I/O and VRAM area contains only ca. 660KB valid addresses, and the ARM7 probably doesn't have a Protection Unit at all. Alltogether, the protection is better than in GBA, but it's still pretty crude compared with software debugging tools.

Region address/size are unified (same for code and data), however, cachabilty and access rights are non-unified (and may be separately defined for code and data).
