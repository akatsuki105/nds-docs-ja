# カートリッジヘッダ

```
  Address Bytes Expl.
  000h    12    ゲームタイトル  (大文字ASCII, 12文字より少ない時は0でパディング)
  00Ch    4     ゲームコード    (大文字ASCII, NTR-<code>)        (自作ソフトでは0)
  010h    2     メーカーコード   (大文字ASCII, eg. "01"=任天堂) (自作ソフトでは0)
  012h    1     Unitcode    (00h=NDS, 02h=NDS+DSi, 03h=DSi) (bit1=DSi)
  013h    1     Encryption Seed Select (00..07h, usually 00h)
  014h    1     Devicecapacity         (Chipsize = 128KB SHL nn) (eg. 7 = 16MB)
  015h    7     予約領域    (zero filled)
  01Ch    1     予約領域    (zero)                      (except, used on DSi)
  01Dh    1     対象地域  (00h=指定なし, 80h=中国, 40h=韓国) (other on DSi)
  01Eh    1     バージョン (基本的に0)
  01Fh    1     Autostart (Bit2: Skip "Press Button" after Health and Safety. Also skips bootmenu, even in Manual mode & even Start pressed)
  020h    4     ARM9 rom_offset    (4000h and up, align 1000h)
  024h    4     ARM9 entry_address (2000000h..23BFE00h)
  028h    4     ARM9 ram_address   (2000000h..23BFE00h)
  02Ch    4     ARM9 size          (max 3BFE00h) (3839.5KB)
  030h    4     ARM7 rom_offset    (8000h and up)
  034h    4     ARM7 entry_address (2000000h..23BFE00h, or 37F8000h..3807E00h)
  038h    4     ARM7 ram_address   (2000000h..23BFE00h, or 37F8000h..3807E00h)
  03Ch    4     ARM7 size          (max 3BFE00h, or FE00h) (3839.5KB, 63.5KB)
  040h    4     File Name Table (FNT) offset
  044h    4     File Name Table (FNT) size
  048h    4     File Allocation Table (FAT) offset
  04Ch    4     File Allocation Table (FAT) size
  050h    4     File ARM9 overlay_offset
  054h    4     File ARM9 overlay_size
  058h    4     File ARM7 overlay_offset
  05Ch    4     File ARM7 overlay_size
  060h    4     Port 40001A4h setting for normal commands (usually 00586000h)
  064h    4     Port 40001A4h setting for KEY1 commands   (usually 001808F8h)
  068h    4     アイコンとタイトルのオフセット (0=None) (8000h and up)
  06Ch    2     Secure Area Checksum, CRC-16 of [[020h]..00007FFFh]
  06Eh    2     Secure Area Delay (in 131kHz units) (051Eh=10ms or 0D7Eh=26ms)
  070h    4     ARM9 Auto Load List Hook RAM Address (?) ;\endaddr of auto-load
  074h    4     ARM7 Auto Load List Hook RAM Address (?) ;/functions
  078h    8     Secure Area Disable (by encrypted "NmMdOnly") (usually zero)
  080h    4     Total Used ROM size (remaining/unused bytes usually FFh-padded)
  084h    4     ROMヘッダサイズ (4000h で固定？)
  088h    4     Unknown, some rom_offset, or zero? (DSi: slightly different)
  08Ch    36    予約領域 (zero filled; except, [88h..93h] used on DSi)
  0B0h    16    予約領域 (zero filled; or "DoNotZeroFillMem"=unlaunch fastboot)
  0C0h    156   任天堂のロゴ (圧縮されたビットマップデータ、GBAのものと全く同じ)
  15Ch    2     任天堂のロゴのチェックサム, [0C0h-15Bh]のCRC-16で CF56h で固定
  15Eh    2     カートリッジヘッダのチェックサム, 要するに[000h-15Dh]のCRC-16
  160h    4     Debug rom_offset   (0=none) (8000h and up)       ;only if debug
  164h    4     Debug size         (0=none) (max 3BFE00h)        ;version with
  168h    4     Debug ram_address  (0=none) (2400000h..27BFE00h) ;SIO and 8MB
  16Ch    4     予約領域 (zero filled) (transferred, and stored, but not used)
  170h    144   予約領域 (zero filled) (transferred, but not stored in RAM)
```

## DSi専用カートリッジ

DSi専用カートリッジではカートリッジヘッダの内容が拡張されています。

- DSi Cartridge Header Some of that new/changed DSi header entries are important even in NDS mode:
- On DSi, ARM9/ARM7 areas are restricted to 2.75MB (instead 3.8MB on real NDS)
- New NDS titles must have RSA signatures (and old titles must be in whitelist)

## CRC-16

For more info about CRC-16, see description of GetCRC16 BIOS function,

BIOS Misc Functions For the Logo checksum, the BIOS verifies only `[15Ch]=CF56h`, it does NOT verify the actual data at `[0C0h-15Bh]` (nor it’s checksum), however, the data is verified by the firmware.
