# OBJs

## OBJ優先度

GBAでは、OBJの2bitの対BG優先度(`OAM2.bit10-11`)に関係なく、OAMの128エントリのインデックスに対してOBJ間の優先順位を割り当てていたため、無効な優先順位を指定することができてしまいました。

DSでは、上記の2つの値を9ビットの優先度値に結合することでこの問題は修正されました。

## OBJタイルマッピング (DISPCNT.4,20-21)

 bit4 | bit20-21 | Dimension Boundary Total | Notes
---- | ---- | ---- | ----
0 | x | 2D | 32  | 32K  | Same as GBA 2D Mapping
1 | 0 | 1D | 32  | 32K  | Same as GBA 1D Mapping
1 | 1 | 1D | 64  | 64K  | --
1 | 2 | 1D | 128 | 128K | --
1 | 3 | 1D | 256 | 256K | Engine B: 128K max

```
  TileVramAddress = TileNumber * BoundaryValue
```

Even if the boundary gets changed, OBJs are kept composed of 8x8 tiles.

## Bitmap OBJ Mapping (DISPCNT.6,5,22)

Bitmap OBJs are 15bit Direct Color data, plus 1bit Alpha flag (in bit15).

Bit6 | Bit5 | Bit22 | Dimension   | Boundary  | Total | Notes
---- | ---- | ---- | ---- | ---- | ---- | ----
0    | 0    | x     | 2D/128 dots | 8x8 dots  | 128K  | Source Bitmap width 128 dots
0    | 1    | x     | 2D/256 dots | 8x8 dots  | 128K  | Source Bitmap width 256 dots
1    | 0    | 0     | 1D          | 128 bytes | 128K  | Source Width = Target Width
1    | 0    | 1     | 1D          | 256 bytes | 256K  | Engine A only
1    | 1    | x     | Reserved    | --        | --    | -- 

In 1D mapping mode, the Tile Number is simply multiplied by the boundary value.

```
  1D_BitmapVramAddress = TileNumber(0..3FFh) * BoundaryValue(128..256)
  2D_BitmapVramAddress = (TileNo AND MaskX)*10h + (TileNo AND NOT MaskX)*80h
```

In 2D mode, the Tile Number is split into X and Y indices, the X index is located in the LSBs (ie. MaskX=0Fh, or MaskX=1Fh, depending on DISPCNT.5).

## OBJ Attribute 0 and 2

OBJモード(`OAM0.10-11`)を3に設定することはGBAでは禁止されていますが、NDSでは3に設定するとビットマップOBJモードが選択されます。このモードでは、色深度ビット(`OAM0.13`)は0である必要があります。また、このモードでは、パレット番号（`OAM2.12-15`）はAlpha-OAM値として使用されます。

## OBJ Vertical Wrap

On the GBA, a large OBJ (with 64pix height, scaled into double-size region of 128pix height) located near the bottom of the screen has been wrapped to the top of the screen (and was NOT displayed at the bottom of the screen).

This problem has been "corrected" in the NDS (except in GBA mode), that is, on the NDS, the OBJ appears BOTH at the top and bottom of the screen. That isn't necessarily better - the advantage is that one can manually enable/disable the OBJ in the desired screen-half on IRQ level; that'd be required only if the wrapped portion is non-transparent.
