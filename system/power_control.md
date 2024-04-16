# [電源制御](https://mgba-emu.github.io/gbatek/#dspowercontrol)

DSにはいくつかのパワーマネージメント機能があり、その一部は後述のI/Oポートを介して、また一部はSPIバスを介してアクセスします。詳しくは[電源管理装置](./power_management_device.md)を参照してください。

## 0x0400_0304 - NDS9 - POWCNT1 - Graphics Power Control Register (R/W)

```
  0     Enable Flag for both LCDs (0=Disable) (Prohibited, see notes)
  1     2D Graphics Engine A      (0=Disable) (Ports 008h-05Fh, Pal 5000000h)
  2     3D Rendering Engine       (0=Disable) (Ports 320h-3FFh)
  3     3D Geometry Engine        (0=Disable) (Ports 400h-6FFh)
  4-8   不使用
  9     2D Graphics Engine B      (0=Disable) (Ports 1008h-105Fh, Pal 5000400h)
  10-14 不使用
  15    Display Swap (0=Send Display A to Lower Screen, 1=To Upper Screen)
  16-31 不使用
```

Use SwapBuffers command once after enabling Rendering/Geometry Engine.

Improper use of Bit0 may damage the hardware?

When disabled, corresponding Ports become Read-only, corresponding (palette-) memory becomes read-only-zero-filled.

## 0x0400_0304 - NDS7 - POWCNT2 - Sound/Wifi Power Control Register (R/W)

```
  0     Sound Speakers (0=Disable, 1=Enable) (Initial setting = 1)
  1     Wifi           (0=Disable, 1=Enable) (Initial setting = 0)
  2-31  不使用
```

Note: Bit0 disables the internal Speaker only, headphones are not disabled.

Bit1 disables Port 4000206h, and Ports 4800000h-480FFFFh.

## 0x0400_0206 - NDS7 - WIFIWAITCNT - Wifi Waitstate Control (R)

このレジスタはファームウェアによって起動時に初期化されて以降は固定です。

POWCNT2 の bit1 でWifiが有効になっている場合のみこのレジスタにアクセスできます。

```
  0-2   Wifi WS0 Control (0-7) (Ports 4800000h-4807FFFh)
  3-5   Wifi WS1 Control (0-7) (Ports 4808000h-480FFFFh)
  4-15  不使用 (0固定)
```

## 0x0400_0301 - NDS7 - HALTCNT - Low Power Mode Control (R/W)

Haltモードでは、`(IE AND IF)=0` である限りCPUが一時停止します。

Sleepモードでは、サウンドやビデオを含むほとんどのハードウェアが一時停止します。

```
  0-5   不使用 (0固定)
  6-7   Power Down Mode  (0=通常, 1=Enter GBA Mode, 2=Halt, 3=Sleep)
```

HALTCNT レジスタには直接アクセスすることは推奨されていません。代わりに SWI で BIOS の Halt、Sleep、CustomHalt、IntrWait、または VBlankIntrWait を使ってアクセスすることが推奨されています。

NDS9 はHALTCNTレジスタを持っていませんが、その代わりに、Haltモードに入る際にはコプロセッサ・オペコード`mcr p15,0,r0,c7,c0,4`を使用します。(this opcode locks up if interrupts are disabled via IME=0 (unlike NDS7 HALTCNT method which doesn’t check IME).)

