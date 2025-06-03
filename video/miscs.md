# 概要

## 0x0400_006C - NDS9 - MASTER_BRIGHT - 16bit - 明るさ制御レジスタ (R/W)

画面の明るさを決めるレジスタ

```
  Bit
  0-4    RGB666 の Factor (0-16で16より大きい場合は16として扱う)
           明るさは次のように決定します。
             bit14-15 が 1 (Up):   New = Old + (63-Old) * Factor/16
             bit14-15 が 2 (Down): New = Old - Old      * Factor/16
  5-13   不使用
  14-15  モード(0=無効, 1=Up, 2=Down, 3=Reserved)
```

## DISPSTAT/VCOUNT

The LY and LYC values are in range 0..262, so LY/LYC values have been expanded to 9bit values: LY = VCOUNT Bit 0..8, and LYC=DISPSTAT Bit8..15,7.

VCOUNT register is write-able, allowing to synchronize linked DS consoles.

For proper synchronization:

- write new LY values only in range of 202..212
- write only while old LY values are in range of 202..212

DISPSTAT/VCOUNT supported by NDS9 (Engine A Ports, without separate Engine B Ports), and by NDS7 (allowing to synchronize NDS7 with display timings).

Similar as on GBA, the VBlank flag isn't set in the last line (ie. only in lines 192..261, but not in line 262).

Although the drawing time is only 1536 cycles (256*6), the NDS9 H-Blank flag is "0" for a total of 1606 cycles (and, for whatever reason, a bit longer, 1613 cycles in total, on NDS7).

## VRAM Waitstates

The display controller performs VRAM-reads once every 6 clock cycles, a 1 cycle waitstate is generated if the CPU simultaneously accesses VRAM. 

With capture enabled, additionally VRAM-writes take place once every 6 cycles, so the total VRAM-read/write access rate is then once every 3 cycles.

## DS Window Glitches

The DS counts scanlines in range 0..262 (0..106h), of which only the lower 8bit are compared with the WIN0V/WIN1V register settings. 

Respectively, Y1 coordinates 00h..06h will be triggered in scanlines 100h-106h by mistake. That means, the window gets activated within VBlank period, and will be active in scanline 0 and up (that is no problem with Y1=0, but Y1=1..6 will appear as if if Y1 would be 0). 

Workaround would be to disable the Window during VBlank, or to change Y1 during VBlank (to a value that does not occur during VBlank period, ie. 7..191).

Also, there's a problem to fit the 256 pixel horizontal screen resolution into 8bit values: X1=00h is treated as 0 (left-most), X2=00h is treated as 100h (right-most). 

However, the window is not displayed if X1=X2=00h; the window width can be max 255 pixels.

## 2Dエンジン

NDSは2つの2Dエンジン(エンジンA,B)を持っています。両エンジンはARM9プロセッサによってアクセスでき、それぞれ異なるメモリアドレスとレジスタアドレスを使用します。

 領域  | エンジンA   | エンジンB
------|--------|--------------
I/Oポート   | 4000000h    | 4001000h
パレット   | 5000000h (1KB)   | 5000400h (1KB)
BG VRAM   | 6000000h (最大512KB) | 6200000h (最大128KB)
OBJ VRAM  | 6400000h (最大256KB) | 6600000h (最大128KB)
OAM       | 7000000h (1KB)       | 7000400h (1KB)

Engine A additionally supports 3D and large-screen 256-color Bitmaps, plus main-memory-display and vram-display modes, plus capture unit.

## 4000070h - NDS9 - TVOUTCNT - Unknown (W)

```
  Bit   Expl.
  0-3   "COMMAND"  (?)
  4-7   "COMMAND2" (?)
  8-11  "COMMAND3" (?)
```

This register has been mentioned in an early I/O map from Nintendo, as far as I know, the register isn't used by any games/firmware/bios, not sure if it does really exist on release-version, or if it's been prototype stuff...?

## DS-Lite Screens

The screens in the DS-Lite seem to allow a wider range of vertical angles.

The bad news is that the colors of the DS-Lite are (no surprise) not backwards compatible with older NDS and GBA displays. The good news is that Nintendo has finally reached near-CRT-quality (without blurred colors), so one could hope that they won't show up with more displays with other colors in future.

Don't know if there's an official/recommended way to detect DS-Lite displays (?) possible methods would be whatever values in Firmware header, or by functionality of Power Managment device, or (not too LCD-related) by Wifi Chip ID.


