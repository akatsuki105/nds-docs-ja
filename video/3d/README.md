# [3Dエンジンの概要](https://mgba-emu.github.io/gbatek/#ds3doverview)

NDSの3Dエンジンは、ジオメトリエンジン と レンダリングエンジン で構成されています。

## ジオメトリエンジン (頂点の座標計算、ポリゴン属性の割り当て)

> [!NOTE]
> クリッピング:  画面外のポリゴンを除外すること  
> カリング:      画面にはあるが、他のポリゴンによって隠されているポリゴンを除外すること

ジオメトリエンジンの役割は、 頂点変換、投影、ライティング、クリッピング、カリング、ポリゴン並べ替え です。

ジオメトリコマンドは、`0x0400_0440..` から送信します。(0x0400_0400 に直接書き込むこともできます。)

ジオメトリコマンドの一例として、行列とベクトルの乗算があるため、ポリゴンの頂点座標の回転/拡大縮小/移動をゲーム側で行う必要はありません。(結果の座標は頂点RAMに保存されます)

他にも、ジオメトリコマンドでポリゴンと頂点に属性を割り当てることができ、これには頂点カラーまたは自動的に計算されるライトカラー、テクスチャ属性、ポリゴンあたりの頂点数（3個または4個）、および多数のフラグが含まれ、これらの属性はポリゴンRAMに格納されます。ポリゴンRAMには、頂点RAMの対応する頂点へのポインタも含まれます。

## スワップバッファ (ジオメトリエンジンからレンダリングエンジンへのデータの受け渡し)

3Dエンジンには 2組の頂点RAM/ポリゴンRAM があり、1つはジオメトリエンジンが使用し、もう1つはレンダリングエンジンが使用します。

`SWAP_BUFFERS`コマンドはこれらのバッファを交換するだけで、新しいジオメトリデータがレンダリングエンジンに渡され、古いバッファは空になり、ジオメトリエンジンが新しいデータを書き込めるようになります。 さらに、前の`SWAP_BUFFERS`コマンドの 2つのパラメータビットがジオメトリエンジンにコピーされます。

注意すべきこととして、`SWAP_BUFFERS`コマンドでは、テクスチャメモリ と レンダリング制御レジスタ(DISP3DCNT, `4000330h..40003BFh`) はスワップされません。そのため、ゲーム側はレンダリング中、VRAMをテクスチャにマップし続けレンダリング制御レジスタの値を保持し続ける必要があります。

## レンダリングエンジン (画面出力)

レンダリングエンジンはさまざまなポリゴンを描画し、BG0レイヤとして2Dエンジン(PPU)に出力します。レンダリング部分はハードウェアが自動的に行うため、ソフトウェアが重い処理を行っていても影響はありません。

レンダリングはスキャンラインごとに行われるため、1スキャンラインあたりのクロックサイクル数には限りがあり、1スキャンラインあたりの最大ポリゴン数が制限されます。ただし、48行キャッシュ（後述）のため、一部のスキャンラインはこの最大値を超えても問題ありません。

レンダリングはVBlank期間中に48ライン前から開始され、全表示期間を通じて継続され、レンダリングの結果は最大48スキャンラインを保持できる小さなキャッシュに書き込まれます。

## Scanline Cache vs Framebuffer

Note: There’s only the 48-line cache (not a full 192-line framebuffer to store the whole rendered image). That is perfectly reasonable since animated data is normally drawn only once (so there would be no need to store it). That, assuming that the Geometry Engine presents new data every frame (otherwise, if the Geometry software is too slow, or if the image isn’t animated, then the hardware is automatically rendering the same image again, and again).


## 参考

- [The DS GPU and its fun quirks - melonDS](https://melonds.kuribo64.net/comments.php?id=56)


