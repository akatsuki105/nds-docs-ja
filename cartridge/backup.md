# バックアップ(セーブデータ)

> [!NOTE]
> NDSは、ファームウェアやタッチ画面などと通信を行うためのSPIバスを持っていますが、バックアップチップとの通信には別のSPIバス(とIOポート)を使用します。

> [!TIP]
> AUXSPI は Auxiliary SPI の略です。

NDSはSPI通信を用いて、バックアップチップと通信します。

## 40001A0h - NDS7/NDS9 - AUXSPICNT - Gamecard ROM and SPI Control (R/W)

```
  Bit   Dir  Expl.
  0-1   R/W  ボーレート (0=4MHz/Default, 1=2MHz, 2=1MHz, 3=512KHz)
  2-5   -    不使用(常に0)
  6     R/W  SPI Hold Chipselect (0=Deselect after transfer, 1=Keep selected)
  7     R    ビジーフラグ (0=Ready, 1=Busy)
  8-12  -    不使用(常に0)
  13    R/W  NDS Slot Mode       (0=Parallel/ROM, 1=Serial/SPI-Backup)
  14    R/W  Transfer Ready IRQ  (0=Disable, 1=Enable) (for ROM, not for AUXSPI)
  15    R/W  NDS Slot Enable     (0=Disable, 1=Enable) (for both ROM and AUXSPI)
```

The “Hold” flag should be cleared BEFORE transferring the LAST data unit, the chipselect will be then automatically cleared after the transfer, the program should issue a WaitByLoop(12) on NDS7 (or longer on NDS9) manually AFTER the LAST transfer.

## 40001A2h - NDS7/NDS9 - AUXSPIDATA - Gamecard SPI Bus Data/Strobe (R/W)

SPI転送は全二重通信なので、SPIバスから読み取るつもりであっても、ダミー値（ゼロであるべき）を書き込む必要があります。

```
  0-7  Data
  8-15 不使用(常に0)
```

転送中は、ビジーフラグ`AUXSPICNT.7`がセットされ、`AUXSPIDATA`に書きこまれたデータ値がデバイスに転送（出力ライン経由）され、同時にデータが受信（入力ライン経由）されます。転送が完了すると、ビジーフラグがオフになり、必要に応じて、受信した値を`AUXSPIDATA`から読み取ることができます。

