# 最終的な画面出力

最終的な3D画像（ポリゴンと[リアプレーン](./rear_plane.md)で構成）は、(DISPCNTがBG0として3Dを使用するように設定されている場合に)BG0レイヤとして2DエンジンAに渡されます。

## スクロール

BG0HOFSレジスタ（4000010h）は、3Dレイヤを水平方向にスクロールするのに使用できます。スクロール領域は512ピクセルで、3Dイメージの256ピクセルと、それに続く256の透明ピクセルで構成され、その後再び3Dイメージに折り返されます。

垂直スクロール（および回転/拡大縮小）は3Dレイヤでは使用できません。

## BG優先度

BG0CNTレジスタ（4000008h）の下位2bitは、他のBGやOBJとの相対的な優先順位を制御するため、3Dレイヤを2Dレイヤの前に置くことも後ろに置くこともできます。(つまり2DのBG0と同じように設定できる)

BG0CNTの他のすべてのビットは3Dに影響しません。(例えば、3Dレイヤではモザイクを使用できない)

## 特殊効果

特殊効果に関するレジスタ(`4000050h..54h`)は次のように使用できます:

```
  Brightness up/down with BG0 as 1st Target via EVY   (as for 2D)
  Blending with BG0 as 2nd Target via EVA/EVB         (as for 2D)
  Blending with BG0 as 1st Target via 3D Alpha-values (unlike as for 2D)
```

The latter method probably (?) uses per-pixel 3D alpha values as such: EVA=A/2, and EVB=16-A/2, without using the EVA/EVB settings in 4000052h.

## ウィンドウ

ウィンドウ機能(4000040h..4Bh)は2Dと同じように使用できます。

“If the 3D screen has highest priority, then alpha-blending is always enabled, regardless of the Window Control register’s color effect enable flag [ie. regardless of Bit5 of WIN0IN, WIN1IN, WINOBJ, WINOUT registers]”… not sure if that is true, and if it superseedes the effect selection in Port 4000050h…?

