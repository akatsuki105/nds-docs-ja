# リアルタイムクロック(RTC)

**NDS**

Seiko Instruments Inc. S-35180 (compatible with S-35190A)  
Miniature 8pin RTC with 3-wire serial bus

**DSi**

Seiko S-35199A01 (12pin BGA, with some extra functions like FOUT and Alarm Date)

## 4000138h - NDS7 - Real Time Clock Register

```
  Bit      Expl.
  0        Data I/O   (0=Low, 1=High)
  1        Clock Out  (0=Low, 1=High)
  2        Select Out (0=Low, 1=High/Select)
  4        Data  Direction  (0=Read, 1=Write)
  5        Clock Direction  (should be 1=Write)
  6        Select Direction (should be 1=Write)
  3,8-11   Unused I/O Lines
  7,12-15  Direction for Bit3,8-11 (usually 0)
  16-31    Not used
```


## Serial Transfer Flowchart

Chipselect and Command/Parameter Sequence:

```
  Init CS=LOW and /SCK=HIGH, and wait at least 1us
  Switch CS=HIGH, and wait at least 1us
  Send the Command byte (see bit-transfer below)
  Send/receive Parameter byte(s) associated with the command (see below)
  Switch CS to LOW
```

Bit transfer (repeat 8 times per cmd/param byte) (bits transferred LSB first):

```
  Output /SCK=LOW and SIO=databit (when writing), then wait at least 5us
  Output /SCK=HIGH, wait at least 5us, then read SIO=databit (when reading)
  In either direction, data is output on (or immediately after) falling edge.
```

Ideally, \<both> commands and parameters should be transmitted LSB-first
(unlike the original Seiko document, which recommends LSB-first for data, and
MSB-first for commands) (actually, later Seiko datasheets are going so far to
recommend MSB-first for everything, eg. to use bit-reversed Data=C8h for
Year=13h).

## Command Register

```
  Command Register
    Fwd  Rev
    0    7   Fixed Code (must be 0)
    1    6   Fixed Code (must be 1)
    2    5   Fixed Code (must be 1)
    3    4   Fixed Code (must be 0, or, DSi only: 1=Extended Command)
    4-6  3-1 Command
             Fwd Rev Parameter bytes (read/write access)
             0   0   1 byte, status register 1
             4   1   1 byte, status register 2
             2   2   7 bytes, date & time (year,month,day,day_of_week,hh,mm,ss)
             6   3   3 bytes, time (hh,mm,ss)
             1*  4*  1 byte, int1, frequency duty setting
             1*  4*  3 bytes, int1, alarm time 1 (day_of_week, hour, minute)
             5   5   3 bytes, int2, alarm time 2 (day_of_week, hour, minute)
             3   6   1 byte, clock adjustment register
             7   7   1 byte, free register
             Extended command (when above "fourth bit" was set, DSi only)
             Fwd Rev Parameter bytes (read/write access)
             0   0   3 byte, up counter (msw,mid,lsw) (read only)
             4   1   1 byte, FOUT register setting 1
             2   2   1 byte, FOUT register setting 2
             6   3   reserved
             1   4   3 bytes, alarm date 1 (year,month,day)
             5   5   3 bytes, alarm date 2 (year,month,day)
             3   6   reserved
             7   7   reserved
    7    0   Parameter Read/Write Access (0=Write, 1=Read)
```

## Control and Status Registers

```
  Status Register 1
    0   W   Reset                (0=Normal, 1=Reset)
    1   R/W 12/24 hour mode      (0=12 hour, 1=24 hour)
    2-3 R/W General purpose bits
    4   R   Interrupt 1 Flag (1=Yes)                      ;auto-cleared on read
    5   R   Interrupt 2 Flag (1=Yes)                      ;auto-cleared on read
    6   R   Power Low Flag (0=Normal, 1=Power is/was low) ;auto-cleared on read
    7   R   Power Off Flag (0=Normal, 1=Power was off)    ;auto-cleared on read
    Power off indicates that the battery was removed or fully discharged,
    all registers are reset to 00h (or 01h), and must be re-initialized.
  Status Register 2
    0-3 R/W INT1 Mode/Enable
            0000b Disable
            0x01b Selected Frequency steady interrupt
            0x10b Per-minute edge interrupt
            0011b Per-minute steady interrupt 1 (duty 30.0 seconds)
            0100b Alarm 1 interrupt
            0111b Per-minute steady interrupt 2 (duty 0.0079 seconds)
            1xxxb 32kHz output
    4-5 R/W General purpose bits
    6   R/W INT2 Enable
            0b    Disable
            1b    Alarm 2 interrupt
    7   R/W Test Mode (0=Normal, 1=Test, don't use) (cleared on Reset)
  Clock Adjustment Register (to compensate oscillator inaccuracy)
    0-7 R/W Adjustment (00h=Normal, no adjustment)
  Free Register
    0-7 R/W General purpose bits
```

## Date Registers


```
  Year Register
    0-7 R/W Year     (BCD 00h..99h = 2000..2099)
  Month Register
    0-4 R/W Month    (BCD 01h..12h = January..December)
    5-7 -   Not used (always zero)
  Day Register
    0-5 R/W Day      (BCD 01h..28h,29h,30h,31h, range depending on month/year)
    6-7 -   Not used (always zero)
  Day of Week Register (septenary counter)
    0-2 R/W Day of Week (00h..06h, custom assignment, usually 0=Monday?)
    3-7 -   Not used (always zero)
```



## Time Registers


