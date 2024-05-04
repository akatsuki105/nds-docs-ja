# ジオメトリコマンド

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

## Sending Commands by Ports 4000440h..40005FFh

GXFIFO に書き込んでコマンドを送信する代わりに、4000440h以降のポートを使ってコマンドを送信することもできます。

例えば、GXFIFOにコマンド`0x15` を書き込む代わりに、`0x0400_0454`に(何らかの値を)書き込むことでジオメトリエンジンにコマンド`0x15`を送信できます。

For a command with N paramters: issue N writes to the port.

For a command without parameters: issue one dummy-write to the port.

That mechanism puts the 8bit command + 32bit parameter into the FIFO/PIPE.

If the FIFO is full, then a wait is generated until data is removed from the FIFO, ie. the STR opcode gets freezed, during the wait, the bus cannot be used even by DMA, interrupts, or by the NDS7 CPU.

## GXFIFO Access via DMA

Larger pre-calculated data blocks can be sent directly to the FIFO. This is usually done via DMA (use DMA in Geometry Command Mode, 32bit units, Dest=4000400h/fixed, Length=NumWords, Repeat=0). The timings are handled automatically, ie. the system (should) doesn’t freeze when the FIFO is full (see below Overkill note though). DMA starts when the FIFO becomes less than half full, the DMA does then write 112 words to the GXFIFO register (or less, if the remaining DMA transfer length gets zero).

## GXFIFO Access via STR,STRD,STM

If desired, STR,STRD,STM opcodes can be used to write to the FIFO.

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
