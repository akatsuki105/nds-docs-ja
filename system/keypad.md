# キー入力

GBAと共通のボタンに関しては、GBA同様、ARM7とARM9にはキーボード入力レジスタがあり、それぞれにキーパッドのIRQ制御レジスタがあります。  
なので、[GBAのキー入力](https://github.com/akatsuki105/gba-docs-ja/blob/main/keypad.md)を見れば、だいたいは理解できます。

GBAとDSの違いとしては、

- XYボタン
- タッチ入力

があります。

タッチ入力については、[こちら](tsc.md)を参照してください。ここではXYボタンについて説明します。

## 4000136h - NDS7 - EXTKEYIN - Key X/Y Input (R)

```
  0      Button X     (0=Pressed, 1=Released)
  1      Button Y     (0=Pressed, 1=Released)
  3      DEBUG button (0=Pressed, 1=Released/None such)
  6      Pen down     (0=Pressed, 1=Released/Disabled) (always 0 in DSi mode)
  7      Hinge/folded (0=Open, 1=Closed)
  2,4,5  Unknown / set
  8..15  Unknown / zero
```

The Hinge stuff is a magnetic sensor somewhere underneath of the Start/Select buttons (NDS) or between A/B/X/Y buttons (DSi), it will be triggered by the magnet field from the right speaker when the console is closed. 

The hinge generates an interrupt request (there seems to be no way to disable this, unlike as for all other IRQ sources), however, the interrupt execution can be disabled in IE register (as for other IRQ sources).

The Pen Down is the /PENIRQ signal from the Touch Screen Controller (TSC), if it is enabled in the TSC control register, then it will notify the program when the screen pressed, the program should then read data from the TSC (if there's no /PENIRQ then doing unneccassary TSC reads would just waste CPU power). 

However, the user may release the screen before the program performs the TSC read, so treat the screen as not pressed if you get invalid TSC values (even if /PENIRQ was LOW).

Not sure if the TSC /PENIRQ is actually triggering an IRQ in the NDS?

The Debug Button should be connected to R03 and GND (on original NDS, R03 is the large soldering point between the SL1 jumper and the VR1 potentiometer) (there is no R03 signal visible on the NDS-Lite board).

Interrupts are reportedly not supported for X,Y buttons.
