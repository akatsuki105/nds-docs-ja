# Rear-Plane

他のドキュメントでは、これをリアプレーン（Rear-plane）ではなくクリアプレーン（Clear-plane）と呼んでいるようです。(プレーンは画像であることもあるので、常にクリアとは限らない)

The view order is as such:

```
  --> 2D Layers --> 3D Polygons --> 3D Rear-plane --> 2D Layers --> 2D Backdrop
```

The rear-plane can be disabled (by making it transparent; alpha=0), so that the 2D layers become visible as background.

2D layers can be moved in front of, or behind the 3D layer-group (which is represented as BG0 to the 2D Engine), 2D layers behind BG0 can be used instead of, or additionally to the rear-plane.

The rear-plane can be initialized via below two registers (so all pixels in the plane have the same colors and attributes), this method is used when DISP3DCNT.14 is zero:

## 4000350h - CLEAR_COLOR - Clear Color Attribute Register (W)

```
  0-4    Clear Color, Red
  5-9    Clear Color, Green
  10-14  Clear Color, Blue
  15     Fog (enables Fog to the rear-plane) (doesn't affect Fog of polygons)
  16-20  Alpha
  21-23  不使用
  24-29  Clear Polygon ID (affects edge-marking, at the screen-edges?)
  30-31  不使用
```

## 4000354h - CLEAR_DEPTH - Clear Depth Register (W)

```
  0-14   Clear Depth (0..7FFFh) (usually 7FFFh = most distant)
  15     不使用
  16-31  See Port 4000356h, CLRIMAGE_OFFSET
```

The 15bit Depth is expanded to 24bit as “X=(X*200h)+((X+1)/8000h)*1FFh”.

## Rear Color/Depth Bitmaps

Alternately, the rear-plane can be initialized by bitmap data (allowing to assign different colors & attributes to each pixel), this method is used when DISP3DCNT.14 is set:

Consists of two bitmaps (one with color data, one with depth data), each containing 256x256 16bit entries, and so, each occupying a whole 128K slot,

```
  Rear Color Bitmap (located in Texture Slot 2)
    0-4    Clear Color, Red
    5-9    Clear Color, Green
    10-14  Clear Color, Blue
    15     Alpha (0=Transparent, 1=Solid) (equivalent to 5bit-alpha 0 and 31)
  Rear Depth Bitmap (located in Texture Slot 3)
    0-14   Clear Depth, expanded to 24bit as X=(X*200h)+((X+1)/8000h)*1FFh
    15     Clear Fog (Initial fog enable value)
```

This method requires VRAM to be allocated to Texture Slot 2 and 3 (see Memory Control chapter). Of course, in that case the VRAM is used as Rear-plane, and cannot be used for Textures.

The bitmap method is restricted to 1bit alpha values (the register-method allows to use a 5bit alpha value).

The Clear Polygon ID is kept defined in the CLEAR_COLOR register, even in bitmap mode.

## 4000356h - CLRIMAGE_OFFSET - Rear-plane Bitmap Scroll Offsets (W)

The visible portion of the bitmap is 256x192 pixels (regardless of the viewport setting, which is used only for polygon clipping). Internally, the bitmap is 256x256 pixels, so the bottom-most 64 rows are usually offscreen, unless scrolling is used to move them into view.

```
  Bit0-7   X-Offset (0..255; 0=upper row of bitmap)
  Bit8-14  Y-Offset (0..255; 0=left column of bitmap)
```

The bitmap wraps to the upper/left edges when exceeding the lower/right edges.
