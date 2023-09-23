# DMA

DSには、各CPUに4つのDMAチャンネル（計8チャンネル）が搭載されており、GBAとほぼ同様に動作しています。

よって、NDSのDMA転送は、[GBAのDMA転送](https://github.com/pokemium/gba-docs-ja/blob/main/dma.md)を見れば、だいたいは理解できます。

## NDS9のDMA と GBAのDMA の違い

NDS9のDMA関連のレジスタは、GBAと違い、全て読み書き可能です。

NDS9では全部のDMAチャネルのワードカウント領域が21bitに拡張され、`1..1FFFFFh`の数のワードを指定できるようになりました。(0にしたときは`200000h`ワード)

またSAD,DADレジスタは全部のDMAチャネルで、`0..0FFFFFFEh`の範囲から指定可能になりました。

GBAと違い`Game Pak DRQ`(CNT_Hのbit11)は存在しません。  
NDS9では、CNT_H.11は、2bitだった転送モード(元々CNT_Hのbit12-13で指定?)を3bitに拡張するために使われています。

転送モードの内容は次のようになります。

```
  0  Start Immediately
  1  Start at V-Blank
  2  Start at H-Blank (paused during V-Blank)
  3  Synchronize to start of display
  4  Main memory display
  5  DS Cartridge Slot
  6  GBA Cartridge Slot
  7  Geometry Command FIFO
```

## NDS7のDMA と GBAのDMA の違い

NDS7のDMA関連のレジスタは、全て読み書き可能になった点以外は、GBAのものと同じです。

よって、DMA転送はGBAと同様、最大4000h(DMA3は10000h)ワードまでで、転送元と転送先アドレスも`0..07FFFFFEh`の範囲に限られます。

GBAと違い`Game Pak DRQ`(CNT_H.11)は存在しません。NDS9と違って転送モードのためのビットフィールドは2bitのままです。 

転送モードの内容は次のようになります。

```
  0  Start Immediately
  1  Start at V-Blank
  2  DS Cartridge Slot
  3  DMA0/DMA2: Wireless interrupt, DMA1/DMA3: GBA Cartridge Slot
```

## 40000E0h - NDS9 only - DMA0FILL - DMA 0 Filldata (R/W)
## 40000E4h - NDS9 only - DMA1FILL - DMA 1 Filldata (R/W)
## 40000E8h - NDS9 only - DMA2FILL - DMA 2 Filldata (R/W)
## 40000ECh - NDS9 only - DMA3FILL - DMA 3 Filldata (R/W)

DMAによるメモリ埋めを行う際に、メモリ埋めをするためのデータを入れておくレジスタです。

```
  Bit0-31 Filldata
```

The DMA Filldata registers contain 16 bytes of general purpose WRAM, intended to be used as fixed source addresses for DMA memfill operations.

This is useful because DMA cannot read from TCM, and reading from Main RAM would require to recurse cache & write buffer.

The DMA Filldata is used with Src=Fixed and SAD=40000Exh (which isn't optimal because it's doing repeated reads from SAD, and, for that reason, a memfill via STMIA opcodes can be faster than DMA; the DSi's new NDMA channels are providing a faster fill method with Src=Fill and SAD=Unused).

## サウンドDMA(NDS7)

NDSには、さらに16個のサウンドDMAチャンネルと2個のサウンドキャプチャーDMAチャンネルが含まれています。

詳細は[サウンド](../sound/README.md)を見てください。

これらのチャンネルの優先順位は不明です。

## NDS9 Cache, Writebuffer, DTCM, and ITCM

Cache and tightly coupled memory are connected directly to the NDS9 CPU, without using the system bus. So that, DMA cannot access DTCM/ITCM, and access to cached memory regions must be handled with care: Drain the writebuffer before DMA-reads, and invalidate the cache after DMA-writes. See,

- [ARM CP15 System Control Coprocessor](#armcp15systemcontrolcoprocessor)
The CPU can be kept running during DMA, provided that it is accessing only TCM
(or cached memory), otherwise the CPU is halted until DMA finishes.

Respectively, interrupts executed during DMA will usually halt the CPU (unless
the IRQ handler uses only TCM and cache; the IRQ vector at FFFF00xxh must be
cached, or relocated to ITCM at 000000xxh, and the IRQ handler may not access
IE, IF, or other I/O ports).

## メインメモリへのDMAのシーケンシャルアクセス

メインメモリ(`0x02000000..03000000`)は、シーケンシャルアクセスとノンシーケンシャルアクセスでアクセス時間が異なります。

通常、DMAは(最初のワードを除いて)シーケンシャルアクセスによって転送を行いますが、転送元アドレスと転送先アドレスの**両方**がメインメモリにある場合は、すべてのアクセスがノンシーケンシャルになります。

この場合、メインメモリからWRAMのスクラッチバッファ(?)への転送と、WRAMのスクラッチバッファからメインメモリへの転送と、DMA転送を2回に分けて使用した方が高速になります。

