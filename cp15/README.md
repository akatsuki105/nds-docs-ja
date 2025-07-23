# CP15

## コプロセッサについて

ARMアーキテクチャは仕様として、16個までのコプロセッサをサポートしています。このコプロセッサは、CP0からCP15までの番号が割り当てられています。

CP0..9, CP12, CP13 は、ARMアーキテクチャの標準仕様ではなく、プロセッサの製造元がユースケースに合わせて好きな機能を持たせることができます。(ARMアーキテクチャはMCR,MRC命令というコプロセッサとやりとりする"インターフェース"を定めてあるだけ)

ただし、CP10, CP11, CP14, CP15はARMアーキテクチャの標準仕様として定義されています。つまり、プロセッサの製造元はCP10やCP15にはARMアーキテクチャが定めた機能を持たせる必要があります。

これら、予約されたコプロセッサの機能は次のようになっています。

```
  CP10: VFPユニット(単精度)の制御を行うコプロセッサ
  CP11: VFPユニット(倍精度)の制御を行うコプロセッサ
  CP14: デバッグ用のコプロセッサ
  CP15: システム制御用のコプロセッサ(キャッシュ制御、MMU制御などのハードウェア全般に関する制御)
```

これらのコプロセッサの提供する機能を使わない場合は、これらのコプロセッサは存在しなくても問題ないようです。

現に、ARM7にはTCMやキャッシュといったCP15で制御する必要がある機能が搭載されていないため、ARM7にはCP15が搭載されていません。

## CP15

上で述べたように、CP15はシステム制御用のコプロセッサで、NDS9(というかARM9)では、TCMやキャッシュの制御を行うために利用されています。

CP15自体が制御用レジスタを持っていて、CPUからこれに値を書き込むことでCP15の動作を制御します。

CP15の役割にはMMUの制御があります。MMUの機能は アドレス変換 と メモリ保護 ですが、NDS9のMMUはアドレス変換機能は持たず、メモリ保護機能のみを持っています。
そのため、NDS9では MMU は `MPU(Protection Unit)` と呼ばれています。(`PU(Protection Unit)` とも呼ばれることも)

またCP15では、キャッシュをデータと命令で分離するか、ユニファイドキャッシュにするかを設定できますが、NDS9ではキャッシュを必ずデータと命令で分離します。ちなみにユニファイドキャッシュとして利用する場合は、データキャッシュの設定をすることでユニファイドキャッシュとして利用できます。(命令キャッシュへの設定は無視されます)

## CP15 オペコード

CP15にはMCRとMRCオペコードを用いてアクセス可能です。その際に、`Pn=P15, <cpopc>=0`としてやる必要があります。

```
  MCR{cond} P15,<cpopc>,Rd,Cn,Cm,<cp>   ; ARMレジスタRdからCP15にデータを転送
  MRC{cond} P15,<cpopc>,Rd,Cn,Cm,<cp>   ; CP15からARMレジスタRdにデータを転送
```

RdはARMレジスタを指しており、R0-R14から選択可能です。R15はP15に対するMCR/MRCでは利用すべきではありません。

\<cpopc\>,Cn,Cm,\<cp\>はCP15のレジスタを指定するのに利用します。

例えば、`<cpopc>,Cn,Cm,<cp> = [0, C0, C0, 0]`とするとMain IDレジスタを指定します。

コプロセッサ用の他のオペコード(CDP, LDC, STC)はP15に対しては利用できません。

## CP15 Register List

```
0,C0,C0,0     :  Main ID Register (R)
0,C0,C0,1     :  Cache Type and Size (R)
0,C0,C0,2     :  TCM Physical Size (R)
0,C0,C0,3     :  ARM11: TLB Type Register
0,C0,C1,0     :  ARM11: Processor Feature Register 0
0,C0,C1,1     :  ARM11: Processor Feature Register 1
0,C0,C1,2     :  ARM11: Debug Feature Register 0
0,C0,C1,3     :  ARM11: Auxiliary Feature Register 0
0,C0,C1,4     :  ARM11: Memory Model Feature Register 0
0,C0,C1,5     :  ARM11: Memory Model Feature Register 1
0,C0,C1,6     :  ARM11: Memory Model Feature Register 2
0,C0,C1,7     :  ARM11: Memory Model Feature Register 3
0,C0,C2,0     :  ARM11: Set Attributes Register 0
0,C0,C2,1     :  ARM11: Set Attributes Register 1
0,C0,C2,2     :  ARM11: Set Attributes Register 2
0,C0,C2,3     :  ARM11: Set Attributes Register 3
0,C0,C2,4     :  ARM11: Set Attributes Register 4
0,C0,C2,5     :  ARM11: Set Attributes Register 5

0,C1,C0,0     :  Control Register (R/W, or R=Fixed)
0,C1,C0,1     :  ARM11: Auxiliary Control Register
0,C1,C0,2     :  ARM11: Coprocessor Access Control Register
0,C2,C0,0     :  ARM11: Translation Table Base Register 0
0,C2,C0,1     :  ARM11: Translation Table Base Register 1
0,C2,C0,2     :  ARM11: Translation Table Base Control Register
0,C3,C0,0     :  ARM11: Domain Access Control Register
0,C5,C0,0     :  ARM11: Data Fault Status Register
0,C5,C0,1     :  ARM11: Instruction Fault Status Register
0,C6,C0,0     :  ARM11: Fault Address Register (FAR)
0,C6,C0,1     :  ARM11: Watchpoint Fault Address Register (WFAR)
0,C2,C0,0     :  MPU Cachability Bits for Data/Unified Protection Region
0,C2,C0,1     :  MPU Cachability Bits for Instruction Protection Region
0,C3,C0,0     :  MPU Cache Write-Bufferability Bits for Data Protection Regions
0,C5,C0,0     :  MPU Access Permission Data/Unified Protection Region
0,C5,C0,1     :  MPU Access Permission Instruction Protection Region
0,C5,C0,2     :  MPU Extended Access Permission Data/Unified Protection Region
0,C5,C0,3     :  MPU Extended Access Permission Instruction Protection Region
0,C6,C0..C7,0 :  MPU Protection Unit Data/Unified Region 0..7
0,C6,C0..C7,1 :  MPU Protection Unit Instruction Region 0..7
0,C7,Cm,Op2   :  Cache Commands and Halt Function (W)
0,C9,C0,0     :  Cache Data Lockdown
0,C9,C0,1     :  Cache Instruction Lockdown
0,C9,C1,0     :  TCM Data TCM Base and Virtual Size
0,C9,C1,1     :  TCM Instruction TCM Base and Virtual Size
0,C13,Cm,Op2  :  Misc Process ID registers
0,C15,Cm,Op2  :  Misc Implementation Defined and Test/Debug registers
```

## Reference

- [ARM946E-S Technical Reference Manual](https://developer.arm.com/documentation/ddi0201/d?lang=en)
- [Arm MPUに関するRA2シリーズ ユーザーズマニュアル記載変更](https://www.renesas.com/ja/document/tcu/correction-arm-mpu-descriptions-ra2-series-users-manual?srsltid=AfmBOoo9FccMV8HCSnWlEIR7xAnOErPz3bzmPGCgxNuSjcgRYpHoQRae)
