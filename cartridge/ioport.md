# IOポート

> [!TIP]
> AUXSPI は Auxiliary SPI の略です。

## 40001A0h - NDS7/NDS9 - AUXSPICNT - Gamecard ROM and SPI Control (R/W)

```
  Bit   Expl.
  0-1   SPIボーレート        (0=4MHz/Default, 1=2MHz, 2=1MHz, 3=512KHz)
  2-5   不使用(常に0)
  6     SPI Hold Chipselect (0=Deselect after transfer, 1=Keep selected)
  7     SPI Busy            (0=Ready, 1=Busy) (presumably Read-only)
  8-12  不使用(常に0)
  13    NDS Slot Mode       (0=Parallel/ROM, 1=Serial/SPI-Backup)
  14    Transfer Ready IRQ  (0=Disable, 1=Enable) (for ROM, not for AUXSPI)
  15    NDS Slot Enable     (0=Disable, 1=Enable) (for both ROM and AUXSPI)
```

The “Hold” flag should be cleared BEFORE transferring the LAST data unit, the chipselect will be then automatically cleared after the transfer, the program should issue a WaitByLoop(12) on NDS7 (or longer on NDS9) manually AFTER the LAST transfer.

## 40001A2h - NDS7/NDS9 - AUXSPIDATA - Gamecard SPI Bus Data/Strobe (R/W)

SPI転送は、このレジスタへの書き込みにより開始されるため、SPIバスから読み取るつもりであってもダミー値（0）を書き込む必要があります。

```
  0-7  Data
  8-15 不使用(常に0)
```

転送中は、ビジーフラグ`AUXSPICNT.7`がセットされ、書かれたデータ値がデバイスに転送（出力ライン経由）され、同時にデータが受信（入力ライン経由）されます。転送が完了すると、ビジーフラグがオフになり、必要に応じて、受信した値をAUXSPIDATAから読み取ることができます。

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

例えば、カートリッジのオフセット`0x12345678`からのデータ読み込みコマンドを送信する場合、コマンドは`0xB7aaaaaaaa000000`なので

```
  0x0400_01A8 = 0xB7
  0x0400_01A9 = 0x12
  0x0400_01AA = 0x34
  0x0400_01AB = 0x56
  0x0400_01AC = 0x78
  0x0400_01AD = 0x00
  0x0400_01AE = 0x00
  0x0400_01AF = 0x00
```

となります。

## 4100010h - NDS7/NDS9 - Gamecard bus 4-byte Data In (R)

```
  Bit   Expl.
  0-7   1st received Data Byte (at 4100010h)
  8-15  2nd received Data Byte (at 4100011h)
  16-23 3rd received Data Byte (at 4100012h)
  24-31 4th received Data Byte (at 4100013h)
```

After sending a command, data can be read from this register manually (when the DRQ bit is set), or by DMA (with DMASAD=4100010h, Fixed Source Address, Length=1, Size=32bit, Repeat=On, Mode=DS Gamecard).
