# ポリゴン属性

## 40004A4h - Cmd 29h - POLYGON_ATTR - ポリゴン属性の設定 (W)

```
  Bit    Expl.
  0-3    光源0~3のライティングの影響を受けるか(bit0: 光源0, bit1: 光源1, bit2: 光源2, bit3: 光源3)
  4-5    Polygon Mode  (0=Modulation, 1=デカール, 2=Toon/Highlight Shading, 3=影ポリゴン)
  6      Polygon Back Surface   (0=Hide, 1=Render)  ;Line-segments are always
  7      Polygon Front Surface  (0=Hide, 1=Render)  ;rendered (no front/back)
  8-10   不使用
  11     半透明なピクセルの場合に深度バッファを半透明ピクセルの深度値で更新するか  (0=元の値のまま, 1=更新する)
  12     Far-plane intersecting polygons       (0=Hide, 1=Render/clipped)
  13     1ドットポリゴンが DISP_1DOT_DEPTH でセットした値より奥の時に表示にするか (0=しない, 1=する)
  14     Depth Test, Draw Pixels with Depth    (0=Less, 1=Equal) (usually 0)
  15     フォグ有効化                            (0=Disable, 1=Enable)
  16-20  アルファ値      (0=ワイヤーフレーム, 1..30=半透明, 31=ソリッド)
  21-23  不使用
  24-29  ポリゴンID (00h..3Fh, used for translucent, shadow, and edge-marking)
  30-31  不使用
```

Writes to POLYGON_ATTR have no effect until next BEGIN_VTXS command.

Changes to the Light bits have no effect until lighting is re-calculated by Normal command. The interior of Wire-frame polygons is transparent (Alpha=0), and only the lines at the polygon edges are rendered, using a fixed Alpha value of 31.

## 4000480h - Cmd 20h - COLOR - 頂点カラー設定 (W)

```
  0-4    赤
  5-9    緑
  10-14  青
  15-31  不使用
```

5ビットのRGB値(RGB555)は、内部的に次のように6ビットのRGBに拡張されます: `X=X*2+(X+31)/32` (つまり、0は0のままだが、それ以外の値(X)は`X=X*2+1`)

このColorコマンド(`COLOR`)の他に、(bit15が1のときの)MaterialColor0コマンド(`DIF_AMB`) や Normalコマンド(`NORMAL`)でも頂点の色を変更できます。

## 深度テスト

The Depth Test compares the depth of the pixels of the polygon with the depth of previously rendered polygons (or of the rear plane if there have been none rendered yet). The new pixels are drawn if the new depth is Less (closer to the camera), or if it is Equal, as selected by POLYGON_ATTR.Bit14.

Normally, Depth Equal would work only exact matches (ie. if the overlapping polygons have exactly the same coordinates; and thus have the same rounding errors), however, the NDS hardware is allowing “Equal” to have a tolerance of +/-200h (within the 24bit depth range of 0..FFFFFFh), that may bypass rounding errors, but it may also cause nearby polygons to be accidently treated to have equal depth.

