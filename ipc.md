# [IPC](https://mgba-emu.github.io/gbatek/#dsinterprocesscommunicationipc)

DSには、ARM7とARM9のCPU間でステータス情報を交換できるIPC機能が備わっています。

IPC関連のレジスタは、両方のCPUから（アクセス違反することなく、どちら側でもウェイトステートなしで）同時にアクセスすることができます。

## 0x0400_0180 - NDS9/NDS7 - IPCSYNC - IPC同期レジスタ (R/W)

```
  Bit   Dir  Expl.
  0-3   R    Data input from IPCSYNC Bit8-11 of remote CPU (00h..0Fh)
  4-7   -    不使用
  8-11  R/W  Data output to IPCSYNC Bit0-3 of remote CPU   (00h..0Fh)
  12    -    不使用
  13    W    Send IRQ to remote CPU      (0=None, 1=Send IRQ)
  14    R/W  Enable IRQ from remote CPU  (0=Disable, 1=Enable)
  15-31 -    不使用
```

## 0x0400_0184 - NDS9/NDS7 - IPCFIFOCNT - IPCFIFO制御レジスタ (R/W)

```
  Bit   Dir  Expl.
  0     R    SendFIFO が空か   (0=空でない, 1=空)
  1     R    SendFIFO が満杯か (0=満杯でない, 1=満杯)
  2     R/W  Send Fifo Empty IRQ         (0=Disable, 1=Enable)
  3     W    Send Fifo Clear             (0=Nothing, 1=Flush Send Fifo)
  4-7   -    不使用
  8     R    ReceiveFIFO が空か   (0=空でない, 1=空)
  9     R    ReceiveFIFO が満杯か (0=満杯でない, 1=満杯)
  10    R/W  Receive Fifo Not Empty IRQ  (0=Disable, 1=Enable)
  11-13 -    不使用
  14    R/W  Error, Read Empty/Send Full (0=No Error, 1=Error/Acknowledge)
  15    R/W  FIFO有効化    (0=無効, 1=有効)
               0のとき、IPCFIFOSENDへの書き込みは無視され(データはFIFOに保存されず、エラービットは1にならない)、
               IPCFIFORECV からの読み出しは最後に(有効時に)読み出したFIFOワードを返す(ただし、FIFOからワードは削除されない)
  16-31 -    不使用
```

## 0x0400_0188 - NDS9/NDS7 - IPCFIFOSEND - IPC SendFIFO (W)

```
Bit0-31  SendFIFO のデータ (max 16 words; 64bytes)
```

## 0x0410_0000 - NDS9/NDS7 - IPCFIFORECV - IPC ReceiveFIFO (R)

```
Bit0-31  ReceiveFIFO のデータ (max 16 words; 64bytes)
           空の場合にこのポートから読み出しを行った場合、エラービットをセットして、以下のいずれかを返す
           1. 直近に受信したワード（もしあれば）を返す
           2. データがなかった場合、または IPCFIFOCNT.3 によって FIFO がクリアされていた場合は、0 を返す
```

## IPCFIFO の注意点

IPCFIFO IRQ はエッジトリガです。

`IF.17`(`IPC Send FIFO Empty`) は `(IPCFIFOCNT.2 & IPCFIFOCNT.0)` が `0`から `1` に変化したときにセットされます。

`IF.18`(`IPC Recv FIFO Not Empty`) は `(IPCFIFOCNT.10 &^ IPCFIFOCNT.8)` が `0`から `1` に変化したときにセットされます。

エッジトリガなので、どちらも`1`のときに割り込みフラグのアクノリッジが可能です。

