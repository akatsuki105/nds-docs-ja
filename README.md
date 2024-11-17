# nds-docs-ja

Nintendo DS(NDS)について、技術的な詳細を日本語でまとめたものです。

突然消えたり非公開にする可能性もあるので心配な方はクローンしておくことをお勧めします。

> [!WARNING]
> このレポジトリは大半が執筆途中です。なので現在、ドキュメントとしての信頼性は皆無です。  
> また、コミット履歴は気まぐれで破壊されることがあります。

> [!NOTE]
> ドキュメント中の  
> &nbsp;&nbsp;NDS9 とは、NDSモードのARM9プロセッサとそのメモリおよびI/Oポートを意味します。  
> &nbsp;&nbsp;NDS7 とは、NDSモードのARM7プロセッサとそのメモリおよびI/Oポートを意味します。  
> &nbsp;&nbsp;GBA  とは、GBAモードのARM7プロセッサとそのメモリおよびI/Oポートを意味します。  

## コンテンツ一覧

### NDS

- [仕様](spec.md)
- [ハードウェア一覧](hardware.md)
- [メモリマップ](memory.md)
- [IOレジスタ](io.md)
- [サウンド](sound/README.md)
- [割り込み](interrupt.md)
- [IPC](ipc.md)
- [キー入力](keypad.md)
- [RTC](rtc.md)
- [タイマー](timer.md)
- [電源管理装置](power_management_device.md)

### カートリッジ

- [カートリッジヘッダ](cartridge/header.md)

### CPU

- [ARM7](https://github.com/akatsuki105/gba-docs-ja/tree/main/arm7tdmi)
- [ARM9](arm9.md)
- [CP15](cp15/README.md)
  - [ID Codes](cp15/id_codes.md)

### グラフィック

NDSは2つの2Dエンジンと1つの3Dエンジンを備えていて、2Dエンジンは(2つとも)基本的にGBAのものと同じです。

詳細は、[GBAのグラフィックについての解説](https://github.com/akatsuki105/gba-docs-ja#グラフィック)を参照してください。

- [VRAM](video/vram.md)

**2Dエンジン**

- [BGMode と BG制御](./video/bg_ctl.md)
- [OBJ](./video/objs.md)
- [拡張パレット](./video/extended_palettes.md)
- [キャプチャ](./video/capture.md)
- [見取り図](./video/block_diagram.md)
- [2Dビデオファイル](./video/files_2d.md)

**3Dエンジン**

- [3Dエンジンの概要](./video/3d/README.md)

[gba-docs-ja](https://github.com/akatsuki105/gba-docs-ja) を参照してください

## 関連するレポジトリ

- [gb-docs-ja](https://github.com/akatsuki105/gb-docs-ja): GameBoyについて
- [gba-docs-ja](https://github.com/akatsuki105/gba-docs-ja): GameBoy Advanceについて
- [snes-docs-ja](https://github.com/akatsuki105/snes-docs-ja): スーパーファミコンについて

## 参考記事

- [GBATEK](https://problemkaputt.de/gbatek.htm)
- [mgba-emu/gbatek](https://github.com/mgba-emu/gbatek)
- [Nintendo DS Architecture](https://www.copetti.org/writings/consoles/nintendo-ds)
