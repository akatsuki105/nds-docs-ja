# [拡張パレット](https://mgba-emu.github.io/gbatek/#dsvideoextendedpalettes)

拡張パレットを割り当てた場合、割り当てたメモリはCPUバスにマッピングされないため、CPUは一時的に割り当てを解除するときのみ拡張パレットにアクセスできます。

すべての標準/拡張パレットのカラー0は透明で、BG標準パレット0のカラー0は背景(Backdrop)色として使用されます。拡張パレットのメモリはVRAMに割り当てる必要があります。

BGの拡張パレットは、DISPCNT.30 で有効化できます。

```
 standard palette --> 16-color tiles (with 16bit bgmap entries) (text)
                      256-color tiles (with 8bit bgmap entries) (rot/scal)
                      256-color bitmaps
                      backdrop-color (color 0)
 extended palette --> 256-color tiles (with 16bit bgmap entries)(text,rot/scal)
```

割り当てられたVRAMは 4つの8KBスロット(合計32KB) に分割され、通常はBG0～3がスロット0～3を使用しますが、BG0とBG1は任意でBG0CNTとBG1CNTを介して BG0=スロット2、BG1=スロット3 に変更できます。

OBJの拡張パレットは、DISPCNT.31 で有効化できます。

```
 standard palette --> 16 colors x 16 palettes (256色)
 extended palette --> 256 colors x 16 palettes (4096色)
```

OBJの拡張パレットは、VRAMのバンク F、G、I（全部16KB）に割り当てられなければならないですが、そのうち最初の8KBだけが拡張パレットに使用されます。(RGB555は2バイトなので、4096色で8KB)


