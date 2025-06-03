# DISPCNT

## 0x0400_0000 - NDS9 - DISPCNT (R/W)

```
  Bit    Engine Expl.
  0-2    A+B    BGモード
  3      A      BG0を2D/3Dエンジンのどちらが描画するか (0=2D, 1=3D)
  4      A+B    Tile OBJ Mapping        (0=2D; max 32KB, 1=1D; max 32KB..256KB)
  5      A+B    Bitmap OBJ 2D-Dimension (0=128x512 dots, 1=256x256 dots)
  6      A+B    Bitmap OBJ Mapping      (0=2D; max 128KB, 1=1D; max 128KB..256KB)
  7-15   A+B    GBAのDISPCNTと同じ
  16-17  A+B    描画モード (Engine A: 0..3, Engine B: 0..1, GBA: Green Swap)
  18-19  A      VRAM block (0..3=VRAM A..D) (For Capture & above Display Mode=2)
  20-21  A+B    Tile OBJ 1D-Boundary   (see Bit4)
  22     A      Bitmap OBJ 1D-Boundary (see Bit5-6)
  23     A+B    OBJ Processing during H-Blank (was located in Bit5 on GBA)
  24-26  A      Character Base (in 64K steps) (merged with 16K step in BGxCNT)
  27-29  A      Screen Base (in 64K steps) (merged with 2K step in BGxCNT)
  30     A+B    BG拡張パレット   (0=Disable, 1=Enable)
  31     A+B    OBJ拡張パレット  (0=Disable, 1=Enable)
```

## BGモード

Engine A BG Mode (DISPCNT LSBs) (0-6, 7=Reserved)

Mode | BG0 | BG1 | BG2 | BG3 
-- | -- | -- | -- | -- 
0 | Text/3D | Text | Text     | Text
1 | Text/3D | Text | Text     | Affine
2 | Text/3D | Text | Affine   | Affine
3 | Text/3D | Text | Text     | Extended
4 | Text/3D | Text | Affine   | Extended
5 | Text/3D | Text | Extended | Extended
6 | 3D      | -    | Large    | - 

Of which, the "Extended" modes are sub-selected by BGxCNT bits:

BGxCNT.Bit7 | BGxCNT.Bit2 | Extended Affine Mode Selection
-- | -- | -- 
0  | CharBaseLsb | rot/scal with 16bit bgmap entries (Text+Affine mixup)
1  | 0           | rot/scal 256 color bitmap
1  | 1           | rot/scal direct color bitmap

Engine B: Same as above, except that: Mode 6 is reserved (no Large screen bitmap), and BG0 is always Text (no 3D support).

Affine = formerly Rot/Scal mode (with 8bit BG Map entries)

Large Screen Bitmap = rot/scal 256 color bitmap (using all 512K of 2D VRAM)

## 描画モード (DISPCNT.16-17):

 bit  |  内容
---- | ----
0 | 描画しない (画面は真っ白になる)
1 | 通常モード (normal BG and OBJ layers)
2 | VRAMを使ったbitmap描画 (エンジンAだけこのモードを利用可能、VRAMのどのバンクを使うかは DISPCNT.18-19 で指定)
3 | メインRAMを使ったbitmap描画 (エンジンAだけこのモードを利用可能)

Mode 2-3 display a raw direct color bitmap (15bit RGB values, the upper bit in each halfword is unused), without any further BG,OBJ,3D layers, these modes are completely bypassing the 2D/3D engines as well as any 2D effects, however the Master Brightness effect can be applied to these modes. 

Mode 2 is particulary useful to display captured 2D/3D images (in that case it can indirectly use the 2D/3D engine).

