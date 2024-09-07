# ライティング

DSでは4つの平行光源(= Directional Light)が利用可能です。

The lighting operation is performed by executing the Normal command (which sets the VertexColor based on the Light/Material parameters) (to the rest of the hardware it doesn’t matter if the VertexColor was set by Color command or by Normal command). Light is calculated only for the Front side of the polygon (assuming that the Normal is matched to that side), so the Back side will be (incorrectly) using the same color.

## 40004C8h - Cmd 32h - LIGHT_VECTOR - 平行光源の向きベクトル (W)

パラメータのbit30-31で指定した番号の光源の向きベクトル(LightVector)を設定します。

```
  0-9   平行光源の向きベクトルのX成分 (1bit sign + 9bit fractional part)
  10-19 平行光源の向きベクトルのY成分 (1bit sign + 9bit fractional part)
  20-29 平行光源の向きベクトルのZ成分 (1bit sign + 9bit fractional part)
  30-31 光源番号                   (0..3)
```

このコマンドを実行すると、パラメータで渡されたベクトルに、現在の方向行列(= Directional Matrix)を掛けた結果が、LightVectorとして設定されます。 これにより光源の向きを回転させることができます。 ただし、方向行列(= Directional Matrix)の設定はこのコマンドを実行する前に行う必要があります。

## 40004CCh - Cmd 33h - LIGHT_COLOR - 光の色 (W)

パラメータのbit30-31で指定した番号の光源の色を指定します。

```
  0-4   赤        (0..31)
  5-9   緑        (0..31)
  10-14 青        (0..31)
  15-29 不使用
  30-31 光源番号 (0..3)
```

このコマンドで設定した光の色は`NORMAL`コマンドを実行するときに、拡散反射、鏡面反射、環境反射と組み合わせられ頂点カラーの計算に使用されます。

## 40004C0h - Cmd 30h - DIF_AMB - MaterialColor0 - 拡散反射,環境反射 (W)

拡散反射は ディフューズ(Diffuse) 、 環境反射は アンビエント(Ambient) とも呼ばれます。

```
  0-4   Diffuse Reflection Red     ;\light(s) that directly hits the polygon,
  5-9   Diffuse Reflection Green   ; ie. max when NormalVector has opposite
  10-14 Diffuse Reflection Blue    ;/direction of LightVector
  15    Set Vertex Color (0=No, 1=Set Diffuse Reflection Color as Vertex Color)
  16-20 Ambient Reflection Red     ;\light(s) that indirectly hits the polygon,
  21-25 Ambient Reflection Green   ; ie. assuming that light is reflected by
  26-30 Ambient Reflection Blue    ;/walls/floor, regardless of LightVector
  31    不使用
```

With Bit15 set, the lower 15bits are applied as VertexColor (exactly as when when executing the Color command), the purpose is to use it as default color (eg. when outcommenting the Normal command), normally, when using lighting, the color setting gets overwritten (as soon as executing the Normal command).

## 40004C4h - Cmd 31h - SPE_EMI - MaterialColor1 - 鏡面反射,エミッション (W)

鏡面反射は スペキュラ(Specular) 、 エミッションは 放射(Emission) とも呼ばれます。

```
  0-4   Specular Reflection Red    ;\light(s) reflected towards the camera,
  5-9   Specular Reflection Green  ; ie. max when NormalVector is in middle of
  10-14 Specular Reflection Blue   ;/LightVector and ViewDirection
  15    Specular Reflection Shininess Table (0=Disable, 1=Enable)
  16-20 Emission Red               ;\light emitted by the polygon itself,
  21-25 Emission Green             ; ie. regardless of light colors/vectors,
  26-30 Emission Blue              ;/and no matter if any lights are enabled
  31    不使用
```

Caution: Specular Reflection WON’T WORK when the ProjectionMatrix is rotated.

## 40004D0h - Cmd 34h - SHININESS - Specular Reflection Shininess Table (W)

Write 32 parameter words (each 32bit word containing four 8bit entries), entries 0..3 in the first word, through entries 124..127 in the last word:

```
  0-7   Shininess 0 (unsigned fixed-point, 0bit integer, 8bit fractional part)
  8-15  Shininess 1 ("")
  16-23 Shininess 2 ("")
  24-31 Shininess 3 ("")
```

If the table is disabled (by MaterialColor1.Bit15), then reflection will act as if the table would be filled with linear increasing numbers.

## 4000484h - Cmd 21h - NORMAL - 法線ベクトル設定 (W)

このコマンドは様々なライティングのパラメータに基づいて頂点カラーを計算します。

このコマンドを実行すると、入力されたベクトルは現在の方向行列(=Directional Matrix)と掛け合わされ、その結果を法線ベクトルとして適用します。(つまり法線ベクトルにも、次のポリゴンの頂点座標に使用される回転が適用されます)

```
  0-9   法線ベクトルのX成分 (1bit sign + 9bit fractional part)
  10-19 法線ベクトルのY成分 (1bit sign + 9bit fractional part)
  20-29 法線ベクトルのZ成分 (1bit sign + 9bit fractional part)
  30-31 不使用
```

