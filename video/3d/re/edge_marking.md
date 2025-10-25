# エッジ強調(Edge Marking)

エッジ強調は、Attributeバッファ内で異なるポリゴンIDを持つ(ワイヤーフレームを含む)不透明ポリゴンの輪郭を強調する機能です。この機能は、`DISP3DCNT.5`をセット・クリアすることで（フレーム単位で）有効・無効の切り替えが可能です。

<details>
  <summary>使用例</summary>

左: エッジ強調OFF, 右: エッジ強調ON

階段あたりがわかりやすいです。

![noedgemarking](../../../images/g3/edge_marking_disable.png)&nbsp;&nbsp;![edgemarking](../../../images/g3/edge_marking_enable.png)

</details>

## 4000330h..33Fh - EDGE_COLOR - エッジカラー 0..7 (W)

エッジ強調に使用される色は、ポリゴンIDの上位3bitをインデックスとして選択される8色です。

この16バイト領域には8つのエッジカラー（RGB555）が格納されます。

