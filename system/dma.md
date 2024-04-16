# DMA

DSには、各CPUに4つのDMAチャンネル（計8チャンネル）が搭載されており、GBAとほぼ同様に動作しています。

よって、NDSのDMA転送は、[GBAのDMA転送](https://github.com/akatsuki105/gba-docs-ja/blob/main/dma.md)を見れば、だいたいは理解できます。

## NDS9のDMA と GBAのDMA の違い

NDS9のDMA関連のレジスタは、GBAと違い、全て読み書き可能です。

NDS9では全部のDMAチャネルのワードカウント領域が21bitに拡張され、`1..1FFFFFh`の数のワードを指定できるようになりました。(0にしたときは`200000h`ワード)

またSAD,DADレジスタは全部のDMAチャネルで、`0..0FFFFFFEh`の範囲から指定可能になりました。

GBAと違い`Game Pak DRQ`(CNT_Hのbit11)は存在しません。  
NDS9では、CNT_H.11は、2bitだった転送モード(元々CNT_Hのbit12-13で指定?)を3bitに拡張するために使われています。

転送モードの内容は次のようになります。

```
  0  即座に
  1  VBlank開始時
  2  HBlank開始時(VBlank期間中には無効)
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
  0  即座に
  1  VBlank開始時
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

(後述の理由で)DMAはTCMからデータを読み出すことができず、メインメモリから読み出すにはキャッシュと書き込みバッファを再帰する必要があるため、これは便利です。

The DMA Filldata is used with Src=Fixed and SAD=40000Exh (which isn't optimal because it's doing repeated reads from SAD, and, for that reason, a memfill via STMIA opcodes can be faster than DMA; the DSi's new NDMA channels are providing a faster fill method with Src=Fill and SAD=Unused).

## サウンドDMA(NDS7)

NDSには、さらに16個のサウンドDMAチャンネルと2個のサウンドキャプチャーDMAチャンネルが含まれています。

詳細は[サウンド](../sound/README.md)を見てください。

これらのチャンネルの優先順位は不明です。

## NDS9キャッシュ, Writebuffer, DTCM, and ITCM

キャッシュとTCMはシステムバスを使用せず、NDS9のCPUに直接接続されているので、DMAはDTCM/ITCMにアクセスできません。

またキャッシュされた部分のメモリ領域へのアクセスには注意が必要です。DMAリードの前にライトバッファをドレインし、DMAライトの後にキャッシュを無効にしてください。

CPUは、TCM（またはキャッシュ）のみにアクセスする場合は、DMA中も動き続けることができます。

DMA中に割り込みが起きた場合、IRQハンドラのプログラムがTCMかキャッシュ以外のところにある場合、CPUが停止してしまいます。0xFFFF_00xx の IRQベクタ はキャッシュに配置するか、0x0000_00xxのITCMに配置されなければならず、IRQハンドラはIE、IF、その他のI/Oポートにアクセスすることはできません。

## メインメモリへのDMAのシーケンシャルアクセス

メインメモリ(`0x02000000..03000000`)は、シーケンシャルアクセスとノンシーケンシャルアクセスでアクセス時間が異なります。

通常、DMAは(最初のワードを除いて)シーケンシャルアクセスによって転送を行いますが、転送元アドレスと転送先アドレスの**両方**がメインメモリにある場合は、すべてのアクセスがノンシーケンシャルになります。

この場合、メインメモリからWRAMのスクラッチバッファ(?)への転送と、WRAMのスクラッチバッファからメインメモリへの転送と、DMA転送を2回に分けて使用した方が高速になります。

