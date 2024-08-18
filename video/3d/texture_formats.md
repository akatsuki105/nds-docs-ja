# テクスチャの種類

## Format 1: A3I5 Translucent Texture (3bit Alpha, 5bit Color Index)

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

```
  0-4: 色番号 (0..31)
  5-7: アルファ値 (0..7; 0=透明, 7=不透明)
```

3bitのアルファ値(0..7)は内部的に5bitのアルファ値(0..31)に拡張されます。 `Alpha=(Alpha*4)+(Alpha/2)`

## Format 2: 4-Color Palette Texture

各テクセルは2ビットで、1番目のテクセルが1バイト目のLSBに位置します。

In this format, the Palette Base is specified in 8-byte steps; all other formats use 16-byte steps (see PLTT_BASE register).

## Format 3: 16-Color Palette Texture

各テクセルは4ビットで、1番目のテクセルが1バイト目のLSBに位置します。

## Format 4: 256-Color Palette Texture

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

## Format 5: 4x4-Texel Compressed Texture

Consists of 4x4 Texel blocks in Slot 0 or 2, 32bit per block, 2bit per Texel,

```
  bit:   Description
  0-7:   Upper 4-Texel row (LSB=first/left-most Texel)
  8-15:  Next  4-Texel row
  16-23: Next  4-Texel row
  24-31: Lower 4-Texel row
```

Additional Palette Index Data for each 4x4 Texel Block is located in Slot 1,

```
  0-13:  Palette Offset in 4-byte steps; Addr=(PLTT_BASE*10h)+(Offset*4)
  14-15: Transparent/Interpolation Mode (0..3, see below)
```

whereas, the Slot 1 offset is related to above Slot 0 or 2 offset,

```c
  slot1_addr = slot0_addr / 2           // lower 64K of Slot1 assoc to Slot0
  slot1_addr = slot2_addr / 2 + 0x10000 // upper 64K of Slot1 assoc to Slot2
```

The 2bit Texel values (0..3) are intepreted depending on the Mode (0..3),

```
  Texel  Mode 0       Mode 1             Mode 2         Mode 3
  0      Color 0      Color0             Color 0        Color 0
  1      Color 1      Color1             Color 1        Color 1
  2      Color 2      (Color0+Color1)/2  Color 2        (Color0*5+Color1*3)/8
  3      Transparent  Transparent        Color 3        (Color0*3+Color1*5)/8
```

Mode 1 and 3 are using only 2 Palette Colors (which requires only half as much Palette memory), the 3rd (and 4th) Texel Colors are automatically set to above values (eg. to gray-shades if color 0 and 1 are black and white).

Note: The maximum size for 4x4-Texel Compressed Textures is 1024x512 or 512x1024 (which are both occupying the whole 128K in slot 0 or 2, plus 64K in slot1), a larger size of 1024x1024 cannot be used because of the gap between slot 0 and 2.

## Format 6: A5I3 Translucent Texture (5bit Alpha, 3bit Color Index)

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

```
  0-2: 色番号 (0..7)
  3-7: アルファ値 (0..31; 0=透明, 31=不透明)
```

## Format 7: Direct Color Texture

各テクセルは2バイトで、1番目のテクセルが最初の2バイトに位置します。

```
  ABBBBBGG_GGGRRRRR
    bit0-4:   赤 (0..31)
    bit5-9:   緑 (0..31)
    bit10-14: 青 (0..31)
    bit15:    透明度 (0=透明, 1=不透明)
```
