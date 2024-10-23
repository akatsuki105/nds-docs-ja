# テクスチャ座標

テクスチャ付きポリゴンでは、ポリゴンの各頂点にテクスチャ座標を関連付ける必要があります。

座標`(S,T)`は、`TEXCOORD`コマンド（通常、各VTXコマンドの前に発行される）によって定義され、オプションとして、`TEXIMAGE_PARAM`レジスタのbit30-31で設定した変換モードによって自動的に変換することが可能です。

## テクスチャ行列

テクスチャ行列は4x4で、値`m[0..15]`ですが、実際に使用されるのはこの行列の左2列だけです。

モード2とモード3では、テクスチャ行列の最下行は、最新の`TEXCOORD`コマンドからの S と T 値で置き換えられます。

## モード0: テクスチャ座標の変換なし

テクスチャ座標には`TEXCOORD`コマンドで指定された値がそのまま使用されます。

```
  ( S' T' )  =  ( S  T )
```

テクスチャ行列は無視されます。

## モード1: TexCoord source

テクスチャ座標には`TEXCOORD`コマンドで指定された値にテクスチャ行列による変換が適用された値が使用されます。

```
                                     | m[0]  m[1]  |
  ( S' T' )  =  ( S  T 1/16 1/16 ) * | m[4]  m[5]  |
                                     | m[8]  m[9]  |
                                     | m[12] m[13] |
```

テクスチャ行列として、平行移動、回転、または拡大縮小を設定することによって、単純なテクスチャのスクロール、回転、またはスケーリングを生成するために使用できます。

## モード2: Normal source

テクスチャ座標は`NORMAL`コマンドが実行されたときに計算されます。

```
                                     | m[0]  m[1]  |
  ( S' T' )  =  ( Nx  Ny  Nz 1.0 ) * | m[4]  m[5]  |
                                     | m[8]  m[9]  |
                                     | S     T     |
```

Can be used to produce spherical reflection mapping by setting the texture matrix to the current directional vector matrix, multiplied by a scaling matrix that expands the directional vector space from -1.0..+1.0 to one half of the texture size. For that purpose, translate the origin of the texture coordinate to the center of the spherical texture by using TexCoord command (spherical texture means a bitmap that contains some circle-shaped image).

## モード3: Vertex source

テクスチャ座標は何らかの`VTX`コマンドが実行されたときに計算されます。

```
                                     | m[0]  m[1]  |
  ( S' T' )  =  ( Vx  Vy  Vz 1.0 ) * | m[4]  m[5]  |
                                     | m[8]  m[9]  |
                                     | S     T     |
```

Can be used to produce texture scrolls dependent on the View coordinates by copying the current position coordinate matrix into the texture matrix. For example, the PositionMatrix can be obtained via CLIPMTX_RESULT (see there for details), and that values can be then manually copied to the TextureMatrix.

## Sign+Integer+Fractional Parts used in above Formulas

```
  Matrix    m[..]     1+19+12 (32bit)
  Vertex    Vx,Vy,Vz  1+3+12  (16bit)
  Normal    Nx,Ny,Nz  1+0+9   (10bit)
  Constant  1.0       0+1+0   (1bit)
  Constant  1/16      0+0+4   (4bit)
  TexCoord  S,T       1+11+4  (16bit)
  Result    S',T'     1+11+4  (16bit) <-------- clipped to that size !
```

Observe that the S’,T’ values are clipped to 16bit size. Ie. after the Vector*Matrix calaction, the result is shifted right (to make it having a 4bit fraction), and the value is then masked to 16bit size.


## 4000488h - Cmd 22h - TEXCOORD - Set Texture Coordinates (W)

Specifies the texture source coordinates within the texture bitmap which are to be associated with the next vertex.

```
  0-15:  S-Coordinate (X-Coordinate in Texture Source)
  16-31: T-Coordinate (Y-Coordinate in Texture Source)
  Both values are 1bit sign + 11bit integer + 4bit fractional part.
  A value of 1.0 (=1 SHL 4) equals to one Texel.
```

With Position 0.0 , 0.0 drawing starts from upperleft of the Texture.

With positive offsets, drawing origin starts more “within” the texture.

With negative offsets, drawing starts “before” the texture.

“When texture mapping, the Geometry Engine works faster if you issue commands in the order TexCoord -> Normal -> Vertex.”
