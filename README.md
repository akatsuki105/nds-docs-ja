# nds-docs-ja

Nintendo DSについて、技術的な詳細を日本語でまとめたものです。

突然消えたり非公開にする可能性もあるので心配な方はクローンしておくことをお勧めします。

## コンテンツ一覧

### NDS

- [仕様](spec.md)
- [メモリマップ](memory.md)
- [IOレジスタ](io.md)
- [メモリアクセス時間](memory_timings.md)

### メモリ制御

- [CacheとTCM](./memctl/cache_tcm.md)
- [カートリッジとMain RAM](./memctl/cart_mainram.md)
- [WRAM](./memctl/wram.md)
- [VRAM](./memctl/vram.md)
- [BIOS](./memctl/bios.md)

### グラフィック

NDSは2つの2Dビデオエンジンを備えていて、両方とも基本的にGBAのものと同じです。

詳細は、[GBAのグラフィックについての解説](https://github.com/pokemium/gba-docs-ja#グラフィック)を参照してください。

**NDS Specific 2D Video Features**

- [概要](./video/stuff.md)
- [BGMode と BG制御](./video/bg_ctl.md)
- [OBJ](./video/objs.md)
- [拡張パレット](./video/extended_palettes.md)
- [キャプチャ](./video/capture.md)
- [見取り図](./video/block_diagram.md)

**NDS/DSi File Formats for 2D video**

- [2Dビデオファイル](./video/files_2d.md)

Display Power Control（およびDisplay Swap）、VRAMのアロケーションについては、こちらをご覧ください。

- [電源制御](./system/power_control.md)
- [電源管理装置](./system/power_management_device.md)
- [メモリ制御 - VRAM](./memctl/vram.md)

## 参考記事

- [GBATEK](https://problemkaputt.de/gbatek.htm)

