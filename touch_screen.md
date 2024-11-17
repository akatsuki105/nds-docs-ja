# [タッチパネル(TSC)](https://mgba-emu.github.io/gbatek/#dstouchscreencontrollertsc)

> [!NOTE]
> TSC = Touch Screen Controller

```
NDS:       Texas Instruments TSC2046
NDS-Lite:  Asahi Kasei Microsystems AK4148AVT
```

## Control Byte (transferred MSB first)

```
  0-1  Power Down Mode Select
  2    Reference Select (0=Differential, 1=Single-Ended)
  3    Conversion Mode  (0=12bit, max CLK=2MHz, 1=8bit, max CLK=3MHz)
  4-6  Channel Select   (0-7, see below)
  7    Start Bit (Must be set to access Control Byte)
```

## Channel

```
  0 Temperature 0 (requires calibration, step 2.1mV per 1'C accuracy)
  1 Touchscreen Y-Position  (somewhat 0B0h..F20h, or FFFh=released)
  2 Battery Voltage         (not used, connected to GND in NDS, always 000h)
  3 Touchscreen Z1-Position (diagonal position for pressure measurement)
  4 Touchscreen Z2-Position (diagonal position for pressure measurement)
  5 Touchscreen X-Position  (somewhat 100h..ED0h, or 000h=released)
  6 AUX Input               (connected to Microphone in the NDS)
  7 Temperature 1 (difference to Temp 0, without calibration, 2'C accuracy)
```

All channels can be accessed in Single-Ended mode.

In differential mode, only channel 1,3,4,5 (X,Z1,Z2,Y) can be accessed.

On AK4148AVT, channel 6 (AUX) is split into two separate channels, IN1 and IN2, separated by Bit2 (Reference Select). IN1 is selected when Bit2=1, IN2 is selected when Bit2=0 (despite of the Bit2 settings, both IN1 and IN2 are using single ended more). On the NDS-Lite, IN1 connects to the mircrophone (as on original NDS), and the new IN2 input is simply wired to VDD3.3 (which is equal to the external VREF voltage, so IN2 is always FFFh).


## Power Down Mode

```
  Mode /PENIRQ   VREF  ADC   Recommended use
  0    Enabled   Auto  Auto  Differential Mode (Touchscreen, Penirq)
  1    Disabled  Off   On    Single-Ended Mode (Temperature, Microphone)
  2    Enabled   On    Off   Don't use
  3    Disabled  On    On    Don't use
```

Allows to enable/disable the /PENIRQ output, the internal reference voltage (VREF), and the Analogue-Digital Converter.

For AK4148AVT, Power Down modes are slightly different (among others, /PENIRQ is enabled in Mode 0..2).


## Reference Voltage (VREF)

VREF is used as reference voltage in single ended mode, at 12bit resolution one ADC step equals to VREF/4096. The TSC generates an internal VREF of 2.5V (+/-0.05V), however, the NDS uses as external VREF of 3.33V (sinks to 3.31V at low battery charge), the external VREF is always enabled, no matter if internal VREF is on or off. Power Down Mode 1 disables the internal VREF, which may reduce power consumption in single ended mode. After conversion, Power Down Mode 0 should be restored to re-enable the Penirq signal.

## Sending the first Command after Chip-Select

Switch chipselect low, then output the command byte (MSB first).

## Reply Data

The following reply data is received (via Input line) after the Command byte has been transferred: One dummy bit (zero), followed by the 8bit or 12bit conversion result (MSB first), followed by endless padding (zero).

Note: The returned ADC value may become unreliable if there are longer delays between sending the command, and receiving the reply byte(s).

## Sending further Commands during/after receiving Reply Data

```
In general, the Output line should be LOW during the reply period, however, once when Data bit6 has been received (or anytime later), a new Command can be invoked (started by sending the HIGH-startbit, ie. Command bit7), simultaneously, the remaining reply-data bits (bit5..0) can be received.

In other words, the new command can be output after receiving 3 bits in 8bit mode (the dummy bit, and data bits 7..6), or after receiving 7 bits in 12bit mode (the dummy bit, and data bits 11..6).

In practice, the NDS SPI register always transfers 8 bits at once, so that one would usually receive 8 bits (rather than above 3 or 7 bits), before outputting a new command.
```

## Touchscreen Position

Read the X and Y positions in 12bit differential mode, then convert the touchscreen values (adc) to screen/pixel positions (scr), as such:

```
  scr.x = (adc.x-adc.x1) * (scr.x2-scr.x1) / (adc.x2-adc.x1) + (scr.x1-1)
  scr.y = (adc.y-adc.y1) * (scr.y2-scr.y1) / (adc.y2-adc.y1) + (scr.y1-1)
```

