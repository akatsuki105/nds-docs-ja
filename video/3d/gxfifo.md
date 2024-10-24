# GXFIFO

## FIFOとPIPE

NDSは、ジオメトリコマンドを送信するために、FIFOとPIPEを使用します。FIFOは256エントリ、PIPEは4エントリの合計260エントリです。

FIFOが空で、PIPEに空きがあれば、データは直接PIPEに移動され、そうでなければFIFOに移動されます。PIPEが半分空（3エントリ未満）で実行されている場合、2エントリがFIFOからPIPEに移動されます。

FIFOの状態は、`GXSTAT.16-26`で取得できます。FIFOが空でも、PIPEにデータが残っている場合があるので注意してください。PIPEまたはFIFOにデータが残っているか、またはコマンドがまだ実行中であるかを確認するには、`GXSTAT.27`のbusyフラグをチェックしてください。

各PIPE/FIFOエントリは、40bitのデータ（8bitのコマンドと32bitのパラメータ値）で構成されます。パラメータのないコマンドは1エントリを占有し、N個のパラメータを持つコマンドはNエントリを占有します。

## 0x0400_0400 - GXFIFO - ジオメトリコマンドFIFO (W) (mirrored up to 400043Fh?

このレジスタに書き込むことで、(圧縮 or 非圧縮の) ジオメトリコマンド と、そのパラメータをジオメトリエンジンに送信します。

0x0400_043Fまで、このレジスタはミラーされています。(つまり、0x0400_0400..0400_043Fのどれかに書き込むことで、0x0400_0400に書き込むことになります)

```
  ジオメトリコマンド(非圧縮, unpacked)
    0-7   コマンド
    8-31  0x00

  ジオメトリコマンド(圧縮, packed)
    0-7   1stコマンド
    8-15  2ndコマンド
    16-23 3rdコマンド
    24-31 4thコマンド

  パラメータ
    0-31  Parameter data for the previously sent (packed) command(s)
```

## ジオメトリコマンド送信レジスタ 4000440h..40005FFh

GXFIFO に書き込んでコマンドを送信する代わりに、4000440h以降のポートを使ってコマンドを送信することもできます。

例えば、GXFIFOにコマンド`0x15` を書き込む代わりに、`0x0400_0454`に(何らかの値を)書き込むことでジオメトリエンジンにコマンド`0x15`を送信できます。

For a command with N paramters: issue N writes to the port.

For a command without parameters: issue one dummy-write to the port.

That mechanism puts the 8bit command + 32bit parameter into the FIFO/PIPE.

If the FIFO is full, then a wait is generated until data is removed from the FIFO, ie. the STR opcode gets freezed, during the wait, the bus cannot be used even by DMA, interrupts, or by the NDS7 CPU.

## GXFIFOへのアクセス

### DMA を使う場合

DMAを使って、事前に計算された大きなデータブロックをFIFOに直接送信することができます。基本的に、モードは7(`CNT_H.11-13=7`)で32bit単位、送信先は4000400h/fixed、長さはNumWords、リピートは0で設定します。

The timings are handled automatically, ie. the system (should) doesn’t freeze when the FIFO is full (see below Overkill note though).

DMAはFIFOが半分以下になったときに開始され、DMAは112ワードをGXFIFOレジスタに書き込みます(もしDMA転送の残りが0になった場合は、112ワードより少ない転送になります)。

### STR,STRD,STM を使う場合

必要であれば、STR,STRD,STM オペコードを使用して FIFO に書き込むことができます。

Opcodes that write more than one 32bit value (ie. STRD and STM) can be used to send ONE UNPACKED command, plus any parameters which belong to that command. After that, there must be a 1 cycle delay before sending the next command (ie. one cannot sent more than one command at once with a single opcode, each command must be invoked by a new opcode). STRD and STM can be used because the GXFIFO register is mirrored to 4000400h..43Fh (16 words).

As with Ports 4000440h and up, the CPU gets stopped if (and as long as) the FIFO is full.

## GXFIFO DMA Overkill on Packed Commands Without Parameters

基本的に、GXFIFO DMAでは、112ワードの制限によりFIFO（256エントリ）が一杯になることはありませんが、多くの「パラメータなしのパックされたコマンド」（例：PUSH、IDENTITY、またはEND）を送信するには、この上限は無駄に多すぎます。

例えば、`0x00151515`のような112ワードのパックされたコマンドをGXFIFOに送信すると、FIFOに336ワードのCmd(`0x15`)が書き込まれるため、FIFOがいっぱいになってしまいます。

これは、FIFOコマンドが処理されて、112ワードの転送が終わるまで発生し、(最悪の場合、数秒間)DMA（およびCPU）が一時停止します。

Not sure if there’s much chance to get Overkills in practice. Normally most commands DO have parameters, and so, usually even LESS than 112 FIFO entries are occupied (since 8bit commands with 32bit parameters are merged into single 40bit FIFO entries).

## GXFIFO / Unpacked Commands

```
  - command1 (upper 24bit zero)
  - parameter(s) for command1 (if any)
  - command2 (upper 24bit zero)
  - parameter(s) for command2 (if any)
  - command3 (upper 24bit zero)
  - parameter(s) for command3 (if any)
```

## GXFIFO / Packed Commands

```
  - command1,2,3,4 packed into one 32bit value (all bits used)
  - parameter(s) for command1 (if any)
  - parameter(s) for command2 (if any)
  - parameter(s) for command3 (if any)
  - parameter(s) for command4 (top-most packed command MUST have parameters)
  - command5,6 packed into one 32bit value (upper 16bit zero)
  - parameter(s) for command5 (if any)
  - parameter(s) for command6 (top-most packed command MUST have parameters)
  - command7,8,9 packed into one 32bit value (upper 8bit zero)
  - parameter(s) for command7 (if any)
  - parameter(s) for command8 (if any)
  - parameter(s) for command9 (top-most packed command MUST have parameters)
```

Packed commands are first decompressed and then stored in command the FIFO.
