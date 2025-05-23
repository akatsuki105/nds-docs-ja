# NSBMD (3Dモデル)

拡張子: `.nsbmd` (他にも`.bmd`, `.bmd0` など)
マジックナンバー: `BMD0`

3Dモデルのファイルフォーマットです。基本的には、DSのGPUにコマンドを送信するためのスクリプトです。

```cpp
struct BMD0 {
  struct {
  	char signature[4]; // "BMD0"
  	u16 byteOrder; // 0xFEFF
  	u16 version;
  	u32 fileSize;
  	u16 headerSize; // 16
  	u16 dataBlocks; // 1 (MDL0 のみ) or 2 (MDL0 + TEX0)
    u32 blockOffsets[BLOCK_NUM]; // .dataBlocks 
  } Header;
};
```

TODO