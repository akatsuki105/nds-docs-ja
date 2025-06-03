# NitroFS

NitroFS は、（少なくとも任天堂のツールで開発された）ニンテンドーDSのゲーム(NitroROM)で使用されるファイルシステムです。

DS では、カートリッジヘッダの `0x20..2F` (ARM9の場合。ARM7 は `0x30..3F`) に記述されるブートコードは、自動的にPSRAMにロードされますが、ROMの他の部分は、プログラム側でロードする必要があります。

```cpp
  FNT = cart_hdr[0x040];  // origin as defined in ROM cartridge header
  FAT = cart_hdr[0x048];
  IMG = 0x00000000;       // ROMファイルの先頭
```

## NitroARC (Nitro Archive, NARC)

NARCファイルは、通常、NitroFS 内に存在します。したがって、NARC は実際のファイルシステム内にネストされた第2の仮想ファイルシステムです。

NARC フォーマットは NitroFS と非常に類似していますが、カートリッジヘッダ の代わりに追加のチャンクヘッダが含まれています。

```
  ...  ...  Optional Header (eg. compression header, or RSA signature)
  000h 4    Chunk Name "NARC" (Nitro Archive)                   ;\
  004h 2    Byte Order (FFFEh) (unlike usually, not FEFFh)      ;
  006h 2    Version (0100h)                                     ; NARC
  008h 4    File Size (from "NARC" ID to end of file)           ; Header
  00Ch 2    Chunk Size (0010h)                                  ;
  00Eh 2    Number of following chunks (0003h)                  ;/
  010h 4    Chunk Name "BTAF" (File Allocation Table Block)     ;\
  014h 4    Chunk Size (including above chunk name)             ; File
  018h 2    Number of Files                                     ; Allocation
  01Ah 2    Reserved (0000h)                                    ; Table
  01Ch ...  FAT (see below)                                     ;/
  ...  4    Chunk Name "BTNF" (File Name Table Block)           ;\
  ...  4    Chunk Size (including above chunk name)             ; File Name
  ...  ...  FNT (see below)                                     ; Table
  ...  ..   Padding for 4-byte alignment (FFh-filled, if any)   ;/
  ...  4    Chunk Name "GMIF" (File Image Block)                ;\
  ...  4    Chunk Size (including above chunk name)             ; File Data
  ...  ...  IMG (File Data)                                     ;/
```

### カスタムNARC

いくつかのゲームでは、NARCフォーマットをカスタマイズしたものを使用しています。("Over the Hedge", "Tony Hawk's Downhill Jam" の `rom:\dwc\utility.bin`)

```
  000h 4   FNT Filename Table Offset (always at 10h)
  004h 4   FNT Filename Table Size
  008h 4   FAT Allocaton Table Offset (at above Offset+Size+Padding)
  00Ch 4   FAT Allocaton Table Size
  010h ..  FNT Filename Table Data
  ...  ..  FAT Allocaton Table Data
  ...  ..  IMG File Data
```

The offsets in FAT are relative to `IMG=0` (as if IMG would start at begin of file).

## FAT (File Allocation Table)

> [!TIP]  
> Windows で使われる FATファイルシステム とは無関係です。

```cpp
  FileCount = cart_hdr[0x04C] / 8; // 8 bytes per entry (sizeof(FATEntry))
```

DSのゲームは、最大で61440個のファイルを持つことができます。ファイルIDは、0スタートです。

```cpp
struct FATEntry {
  u32 startAddr; // Start address (originated at IMG base) (0=Unused Entry)
  u32 endAddr;   // End address   (Start+Len)              (0=Unused Entry)
}; // 8バイト
```


For NitroROM, addresses must be after Secure Area (at 8000h and up).

For NitroARC, addresses can be anywhere in the IMG area (at 0 and up).

FAT は、NitroFS の中の全てのファイルの位置データ(とサイズ)を持っているだけで、ディレクトリツリーやファイル名の保持は FNT が行います。

## FNT (File Name Table)

FNT は ディレクトリテーブル と その後に続く 1つ以上のFNTサブテーブル から構成されます。

To interprete the directory tree: Start at the 1st Main-Table entry, which is referencing to a Sub-Table, any directories in the Sub-Table are referencing to Main-Table entries, which are referencing to further Sub-Tables, and so on.

### FNTディレクトリテーブル (base=FNT+0, size=[FNT+06h]\*8)

最大4096個のディレクトリのリストです。ディレクトリIDは、`0xF000`からスタートします。

```
  Addr      Size    Expl.
  00h       8       1st Entry (ID F000h, ルートディレクトリ)
      00h     4       サブテーブルのオフセット (originated at FNT base)
      04h     2       ID of first file in Sub-table   (0000h..EFFFh)
      06h     2       ディレクトリの総数; ルートディレクトリも含めて 1..4096 個
  08h       8       2nd Entry (ID F001h, サブディレクトリ)
      00h     4       サブテーブルのオフセット (originated at FNT base)
      04h     2       ID of first file in Sub-table   (0000h..EFFFh)
      06h     2       親ディレクトリのID (F000h..FFFEh)
  10h       8       3rd Entry (ID F002h, サブディレクトリ)
  ...
```

### FNTサブテーブル (base=FNT+offset, ends at Type/Length=00h)

特定のディレクトリ の内容を表すテーブルです。(どのディレクトリに対応しているかは、FNTディレクトリテーブルのエントリから分かります。)

ディレクトリ内のすべてのファイルとサブディレクトリのファイル名(ASCII形式)を持っています。

```
  Addr   Size  Expl.
  00h    1     Type/Length
               01h..7Fh File Entry          (Length=1..127, without ID field)
               81h..FFh Sub-Directory Entry (Length=1..127, plus ID field)
               00h      End of Sub-Table
               80h      Reserved
  01h    LEN   File or Sub-Directory Name
               Case-sensitive, without any ending zero, ASCII 20h..7Eh, except for characters \/?"<>*:;|
```

さらにサブディレクトリのエントリには追加で以下のフィールドがあります。

```
  Addr   Size  Expl.
  LEN+1  2     サブディレクトリID (F001h..FFFFh) ;see FNT+(ID AND FFFh)*8
```

ファイルエントリには上記のIDフィールドはありません。代わりに、ファイルIDはディレクトリテーブルで指定された「最初のID」値から開始して、連番で割り当てられます。

