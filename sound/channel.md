# チャンネル0..15

各チャンネルには、`0x40004x0..0x40004xF` の16バイトにIOレジスタが割り当てられています。(xはチャンネル番号)

## 40004x0h - NDS7 - SOUNDxCNT - CHx制御レジスタ (R/W)

```
  0-6    Volume Mul   (0..127=silent..loud)
  7      不使用     (常に0)
  8-9    Volume Div   (0=Normal, 1=Div2, 2=Div4, 3=Div16)
  10-14  不使用     (常に0)
  15     Hold         (0=Normal, 1=Hold last sample after one-shot sound)
  16-22  Panning      (0..127=left..right) (64=half volume on both speakers)
  23     不使用     (常に0)
  24-26  デューティ比   (0..7) ;HIGH=(N+1)*12.5%, LOW=(7-N)*12.5% (PSG only)
  27-28  Repeat Mode  (0=Manual, 1=Loop Infinite, 2=One-Shot, 3=Prohibited)
  29-30  モード        (0=PCM8, 1=PCM16, 2=IMA-ADPCM, 3=PSG(矩形波/ノイズ))
  31     Start/Status (0=Stop, 1=Start/Busy)
```

PCM/ADPCM は全てのチャンネルでサポートされていますが、PSGの矩形波はチャンネル 8..13 のみ、(ホワイト)ノイズはチャンネル 14,15 のみ使用できます。

## 40004x4h - NDS7 - SOUNDxSAD - 音源アドレスレジスタ (W)

PCM/ADPCMの場合、音源データはメモリ上に配置されている必要があり、このレジスタではメモリに配置された音源データの開始アドレスを指定します。

```
  0-26  音源データの開始アドレス (4バイトでアラインメントされる)
  27-31 不使用
```

## 40004x8h - NDS7 - SOUNDxTMR - Sound Channel X Timer Register (W)

```
  0-15  Timer Value, Sample frequency, timerval=-(33513982Hz/2)/freq
```

The PSG Duty Cycles are composed of eight “samples”, and so, the frequency for Rectangular Wave is 1/8th of the selected sample frequency.

PSG音源によるノイズの場合、ノイズ周波数はサンプル周波数と同じになります。

## 40004xAh - NDS7 - SOUNDxPNT - ループ開始位置 (W)

音源をループさせたい場合にループ開始位置を指定するためのレジスタです。

```
  0-15  音源データのループ開始オフセット (ワード単位, つまり(N*4)バイト)
```

## 40004xCh - NDS7 - SOUNDxLEN - PCM音源サイズレジスタ (W)

PCM音源のサイズを指定するためのレジスタです。

The number of samples for N words is `4*N` PCM8 samples, `2*N` PCM16 samples, or `8*(N-1)` ADPCM samples (the first word containing the ADPCM header).

PSGモードでは、このレジスタは使用されません。

```
  0-21  サウンドの長さ (ワード単位, つまり(N*4)バイト)
  22-31 不使用
```

Minimum length (the sum of PNT+LEN) is 4 words (16 bytes), smaller values (0..3 words) are causing hang-ups (busy bit remains set infinite, but no sound output occurs).

In One-shot mode, the sound length is the sum of (PNT+LEN).

In Looped mode, the length is (1*PNT+Infinite*LEN), ie. the first part (PNT) is played once, the second part (LEN) is repeated infinitely.
