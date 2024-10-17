# サウンド

DSには16個の[サウンドチャンネル](./channel.md)が搭載されています。

DSの上画面には、左右に配置された2つのスピーカーがあり、ヘッドフォン端子を使わなくてもステレオサウンドを楽しむことができます。

## 制御レジスタ

### 4000500h - NDS7 - SOUNDCNT - サウンド制御レジスタ (R/W)

サウンド全体を制御するためのレジスタです。

```
  0-6   Master Volume       (0..127=silent..loud)
  7     不使用            (常に0)
  8-9   Left Output from    (0=Left Mixer, 1=Ch1, 2=Ch3, 3=Ch1+Ch3)
  10-11 Right Output from   (0=Right Mixer, 1=Ch1, 2=Ch3, 3=Ch1+Ch3)
  12    Output Ch1 to Mixer (0=Yes, 1=No) (both Left/Right)
  13    Output Ch3 to Mixer (0=Yes, 1=No) (both Left/Right)
  14    不使用            (常に0)
  15    Master Enable    (0=Disable, 1=Enable)
  16-31 不使用            (常に0)
```

### 4000504h - NDS7 - SOUNDBIAS - バイアス調整レジスタ (R/W)

サウンドのバイアスについては[こちら](https://nishimurasound.jp/blog/archives/14794)の記事が参考になります。

```
  0-9   バイアス値 (0..3FFh, usually 200h)
  10-31 不使用    (常に0)
```

After applying the master volume, the signed left/right audio signals are in range -200h..+1FFh (with medium level zero), the Bias value is then added to convert the signed numbers into unsigned values (with medium level 200h).

サウンドがオフ(`SOUNDCNT.15==0`)の場合でも、バイアスの出力は常に有効です。

ミキサーのサンプリング周波数は1.04876MHzで、振幅分解能は24bitです。しかし、PWMとの混合後のサンプリング周波数は32.768kHzで、振幅分解能は10bitです。


