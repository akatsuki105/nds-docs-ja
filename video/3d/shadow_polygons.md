# Shadow Polygons

DSのライティング機能では、カメラに光を反射させることはできますが、他のポリゴンに光を反射させたり、影を落としたりすることはできません。固定された位置に影をつけるには、その形と位置を事前に計算し、影をつけるポリゴンの頂点カラーを変更するのがベストでしょう。

Additionally, the Shadow Polygon feature can be used to create animated shadows, ie. moved objects and variable light sources.

## Shadow Polygons and Shadow Volume

プログラム側はシャドウボリューム（光を含まない領域）を定義する必要があり、ハードウェアはその領域内にあるxyz座標を持つすべてのピクセルに自動的にシャドーを描画します。

The Shadow Volume must be defined by several Shadow Polygons which are enclosing the shaded region. The ‘top’ of the shadow volume should be usually translated to the position of the object that casts the shadow, if the light direction changes then the shadow volume should be also rotated to match the light direction. The ‘length’ of the shadow volume should be (at least) long enough to reach from the object to the walls/floor where the shadow is to be drawn. The shadow volume must be passed TWICE to the hardware:

## Step 1 - Shadow Volume for Mask

Set Polygon_Attr Mode=Shadow, PolygonID=00h, Back=Render, Front=Hide, Alpha=01h..1Eh, and pass the shadow volume (ie. the shadow polygons) to the geometry engine.

The Back=Render / Front=Hide setting causes the ‘rear-side’ of the shadow volume to be rendered, of course only as far as it is in front of other polygons. The Mode=Shadow / ID=00h setting causes the polygon NOT to be drawn to the Color Buffer - instead, flags are set in the Stencil Buffer (to be used in Step 2).

## Step 2 - Shadow Volume for Rendering

Simply repeat step 1, but with Polygon_Attr Mode=Shadow, PolygonID=01h..3Fh, Back=Render(what/why?), Front=Render, Alpha=01h..1Eh.

The Front=Render setting causes the ‘front-side’ of the shadow volume to be rendered, again, only as far as it is in front of other polygons. The Mode=Shadow / ID>00h setting causes the polygon to be drawn to the Color Buffer as usually, but only if the Stencil Buffer bits are zero (ie. the portion from Step 1 is excluded) (additionally, Step 2 resets the stencil bits after checking them). Moreover, the shadow is rendered only if its Polygon ID differs from the ID in the Attribute Buffer.

## Shadow Alpha and Shadow Color

The Alpha=Translucent setting in Step 1 and 2 ensures that the Shadow is drawn AFTER the normal (opaque) polygons have been rendered. In Step 2 it does additionally specify the ‘intensity’ of the shadow. For normal shadows, the Vertex Color should be usually black, however, the shadow volume may be also used as ‘spotlight volume’ when using other colors.

## Rendering Order

The Mask Volume must be rendered prior to the Rendering Volume, ie. Step 1 and 2 must be performed in that order, and, to keep that order intact, Auto-sorting must have been disabled in the previous Swap_Buffers command.

The shadow volume must be rendered after the ‘target’ polygons have been rendered, for opaque targets this is done automatically (due to the translucent alpha setting; translucent polygons are always rendered last, even with auto-sort disabled).

## Translucent Targets

Casting shadows on Translucent Polygons. First draw the translucent target (with update depth buffer enabled, required for the shadow z-coordinates), then draw the Shadow Mask/Rendering volumes.

Due to the updated depth buffer the shadow will be cast only on the translucent target (not on any other polygons underneath of the translucent polygon). If you want the shadow to appear on both: Draw draw the Shadow Mask/Rendering volume TWICE (once before, and once after drawing the translucent target).

## Polygon ID and Fog Enable

The “Render only if Polygon ID differs” feature (see Step 2) allows to prevent the shadow to be cast on the object that casts the shadow (ie. the object and shadow should have the same IDs). The feature also allows to select whether overlapping shadows (with same/different IDs) are shaded once or twice.

The old Fog Enable flag in the Attribute Buffer is ANDed with the Fog Enable flag of the Shadow Polygons, this allows to exclude Fog in shaded regions.

## Shadow Volume Open/Closed Shapes

Normally, the shadow volume should have a closed shape, ie. should have rear-sides (step 1), and corresponding front-sides (step 2) for all possible viewing angles. That is required for the shadow to be drawn correctly, and also for the Stencil Buffer to be reset to zero (in step 2, so that the stencil bits won’t disturb other shadow volumes).

Due to that, drawing errors may occur if the shadow volume’s front or rear side gets clipped by near/far clip plane.

One exception is that the volume doesn’t need a bottom-side (with a suitable volume length, the bottom may be left open, since it vanishes in the floor/walls anyways).

