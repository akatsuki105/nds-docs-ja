# SWRAM (共有WRAM)

> [!NOTE]  
> SWRAM は Shared WRAM の略で、公式資料では WRAM と呼ばれていますが、ここでは便宜上 SWRAM と呼びます。

NDSは、32KBの SWRAM を持っています。 SWRAMは NDS9とNDS7の間で共有されており、以下のレジスタを使って制御されます。

## 4000247h - NDS9 - WRAMCNT - 8bit - WRAM Bank Control (R/W)
## 4000241h - NDS7 - WRAMSTAT - 8bit - WRAM Bank Status (R)

任天堂の公式資料によるとこのレジスタの値は変更しない方がよさそうです。

```
  Bit
  0-1  マッピング
          0b00: ARM9: 32KB,     ARM7: 0KB
          0b01: ARM9: 後半16KB, ARM7: 前半16KB
          0b10: ARM9: 前半16KB, ARM7: 後半16KB
          0b11: ARM9: 0KB,      ARM7: 32KB
  2-7  不使用
```

ARM9は、`0x0300_0000..03FF_FFFF`、 ARM7は、`0x0300_0000..037F_FFFF` に対して、SWRAMがマッピングされています。

割り当てられた16KBまたは32KBは、上記のすべての領域でミラーリングされます。

bit0-1を`0b11`にしてARM9にSWRAMを割り当てなかった場合、SWRAMは未定義状態になります。

bit0-1を`0b00`にしてARM7にSWRAMを割り当てなかった場合、SWRAMのメモリ空間(`0x0300_0000..037F_FFFF`)は、`0x0380_0000..03FF_FFFF`(ARM7専用のWRAM)のミラーになります。

> [!NOTE]  
> SWRAMは`0x0300_0000`から始まりますが、プログラムでは（ARM9、ARM7とも）ミラーされている`37F_8000h`を使うのが一般的です。これにより、ARM7側では、32KBのSWRAM と 64KBのARM7-WRAM を96KBの連続したRAMブロックとして使用することができます。
