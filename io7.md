## ARM7 I/Oポート

```
  4000004h  2   DISPSTAT
  4000006h  2   VCOUNT
  40000B0h  30h DMA Channels 0..3
  4000100h  10h Timers 0..3
  4000120h  4   Debug SIODATA32
  4000128h  4   Debug SIOCNT
  4000130h  2   KEYINPUT
  4000132h  2   KEYCNT
  4000134h  2   Debug RCNT
  4000136h  2   EXTKEYIN
  4000138h  1   RTC Realtime Clock Bus
  4000180h  2   IPCSYNC - IPC Synchronize Register (R/W)
  4000184h  2   IPCFIFOCNT - IPC Fifo Control Register (R/W)
  4000188h  4   IPCFIFOSEND - IPC Send Fifo (W)
  40001A0h  2   AUXSPICNT - Gamecard ROM and SPI Control
  40001A2h  2   AUXSPIDATA - Gamecard SPI Bus Data/Strobe
  40001A4h  4   Gamecard bus timing/control
  40001A8h  8   Gamecard bus 8-byte command out
  40001B0h  4   Gamecard Encryption Seed 0 Lower 32bit
  40001B4h  4   Gamecard Encryption Seed 1 Lower 32bit
  40001B8h  2   Gamecard Encryption Seed 0 Upper 7bit (bit7-15 unused)
  40001BAh  2   Gamecard Encryption Seed 1 Upper 7bit (bit7-15 unused)
  40001C0h  2   SPI bus Control (Firmware, Touchscreen, Powerman)
  40001C2h  2   SPI bus Data
  4000204h  2   EXMEMSTAT - External Memory Status
  4000206h  2   WIFIWAITCNT
  4000208h  4   IME - Interrupt Master Enable (R/W)
  4000210h  4   IE  - Interrupt Enable (R/W)
  4000214h  4   IF  - Interrupt Request Flags (R/W)
  4000218h  -   IE2  ;\DSi only (additional ARM7 interrupt sources)
  400021Ch  -   IF2  ;/
  4000240h  1   VRAMSTAT - VRAM-C,D Bank Status (R)
  4000241h  1   WRAMSTAT - WRAM Bank Status (R)
  4000300h  1   POSTFLG
  4000301h  1   HALTCNT (different bits than on GBA) (plus NOP delay)
  4000304h  2   POWCNT2  Sound/Wifi Power Control Register (R/W)
  4000308h  4   BIOSPROT - Bios-data-read-protection address
  Sound:
    4000400h 100h Sound Channel 0..15 (10h bytes each)
    40004x0h  4   SOUNDxCNT - Sound Channel X Control Register (R/W)
    40004x4h  4   SOUNDxSAD - Sound Channel X Data Source Register (W)
    40004x8h  2   SOUNDxTMR - Sound Channel X Timer Register (W)
    40004xAh  2   SOUNDxPNT - Sound Channel X Loopstart Register (W)
    40004xCh  4   SOUNDxLEN - Sound Channel X Length Register (W)
    4000500h  2   SOUNDCNT - Sound Control Register (R/W)
    4000504h  2   SOUNDBIAS - Sound Bias Register (R/W)
    4000508h  1   SNDCAP0CNT - Sound Capture 0 Control Register (R/W)
    4000509h  1   SNDCAP1CNT - Sound Capture 1 Control Register (R/W)
    4000510h  4   SNDCAP0DAD - Sound Capture 0 Destination Address (R/W)
    4000514h  2   SNDCAP0LEN - Sound Capture 0 Length (W)
    4000518h  4   SNDCAP1DAD - Sound Capture 1 Destination Address (R/W)
    400051Ch  2   SNDCAP1LEN - Sound Capture 1 Length (W)
  40021Axh  ..  DSi Registers
  4004xxxh  ..  DSi Registers
  4004700h  2   DSi SNDEXCNT Register  ;\mapped even in DS mode
  4004C0xh  ..  DSi GPIO Registers     ;/
  4100000h  4   IPCFIFORECV - IPC Receive Fifo (R)
  4100010h  4   Gamecard bus 4-byte data in, for manual or dma read
  4700000h  4   Disable ARM7 bootrom overlay (W) (3DS only)
  WiFi:
    4800000h  ..  Wifi WS0 Region (32K) (Wifi Ports, and 8K Wifi RAM)
    4808000h  ..  Wifi WS1 Region (32K) (mirror of above, other waitstates)
```