```
  Hour Register
    0-5 R/W Hour     (BCD 00h..23h in 24h mode, or 00h..11h in 12h mode)
    6   *   AM/PM    (0=AM before noon, 1=PM after noon)
            * 24h mode: AM/PM flag is read only (PM=1 if hour = 12h..23h)
            * 12h mode: AM/PM flag is read/write-able
            * 12h mode: Observe that 12 o'clock is defined as 00h (not 12h)
    7   -   Not used (always zero)
  Minute Register
    0-6 R/W Minute   (BCD 00h..59h)
    7   -   Not used (always zero)
  Second Register
    0-6 R/W Minute   (BCD 00h..59h)
    7   -   Not used (always zero)
```



## Alarm 1 and Alarm 2 Registers


```
  Alarm1 and Alarm2 Day of Week Registers (INT1 and INT2 each)
    0-2 R/W Day of Week (00h..06h)
    3-6 -   Not used (always zero)
    7   R/W Compare Enable (0=Alarm every day, 1=Alarm only at specified day)
  Alarm1 and Alarm2 Hour Registers (INT1 and INT2 each)
    0-5 R/W Hour     (BCD 00h..23h in 24h mode, or 00h..11h in 12h mode)
    6   R/W AM/PM    (0=AM, 1=PM) (must be correct even in 24h mode?)
    7   R/W Compare Enable (0=Alarm every hour, 1=Alarm only at specified hour)
  Alarm1 and Alarm2 Minute Registers (INT1 and INT2 each)
    0-6 R/W Minute   (BCD 00h..59h)
    7   R/W Compare Enable (0=Alarm every min, 1=Alarm only at specified min)
  Selected Frequency Steady Interrupt Register (INT1 only) (when Stat2/Bit2=0)
    0   R/W Enable 1Hz Frequency  (0=Disable, 1=Enable)
    1   R/W Enable 2Hz Frequency  (0=Disable, 1=Enable)
    2   R/W Enable 4Hz Frequency  (0=Disable, 1=Enable)
    3   R/W Enable 8Hz Frequency  (0=Disable, 1=Enable)
    4   R/W Enable 16Hz Frequency (0=Disable, 1=Enable)
            The signals are ANDed when two or more frequencies are enabled,
            ie. the /INT signal gets LOW when either of the signals is LOW.
    5-7 R/W General purpose bits
```

Note: There is only one register shared as "Selected Frequency Steady
Interrupt" (accessed as single byte parameter when Stat2/Bit2=0) and as "Alarm1
Minute" (accessed as 3rd byte of 3-byte parameter when Stat2/Bit2=1), changing
either value will also change the other value.



## Up Counter (DSi only)


```
  Up Counter Msw
    0-7 R   Up Counter bit16-23 (non-BCD, 00h..FFh)
  Up Counter Mid
    0-7 R   Up Counter bit8-15  (non-BCD, 00h..FFh)
  Up Counter Lsw
    0-7 R   Up Counter bit0-7   (non-BCD, 00h..FFh)
```

The 24bit Up Counter is incremented when seconds=00h (that is, once per minute;
unless the Time is getting getting changed by write commands, which may cause
some stuttering). The Up Counter starts at 000000h upon power-up, and, if the
battery lasts that long: wraps from FFFFFFh to 000000h after about 30 years.



## Alarm 1 and Alarm 2 Date Registers (DSi only)


```
  Alarm 1 and Alarm 2 Year Register
    0-7 R/W Year     (BCD 00h..99h = 2000..2099)
  Alarm 1 and Alarm 2 Month Register
    0-4 R/W Month    (BCD 01h..12h = January..December)
    5   -   Not used (always zero)
    6   R/W Year Compare Enable (0=Ignore, 1=Enable)
    7   R/W Month Compare Enable (0=Ignore, 1=Enable)
  Alarm 1 and Alarm 2 Day Register
    0-5 R/W Day      (BCD 01h..28h,29h,30h,31h, range depending on month/year)
    6   -   Not used (always zero)
    7   R/W Day Compare Enable (0=Ignore, 1=Enable)
```

XXX unspecified if above Alarm Date stuff is really R/W (or write only)



## FOUT Register (DSi only)


```
  FOUT Register Setting 1
    0-7 R/W  Enable bits (bit0=256Hz, bit1=512Hz, ..., bit7=32768Hz)
  FOUT Register Setting 2
    0-7 R/W  Enable bits (bit0=1Hz,   bit1=2Hz, ...,   bit7=128Hz)
  The above sixteen FOUT signals are ANDed when two or more frequencies are
  enabled, ie. the FOUT signal gets LOW when either of the signals is LOW.
```

Note: The FOUT pin goes to the DSi's wifi daughterboard (FOUT is configured by
firmware (needed if it was changed, or when the battery was removed), FOUT is
required for exchanging Atheros WMI commands/events).



## 割り込み

/INT信号のピンは1つしか存在せず、INT1とINT2の両方で共有されています。

In the NDS, it is connected to the SI-input of the SIO unit (and so, also shared with SIO interrupts). To enable the interrupt, RCNT should be set to 8144h (Bit14-15=General Purpose mode, Bit8=SI Interrupt Enable, Bit6,2=SI Output/High).

The Output/High settings seems to be used as pullup (giving faster reactions on
low-to-high transitions) (nethertheless, in most cases it seems to be also
working okay as Input, ie. with RCNT=8100h).

The RCNT interrupt is generated on high-to-low transitions on the SI line (but
only if the IRQ is enabled in RCNT.8, and only if RCNT is set to general
purpose mode) (note: changing RCNT.8 from off-to-on does NOT generate IRQs,
even when SI is LOW).



## Pin-Outs


```
  1 /INT      8 VDD
  2 XOUT      7 SIO
  3 XIN       6 /SCK
  4 GND       5 CS
```

