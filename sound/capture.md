# サウンドキャプチャ

NDSには2つのサウンドキャプチャデバイスが内蔵されており、出力波形データをメモリに取り込むことができます。 

サウンドキャプチャ0は左ミキサーまたはチャンネル0からの出力を、サウンドキャプチャ1は右ミキサーまたはチャンネル2からの出力を取り込むことができます。

サンプリング周波数は最大1.04876MHzまで設定可能です。振幅分解能も8ビットから16ビットまで設定できます。

## 利用例

### リバーブ

サウンドキャプチャ機能を用いてリバーブ効果（エコー）を実現できます。

キャプチャ機能でミキサー出力をメモリに保存し、その保存したサウンドデータをチャンネルで再生し、ミキサー経由でスピーカーに出力することでリバーブ効果を実現できます。

### [エフェクター](https://ja.wikipedia.org/wiki/エフェクター)

キャプチャしたデータをメモリに保存し、サブプロセッサでメモリのキャプチャしたデータを変更した後、チャンネルからデータを出力します。

## 4000508h - NDS7 - SNDCAP0CNT - Sound Capture 0 Control Register (R/W)

## 4000509h - NDS7 - SNDCAP1CNT - Sound Capture 1 Control Register (R/W)

```
  Bit
  0     Control of Associated Sound Channels (ANDed with Bit7)
          SNDCAP0CNT: Output Sound Channel 1 (0=As such, 1=Add to Channel 0)
          SNDCAP1CNT: Output Sound Channel 3 (0=As such, 1=Add to Channel 2)
          Caution: Addition mode works only if BOTH Bit0 and Bit7 are set.
  1     Capture Source Selection
          SNDCAP0CNT: Capture 0 Source (0=Left Mixer, 1=Channel 0/Bugged)
          SNDCAP1CNT: Capture 1 Source (0=Right Mixer, 1=Channel 2/Bugged)
  2     Capture Repeat        (0=Loop, 1=One-shot)
  3     Capture Format        (0=PCM16, 1=PCM8)
  4-6   不使用              (常に0)
  7     Capture Start/Status  (0=Stop, 1=Start/Busy)
```

## 4000510h - NDS7 - SNDCAP0DAD - Sound Capture 0 Destination Address (R/W)

## 4000518h - NDS7 - SNDCAP1DAD - Sound Capture 1 Destination Address (R/W)

```
  Bit
  0-26  Destination address (word aligned, bit0-1 are always zero)
  27-31 不使用 (常に0)
```

Capture start address (also used as re-start address for looped capture).

## 4000514h - NDS7 - SNDCAP0LEN - Sound Capture 0 Length (W)

## 400051Ch - NDS7 - SNDCAP1LEN - Sound Capture 1 Length (W)

```
  Bit
  0-15  Buffer length (1..FFFFh words) (ie. N*4 bytes)
  16-31 不使用
```

Minimum length is 1 word (attempts to use 0 words are interpreted as 1 word).

## SOUND1TMR - NDS7 - Sound Channel 1 Timer shared as Capture 0 Timer

## SOUND3TMR - NDS7 - Sound Channel 3 Timer shared as Capture 1 Timer

キャプチャのサンプリング周波数は (`SOUNDxTMR` で設定した) チャンネル1,3 のサンプル周波数と同じになります。

キャプチャされたデータは、同じチャンネルが再利用することを想定しているため(利用例参照)、キャプチャと音声出力で同じ周波数を使用しても問題はないということすかね...(サウンド何もわからん)

## Capture Bugs

The NDS contains two hardware bugs which do occur when capturing data from ch(a) (SNDCAPxCNT.Bit1=1), if so, either bug occurs depending on whether ch(a)+ch(b) addition is enabled or disabled (SNDCAPxCNT.Bit0).

```
  1) Both Negative Bug - SNDCAPxCNT Bit1=1, Bit0=0 (addition disabled)
   Capture data is accidently set to -8000h if ch(a) and ch(b) are both <0.
   Otherwise the correct capture result is returned, ie. plain ch(a) data,
   not being affected by ch(b) (since addition is disabled).
   Workaround: Ensure that ch(a) and/or ch(b) are >=0 (or disabled).
 2) Overflow Bug - SNDCAPxCNT Bit1=1, Bit0=1 (addition enabled)
   In this mode, Capture data isn't clipped to MinMax(-8000h,+7FFFh),
   instead, it is ANDed with FFFFh, so the sign bit is lost if the
   addition result ch(a)+ch(b) is less/greater than -8000h/+7FFFh.
   Workaround: Reduce ch(a)/ch(b) volume or data to avoid overflows.
```

These bugs occur only for capture (speaker output remains intact), and they occur only when capturing ch(a) (capturing mixer-output works flawless).

## ch(a)+ch(b) Channel Addition

The ch(a)+ch(b) addition unit has 2 outputs, with slightly different results:

1. Addition Result for Capture(x) when using capture source=ch(a):
    - Addition is performed always, no matter of SOUNDCNT.Bit12/13.
    - And, no matter of ch(a) enable, result is plain ch(b) if ch(a) is disabled.
    - Result is 16bit (plus fraction) with overflow error (see Capture Bugs).
2. Addition Result for Mixer (towards speakers, and capture source=mixer):
    - Ch(b) is muted if ch(a) is disabled.
    - Ch(b) is muted if ch(b) SOUNDCNT.Bit12/13 is set to “Ch(b) not to mixer”.
    - Result is 17bit (plus fraction) without overflow error.

Addition mode can be used only if the "corresponding" capture unit is enabled, ie. if SNDCAPxCNT (Bit0 AND Bit7)=1. If so, addition affects both mixers (and so, may also affect the "other" capture unit if it reads from mixer).