このコマンドを実行すると、コマンドで渡した法線ベクトルと、すでに設定済みのライト/マテリアルパラメータに基づいて頂点カラーが更新されます。 コマンドの実行時間は、有効な光源の数によって異なります。

## Additional Light Registers

上記のレジスタに加えて、`POLYGON_ATTR`でライトを有効にする必要があります（`POLYGON_ATTR`の変更は次のBeginコマンドまで適用されないことに注意してください）。

また、`NORMAL`と`LIGHT_VECTOR`コマンドを実行する前に、方向行列(=Directional Matrix)が正しく設定されている必要があります（MtxMode=2）。

## 法線ベクトル

The Normal vector must point “away from the polygon surface” (eg. for the floor, the Normal should point upwards). That direction is implied by the polygon vertices, however, the hardware cannot automatically calculate it, so it must be set manually with the Normal command (prior to the VTX-commands).

When using lighting, the Normal command must be re-executed after switching Lighting on/off, or after changing light/material parameters. And, of course, also before defining polygons with different orientation. Polygons with same orientation (eg. horizontal polygon surfaces) and same material color can use the same Normal. Changing the Normal per polygon gives differently colored polygons with flat surfaces, changing the Normal per vertex gives the illusion of curved surfaces.

## Light Vector

Each light consists of parallel beams; similar to sunlight, which appears to us (due to the great distance) to consist of parallel beams, all emmitted into the same direction; towards Earth.

In reality, light is emitted into ALL directions, originated from the light source (eg. a candle), the hardware doesn’t support that type of non-parallel light. However, the light vectors can be changed per polygon, so a polygon that is located north of the light source may use different light direction than a polygon that is east of the light source.

言うまでもなく、光源0~3はそれぞれ異なる向きを持つことができます。

## ベクトルの正規化

法線ベクトル と 光源の向きベクトル は、長さが1.0になるように正規化されている必要があります。 (実際の場合、レジスタは小数部のみを持つため、1.0の長さはオーバーフローを引き起こす可能性があるため、0.99のような値が適しています)。

## Lighting Limitations

The functionality of the light feature is limited to reflecting light to the camera (light is not reflected to other polygons, nor does it cast shadows on other polygons). However, independently of the lighting feature, the DS hardware does allow to create shadows, see [Shadow Polygons](./shadow_polygons.md).

## ライティングの内部処理

`NORMAL`コマンドを実行すると、以下のような処理が行われます。

```c
  if (TexCoordTransformMode==2) TexCoord=NormalVector*Matrix; // see TexCoord

  NormalVector=NormalVector*DirectionalMatrix;
  VertexColor = EmissionColor;

  for (i = 0; i < 4; i++) { // for each light
    if (PolygonAttrLight[i]) {
      DiffuseLevel = max(0, -(LightVector[i]*NormalVector));
      ShininessLevel = max(0, (-HalfVector[i])*(NormalVector))^2;
      if (TableEnabled) ShininessLevel = ShininessTable[ShininessLevel];

      // NOTE: below processed separately for the R,G,B color components...
      VertexColor = VertexColor + (SpecularColor*LightColor[i]*ShininessLevel);
      VertexColor = VertexColor + (DiffuseColor*LightColor[i]*DiffuseLevel);
      VertexColor = VertexColor + (AmbientColor*LightColor[i]);
    }
  }
```

`LIGHT_VECTOR`コマンドを実行すると、以下のような処理が行われます。

```c
  LightVector[i] = (LightVector*DirectionalMatrix)
  HalfVector[i] = (LightVector[i]+LineOfSightVector)/2
```

## LineOfSightVector

### how it SHOULD work

Ideally, the LineOfSightVector should point from the camera to the vertic(es), however, the vertic(es) are still unknown at time of normal command, so it is just pointing from the camera to the screen, ie. `LineOfSightVector = (0,0,-1.0)`

Moreover, the LineOfSightVector should be multiplied by the Projection Matrix (so the vector would get rotated accordingly when the camera gets rotated), and, after multiplication by a scaled matrix, it’d be required to normalize the resulting vector.

### how it DOES actually work

However, the NDS cannot normalize vectors by hardware, and therefore, it does completely leave out the LineOfSightVector*ProjectionMatrix multiplication. So, the LineOfSightVector is always (0,0,-1.0), no matter of any camera rotation. That means `Specular Reflection WON'T WORK when the ProjectionMatrix is rotated (!)`.

So, if you want to rotate the “camera” (in MTX_MODE=0), then you must instead rotate the “world” in the opposite direction (in MTX_MODE=2).

That problem applies only to Specular Reflection, ie. only if Lighting is used, and only if the Specular Material Color is nonzero.

## Maths Notes

Note on `Vector*Vector` multiplication: Processed as `LineVector*RowVector`, so the result is a number (aka scalar) (aka a matrix with only 1x1 elements), multiplying two (normalized) vectors results in: `cos(angle)=vec1*vec2`, ie. the cosine of the angle between the two vectors.

光源関係のベクトル(Normal/Light/Half/Sight)は、全て3次元ベクトル(x,y,z)であり、4x4の方向行列(=Directional Matrix)との乗算時には、左上の3x3行列要素のみが使用されます。

## 参考

- [How lighting works - OpenGL Wiki](https://www.khronos.org/opengl/wiki/How_lighting_works)
