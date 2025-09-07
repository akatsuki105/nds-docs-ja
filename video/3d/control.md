# 制御レジスタ

## 4000060h - DISP3DCNT - 3D制御レジスタ (R/W)

```
  Bit
  0     テクスチャマッピング     (0=Disable, 1=Enable)
  1     PolygonAttr Shading  (0=トゥーンシェーディング, 1=Highlight Shading)
  2     アルファテスト有効化    (0=Disable, 1=Enable) (see ALPHA_TEST_REF)
  3     アルファブレンド       (0=Disable, 1=Enable) (see various Alpha values)
  4     アンチエイリアス       (0=Disable, 1=Enable)
  5     Edge-Marking         (0=Disable, 1=Enable) (see EDGE_COLOR)
  6     Fog Color/Alpha Mode (0=Alpha and Color, 1=Only Alpha) (see FOG_COLOR)
  7     Fog Master Enable    (0=Disable, 1=Enable)
  8-11  Fog Depth Shift      (FOG_STEP=400h shr FOG_SHIFT) (see FOG_OFFSET)
  12    Color Buffer RDLINES Underflow (0=None, 1=Underflow/Acknowledge)
  13    Polygon/Vertex RAM Overflow    (0=None, 1=Overflow/Acknowledge)
  14    リアプレーンモード       (0=Blank, 1=Bitmap)
  15-31 不使用
```

## 4000540h - Cmd 50h - SWAP_BUFFERS - Swap Rendering Engine Buffer (W)

`SWAP_BUFFERS`コマンドでは、2組のポリゴン/頂点バッファを交換します。つまり、新しく定義されたポリゴン/頂点がレンダリングエンジンに渡され、その後のフレームで描画されます。

レンダリングエンジンからジオメトリエンジンに返却された方のバッファは空になり、ジオメトリエンジンに渡されます。(そして、ジオメトリコマンドによって新しいポリゴン/頂点で埋められます)。

```
  0     Translucent polygon Y-sorting (0=自動ソート, 1=手動ソート)
  1     深度バッファ  (0=Zバッファ, 1=Wバッファ; 平行投影時(orthogonal projections)にWバッファを使っても機能しません)
  2-31  不使用
```

SwapBuffers は次のVBlank(スキャンライン=192)まで実行されず、その間ジオメトリエンジンはHaltします。

`SWAP_BUFFERS` はBegin/End内で発行してはいけません。

2つのパラメータビットは、`SWAP_BUFFERS`コマンドの後のgxcommandsに使用されます(つまり、`SWAP_BUFFERS`コマンドより前のgxcommandsには使用されません)。

SwapBuffersは、(2頂点しかない三角ポリゴンのような)不完全なポリゴンリストが定義された場合、3Dハードウェアをロックします。

On lock-up, only 2D video is kept working, any wait-loops for GXSTAT.27 will hang the program. Once lock-up has occured, there seems to be no way to recover by software, not by sending the missing veric(es), and not even by pulsing POWCNT1.Bit2-3.

## 4000580h - Cmd 60h - VIEWPORT - ビューポート設定 (W)

```
  0-7   Screen/BG0 Coordinate X1 (0..255) (For Fullscreen: 0=左端)
  8-15  Screen/BG0 Coordinate Y1 (0..191) (For Fullscreen: 0=画面下)
  16-23 Screen/BG0 Coordinate X2 (0..255) (For Fullscreen: 255=右端)
  24-31 Screen/BG0 Coordinate Y2 (0..191) (For Fullscreen: 191=画面上)
```

座標`(0, 0)`は**左下**です。2Dの場合(左上)とは異なります。

ビューボリューム(大きさはクリップ行列で定義)は、ビューポート領域に合わせて自動的にスケーリングされます。ポリゴンの頂点はビューボリュームにクリップされますが、いくつかの頂点は丸め誤差のためにX2,Y1(右上)境界を1ピクセル超える場合があります。ビューポート設定は3Dリアプレーンのサイズや位置に影響しません。

`VIEWPORT`コマンドは`BEGIN_VTXS/END_VTXS`内で発行してはいけません。

## 4000610h - DISP_1DOT_DEPTH - 1-Dot Polygon Display Boundary Depth (W)

1ドットポリゴンはジオメトリ処理を行った後のポリゴンの頂点のxy座標が全て同じ値を取るポリゴンのことです。(= 画面上で1ドットしか映らないポリゴン)

`DISP_1DOT_DEPTH` でセットした値よりも奥行きの値が大きい（遠くにある）1ドットポリゴンは、自動的に非表示にすることができます。(メモリ消費を抑えたり、画面の汚れを軽減する際に役に立ちます)

```
  0-14  W座標 (Unsigned, 12bit integer, 3bit fractional part)
  15-31 不使用 (0000h=Closest, 7FFFh=Most Distant)
```

The DISP_1DOT_DEPTH comparision can be enabled/disabled per polygon (via POLYGON_ATTR.Bit13), so “important” polygons can be displayed regardless of their size and distance.

Note: The comparision is always using the W-coordinate of the vertex (not the Z-coordinate) (ie. no matter if using Z-buffering, or W-buffering). The polygon is rendered if at least one of its vertices is having a w-coordinate less or equal than DISP_1DOT_DEPTH. NB. despite of checking the w-coords of ALL vertices, the polygon is rendered using the color/depth/texture of its FIRST vertex.

Note: The hardware does round-up the width and height of all polygons to at least 1, so polygons of 0x0, 1x0, 0x1, and 1x1 dots will be all rounded-up to a size of 1x1. Of which, the so-called “1dot” depth check is applied only to the 0x0 dot variant (so “0dot” depth check would be a better name for it).

Caution: Although DISP_1DOT_DEPTH is a Geometry Engine parameter, it is NOT routed through GXFIFO, ie. changes will take place immediately, and will affect all following polygons, including such that are still in GXFIFO. Workaround: ensure that GXFIFO is empty before changing this parameter.

## 4000340h - ALPHA_TEST_REF - アルファテスト閾値 (W)

アルファテストは`DISP3DCNT.2`で有効にでき、有効にすると、ピクセルはそのアルファ値が`ALPHA_TEST_REF`より大きい場合にのみレンダリングされます。

アルファテストが無効なら、このレジスタの値は使われず、ピクセルはアルファ値が0より大きいならレンダリングされます。

アルファテストは、最終的なポリゴンのピクセル(=テクスチャブレンドの後)で実行されます。

```
  0-4   閾値   (0..31, アルファ値がこの値より大きい場合にレンダリングされる)
  5-31  不使用
```

`0x00`にした場合、`DISP3DCNT.2`でアルファテストを無効にした状態と同じになります。

`0x1F`にした場合、全てのポリゴンのピクセルがレンダリングされなくなります。


