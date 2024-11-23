# NDSスロット

NDSスロットは

- ROMとの8bitバス(Gamecardバス)
- バックアップチップ(セーブデータ)とのSPIバス

の2つのバスを持っています。

ROMとのやりとりは、[こちら](./rom/)を、バックアップチップとのやりとりは[こちら](./backup.md)を参照してください。

## 40001A0h - NDS7/NDS9 - AUXSPICNT - NDSスロット制御レジスタ (R/W)

このIOポートではセーブデータとのSPIバスだけでなく、Gamecardバスの制御も行っています。

```
  Bit   Dir  Expl.
  0-1   R/W  ボーレート (0=4MHz/Default, 1=2MHz, 2=1MHz, 3=512KHz)
  2-5   -    不使用(常に0)
  6     R/W  SPI Hold Chipselect (0=Deselect after transfer, 1=Keep selected)
  7     R    SPIビジーフラグ (0=Ready, 1=Busy)
  8-12  -    不使用(常に0)
  13    R/W  NDSスロットのモード       (0=Parallel/ROM, 1=Serial/SPI-Backup)
  14    R/W  Transfer Ready IRQ  (0=Disable, 1=Enable) (for ROM, not for AUXSPI)
  15    R/W  NDS Slot Enable     (0=Disable, 1=Enable) (for both ROM and AUXSPI)
```

The “Hold” flag should be cleared BEFORE transferring the LAST data unit, the chipselect will be then automatically cleared after the transfer, the program should issue a WaitByLoop(12) on NDS7 (or longer on NDS9) manually AFTER the LAST transfer.
