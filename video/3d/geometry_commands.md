# ジオメトリコマンド

## ジオメトリコマンド

テーブルは、ポートアドレス、コマンドID、パラメータ数、クロックサイクルを表しています。

```
  Address  Cmd Pa.Cy.
  N/A      00h -  -   NOP - No Operation (for padding packed GXFIFO commands)
  4000440h 10h 1  1   MTX_MODE - Set Matrix Mode (W)
  4000444h 11h -  17  MTX_PUSH - Push Current Matrix on Stack (W)
  4000448h 12h 1  36  MTX_POP - Pop Current Matrix from Stack (W)
  400044Ch 13h 1  17  MTX_STORE - Store Current Matrix on Stack (W)
  4000450h 14h 1  36  MTX_RESTORE - Restore Current Matrix from Stack (W)
  4000454h 15h -  19  MTX_IDENTITY - Load Unit Matrix to Current Matrix (W)
  4000458h 16h 16 34  MTX_LOAD_4x4 - Load 4x4 Matrix to Current Matrix (W)
  400045Ch 17h 12 30  MTX_LOAD_4x3 - Load 4x3 Matrix to Current Matrix (W)
  4000460h 18h 16 35* MTX_MULT_4x4 - Multiply Current Matrix by 4x4 Matrix (W)
  4000464h 19h 12 31* MTX_MULT_4x3 - Multiply Current Matrix by 4x3 Matrix (W)
  4000468h 1Ah 9  28* MTX_MULT_3x3 - Multiply Current Matrix by 3x3 Matrix (W)
  400046Ch 1Bh 3  22  MTX_SCALE - Multiply Current Matrix by Scale Matrix (W)
  4000470h 1Ch 3  22* MTX_TRANS - Mult. Curr. Matrix by Translation Matrix (W)
  4000480h 20h 1  1   COLOR - Directly Set Vertex Color (W)
  4000484h 21h 1  9*  NORMAL - Set Normal Vector (W)
  4000488h 22h 1  1   TEXCOORD - Set Texture Coordinates (W)
  400048Ch 23h 2  9   VTX_16 - Set Vertex XYZ Coordinates (W)
  4000490h 24h 1  8   VTX_10 - Set Vertex XYZ Coordinates (W)
  4000494h 25h 1  8   VTX_XY - Set Vertex XY Coordinates (W)
  4000498h 26h 1  8   VTX_XZ - Set Vertex XZ Coordinates (W)
  400049Ch 27h 1  8   VTX_YZ - Set Vertex YZ Coordinates (W)
  40004A0h 28h 1  8   VTX_DIFF - Set Relative Vertex Coordinates (W)
  40004A4h 29h 1  1   POLYGON_ATTR - Set Polygon Attributes (W)
  40004A8h 2Ah 1  1   TEXIMAGE_PARAM - Set Texture Parameters (W)
  40004ACh 2Bh 1  1   PLTT_BASE - Set Texture Palette Base Address (W)
  40004C0h 30h 1  4   DIF_AMB - MaterialColor0 - Diffuse/Ambient Reflect. (W)
  40004C4h 31h 1  4   SPE_EMI - MaterialColor1 - Specular Ref. & Emission (W)
  40004C8h 32h 1  6   LIGHT_VECTOR - Set Light's Directional Vector (W)
  40004CCh 33h 1  1   LIGHT_COLOR - Set Light Color (W)
  40004D0h 34h 32 32  SHININESS - Specular Reflection Shininess Table (W)
  4000500h 40h 1  1   BEGIN_VTXS - Start of Vertex List (W)
  4000504h 41h -  1   END_VTXS - End of Vertex List (W)
  4000540h 50h 1  392 SWAP_BUFFERS - Swap Rendering Engine Buffer (W)
  4000580h 60h 1  1   VIEWPORT - Set Viewport (W)
  40005C0h 70h 3  103 BOX_TEST - Test if Cuboid Sits inside View Volume (W)
  40005C4h 71h 2  9   POS_TEST - Set Position Coordinates for Test (W)
  40005C8h 72h 1  5   VEC_TEST - Set Directional Vector for Test (W)
```

サイクルタイミングはすべて33.51MHz単位でカウントされます。

NORMAL commands takes 9..12 cycles, depending on the number of enabled lights in PolyAttr (Huh, 9..12 (four timings) cycles for 0..4 (five settings) lights?) Total execution time of SwapBuffers is Duration until VBlank, plus 392 cycles.

`MTX_MODE=2`のとき、`MTX_MULT/TRANS`には追加で30サイクルかかります。

## 0x0400_0400 - GXFIFO - ジオメトリコマンドFIFO (W) (mirrored up to 400043Fh?

このレジスタに書き込むことで、(圧縮 or 非圧縮の) ジオメトリコマンド と、そのパラメータをジオメトリエンジンに送信します。

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

## FIFO / PIPE Number of Entries

The FIFO has 256 entries, additionally, there is a PIPE with four entries (giving a total of 260 entries). If the FIFO is empty, and if the PIPE isn’t full, then data is moved directly into the PIPE, otherwise it is moved into the FIFO. If the PIPE runs half empty (less than 3 entries) then 2 entries are moved from the FIFO to the PIPE. The state of the FIFO can be obtained in GXSTAT.Bit16-26, observe that there may be still data in the PIPE, even if the FIFO is empty. Check the busy flag in GXSTAT.Bit27 to see if the PIPE or FIFO contains data (or if a command is still executing).

Each PIPE/FIFO entry consists of 40bits of data (8bit command code, plus 32bit parameter value). Commands without parameters occupy 1 entry, and Commands with N parameters occupy N entries.

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

## GXFIFO DMA Overkill on Packed Commands Without Parameters

Normally, the 112 word limit ensures that the FIFO (256 entries) doesn’t get full, however, this limit is much too high for sending a lot of “Packed Commands Without Parameters” (ie. PUSH, IDENTITY, or END) - eg. sending 112 x Packed(00151515h) to GXFIFO would write 336 x Cmd(15h) to the FIFO, which is causing the FIFO to get full, and which is causing the DMA (and CPU) to be paused (for several seconds, in WORST case) until enough FIFO commands have been processed to allow the DMA to finish the 112 word transfer.

Not sure if there’s much chance to get Overkills in practice. Normally most commands DO have parameters, and so, usually even LESS than 112 FIFO entries are occupied (since 8bit commands with 32bit parameters are merged into single 40bit FIFO entries).

## Invalid GX commands

Invalid commands (anything else than 10h..1Ch, 20h..2Bh, 30h..33h, 40h..41h, 50h, 60h, or 70h..72h) seem to be simply ignored by the hardware (at least, testing has confirmed that they do not fetch any parameters from the gxfifo).
