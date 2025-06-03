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
  13    R/W  NDSスロットのモード (0=Gamecardバス, 1=SPIバス)
  14    R/W  Transfer Ready IRQ  (0=Disable, 1=Enable) (for ROM, not for AUXSPI)
  15    R/W  NDSスロット有効化フラグ  (0=Disable, 1=Enable) (for both ROM and AUXSPI)
```

The “Hold” flag should be cleared BEFORE transferring the LAST data unit, the chipselect will be then automatically cleared after the transfer, the program should issue a WaitByLoop(12) on NDS7 (or longer on NDS9) manually AFTER the LAST transfer.

## GBAスロット

GBAスロット(`0x8000000..AFFFFFF`)は、`EXMEMCNT.7`によって ARM9 または ARM7 にマッピングできます。

マッピングされたCPUのアドレス空間では、`0x8000000..9FFFFFF` でGBAのROM、`0xA000000..AFFFFFF` でGBAのSRAM（64KBごとにミラー）にアクセスできます。

GBA スロットにカートリッジがない場合、ROM,SRAM 領域にはオープンバス値が格納されます。

- SRAM: 0xFF で埋められています (Hi-Z)
- ROM:  ROMへのアクセス時間 と アドレスによって以下の値を返します。

```
  6 clks   --> returns "Addr/2"
  8 clks   --> returns "Addr/2"
  10 clks  --> returns "(Addr/2) | 0xFE08" (or similar garbage)
  18 clks  --> returns "FFFFh" (Hi-Z)
```

マッピングされていないCPUでは、`0x8000000..AFFFFFF` は `0x00` で埋められています。

This is required for bugged games like Digimon Story: Super Xros Wars (which is accidently reading deselected GBA SRAM at [main\_ram\_base+main\_ram\_addr\*4], whereas it does presumably want to read Main RAM at [main\_ram\_base+index\*4]).

