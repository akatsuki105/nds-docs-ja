# 影ポリゴン

> [!NOTE]  
> Nintendo DSの影の描画は、ステンシルシャドウボリュームという手法を使用しています。  
> [こちらの記事](https://news.mynavi.jp/article/graphics-22/)が参考になります。

DSのライティング機能では、カメラに光を反射させることはできますが、他のポリゴンに光を反射させたり、影を落としたりすることはできません。固定された位置に影をつけるには、その形と位置を事前に計算し、影をつけるポリゴン(影ポリゴン)の頂点カラーを変更するのがベストでしょう。

Additionally, the Shadow Polygon feature can be used to create animated shadows, ie. moved objects and variable light sources.

## 影ポリゴン と シャドウボリューム

プログラム側はシャドウボリュームを定義する必要があり、ハードウェアはその領域内にあるxyz座標を持つすべてのピクセルに自動的に影を描画します。

The Shadow Volume must be defined by several Shadow Polygons which are enclosing the shaded region. The ‘top’ of the shadow volume should be usually translated to the position of the object that casts the shadow, if the light direction changes then the shadow volume should be also rotated to match the light direction. The ‘length’ of the shadow volume should be (at least) long enough to reach from the object to the walls/floor where the shadow is to be drawn. The shadow volume must be passed TWICE to the hardware:

## Step 1 - Shadow Volume for Mask

`POLYGON_ATTR`コマンドで ポリゴン属性を `Mode=Shadow, PolygonID=00h, Back=Render, Front=Hide, Alpha=01h..1Eh` に設定し、シャドウボリューム (つまり、影ポリゴン) をジオメトリエンジンに渡します。

`Back=Render, Front=Hide` の設定では、このポリゴンの裏側が、(もちろん他のポリゴンの前にある範囲のみ)レンダリングされます。ただし、このポリゴンはフレームバッファには描画されず、代わりに、ステンシルバッファにフラグが設定されます。(ステップ2で使用)

> [!NOTE]  
> このステップ1で渡したポリゴンを、シャドウマスク と呼ぶこともあります。

## Step 2 - Shadow Volume for Rendering

ステップ1と同様に影ポリゴンをジオメトリエンジンに渡しますが、ポリゴン属性は `Mode=Shadow、PolygonID=01h..3Fh、Back=Render(?), Front=Render、Alpha=01h..1Eh` に設定します。

The Front=Render setting causes the ‘front-side’ of the shadow volume to be rendered, again, only as far as it is in front of other polygons. The Mode=Shadow / ID>00h setting causes the polygon to be drawn to the Color Buffer as usually, but only if the Stencil Buffer bits are zero (ie. the portion from Step 1 is excluded) (additionally, Step 2 resets the stencil bits after checking them). 

さらに、そのポリゴンID が属性バッファ(Attributeバッファ)のIDと異なる場合にのみ、影がフレームバッファにレンダリングされます。

## 影の透明度と色

ステップ1,2 での半透明設定(`Alpha=01h..1Eh`)により、影は通常の(不透明な)ポリゴンがレンダリングされた後に描画されます。

In Step 2 it does additionally specify the ‘intensity’ of the shadow. 

For normal shadows, the Vertex Color should be usually black, however, the shadow volume may be also used as ‘spotlight volume’ when using other colors.

## 描画順

ステップ1と2は必ずでこの順番に実行する必要があります。そのため直前の`SWAP_BUFFERS`コマンドでは自動ソートを無効にしておく必要があります。

影ポリゴンは、ターゲット(影の落ちる先)のポリゴンがレンダリングされた後にレンダリングする必要があります。不透明(opaque)なターゲットの場合、これは自動的に行われます。(自動ソートが無効でも、半透明ポリゴンは必ず不透明ポリゴンの後に描画されるため)

## ターゲットが半透明ポリゴンの場合

Casting shadows on Translucent Polygons. First draw the translucent target (with update depth buffer enabled, required for the shadow z-coordinates), then draw the Shadow Mask/Rendering volumes.

Due to the updated depth buffer the shadow will be cast only on the translucent target (not on any other polygons underneath of the translucent polygon). If you want the shadow to appear on both: Draw draw the Shadow Mask/Rendering volume TWICE (once before, and once after drawing the translucent target).

## ポリゴンID と フォグ

The “Render only if Polygon ID differs” feature (see Step 2) allows to prevent the shadow to be cast on the object that casts the shadow (ie. the object and shadow should have the same IDs). The feature also allows to select whether overlapping shadows (with same/different IDs) are shaded once or twice.

The old Fog Enable flag in the Attribute Buffer is ANDed with the Fog Enable flag of the Shadow Polygons, this allows to exclude Fog in shaded regions.

## Shadow Volume Open/Closed Shapes

Normally, the shadow volume should have a closed shape, ie. should have rear-sides (step 1), and corresponding front-sides (step 2) for all possible viewing angles. That is required for the shadow to be drawn correctly, and also for the Stencil Buffer to be reset to zero (in step 2, so that the stencil bits won’t disturb other shadow volumes).

Due to that, drawing errors may occur if the shadow volume’s front or rear side gets clipped by near/far clip plane.

One exception is that the volume doesn’t need a bottom-side (with a suitable volume length, the bottom may be left open, since it vanishes in the floor/walls anyways).

