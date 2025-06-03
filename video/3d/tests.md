# テスト機能

## 40005C0h - Cmd 70h - BOX_TEST - Test if Cuboid Sits inside View Volume (W)

The BoxTest result indicates if one or more of the 6 faces of the box are fully or parts of inside of the view volume. Can be used to reduce unnecessary overload, ie. if the result is false, then the program can skip drawing of objects which are inside of the box.

BoxTest verifies only if the faces of the box are inside view volume, and so, it will return false if the whole view volume is located inside of the box (still objects inside of the box may be inside of view).

```
  Parameter 1, Bit 0-15   X-Coordinate
  Parameter 1, Bit 16-31  Y-Coordinate
  Parameter 2, Bit 0-15   Z-Coordinate
  Parameter 2, Bit 16-31  Width  (presumably: X-Offset?)
  Parameter 3, Bit 0-15   Height (presumably: Y-Offset?)
  Parameter 3, Bit 16-31  Depth  (presumably: Z-Offset?)
  All values are 1bit sign, 3bit integer, 12bit fractional part
```

The result of the “coordinate+offset” additions should not overflow 16bit vertex coordinate range (1bit sign, 3bit integer, 12bit fraction).

Before using BoxTest, be sure that far-plane-intersecting & 1-dot polygons are enabled, if they aren’t: Send the PolygonAttr command (with bit12,13 set to enable them), followed by dummy Begin and End commands (required to apply the new PolygonAttr settings). BoxTest should not be issued within Begin/End.

After sending the BoxTest command, wait until GXSTAT.Bit0 indicates Ready, then read the result from GXSTAT.Bit1.

## 40005C4h - Cmd 71h - POS_TEST - Set Position Coordinates for Test (W)

```
  座標は全て、小数部が12bitの固定小数点数 (1bit sign, 3bit integer, 12bit fractional part)
  Parameter 1
    Bit
    0-15   X座標
    16-31  Y座標
  Parameter 2
    Bit
    0-15   Z座標
    16-31  不使用
```

Multiplies the specified line-vector (x,y,z,1) by the clip coordinate matrix.

After sending the command, wait until GXSTAT.Bit0 indicates Ready, then read the result from POS_RESULT registers. POS_TEST can be issued anywhere (except within polygon strips, huh?).

Caution: POS_TEST overwrites the internal VTX registers, so the next vertex should be <fully> defined by VTX_10 or VTX_16, otherwise, when using VTX_XY, VTX_XZ, VTX_YZ, or VTX_DIFF, then the new vertex will be relative to the POS_TEST coordinates (rather than to the previous vertex).

## 4000620h..62Fh - POS_RESULT - Position Test Results (R)

This 16-byte region (4 words) contains the resulting clip coordinates (x,y,z,w) from the POS_TEST command. 

各値は1ビットの符号、19ビットの整数、12ビットの小数部を持った固定小数点数です。

## 40005C8h - Cmd 72h - VEC_TEST - Set Directional Vector for Test (W)

```
  Bit
  0-9    X座標 (1bit sign + 9bit fractional part, YZも同様)
  10-19  Y座標
  20-29  Z座標
  30-31  不使用
```

Multiplies the specified line-vector (x,y,z,0) by the directional vector matrix. Similar as for the NORMAL command, it does require Matrix Mode 2 (ie. Position & Vector Simultaneous Set mode).

After sending the command, wait until GXSTAT.Bit0 indicates Ready, then read the result (“the directional vector in the View coordinate space”) from VEC_RESULT registers.

## 4000630h..635h - VEC_RESULT - Vector Test Results (R)

This 6-byte region (3 halfwords) contains the resulting vector (x,y,z) from the VEC_TEST command. Each value is 4bit sign, 0bit integer, 12bit fractional part. The 4bit sign is either 0000b (positive) or 1111b (negative).

There is no integer part, so values >=1.0 or <-1.0 will cause overflows.

(Eg. +1.0 aka 1000h will be returned as -1.0 aka F000h due to overflow and sign-expansion).

