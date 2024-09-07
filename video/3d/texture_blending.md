# テクスチャのブレンド

ポリゴンのピクセルは、頂点カラーとテクスチャで色付けされます。

頂点カラーとテクスチャは、どちらか一方だけを選択することもできますが、両方をブレンドして使うこともできます。

頂点カラーだけを使う場合、`TEXIMAGE_PARAM.30-31`に`0`(テクスチャなし)をセットします。

テクスチャだけを使う場合は、`POLYGON_ATTR`のbit4-5(モード)に`0`(Modulation)をセットし、bit16-20(アルファ値)に`31`をセットします。さらに`COLOR`コマンドで頂点カラーとして`0x7FFF`(白)またはグレー値をセットします。(グレー値にするとテクスチャの明度が下がります)

## 頂点カラー (Rv,Gv,Bv,Av)

頂点カラー(Rv,Gv,Bv) は頂点ごとに `COLOR`, `NORMAL`, `DIF_AMB` コマンドで変更できます。

頂点と頂点の間のピクセルは、周囲の頂点の中間の値にシェーディングされます。

頂点アルファ（Av）は、ポリゴンごとに（`POLYGON_ATTR`コマンドで）変更できます。

## テクスチャカラー (Rt,Gt,Bt,At)

テクスチャの色(Rt,Gt,Bt)とアルファ値(At)は、テクスチャビットマップで定義されます。

アルファ値のないフォーマット(フォーマット2の4色カラーなど)では `At=31` とし、アルファ値が1bitのフォーマット(フォーマット7のダイレクトカラー)では `At= A ? 31 : 0` とします。

## Shading Table Colors (Rs,Gs,Bs)

トゥーンまたはハイライト シェーディングモードでは、頂点カラー(Rv)の赤成分をシェーディングテーブルのインデックスとして使用します。つまり、Rvはシェーディングテーブルからシェーディングカラー(Rs,Gs,Bs)を読み取るために使われ、頂点カラーの緑と青の成分(Gv,Bv)はこのモードでは使われません。頂点アルファ（Av）は使用され続けます。

ポリゴンモード2でシェーディングを使用する場合は、レジスタ`DISP3DCNT` でトゥーンシェーディングかハイライトシェーディングかを選択します。これは1フレームごとの選択なので、どちらか一方しか使用できません。

## モード (POLYGON_ATTR.4-5)

### モード0: Modulation (a.k.a. Multiplicative blending)

```c
  R = ((Rt+1)*(Rv+1)-1)/64;
  G = ((Gt+1)*(Gv+1)-1)/64;
  B = ((Bt+1)*(Bv+1)-1)/64;
  A = ((At+1)*(Av+1)-1)/64;
```

乗算の結果、色の強度は低下します。(両方の要素が63の場合を除く)

### モード1: デカール

> デカール(Decal): テクスチャを3Dオブジェクトに貼り付けること。水たまりや弾痕などの効果を表現するのに使われる。

```c
  R = (Rt*At + Rv*(63-At))/64;  // except, when At=0: R=Rv, when At=31: R=Rt
  G = (Gt*At + Gv*(63-At))/64;  // except, when At=0: G=Gv, when At=31: G=Gt
  B = (Bt*At + Bv*(63-At))/64;  // except, when At=0: B=Bv, when At=31: B=Bt
  A = Av;
```

テクスチャのアルファ値(At)は、テクスチャカラーと頂点カラーの比率として使用されます。(それ以外の用途では使用しません)

### モード2: シェーディング(トゥーン/ハイライト)

#### トゥーンシェーディング (DISP3DCNT.1 が 0 の場合)

頂点カラーの赤成分（Rv）は、トゥーンテーブルのインデックスとして使用されます。

```c
  R = ((Rt+1)*(Rs+1)-1)/64;   // Rs=ToonTableRed[Rv]
  G = ((Gt+1)*(Gs+1)-1)/64;   // Gs=ToonTableGreen[Rv]
  B = ((Bt+1)*(Bs+1)-1)/64;   // Bs=ToonTableBlue[Rv]
  A = ((At+1)*(Av+1)-1)/64;
```

頂点カラー(Rv,Gv,Bv)の代わりに、トゥーンテーブルの色(Rs,Gs,Bs)が使用される点以外は、モード0(モジュレーション)と同じです。

#### ハイライトシェーディング (DISP3DCNT.1 が 1 の場合)

```c
  R = ((Rt+1)*(Rs+1)-1)/64 + Rs; // 63より大きい場合は63になる
  G = ((Gt+1)*(Gs+1)-1)/64 + Gs; // 63より大きい場合は63になる
  B = ((Bt+1)*(Bs+1)-1)/64 + Bs; // 63より大きい場合は63になる
  A = ((At+1)*(Av+1)-1)/64;
```

基本的にはトゥーンシェーディングと同じですが、オフセットを追加されており、強度を増加させ、色相を変更する可能性があります。

上記の式は6ビットのRGBA値に対するもので、内部的に5ビットの値を次の式で6ビットに展開したものです： `IF X>0 THEN X=X*2+1.`

## Uni-Colored Textures

テクスチャは普通は画像データですが、場合によっては、単色で塗りつぶされたまっさらなテクスチャを使う方が効果的な場合があります。

Wire-frame polygons are always having Av=31, however, they can be made transparent by using Translucent Textures (ie. A5I3 or A3I5 formats) with At<31.

In Toon/Highlight shading modes, the Vertex Color is mis-used as table index, however, Toon/Highlight shading can be used on uni-colored textures, which is more or less the same as using Toon/Highlight shading on uni-colored Vertex-colors.

