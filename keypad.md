# キー入力

ここに書いていない`KEYINPUT` と `KEYCNT` はGBAと(アドレスも含めて)全く同じです。(NDS9とNDS7両方からアクセス可能)

## 0x0400_0136 - NDS7 - EXTKEYIN - Key X/Y Input (R)

```
  0      Button X     (0=Pressed, 1=Released)
  1      Button Y     (0=Pressed, 1=Released)
  3      DEBUG button (0=Pressed, 1=Released/None such)
  6      Pen down     (0=Pressed, 1=Released/Disabled) (always 0 in DSi mode)
  7      Hinge/folded (0=Open, 1=Closed)
  2,4,5  Unknown / set
  8..15  Unknown / zero
```

