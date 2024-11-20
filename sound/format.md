# サウンドフォーマット

`SOUNDxCNT.29-30`で指定するサウンドフォーマットには以下の4種類があります。

```
  SOUNDxCNT.29-30:
    0: PCM8
    1: PCM16
    2: IMA-ADPCM
    3: PSG(矩形波/ノイズ)
```

## PCM8

サンプルは符号あり8ビットで表現されます。(つまり, -128..127)

最終的な出力は`PCM8=0xNN`の場合、 `PCM16=0xNN00`に拡張されます。

## PCM16

サンプルは符号あり16ビットで表現されます。(つまり, -32768..32767)

## IMA-ADPCM

IMA-ADPCM は [ADPCM](https://ja.wikipedia.org/wiki/適応的差分パルス符号変調) の一種で、International Multimedia Association (IMA) によって設計されました。 

NDSで使われる IMA-ADPCM 音源データは、32bitのヘッダの後に4bitの音源データが続いています。ヘッダの内容は次のようになっています。

```
  Bit:
  0-15    初期サンプル (PCM16フォーマット, -7FFFh..+7FFF (-8000h は指定できない))
  16-22   Initial Table Index Value (Index = 0..88)
  23-31   不使用 (0)
```

In theory, the 4bit values are decoded into PCM16 values, as such:

```
  Diff = ((Data4bit AND 7)*2+1)*AdpcmTable[Index]/8      ;see rounding-error
  IF (Data4bit AND 8)=0 THEN Pcm16bit = Max(Pcm16bit+Diff,+7FFFh)
  IF (Data4bit AND 8)=8 THEN Pcm16bit = Min(Pcm16bit-Diff,-7FFFh)
  Index = MinMax (Index+IndexTable[Data4bit AND 7],0,88)
```

In practice, the first line works like so (with rounding-error):

```
  Diff = AdpcmTable[Index]/8
  IF (data4bit AND 1) THEN Diff = Diff + AdpcmTable[Index]/4
  IF (data4bit AND 2) THEN Diff = Diff + AdpcmTable[Index]/2
  IF (data4bit AND 4) THEN Diff = Diff + AdpcmTable[Index]/1
```

And, a note on the second/third lines (with clipping-error):

```
  Max(+7FFFh) leaves -8000h unclipped (can happen if initial PCM16 was -8000h)
  Min(-7FFFh) clips -8000h to -7FFFh (possibly unlike windows .WAV files?)
```

`IndexTable` と `AdpcmTable` の内容は次のようになっています。

```c
const s8 IndexTable[8] = {-1, -1, -1, -1, 2, 4, 6, 8};

const u16 AdpcmTable[89] = {
    0x0007, 0x0008, 0x0009, 0x000A, 0x000B, 0x000C, 0x000D, 0x000E,
    0x0010, 0x0011, 0x0013, 0x0015, 0x0017, 0x0019, 0x001C, 0x001F,
    0x0022, 0x0025, 0x0029, 0x002D, 0x0032, 0x0037, 0x003C, 0x0042,
    0x0049, 0x0050, 0x0058, 0x0061, 0x006B, 0x0076, 0x0082, 0x008F,
    0x009D, 0x00AD, 0x00BE, 0x00D1, 0x00E6, 0x00FD, 0x0117, 0x0133,
    0x0151, 0x0173, 0x0198, 0x01C1, 0x01EE, 0x0220, 0x0256, 0x0292,
    0x02D4, 0x031C, 0x036C, 0x03C3, 0x0424, 0x048E, 0x0502, 0x0583,
    0x0610, 0x06AB, 0x0756, 0x0812, 0x08E0, 0x09C3, 0x0ABD, 0x0BD0,
    0x0CFF, 0x0E4C, 0x0FBA, 0x114C, 0x1307, 0x14EE, 0x1706, 0x1954,
    0x1BDC, 0x1EA5, 0x21B6, 0x2515, 0x28CA, 0x2CDF, 0x315B, 0x364B,
    0x3BB9, 0x41B2, 0x4844, 0x4F7E, 0x5771, 0x602F, 0x69CE, 0x7462,
    0x7FFF,
};
```

The closest way to reproduce the AdpcmTable with 32bit integer maths appears:

```
  X=000776d2h, FOR I=0 TO 88, Table[I]=X SHR 16, X=X+(X/10), NEXT I
  Table[3]=000Ah, Table[4]=000Bh, Table[88]=7FFFh, Table[89..127]=0000h
```

When using ADPCM and loops, set the loopstart position to the data part, rather than the header. At the loop end, the SAD value is reloaded to the loop start location, additionally index and pcm16 values are reloaded to the values that have originally appeared at that location. Do not change the ADPCM loop start position during playback.

## PSG

GBA以前のPSG音源とは大きく異なり、NDSのPSG音源は単純な矩形波とノイズの2種類の音色しかありません。

出力は、PCM16と同じく符号あり16ビットで表現されます。(つまり, -32768..32767)

PSG sound is always Infinite (the SOUNDxLEN Register, and the SOUNDxCNT Repeat Mode bits have no effect). The PSG hardware doesn’t support sound length, sweep, or volume envelopes, however, these effects can be produced by software with little overload (or, more typically, with enormous overload, depending on the programming language used).

### 矩形波

Each duty cycle consists of eight HIGH or LOW samples, so the sound frequency is 1/8th of the selected sample rate. The duty cycle always starts at the begin of the LOW period when the sound gets (re-)started.

```
  0  12.5% "_______-_______-_______-"
  1  25.0% "______--______--______--"
  2  37.5% "_____---_____---_____---"
  3  50.0% "____----____----____----"
  4  62.5% "___-----___-----___-----"
  5  75.0% "__------__------__------"
  6  87.5% "_-------_-------_-------"
  7   0.0% "________________________"
```

The Wave Duty bits exist and are read/write-able on all channels (although they are actually used only in PSG mode on channels 8-13).

### ノイズ

Noise randomly switches between HIGH and LOW samples, the output levels are calculated, at the selected sample rate, as such:

```
  X=X SHR 1, IF carry THEN Out=LOW, X=X XOR 6000h ELSE Out=HIGH
```

The initial value when (re-)starting the sound is X=7FFFh. The formula is more or less same as “15bit polynomial counter” used on 8bit Gameboy and GBA.
