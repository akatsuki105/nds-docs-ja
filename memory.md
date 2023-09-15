# メモリマップ

## NDS9メモリマップ

NDS9から見たメモリマップです。

```
  00000000h  ITCM (32KB) (not moveable) (mirror-able to 1000000h)
  0xxxx000h  DTCM (16KB) (moveable)
  02000000h  メインメモリ(NDS9と共有) (4MB)
  03000000h  WRAM (32KB, NDS7と共有, CPUから直接アクセス可能)
  04000000h  ARM9 I/Oポート
  05000000h  Standard Palettes (2KB) (Engine A BG/OBJ, Engine B BG/OBJ)
  06000000h  VRAM - Engine A, BG VRAM  (max 512KB)
  06200000h  VRAM - Engine B, BG VRAM  (max 128KB)
  06400000h  VRAM - Engine A, OBJ VRAM (max 256KB)
  06600000h  VRAM - Engine B, OBJ VRAM (max 128KB)
  06800000h  VRAM - "LCDC"-allocated (max 656KB)
  07000000h  OAM (2KB) (Engine A, Engine B)
  08000000h  GBA Slot ROM (max 32MB)
  0A000000h  GBA Slot RAM (max 64KB)
  FFFF0000h  ARM9 BIOS (32KB, 使用するのは32KBのうち3KBのみ)
```

ARM9のExceptionベクタは`FFFF_0000h`に存在しています。 IRQハンドラは`[DTCM+3FFCh]`にリダイレクトします。 (DTCM=Data TCM)

## NDS7メモリマップ

NDS7から見たメモリマップです。

```
  00000000h  ARM7 BIOS (16KB)
  02000000h  メインメモリ(NDS9と共有) (4MB)
  03000000h  WRAM (32KB, NDS9と共有, CPUから直接アクセス可能)
  03800000h  WRAM (64KB, NDS7専用, CPUから直接アクセス可能)
  04000000h  ARM7 I/Oポート
  04800000h  Wireless Communications Wait State 0 (8KB RAM at 4804000h)
  04808000h  Wireless Communications Wait State 1 (I/O Ports at 4808000h)
  06000000h  VRAM allocated as Work RAM to ARM7 (max 256K)
  08000000h  GBA Slot ROM (max 32MB)
  0A000000h  GBA Slot RAM (max 64KB)
```

ARM7のExceptionベクタは`0000_0000h`に存在しています。IRQハンドラは`[3FFFFFCh aka 380FFFCh]`へとリダイレクトします。

## メインメモリ

メインメモリは、NDS7/9の両方からアクセスできる4MBのPSRAM<sup>[1](#psram)</sup>です。

CPUが直接アクセスするのではなく、メモリインターフェイスを介してアクセスする必要があります。

## 共有WRAM

共有WRAMは、ARM9とARM7の両方からアクセスできる32KBのRAMです。(一度にアクセスできるのは片方のみです)

共有WRAMは`3000000h`から始まりますが、プログラムでは（ARM9、ARM7とも）ミラーされている`37F8000h`を使うのが一般的です。

これにより、ARM7側では、32KBの共有WRAM と 64KBのARM7-WRAM を96KBの連続したRAMブロックとして使用することができます。

## その他のメモリ

ARM9/ARM7のバスにはマッピングされていません。

```
  3D Engine Polygon RAM (52KBx2)
  3D Engine Vertex RAM (72KBx2)
  Firmware (256KB) (built-in serial flash memory)
  GBA-BIOS (16KB) (not used in NDS mode)
  NDS Slot ROM (serial 8bit-bus, max 4GB with default protocol)
  NDS Slot FLASH/EEPROM/FRAM (serial 1bit-bus)
```

## 未定義のI/Oポート

NDSモードでは未定義のI/Oポートは常に0となります。

## 未定義のメモリ領域について

16MB blocks that do not contain any defined memory regions (or that contain only mapped TCM regions) are typically completely undefined.

16MB blocks that do contain valid memory regions are typically containing mirrors of that memory in the unused upper part of the 16MB area (only exceptions are TCM and BIOS which are not mirrored).

## 注釈

<sup id="psram">1: 疑似SRAM（PSRAM）はDRAMの一種で、チップ内部でリフレッシュを行う。そのため、SRAM（DRAMより高速だが高価な代替品）のように動作する。</sup>
