# GXFIFO

## FIFOとPIPE

NDSは、ジオメトリコマンドを送信するために、FIFOとPIPEを使用します。FIFOは256エントリ、PIPEは4エントリの合計260エントリです。

FIFOが空で、PIPEに空きがあれば、データは直接PIPEに移動され、そうでなければFIFOに移動されます。PIPEが半分空（3エントリ未満）で実行されている場合、2エントリがFIFOからPIPEに移動されます。

FIFOの状態は、`GXSTAT.16-26`で取得できます。FIFOが空でも、PIPEにデータが残っている場合があるので注意してください。PIPEまたはFIFOにデータが残っているか、またはコマンドがまだ実行中であるかを確認するには、`GXSTAT.27`のbusyフラグをチェックしてください。

各PIPE/FIFOには、40bitのデータ（8bitのコマンドと32bitのパラメータ値）がエントリとして格納されています。パラメータのないコマンドは1エントリを占有(パラメータ部分は`0`)し、N個のパラメータを持つコマンドはNエントリを占有します。

## 0x0400_0400 - NDS9 - GXFIFO - ジオメトリコマンドFIFO (W)

このレジスタに書き込むことで、ジオメトリコマンド と、そのパラメータをジオメトリエンジンに送信します。

`0x0400_043F`まで、このレジスタはミラーされています。(つまり、`0x0400_0400..0400_043F`のどれかに書き込むことで、`0x0400_0400`に書き込むことになります)

ジオメトリコマンドをGXFIFOに送信する際には、次のような順序で送信する必要があります。コマンドは4つまでまとめることができます。パラメータの個数は、コマンドごとに異なります。

```
  bytes:
    0:   1stコマンド
    1:   2ndコマンド
    2:   3rdコマンド
    3:   4thコマンド
    4-a: 1stコマンドのパラメータ列
    a-b: 2ndコマンドのパラメータ列
    b-c: 3rdコマンドのパラメータ列
    c-d: 4thコマンドのパラメータ列
```

例えば、以下のように送信します。

```
例1:
  PLTT_BASE 0xA6F
  VTX_16 0x100, 0x280, 0x100
  MTX_IDENTITY
  MTX_MODE 2
  を4つまとめて送信する場合 のバイト列は

    0x2B, 0x23, 0x15, 0x10  ; 1st, 2nd, 3rd, 4thコマンド
    0x6F, 0x0A, 0x00, 0x00  ; 1stコマンド(PLTT_BASE)のパラメータ
    0x00, 0x01, 0x80, 0x02  ; 2ndコマンド(VTX_16)のパラメータ1
    0x00, 0x01, 0x00, 0x00  ; 2ndコマンド(VTX_16)のパラメータ2
    0x02, 0x00, 0x00, 0x00  ; 4thコマンド(MTX_MODE)のパラメータ

  となります。(MTX_IDENTITYはパラメータがないので、パラメータ列はありません)

例2:
  VTX_16 0x100, 0x280, 0x100
  END_VTXS
  を2つまとめて送信する場合 のバイト列は

    0x23, 0x41, 0x00, 0x00  ; 1st, 2ndコマンド
    0x00, 0x01, 0x80, 0x02  ; 2ndコマンド(VTX_16)のパラメータ1
    0x00, 0x01, 0x00, 0x00  ; 2ndコマンド(VTX_16)のパラメータ2

  となります。(3rd, 4thコマンドが NOP(0x00, パラメータなし) になっているのと同じです)
```

## ジオメトリコマンド送信レジスタ 0x0400_0440..0400_05FF 

GXFIFO に書き込んでコマンドを送信する代わりに、`0x0400_0440`以降のI/Oポートを使ってコマンドを送信することもできます。

例えば、GXFIFOにコマンド`0x15` を書き込む代わりに、`0x0400_0454`に(何らかの値を)書き込むことでジオメトリエンジンにコマンド`0x15`(`MTX_IDENTITY`)を送信できます。

For a command with N paramters: issue N writes to the port.

For a command without parameters: issue one dummy-write to the port.

That mechanism puts the 8bit command + 32bit parameter into the FIFO/PIPE.

If the FIFO is full, then a wait is generated until data is removed from the FIFO, ie. the STR opcode gets freezed, during the wait, the bus cannot be used even by DMA, interrupts, or by the NDS7 CPU.

## GXFIFOへのアクセス

### DMA を使う場合

DMAを使って、事前に計算された大きなデータブロックをFIFOに直接送信することができます。基本的に、モードは7(`DMAnCNT.27-29=7`)で32bit単位、送信先は`0x04000400/fixed`、長さはNumWords、リピートは0にします。

The timings are handled automatically, ie. the system (should) doesn’t freeze when the FIFO is full (see below Overkill note though).

DMAはFIFOが半分以下になったときに開始され、DMAは112ワードをGXFIFOレジスタに書き込みます(もしDMA転送の残りが0になった場合は、112ワードより少ない転送になります)。

### STR,STRD,STM を使う場合

必要であれば、STR,STRD,STM オペコードを使用して FIFO に書き込むことができます。

Opcodes that write more than one 32bit value (ie. STRD and STM) can be used to send ONE UNPACKED command, plus any parameters which belong to that command. After that, there must be a 1 cycle delay before sending the next command (ie. one cannot sent more than one command at once with a single opcode, each command must be invoked by a new opcode). STRD and STM can be used because the GXFIFO register is mirrored to 4000400h..43Fh (16 words).

As with Ports 4000440h and up, the CPU gets stopped if (and as long as) the FIFO is full.

## GXFIFO DMA Overkill on Packed Commands Without Parameters

基本的に、GXFIFO DMAでは、112ワードの制限によりFIFO（256エントリ）が一杯になることはありませんが、多くの「パラメータなしのパックされたコマンド」（例：`MTX_PUSH`、`MTX_IDENTITY`、または`END_VTXS`）を送信するには、この上限は無駄に多すぎます。

例えば、`0x00151515`のような112ワードのパックされたコマンドをGXFIFOに送信すると、FIFOに336ワードのCmd(`0x15`)が書き込まれるため、FIFOがいっぱいになってしまいます。

つまり、パラメータなしのコマンドは、"密度"(データあたりのコマンド数)が高いので、FIFOが満杯になる可能性があります。

112ワードのDMA転送中、45ワード転送したところでFIFOが満杯になると、ジオメトリコマンドが処理されFIFOに(残った67ワード分の)十分な空きができるまで、DMA（およびCPU）が一時停止してしまいます。

Not sure if there’s much chance to get Overkills in practice. Normally most commands DO have parameters, and so, usually even LESS than 112 FIFO entries are occupied (since 8bit commands with 32bit parameters are merged into single 40bit FIFO entries).

