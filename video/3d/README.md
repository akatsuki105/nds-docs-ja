# [3Dエンジンの概要](https://mgba-emu.github.io/gbatek/#ds3doverview)

>**Warning**  
> 筆者(Akatsuki105)は3Dエンジンについての知識が乏しいため、3Dについてのページの信頼性は特に低いです。

3Dエンジンは、ジオメトリエンジン と レンダリングエンジン で構成されています。

## ジオメトリエンジン (頂点の座標計算、ポリゴンアトリビュートの割り当て)

>**Note**  
> ジオメトリ:   「形状」または「形状データ」のことを指す  
> クリッピング:  画面外のポリゴンを除外すること  
> カリング:      画面にはあるが、他のポリゴンによって隠されているポリゴンを除外すること

ジオメトリエンジンの役割は、 頂点変換、投影、ライティング、クリッピング、カリング、ポリゴン並べ替え です。

ジオメトリコマンドは、`0x0400_0440..` から送信します。(0x0400_0400 に直接書き込むこともできます。)

コマンドには行列とベクトルの乗算が含まれ、その目的は頂点座標の回転/拡大縮小/移動で、結果の座標は頂点RAMに保存されます。

さらに、ポリゴンと頂点にアトリビュートを割り当てることができ、これには、頂点カラーまたは自動的に計算されるライトカラー、テクスチャ属性、ポリゴンあたりの頂点数（3個または4個）、および多数のフラグが含まれ、これらのアトリビュートはポリゴンRAMに格納されます。ポリゴンRAMには、頂点RAMの対応する頂点へのポインタも含まれます。

## スワップバッファ (ジオメトリエンジンからレンダリングエンジンへのデータの受け渡し)

3Dエンジンには 2組の頂点RAM/ポリゴンRAM があり、1つはジオメトリエンジンが使用し、もう1つはレンダリングエンジンが使用します。

スワップバッファコマンドはこれらのバッファを交換するだけで、新しいジオメトリデータがレンダリングエンジンに渡され、古いバッファは空になり、ジオメトリエンジンが新しいデータを書き込めるようになります。

さらに、前のスワップバッファコマンドの 2つのパラメータビットがジオメトリエンジンにコピーされます。

Data that is NOT swapped: SwapBuffers obviously can’t swap Texture memory (so software must take care that Texture memory is kept mapped throughout rendering). Moreover, the rendering control registers (ports 4000060h, and 4000330h..40003BFh) are not swapped (so that values must be kept intact during rendering, too).

## レンダリングエンジン (画面出力)

レンダリングエンジンはさまざまなポリゴンを描画し、BG0レイヤとして2Dエンジン(PPU)に出力します。レンダリング部分はハードウェアが自動的に行うため、ソフトウェアが重い処理を行っていても影響はありません。

レンダリングはスキャンラインごとに行われるため、1スキャンラインあたりのクロックサイクル数には限りがあり、1スキャンラインあたりの最大ポリゴン数が制限されます。ただし、48行キャッシュ（後述）のため、一部のスキャンラインはこの最大値を超えても問題ありません。

レンダリングはVBlank期間中に48ライン前から開始され、全表示期間を通じて継続され、レンダリングの結果は最大48スキャンラインを保持できる小さなキャッシュに書き込まれます。

## Scanline Cache vs Framebuffer

Note: There’s only the 48-line cache (not a full 192-line framebuffer to store the whole rendered image). That is perfectly reasonable since animated data is normally drawn only once (so there would be no need to store it). That, assuming that the Geometry Engine presents new data every frame (otherwise, if the Geometry software is too slow, or if the image isn’t animated, then the hardware is automatically rendering the same image again, and again).
