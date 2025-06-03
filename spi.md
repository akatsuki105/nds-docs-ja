# SPI

> [!TIP]
> SPI は Serial Peripheral Interface の略です。 

## 40001C0h - NDS7 - SPICNT - SPIバス制御 (R/W)

```
  Bit   Dir  Expl.
  0-1   R/W  ボーレート
               0: 4MHz (Firmware)
               1: 2MHz (Touchscreen)
               2: 1MHz (Powerman)
               3: 512KHz
  2     R/W  ボーレートの拡張bit (DSiのみ、SCFG_EXT7.9 が1の場合, NDSでは不使用(0))
               4:   8MHz
               5-7: None (0Hz)
               NDSでは不使用(0)
  3-6   -    不使用 (0)
  7     R    ビジーフラグ (0=Ready, 1=Busy)
  8-9   R/W  スレーブセレクト (0=Powerman, 1=Firmware, 2=Touchscr, 3=Reserved)
  10    R/W  Transfer Size       (0=8bit/Normal, 1=16bit/Bugged)
  11    R/W  Chipselect Hold     (0=Deselect after transfer, 1=Keep selected)
  12-13 -    不使用 (0)
  14    R/W  Interrupt Request   (0=Disable, 1=Enable)
  15    R/W  SPI Bus Enable      (0=Disable, 1=Enable)
```

The “Hold” flag should be cleared BEFORE transferring the LAST data unit, the chipselect will be then automatically cleared after the transfer, the program should issue a WaitByLoop(3) manually AFTER the LAST transfer.

## 40001C2h - NDS7 - SPIDATA - SPI Bus Data/Strobe Register (R/W)

SPI転送は全二重通信なので、SPIバスから読み取るつもりであっても、ダミー値（ゼロであるべき）を書き込む必要があります。

```
  0-7   Data
  8-15  不使用 (常に0, even in bugged-16bit mode)
```

転送中は、ビジーフラグ`SPICNT.7`がセットされ、`SPIDATA`に書きこまれたデータ値がデバイスに転送（出力ライン経由）され、同時にデータが受信（入力ライン経由）されます。転送が完了すると、ビジーフラグがオフになり(さらに`SPICNT.14`が1の場合はIRQ発生)、必要に応じて、受信した値を`SPIDATA`から読み取ることができます。


