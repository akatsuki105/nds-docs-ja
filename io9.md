## ARM9 I/Oポート

## ARM9 Display Engine A

```
  4000000h  4    2D Engine A - DISPCNT - LCD Control (Read/Write)
  4000004h  2    2D Engine A+B - DISPSTAT - General LCD Status (Read/Write)
  4000006h  2    2D Engine A+B - VCOUNT - Vertical Counter (Read only)
  4000008h  80   2D Engine A (same registers as GBA, some changed bits)
  4000060h  2    DISP3DCNT - 3D Display Control Register (R/W)
  4000064h  4    DISPCAPCNT - Display Capture Control Register (R/W)
  4000068h  4    DISP_MMEM_FIFO - Main Memory Display FIFO (R?/W)
  400006Ch  2    2D Engine A - MASTER_BRIGHT - Master Brightness Up/Down
```

## ARM9 DMA, Timers, and Keypad

```
  40000B0h  30h  DMA Channel 0..3
  40000E0h  10h  DMA FILL Registers for Channel 0..3
  4000100h  10h  Timers 0..3
  4000130h  2    KEYINPUT
  4000132h  2    KEYCNT
```

## ARM9 IPC/ROM

```
  4000180h  2  IPCSYNC - IPC Synchronize Register (R/W)
  4000184h  2  IPCFIFOCNT - IPC Fifo Control Register (R/W)
  4000188h  4  IPCFIFOSEND - IPC Send Fifo (W)
  40001A0h  2  AUXSPICNT - Gamecard ROM and SPI Control
  40001A2h  2  AUXSPIDATA - Gamecard SPI Bus Data/Strobe
  40001A4h  4  Gamecard bus timing/control
  40001A8h  8  Gamecard bus 8-byte command out
  40001B0h  4  Gamecard Encryption Seed 0 Lower 32bit
  40001B4h  4  Gamecard Encryption Seed 1 Lower 32bit
  40001B8h  2  Gamecard Encryption Seed 0 Upper 7bit (bit7-15 unused)
  40001BAh  2  Gamecard Encryption Seed 1 Upper 7bit (bit7-15 unused)
```

## ARM9 Memory and IRQ Control

```
  4000204h  2  EXMEMCNT - External Memory Control (R/W)
  4000208h  2  IME - Interrupt Master Enable (R/W)
  4000210h  4  IE  - Interrupt Enable (R/W)
  4000214h  4  IF  - Interrupt Request Flags (R/W)
  4000240h  1  VRAMCNT_A - VRAM-A (128K) Bank Control (W)
  4000241h  1  VRAMCNT_B - VRAM-B (128K) Bank Control (W)
  4000242h  1  VRAMCNT_C - VRAM-C (128K) Bank Control (W)
  4000243h  1  VRAMCNT_D - VRAM-D (128K) Bank Control (W)
  4000244h  1  VRAMCNT_E - VRAM-E (64K) Bank Control (W)
  4000245h  1  VRAMCNT_F - VRAM-F (16K) Bank Control (W)
  4000246h  1  VRAMCNT_G - VRAM-G (16K) Bank Control (W)
  4000247h  1  WRAMCNT   - WRAM Bank Control (W)
  4000248h  1  VRAMCNT_H - VRAM-H (32K) Bank Control (W)
  4000249h  1  VRAMCNT_I - VRAM-I (16K) Bank Control (W)
```

## ARM9 Maths

```
  4000280h  2  DIVCNT - Division Control (R/W)
  4000290h  8  DIV_NUMER - Division Numerator (R/W)
  4000298h  8  DIV_DENOM - Division Denominator (R/W)
  40002A0h  8  DIV_RESULT - Division Quotient (=Numer/Denom) (R)
  40002A8h  8  DIVREM_RESULT - Division Remainder (=Numer MOD Denom) (R)
  40002B0h  2  SQRTCNT - Square Root Control (R/W)
  40002B4h  4  SQRT_RESULT - Square Root Result (R)
  40002B8h  8  SQRT_PARAM - Square Root Parameter Input (R/W)
  4000300h  4  POSTFLG - Undoc
  4000304h  2  POWCNT1 - Graphics Power Control Register (R/W)
```

## ARM9 3D Display Engine

```
4000320h..6A3h
```

## ARM9 Display Engine B

```
  4001000h  4    2D Engine B - DISPCNT - LCD Control (Read/Write)
  4001008h  50h  2D Engine B (same registers as GBA, some changed bits)
  400106Ch  2    2D Engine B - MASTER_BRIGHT - 16bit - Brightness Up/Down
```

## ARM9 DSi Extra Registers

```
  40021Axh  ..  DSi Registers
  4004xxxh  ..  DSi Registers
```

## ARM9 IPC/ROM

```
  4100000h  4    IPCFIFORECV - IPC Receive Fifo (R)
  4100010h  4    Gamecard bus 4-byte data in, for manual or dma read
```

## ARM9 DS Debug Registers (Emulator/Devkits)

```
  4FFF0xxh  ..   Ensata Emulator Debug Registers
  4FFFAxxh  ..   No$gba Emulator Debug Registers
```

## ARM9 Hardcoded RAM Addresses for Exception Handling

```
  27FFD9Ch   ..  NDS9 Debug Stacktop / Debug Vector (0=None)
  DTCM+3FF8h 4   NDS9 IRQ Check Bits (hardcoded RAM address)
  DTCM+3FFCh 4   NDS9 IRQ Handler (hardcoded RAM address)
```

## Main Memory Control

```
  27FFFFEh  2    Main Memory Control
```

