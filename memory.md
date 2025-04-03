# メモリマップ

## NDS9メモリマップ

```
  0000_0000h  ITCM (32KB) (100_0000hごとにミラー)
  0xxx_x000h  DTCM (16KB) (CP15によって移動可能, デフォルトでは 27C0000h)
  0200_0000h  メインメモリ(NDS9と共有) (4MB)
  0300_0000h  WRAM (32KB, NDS7と共有, CPUから直接アクセス可能)
  0400_0000h  ARM9 I/Oポート
  0500_0000h  パレット (2KB, GBAと同じ1KBのパレットが2つ(エンジンA/B))
  0600_0000h  VRAM - エンジンA, BG  (最大 512KB)
  0620_0000h  VRAM - エンジンB, BG  (最大 128KB)
  0640_0000h  VRAM - エンジンA, OBJ (最大 256KB)
  0660_0000h  VRAM - エンジンB, OBJ (最大 128KB)
  0680_0000h  VRAM - "LCDC"-allocated (最大 656KB)
  0700_0000h  OAM (2KB, GBAと同じ1KBのOAMが2つ(エンジンA/B))
  0800_0000h  GBA Slot ROM (最大 32MB)
  0A00_0000h  GBA Slot RAM (最大 64KB)
  FFFF_0000h  ARM9 BIOS (32KB, 使用するのは32KBのうち3KBのみ)
```

ARM9のExceptionベクタは`FFFF_0000h`に存在しています。 IRQハンドラは`[DTCM+3FFCh]`にリダイレクトします。 

## NDS7メモリマップ

```
  0000_0000h  ARM7 BIOS (16KB)
  0200_0000h  メインメモリ(NDS9と共有) (4MB)
  0300_0000h  WRAM (32KB, NDS9と共有, CPUから直接アクセス可能)
  0380_0000h  WRAM (64KB, NDS7専用, CPUから直接アクセス可能)
  0400_0000h  ARM7 I/Oポート
  0480_0000h  Wireless Communications Wait State 0 (8KB RAM at 4804000h)
  0480_8000h  Wireless Communications Wait State 1 (I/O Ports at 4808000h)
  0600_0000h  VRAM allocated as Work RAM to ARM7 (最大 256KB)
  0800_0000h  GBA Slot ROM (最大 32MB)
  0A00_0000h  GBA Slot RAM (最大 64KB)
```

ARM7のExceptionベクタは`0000_0000h`に存在しています。IRQハンドラは`[3FFFFFCh](=[380FFFCh])`へとリダイレクトします。

## メインメモリ

メインメモリは、NDS7/9の両方からアクセスできる4MBのPSRAM<sup>[1](#psram)</sup>です。

CPUが直接アクセスするのではなく、メモリインターフェイスを介してアクセスする必要があります。

## その他のメモリ

ARM9/ARM7のバスにはマッピングされていません。

```
  3D Engine Polygon RAM (52KBx2)
  3D Engine Vertex RAM (72KBx2)
  Firmware (256KB) (built-in serial flash memory)
  GBA-BIOS (16KB) (NDSモードでは不使用)
  NDS Slot ROM (serial 8bit-bus, max 4GB with default protocol)
  NDS Slot FLASH/EEPROM/FRAM (serial 1bit-bus)
```

## 未定義のI/Oポート

NDSモードでは未定義のI/Oポートは常に0となります。

## 未定義のメモリ領域について

16MB blocks that do not contain any defined memory regions (or that contain only mapped TCM regions) are typically completely undefined.

16MB blocks that do contain valid memory regions are typically containing mirrors of that memory in the unused upper part of the 16MB area (only exceptions are TCM and BIOS which are not mirrored).

## 注釈

<sup id="psram">1: 疑似SRAM（PSRAM）はDRAMの一種で、チップ内部でリフレッシュを行います。そのため、SRAM（DRAMより高速だが高価な代替品）のように動作します。</sup>
