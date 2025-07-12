# テクスチャ

## 40004A8h - Cmd 2Ah - TEXIMAGE_PARAM - Set Texture Parameters (W)

```
  Bit     Expl.
  0-15    Texture VRAM Offset div 8 (0..FFFFh -> 512K RAM in Slot 0,1,2,3)
          (VRAM must be allocated as Texture data, see Memory Control chapter)
  16      Repeat in S Direction (0=Clamp Texture, 1=Repeat Texture)
  17      Repeat in T Direction (0=Clamp Texture, 1=Repeat Texture)
  18      Flip in S Direction   (0=No, 1=Flip each 2nd Texture) (requires Repeat)
  19      Flip in T Direction   (0=No, 1=Flip each 2nd Texture) (requires Repeat)
  20-22   Texture S-Size        (for N=0..7: Size=(8 SHL N); ie. 8..1024 texels)
  23-25   Texture T-Size        (for N=0..7: Size=(8 SHL N); ie. 8..1024 texels)
  26-28   テクスチャのフォーマット (0..7)
            0: テクスチャなし
            1: A3I5半透明
            2: 4色パレット(2bpp)
            3: 16色パレット(4bpp)
            4: 256色パレット(8bpp)
            5: 圧縮テクスチャ
            6: A5I3半透明
            7: ダイレクトカラー
  29      パレット使用時(フォーマット2,3,4) の色番号0の扱い (0=表示, 1=透明色として扱う)
  30-31   Texture Coordinates Transformation Mode (0..3, see below)
```

## フォーマット(`TEXIMAGE_PARAM.26-28`)

### Format 1: A3I5半透明テクスチャ (3bit Alpha, 5bit Color Index)

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

```
  Bit  Expl.
  0-4  色番号 (0..31)
  5-7  アルファ値 (0..7; 0=透明, 7=不透明)
```

3bitのアルファ値(0..7)は内部的に5bitのアルファ値(0..31)に拡張されます。 `Alpha=(Alpha*4)+(Alpha/2)`

### Format 2: 4色パレットテクスチャ

各テクセルは2ビットで、1番目のテクセルが1バイト目のLSBに位置します。

In this format, the Palette Base is specified in 8-byte steps; all other formats use 16-byte steps (see PLTT_BASE register).

### Format 3: 16色パレットテクスチャ

各テクセルは4ビットで、1番目のテクセルが1バイト目のLSBに位置します。

### Format 4: 256色パレットテクスチャ

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

### Format 5: 圧縮テクスチャ

圧縮テクスチャについては、[この記事](https://www.webtech.co.jp/blog/optpix_labs/format/4013/)を読んでおくと理解の助けになります。

テクスチャのデータは、4x4を圧縮単位(ブロック)としています。

ブロックのデータは、インデックス情報 と 色情報 の2つでそれぞれVRAM上に分かれて配置されます。インデックス情報は スロット0またはスロット2 に、色情報はスロット1に配置されます。(スロットはVRAMの128KBのスロットのこと)

インデックス情報は、1テクセルあたり2ビット(`0..3`)です。つまり1ブロックあたり4バイトになります。

```
  Bit    Expl.
  0-7    Upper 4-Texel row (LSB=first/left-most Texel, 0b33221100)
  8-15   Next  4-Texel row
  16-23  Next  4-Texel row
  24-31  Lower 4-Texel row
```

例えば、テクスチャサイズが`16x8`の場合、4x4のブロックが`4x2`個、つまり合計8個含まれます。 このとき、

```
  Byte   Expl.
  0-3    1番目の4x4ブロック ; (0, 0)
  4-7    2番目の4x4ブロック ; (4, 0)
  8-11   3番目の4x4ブロック ; (8, 0)
  12-15  4番目の4x4ブロック ; (12, 0)
  16-19  5番目の4x4ブロック ; (0, 4)
  20-23  6番目の4x4ブロック ; (4, 4)
  24-27  7番目の4x4ブロック ; (8, 4)
  28-31  8番目の4x4ブロック ; (12, 4)
```

スロット1には、ブロックの色情報があります。

```
  Bit    Expl.
  0-13   パレットのオフセット(4バイト単位); Addr = (PLTT_BASE*16) + (Offset*4)
  14-15  モード (0..3, 後述)
```

スロット1の色情報のオフセットは、スロット0 or スロット2 のインデックス情報のオフセットから次のように計算されます。

```c
  slot1_addr = slot0_addr / 2;             // スロット1の前半64KBはスロット0に関連付けられる
  slot1_addr = slot2_addr / 2 + (64*1024); // スロット1の後半64KBはスロット2に関連付けられる
```

インデックス値(0..3) と 色情報のモード(bit14-15) の関係は以下の通りです。

```
  Texel  Mode 0       Mode 1             Mode 2         Mode 3
  0      Color 0      Color0             Color 0        Color 0
  1      Color 1      Color1             Color 1        Color 1
  2      Color 2      (Color0+Color1)/2  Color 2        (Color0*5+Color1*3)/8
  3      Transparent  Transparent        Color 3        (Color0*3+Color1*5)/8
```

Mode 1 and 3 are using only 2 Palette Colors (which requires only half as much Palette memory), the 3rd (and 4th) Texel Colors are automatically set to above values (eg. to gray-shades if color 0 and 1 are black and white).

Note: スロット0とスロット2はメモリ領域が連続していないため、テクスチャデータの最大サイズは 1024x512 または 512x1024 です。(この場合は、スロット0または2の128KB全体とスロット1の64KBを占有する) つまり、1024x1024のような大きなサイズは使用できません。

### Format 6: A5I3半透明テクスチャ (5bit Alpha, 3bit Color Index)

各テクセルは1バイトで、1番目のテクセルが1バイト目に位置します。

```
  Bit   Expl.
  0-2   色番号 (0..7)
  3-7   アルファ値 (0..31; 0=透明, 31=不透明)
```

### Format 7: ダイレクトカラーテクスチャ

各テクセルは2バイトで、1番目のテクセルが最初の2バイトに位置します。

```
  ABBBBBGG_GGGRRRRR
    bit0-4:   赤 (0..31)
    bit5-9:   緑 (0..31)
    bit10-14: 青 (0..31)
    bit15:    透明度 (0=透明, 1=不透明)
```

