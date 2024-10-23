# BIOS

BIOS には、SWI命令を通して利用可能ないくつかのシステムコール関数が含まれています。

システムコールの引数は通常、レジスタR0,R1,R2,R3に渡されます。

システムコール終了後、R0,R1,R3には、ゴミか戻り値のいずれかが含まれます。他のレジスタ(R2,R4-R14)は全て変更されません。

## BIOS関数の一覧

NDSでは、GBAのBIOSから、いくつかの機能が削除され、新しい機能が追加されています。

### 標準機能

nn | 関数
-- | -- 
0x00 | SoftReset
0x01 | RegisterRamReset
0x02 | Halt
0x03 | Stop/Sleep
0x04 | IntrWait
0x05 | VBlankIntrWait
0x06 | Div
0x07 | DivArm
0x08 | Sqrt
0x09 | ArcTan
0x0a | ArcTan2
0x0b | CpuSet
0x0c | CpuFastSet
0x0d | GetBiosChecksum
0x0e | BgAffineSet
0x0f | ObjAffineSet

### 解凍

nn | 関数
-- | -- 
0x10 | BitUnPack
0x11 | LZ77UnCompReadNormalWrite8bit
0x12 | LZ77UnCompReadNormalWrite16bit
0x13 | HuffUnCompReadNormal
0x14 | RLUnCompReadNormalWrite8bit
0x15 | RLUnCompReadNormalWrite16bit
0x16 | Diff8bitUnFilterWrite8bit
0x17 | Diff8bitUnFilterWrite16bit
0x18 | Diff16bitUnFilter

### サウンド・その他

nn | 関数
-- | -- 
0x19 | SoundBias
0x1a | SoundDriverInit
0x1b | SoundDriverMode
0x1c | SoundDriverMain
0x1d | SoundDriverVSync
0x1e | SoundChannelClear
0x1f | MidiKey2Freq
0x20 | SoundWhatever0
0x21 | SoundWhatever1
0x22 | SoundWhatever2
0x23 | SoundWhatever3
0x24 | SoundWhatever4
0x25 | MultiBoot
0x26 | HardReset
0x27 | CustomHalt
0x28 | SoundDriverVSyncOff
0x29 | SoundDriverVSyncOn
0x2a | SoundGetJumpList

### GBAのBIOSとの違い

NDSではBIOSに以下の変更が加えられています。

- SoftResetで使うアドレスが異なる
- Halt, Stop/Sleep, Div, Sqrtを呼び出すためのSWIの番号が異なる
- NDS9ではHaltを使うと、r0の内容が破壊される
- NDS9ではIntrWaitにバグがある
- CpuFastSet allows 4-byte blocks (nice), but...
- CpuFastSet works very SLOW because of a programming bug (uncool)
- 解凍を行う関数でコールバックを利用するようになった
- SoundBiasにdelayを指定するための引数が追加された

## 0x0400_0308 - NDS7 - BIOSPROT - BIOS読み取り保護アドレス

`BIOSPROT`レジスタはNDS7のBIOS領域の最初の数KBを`double-protect`する際にその制御を行うレジスタです。

BIOSの保護領域は2つに分けられます。1つは常にアクティブで、もう1つは`BIOSPROT`レジスタによって制御されます。

まとめると、BIOS(を実行中のCPU)のみがBIOS領域からの読み取りが可能で、他がBIOS領域からデータを読み出すと`0xFF`となります。

 Opcodes at... | Can read from | Expl.
 -- | -- | --
 0..\[BIOSPROT\]-1   | 0..3FFFh            | Double-protected (when BIOSPROT is set)
 \[BIOSPROT\]..3FFFh | \[BIOSPROT\]..3FFFh | Normal-protected (常にアクティブ)

電源オン時は、`BIOSPROT=0(disabled)`です。

カートリッジの起動前に、BIOSのブートコードはこのレジスタを`0x1204`で初期化します。(実際は`0x1205`を書き込んでいますが、2バイトでアラインメントされるため最下位ビットは無視されます)

初期化後は、このレジスタへの書き込みは無視されます。

The double-protected region contains the exception vectors, some bytes of code, and the cartridge KEY1 encryption seed (about 4KBytes). As far as I know, it is impossible to unlock the memory once when it is locked, however, with some trickery, it is possible execute code before it gets locked. Also, the two THUMB opcodes at 05ECh can be used to read all memory at 0..3FFFh,

```
  05ECh  ldrb r3,[r3,12h]      ; requires incoming r3=src-12h
  05EEh  pop  r2,r4,r6,r7,r15  ; requires dummy values & THUMB retadr on stack
```

また、ほとんどのBIOS関数（`CpuSet`など）には、BIOS領域をソースアドレスとして使うのを拒否するソフトウェアベースの保護機能があります。唯一の例外は`GetCRC16`ですが、`BIOSPROT`の設定は無視することはできません。

`BIOSPROT`が保護するのはNDS7のBIOSで、NDS9のBIOSには、ソフトウェアやハードウェアによる読み取り保護機能はありません。
