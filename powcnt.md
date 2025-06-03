# [電源制御](https://mgba-emu.github.io/gbatek/#dspowercontrol)

DSの各ユニットには、電源オンオフを行えるものがあり、それらはI/Oポートを介してアクセスできます。

ユニットの電源がオフの場合、そのユニットにマッピングされたI/Oポートは読み取り専用になり、そのユニットのメモリ（パレットメモリなど）は読み取り専用でゼロで埋められます。

## 0x0400_0304 - NDS9 - POWCNT1 - Graphics Power Control Register (R/W)

```
  0     液晶画面(上下両方, 勝手な変更は禁止？)
  1     2DエンジンA  (IO: 0x0400_0008..0400_005F, 0x0500_0000..0500_03FF)
  2     3Dレンダリングエンジン (IO: 0x0400_0320..0400_03FF)
  3     3Dジオメトリエンジン (IO: 0x0400_0400..0400_06FF)
  4-8   不使用
  9     2DエンジンB  (IO: 0x0400_1008..0400_105F, 0x0500_0400..0500_07FF)
  10-14 不使用
  15    Display Swap (0=Send Display A to Lower Screen, 1=To Upper Screen)
  16-31 不使用
```

Use SwapBuffers command once after enabling Rendering/Geometry Engine.

Bit0を不用意に変更するとハードウェアに損傷を与える可能性があります。

## 0x0400_0304 - NDS7 - POWCNT2 - Sound/Wifi Power Control Register (R/W)

```
  0     内蔵スピーカー (初期値は1, ヘッドフォンからの音は無効化されない)
  1     WiFi  (初期値は0, IO: 0x0400_0206 と 0x0480_0000..0480_FFFF)
  2-31  不使用
```

## 0x0400_0206 - NDS7 - WIFIWAITCNT - WiFiウェイトステート (R)

WiFiモジュールへのメモリアクセスの際にかかるウェイトステートを制御します。

このレジスタはファームウェアによって起動時に初期化されて以降は固定です。

`POWCNT2.1` でWiFiが有効になっている場合のみこのレジスタにアクセスできます。

```
  0-2   0x0480_0000..0480_7FFF の ウェイトステート (WS0)
  3-5   0x0480_8000..0480_FFFF の ウェイトステート (WS1)
  4-15  不使用 (0)
```

## 0x0400_0301 - NDS7 - HALTCNT - 動作モード (R/W)

Haltモードでは、`(IE AND IF)=0` である限りCPUが一時停止します。

Sleepモードでは、サウンドやビデオを含むほとんどのハードウェアが一時停止します。

```
  0-5   不使用 (0)
  6-7   動作モード  (0=通常, 1=Enter GBA Mode, 2=Halt, 3=Sleep)
```

`HALTCNT`レジスタには直接アクセスすることは推奨されておらず、代わりに SWI で BIOS の `Halt, Sleep, CustomHalt, IntrWait, VBlankIntrWait` を使ってアクセスすることが推奨されています。

NDS9 は`HALTCNT`レジスタを持っていませんが、その代わりに、Haltモードに入る際にはコプロセッサ・オペコード`mcr p15,0,r0,c7,c0,4`を使用します。(this opcode locks up if interrupts are disabled via IME=0 (unlike NDS7 HALTCNT method which doesn’t check IME).)