The X1,Y1 and X2,Y2 calibration points are found in Firmware User Settings,
- DS Firmware User Settings scr.x1,y1,x2,y2 are originated at 1,1 (converted to 0,0 by above formula).


## Touchscreen Pressure (not supported on DSi)

To calculate the pressure resistance, in respect to X/Y/Z positions and X/Y plate resistances, either of below formulas can be used,

```
  Rtouch = (Rx_plate*Xpos*(Z2pos/Z1pos-1))/4096
  Rtouch = (Rx_plate*Xpos*(4096/Z1pos-1)-Ry_plate*(1-Ypos))/4096
```

The second formula requires less CPU load (as it doesn’t require to measure Z2), the downside is that one must know both X and Y plate resistance (or at least their ratio). The first formula doesn’t require that ratio, and so Rx_plate can be set to any value, setting it to 4096 results in

```
  touchval = Xpos*(Z2pos/Z1pos-1)
```

Of course, in that case, touchval is just a number, not a resistance in Ohms.

## Touchscreen Notes

It may be impossible to press locations close to the screen borders.

When pressing two or more locations the TSC values will be somewhere in the middle of these locations.

The TSC values may be garbage if the screen becomes newly pressed or released, to avoid invalid inputs: read TSC values at least two times, and ignore BOTH positions if ONE position was invalid.

## Microphone / AUX Channel

Observe that the microphone amplifier is switched off after power up, see:

- [電源管理装置](../power_management_device.md)
- [電源制御](../power_control.md)


## Temperature Calculation (not supported on DSi)

TP0 decreases by circa 2.1mV per degree Kelvin. The voltage difference between TP1 minus TP0 increases by circa 0.39mV (1/2573 V) per degree Kelvin. At VREF=3.33V, one 12bit ADC step equals to circa 0.8mV (VREF/4096).

Temperature can be calculated at best resolution when using the current TP0 value, and two calibration values (an ADC value, and the corresponding temperature in degrees kelvin):

```
  K = (CAL.TP0-ADC.TP0) * 0.4 + CAL.KELVIN
```

Alternately, temperature can be calculated at rather bad resolution, but without calibration, by using the difference between TP1 and TP0:

```
  K = (ADC.TP1-ADC.TP0) * 8568 / 4096
```

To convert Kelvin to other formats,

```
  Celsius:     C = (K-273.15)
  Fahrenheit:  F = (K-273.15)*9/5+32
  Reaumur:     R = (K-273.15)*4/5
  Rankine:     X = (K)*9/5
```

The Temperature Range for the TSC 2046 chip is -40’C..+85’C (for AK4181AVT only -20’C..+70’C). According to Nintendo, the DS should not be exposed to “extreme” heat or cold, the optimal battery charging temperature is specified as +10’C..+40’C.

The original firmware does not support temperature calibration, calibration is supported by nocash firmware (if present). See Extended Settings,

- [DS Firmware Extended Settings](https://mgba-emu.github.io/gbatek/#dsfirmwareextendedsettings)

## Pin-Outs

```
         ________
  VCC  1|o       |16 DCLK
  X+   2|        |15 /CS
  Y+   3|  TSC   |14 DIN
  X-   4|  2046  |13 BUSY
  Y-   5|        |12 DOUT
  GND  6|        |11 /PENIRQ
  VBAT 7|        |10 IOVDD
  AUX  8|________|9  VREF
```

For AK4181AVT, same pins as above, except that IOVDD replaced by the new IN2 input, the pin is wired to VDD3.3 (so IN2 is always equal to VREF, which is wired to VDD3.3, too) (and AUX is renamed to IN1, and is kept used for MIC input).

## DSi Touchscreen Controller (in NDS mode)

DSi in NDS mode does support only X, Y, and MIC (all other channels do return FFFh in 12bit mode, and FFh in 8bit mode, ie. no pressure, no temperature, and no GNDed battery sensor). On DSi, MIC does return data in both single-ended and differential mode (unlike as on real NDS).

## DSi Touchscreen Controller (in DSi mode)

The DSi touchscreen controller supports a NDS backwards compatibility mode. But, in DSi mode, it is working entirely different (it’s still accessed via SPI bus, but with some new MODE/INDEX values).

- [DSi Touchscreen/Sound Controller](https://mgba-emu.github.io/gbatek/#dsitouchscreensoundcontroller): The NDS Touchscreen controller did additionally allow to read Temperature and Touchscreen Pressure - unknown if the DSi is also supporting such stuff (via whatever DSi-specific registers).

The touchscreen hardware can be switched to NDS compatibility mode (for older games), but unknown how to do that.

