# 行列計算

## 4000440h - Cmd 10h - MTX_MODE - マトリックスモード (W)

```
  0-1   マトリックスモード (0..3)
         0  投影行列 (MVP変換のP)
         1  モデルビュー行列 (Position Matrix とも呼ばれる, MVP変換のMV)
         2  モデルビュー行列 & 方向行列 (ライティングとVEC_TESTで使用)
         3  テクスチャ変換行列
  2-31  不使用
```

このコマンド以降の `MTX_xx`コマンドを適用する行列の種類を選択します。

In Mode 2, all MTX commands are applied to both the Position and Vector matrices. There are two special cases:

```
  MTX_SCALE in Mode 2:                  uses ONLY Position Matrix
  MTX_PUSH/POP/STORE/RESTORE in Mode 1: uses BOTH Position AND Vector Matrices
```

Ie. the four stack commands act like mode 2 (even when in mode 1; keeping the two stacks somewhat in sync), and scale acts like mode 1 (even when in mode 2; keeping the light vector length’s intact).

## 4000454h - Cmd 15h - MTX_IDENTITY - Load Unit Matrix to Current Matrix (W)

Sets C=I. Parameters: None

The Identity Matrix (I), aka Unit Matrix, consists of all zeroes, with a diagonal row of ones. A matrix multiplied by the Unit Matrix is left unchanged.

## 4000458h - Cmd 16h - MTX_LOAD_4x4 - Load 4x4 Matrix to Current Matrix (W)

Sets C=M. Parameters: 16, m[0..15]

## 400045Ch - Cmd 17h - MTX_LOAD_4x3 - Load 4x3 Matrix to Current Matrix (W)

Sets C=M. Parameters: 12, m[0..11]

## 4000460h - Cmd 18h - MTX_MULT_4x4 - Multiply Current Matrix by 4x4 Matrix (W)

Sets C=M*C. Parameters: 16, m[0..15]

## 4000464h - Cmd 19h - MTX_MULT_4x3 - Multiply Current Matrix by 4x3 Matrix (W)

Sets C=M*C. Parameters: 12, m[0..11]

## 4000468h - Cmd 1Ah - MTX_MULT_3x3 - Multiply Current Matrix by 3x3 Matrix (W)

Sets C=M*C. Parameters: 9, m[0..8]

## 400046Ch - Cmd 1Bh - MTX_SCALE - Multiply Current Matrix by Scale Matrix (W)

Sets C=M*C. Parameters: 3, m[0..2]

注意: `MTX_SCALE`コマンドは、`MTX_MODE=2`のときでも Vector Matrix を変更しません。(that’s done so for keeping the length of the light vector’s intact)

## 4000470h - Cmd 1Ch - MTX_TRANS - Mult. Curr. Matrix by Translation Matrix (W)

Sets C=M*C. Parameters: 3, m[0..2] (x,y,z position)

## 4000640h..67Fh - CLIPMTX_RESULT - MVP行列読み出しレジスタ (R)

This 64-byte region (16 words) contains the m[0..15] values of the Current Clip Coordinates Matrix, arranged in 4x4 Matrix format. Make sure that the Geometry Engine is stopped (GXSTAT.27) before reading from these registers.

The Clip Matrix is internally used to convert vertices to screen coordinates, and is internally re-calculated anytime when changing the Position or Projection matrices:

```c
  ClipMatrix = MV * P; // MV = Modelview Matrix, P = Projection Matrix
```

MV行列か、P行列のみを読み出したい場合は、もう一方の行列を単位行列に設定してからこのレジスタを読み出してください。

## 4000680h..6A3h - VECMTX_RESULT - Read Current Directional Vector Matrix (R)

This 36-byte region (9 words) contains the m[0..8] values of the Current Directional Vector Matrix, arranged in 3x3 Matrix format (the fourth row/column may contain any values).

Make sure that the Geometry Engine is stopped (GXSTAT.27) before reading from these registers.

