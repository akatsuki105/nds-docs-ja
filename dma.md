# DMA

DSには、各CPUに4つのDMAチャンネル（計8チャンネル）が搭載されており、GBAとほぼ同様に動作しています。

よって、NDSのDMA転送は、[GBAのDMA転送](https://github.com/akatsuki105/gba-docs-ja/blob/main/dma.md)を見れば、だいたいは理解できます。(レジスタのアドレスも同じです)

## GBAのDMAとの違い

GBAでは、`DMAnSAD` と `DMAnDAD` は書き込み専用で、読み取りはできませんでしたが、NDSでは読み書き可能になりました。

さらにNDS9のDMAでは、`DMAnSAD` と `DMAnDAD` で指定できる範囲が、`0..0FFFFFFEh` に拡張されました。(NDS7ではGBAと変わらず`0..07FFFFFEh`のまま)

`DMAnCNT`は、GBAと同じく32bitですが、次のように変更されています。

さらにNDS9では、DMAでデータ埋め(fill)を行う際に、メモリ埋めをするためのデータを入れておくレジスタ`DMAnFILL`が追加されました。

## 40000B8h - NDS9/NDS7 - DMA0CNT - 32bit - DMA0制御レジスタ (R/W)
## 40000C4h - NDS9/NDS7 - DMA1CNT - 32bit - DMA1制御レジスタ (R/W)
## 40000D0h - NDS9/NDS7 - DMA2CNT - 32bit - DMA2制御レジスタ (R/W)
## 40000DCh - NDS9/NDS7 - DMA3CNT - 32bit - DMA3制御レジスタ (R/W)

```
  0-20    転送するワード数(0..1FFFFFh)
            NDS9: 0にすると 200000hワード
            NDS7: bit0-15のみが使用され、 0にすると 4000hワード (DMA3のみ 10000hワード)
  21-22   ターゲットアドレス制御(0..3)
            0b00: +1
            0b01: -1
            0b10: ±0
            0b11: +1/Reload
  23-24   ソースアドレス制御(0..3)
            0b00: +1
            0b01: -1
            0b10: ±0
            0b11: 指定禁止 (指定した場合は、 +1 ?)
  25      DMAリピート
  26      ワードサイズ (0=16bit, 1=32bit)
  27      DMA転送モード拡張bit (NDS9のみ, NDS7ではこのbitは不使用)
  28-29   DMA転送モード (0..3)
            NDS9: (= (DMAnCNT >> 27) & 7)
             0: 即座に
             1: VBlank開始時
             2: HBlank開始時(VBlank期間中には無効)
             3: Synchronize to start of display
             4: Main memory display
             5: DS Cartridge Slot
             6: GBA Cartridge Slot
             7: Geometry Command FIFO
            NDS7: (= (DMAnCNT >> 28) & 3)
              0: 即座に
              1: VBlank開始時
              2: DS Cartridge Slot
              3: DMA0/DMA2: Wireless interrupt, DMA1/DMA3: GBA Cartridge Slot
  30      転送終了時にIRQを発生させるかどうか
  31      このDMAチャンネルが有効かどうか
```

## 40000E0h - NDS9 only - DMA0FILL - DMA 0 Filldata (R/W)
## 40000E4h - NDS9 only - DMA1FILL - DMA 1 Filldata (R/W)
## 40000E8h - NDS9 only - DMA2FILL - DMA 2 Filldata (R/W)
## 40000ECh - NDS9 only - DMA3FILL - DMA 3 Filldata (R/W)

DMAによるメモリ埋めを行う際に、メモリ埋めをするためのデータを入れておくレジスタです。

```
  0-31   Filldata
```

実態は(4x4の合計で)16バイトの汎用WRAMで、特殊な操作をしないと利用できないなどはなく、メモリ埋めをしたい場合には、`DMAnSAD` に `0x0400_00Ex` を指定し `DMAnCNT.23-24` で `0b10` を指定します。

(後述の理由で)DMAでは高速なTCM/キャッシュを使用できないため、メモリ埋めの際には比較的高速にアクセスできるこの`DMAnFILL`を使用することをお勧めします。

## TCM, キャッシュ, ライトバッファ

キャッシュとTCMはシステムバスを使用せず、内部バスでアクセスするため、DMAは TCM と キャッシュ にアクセスできません。

キャッシュされた部分のメモリ領域をDMA転送するときは、DMAリードの前にライトバッファをドレインし、DMAライトの後にキャッシュを無効にしてください。

CPUは、TCM（またはキャッシュ）のみにアクセスする場合は、DMA中も動き続けることができます。

DMA中に割り込みが起きた場合、IRQハンドラのプログラムがTCMかキャッシュ以外のところにある場合、CPUが停止してしまいます。`0xFFFF_00xx` の IRQベクタ はキャッシュに配置するか、`0x0000_00xx`のITCMに配置されなければならず、IRQハンドラは`IE, IF`、その他のI/Oポートにアクセスすることはできません。

## メインメモリへのDMAのシーケンシャルアクセス

メインメモリ(`0x02000000..03000000`)は、アクセスがシーケンシャルかどうかでアクセス時間が異なります。

通常、DMAは(最初のワードを除いて)シーケンシャルアクセスによって転送を行いますが、転送元アドレスと転送先アドレスの**両方**がメインメモリにある場合は、すべてのアクセスがノンシーケンシャルになります。

この場合、メインメモリからWRAMのスクラッチバッファへの転送と、WRAMのスクラッチバッファからメインメモリへの転送と、DMA転送を2回に分けて使用した方がメインメモリへの読み書きがシーケンシャルになるため、高速になります。

