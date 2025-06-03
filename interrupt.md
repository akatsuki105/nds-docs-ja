# 割り込み

## 0x0400_0208 - NDS9/NDS7 - IME - 割り込み有効フラグ (R/W)

割り込み機能を使用するか使用しないかを設定します。`0`か`1`を指定するだけです。

```
  Bit   Expl.
  0     割り込み有効フラグ (0=全ての割り込み無効, 1=有効(IEで設定した割り込みを受け付ける))
  1-31  不使用
```

## 0x0400_0210 - NDS9/NDS7 - IE - 割り込み許可フラグ (R/W)

GBA では16bitでしたが割り込みソースの数が増えたため32bitに拡張されました。

許可する割り込みの種類を管理するフラグです。 対応するビットが`0`の場合、IRQは無視されます。

どのビットとどの割り込みが対応しているかは、後述です。

## 0x0400_0214 - NDS9/NDS7 - IF - IRQフラグ (R/W)

こちらも32bitに拡張されました。

発生した割り込みリクエスト(IRQ)に対応するビットが立てられます。

`IF`の対応するビットを`1`にして書き込むと、`IF`のそのビットをアクノリッジします。(`IF`に対しての書き込みは、bitが`0`の場合は`IF`を何も変更せず、1の場合は`IF`のそのbitをクリアします。)

例えば、 `IF`が `0xFFFF_FFFF` のときに `0x0010_0001` を書き込むと `0xFFEF_FFFE` になります。

## 割り込みソース

IE と IF のビットと割り込みの対応は以下の通りです。

```
  Bit   割り込み
  0     VBlank
  1     HBlank
  2     LCD V-Counter Match
  3     Timer0 オーバーフロー
  4     Timer1 オーバーフロー
  5     Timer2 オーバーフロー
  6     Timer3 オーバーフロー
  7     NDS7 only: SIO/RCNT/RTC (Real Time Clock)
  8     DMA 0
  9     DMA 1
  10    DMA 2
  11    DMA 3
  12    キー入力
  13    GBA-Slot (external IRQ source) / DSi: None such
  14    不使用                          / DSi9: NDS-Slot Card change?
  15    不使用                          / DSi: dito for 2nd NDS-Slot?
  16    IPC Sync
  17    IPC Send FIFO Empty
  18    IPC Recv FIFO Not Empty
  19    NDS-Slot Game Card Data Transfer Completion
  20    NDS-Slot Game Card IREQ_MC
  21    NDS9 only: Geometry Command FIFO
  22    NDS7 only: Screens unfolding
  23    NDS7 only: SPI bus
  24    NDS7 only: Wifi    / DSi9: XpertTeak DSP
  25    不使用              / DSi9: Camera
  26    不使用              / DSi9: Undoc, IF.26 set on FFh-filling 40021Axh
  27    不使用              / DSi:  Maybe IREQ_MC for 2nd gamecard?
  28    不使用              / DSi: NewDMA0
  29    不使用              / DSi: NewDMA1
  30    不使用              / DSi: NewDMA2
  31    不使用              / DSi: NewDMA3
```
