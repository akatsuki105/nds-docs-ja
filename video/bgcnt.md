# BGnCNT

タイルデータのベースアドレス指定は、GBAのときの Bit2-3 から Bit2-5 に拡張されました。(Bit4-5 はGBAでは使っていなかった)

```
  engine A screen base: BGxCNT.bits*2K + DISPCNT.bits*64K
  engine B screen base: BGxCNT.bits*2K + 0
  engine A char base: BGxCNT.bits*16K + DISPCNT.bits*64K
  engine B char base: BGxCNT.bits*16K + 0
```

- タイルベースはタイルモードでのみ使用します(ビットマップモードでは使用しない)
- マップベースも同様にタイルモードでのみ使用します
- screen base used in bitmap modes as BGxCNT.bits\*16K, without DISPCNT.bits\*64K
- screen base however NOT used at all for Large screen bitmap mode

bgcnt size | text | rotscal | bitmap | large bmp
-- | -- | -- | -- | -- 
0 | 256x256 | 128x128   | 128x128 | 512x1024
1 | 512x256 | 256x256   | 256x256 | 1024x512
2 | 256x512 | 512x512   | 512x256 | -
3 | 512x512 | 1024x1024 | 512x512 | -

128KBより大きなVRAMを必要とするビットマップはエンジンAのみでサポートされます。

For BGxCNT.Bit7 and BGxCNT.Bit2 in Extended Affine modes, see above BG Mode description (extended affine doesn't include 16-color modes, so color depth bit can be used for mode selection. Also, bitmap modes do not use charbase, so charbase.0 can be used for mode selection as well).

```
  for BG0CNT, BG1CNT only: bit13 selects extended palette slot
                           (BG0: 0=Slot0, 1=Slot2, BG1: 0=Slot1, 1=Slot3)
```

**Direct Color Bitmap BG, and Direct Color Bitmap OBJ**

BG/OBJ Supports 32K colors (15bit RGB value) - so far same as GBAs BG.

However, the upper bit (Bit15) is used as Alpha flag. That is, Alpha=0=Transparent, Alpha=1=Normal (ie. on the NDS, Direct Color values 0..7FFFh are NOT displayed).

Unlike GBA bitmap modes, NDS bitmap modes are supporting the Area Overflow bit (BG2CNT and BG3CNT, Bit 13).