# IOレジスタ

```
  Address  Siz Name            Expl.

  Rendering Engine (per Frame settings)
  4000060h 2   DISP3DCNT       3D Display Control Register (R/W)
  4000320h 1   RDLINES_COUNT   Rendered Line Count Register (R)
  4000330h 10h EDGE_COLOR      Edge Colors 0..7 (W)
  4000340h 1   ALPHA_TEST_REF  アルファテスト閾値 (W)
  4000350h 4   CLEAR_COLOR     Clear Color Attribute Register (W)
  4000354h 2   CLEAR_DEPTH     Clear Depth Register (W)
  4000356h 2   CLRIMAGE_OFFSET Rear-plane Bitmap Scroll Offsets (W)
  4000358h 4   FOG_COLOR       Fog Color (W)
  400035Ch 2   FOG_OFFSET      Fog Depth Offset (W)
  4000360h 20h FOG_TABLE       Fog Density Table, 32 entries (W)
  4000380h 40h TOON_TABLE      Toon Table, 32 colors (W)

  Geometry Engine (per Polygon/Vertex settings)
  4000400h 40h GXFIFO          ジオメトリコマンドFIFO (W)
  4000440h ... ...             ジオメトリコマンド送信レジスタ (W)
  4000600h 4   GXSTAT          Geometry Engine Status Register (R and R/W)
  4000604h 4   RAM_COUNT       Polygon List & Vertex RAM Count Register (R)
  4000610h 2   DISP_1DOT_DEPTH 1-Dot Polygon Display Boundary Depth (W)
  4000620h 10h POS_RESULT      Position Test Results (R)
  4000630h 6   VEC_RESULT      Vector Test Results (R)
  4000640h 40h CLIPMTX_RESULT  Read Current Clip Coordinates Matrix (R)
  4000680h 24h VECMTX_RESULT   Read Current Directional Vector Matrix (R)
```
