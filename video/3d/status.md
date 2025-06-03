# ステータス

## 0x0400_0600 - GXSTAT - ジオメトリエンジンステータスレジスタ (R/W)

bit15に`1`を書き込むとエラーフラグ(bit15)がリセットされ、さらに射影行列スタックポインタ(bit13)がリセットされ、おそらく(？)テクスチャスタックポインタもリセットされます。

```
  Bit
  0     R   BoxTest,PositionTest,VectorTest Busy (0=Ready, 1=Busy)
  1     R   BoxTestの結果  (0=All Outside View, 1=Parts or Fully Inside View)
  2-7   R   不使用
  8-12  R   Position & Vector Matrix Stack Level (0..31) (lower 5bit of 6bit value)
  13    R   Projection Matrix Stack Level        (0..1)
  14    R   Matrix Stack Busy (0=No, 1=Yes; Currently executing a Push/Pop command)
  15    R/W Matrix Stack Overflow/Underflow Error (0=No, 1=Error/Acknowledge/Reset)
  16-24 R   Number of 40bit-entries in Command FIFO  (0..256)
 (24)   R   GXFIFO が満杯か (MSB of above)  (0=No, 1=Yes; Full)
  25    R   GXFIFO が半分未満か  (0=No, 1=Yes; Less than Half-full)
  26    R   GXFIFO が空か                (0=No, 1=Yes; Empty)
  27    R   Busy状態か(コマンド実行中か, 1=Busy)
  28-29 R   不使用
  30-31 R/W GXFIFO IRQのタイミング (0=なし, 1=(コマンドキューが)半分未満になったとき, 2=(コマンドキューが)空のとき, 3=不使用(予約))
```

When GXFIFO IRQ is enabled (setting 1 or 2), the IRQ flag (IF.Bit21) is set while and as long as the IRQ condition is true (and attempts to acknowledge the IRQ by writing to IF.Bit21 have no effect). So that, the IRQ handler must either fill the FIFO, or disable the IRQ (setting 0), BEFORE trying to acknowledge the IRQ.

## 0x0400_0604 - RAM_COUNT - Polygon List & Vertex RAM Count Register (R)

```
  0-11   ポリゴンRAMに格納されているポリゴンの数 (0..2048)
  12-15  不使用
  16-28  頂点RAMに格納されている頂点の数       (0..6144)
  29-31  不使用
```

`SWAP_BUFFERS`コマンドが送信された場合、カウンタは次のVBlankの10サイクル後にリセットされます。(1サイクルは33.51MHzクロック)

## 0x0400_0320 - RDLINES_COUNT - Rendered Line Count Register (R)

レンダリングはスキャンライン214から始まり、レンダリングされたラインは最大48スキャンラインを保持できるバッファに格納されます。実際の画面出力はスキャンライン262以降に開始され、ラインはバッファから読み出されてディスプレイに送られます。 Simultaneously, the rendering engine keeps writing new lines to the buffer (ideally at the same speed than display output, so the buffer would always contain 48 pre-calculated lines).

```
  0-5    Minimum Number (minus 2) of buffered lines in previous frame (0..46)
  6-31   不使用
```

If rendering becomes slower than the display output, then the number of buffered lines decreases. Smaller values in RDLINES indicate that additional load to the rendering engine may cause buffer underflows in further frames, if so, the program should reduce the number of polygons to avoid display glitches.

Even if RDLINES becomes zero, it doesn’t indicate whether actual buffer underflows have occured or not (underflows are indicated in DISP3DCNT.12).
