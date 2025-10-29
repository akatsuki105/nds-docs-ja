# 制御

## 40001A0h - NDS7/NDS9 - AUXSPICNT - Gamecard ROM and SPI Control (R/W)

[こちら](../backup.md)を参照してください。

## 40001A4h - NDS7/NDS9 - ROMCTRL - Gamecard Bus ROMCTRL (R/W)

```
  0-12  R/W  KEY1 gap1 length  (0-1FFFh) (forced min 08F8h by BIOS) (leading gap)
  13    R/W  KEY2 encrypt data (0=Disable, 1=Enable KEY2 Encryption for Data)
  14    R/W  "SE" Unknown? (usually same as Bit13) (does NOT affect timing?)
  15    W    KEY2 Apply Seed   (0=No change, 1=Apply Encryption Seed) (Write only)
  16-21 R/W  KEY1 gap2 length  (0-3Fh)   (forced min 18h by BIOS) (200h-byte gap)
  22    R/W  KEY2 encrypt cmd  (0=Disable, 1=Enable KEY2 Encryption for Commands)
  23    R    Data-Word Status  (0=Busy, 1=Ready/DRQ) (Read-only)
  24-26 R/W  Data Block size   (0=None, 1..6=100h SHL (1..6) bytes, 7=4 bytes)
  27    R/W  Transfer CLK rate (0=6.7MHz=33.51MHz/5, 1=4.2MHz=33.51MHz/8)
  28    R/W  KEY1 Gap CLKs (0=Hold CLK High during gaps, 1=Output Dummy CLK Pulses)
  29    R/W  RESB Release Reset (0=Reset, 1=Release) (cannot be cleared once set)
  30    R/W  "WR"   Unknown, maybe data-write? (usually 0) (read/write-able)
  31    R/W  Block Start/Status (0=Ready, 1=Start/Busy) (IRQ See 40001A0h/Bit14)
```

The cartridge header is booted at 4.2MHz CLK rate, and following transfers are then using ROMCTRL settings specified in cartridge header entries [060h] and [064h], which are usually using 6.7MHz CLK rate for the main data transfer phase (whereof, older MROM carts can actually transfer 6.7Mbyte/s, but newer 1T-ROM carts default to reading 200h-byte blocks with gap1=657h, thus reaching only 1.6Mbyte/s).

Transfer length of null, four, and 200h..4000h bytes are supported by the console, however, retail cartridges cannot cross 1000h-byte boundaries (and, SanDisk ROMs cannot transfer more than 200h bytes).

## 40001A8h - NDS7/NDS9 - Gamecard bus 8-byte Command Out (R/W)

カートリッジに送信する8バイトのコマンドを書き込むためのレジスタです。

```
  Bit    Expl.
  0-7    1st Command Byte (eg. B7h) (MSB)
  8-15   2nd Command Byte (eg. addr bit 24-31)
  16-23  3rd Command Byte (eg. addr bit 16-23)
  24-31  4th Command Byte (eg. addr bit 8-15) (when aligned=even)
  32-39  5th Command Byte (eg. addr bit 0-7)  (when aligned=00h)
  40-47  6th Command Byte (eg. 00h)
  48-57  7th Command Byte (eg. 00h)
  56-63  8th Command Byte (eg. 00h) (LSB)
```

コマンドレジスタ`0x040001A8`のバイトオーダーが逆転していることに注意してください。

例えば、カートリッジのオフセット`0x12345678`からのデータ読み込みコマンドを送信する場合、コマンドは`0xB712345678000000`なので、そのままリトルエンディアンで書き込みをするならば

```
  0x040001A8 = 0x00;
  0x040001A9 = 0x00;
  0x040001AA = 0x00;
  0x040001AB = 0x78;
  0x040001AC = 0x56;
  0x040001AD = 0x34;
  0x040001AE = 0x12;
  0x040001AF = 0xB7;
```

となりますが、コマンドを解釈する際のバイトオーダーが逆転しているため、実際には

```
  0x040001A8 = 0xB7;
  0x040001A9 = 0x12;
  0x040001AA = 0x34;
  0x040001AB = 0x56;
  0x040001AC = 0x78;
  0x040001AD = 0x00;
  0x040001AE = 0x00;
  0x040001AF = 0x00;
```

と書き込む必要があります。

## 4100010h - NDS7/NDS9 - Gamecard bus 4-byte Data In (R)

コマンド送信後、カートリッジから送られてきたデータは、このI/Oポートから読み出すことができます。

読み出すには
- (DRQビットが立っているときに)手動でこのI/Oポートを読み込む
- `DMASAD=0x04100010, Fixed Source Address, Length=1, Size=32bit, Repeat=On, Mode=DS Gamecard` に設定したDMAで読み込む
の2種類の方法があります。

```
  Bit   Expl.
  0-7   1st received Data Byte (at 4100010h)
  8-15  2nd received Data Byte (at 4100011h)
  16-23 3rd received Data Byte (at 4100012h)
  24-31 4th received Data Byte (at 4100013h)
```


