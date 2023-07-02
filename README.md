# SqueezeNet
## ポイント
- CNNで以下の利点を獲得するために小規模、かつAlexNetや他アーキテクチャと同等精度のSqueezeNetを考案
1. 分散トレーニング中に必要なサーバー間の通信が少なくする
2. 新しいモデルをクラウドから自動運転車等にエクスポートするために必要な帯域幅が少なくする
3. メモリが限られている FPGA やその他のハードウェアに展開しやすくする
- パラメータ数を削減するために以下の戦略を用いた
1. 3x3フィルターを 1x1フィルターに置き換える
2. 1x1フィルターで3x3フィルターへの入力チャンネル数を減らす
3. 遅延ダウンサンプリングを適用し、分類精度を向上させる（筆者の経験則によるもの）
- 結果、AlexNetの50分の1のパラメータ数で、同程度の精度を記録
## Fireモジュール
- Fire モジュールは次のもので構成されます
- スクイーズコンボリューションレイヤー (1x1 フィルター):入力チャンネル数を減らす
- 1x1 と 3x3 コンボリューション フィルターが混在するエキスパンド レイヤー
<img alt="Fire module" src=./image/Fire_module.png></img>
- Fireモジュールで 3 つの調整可能なハイパーパラメータがある: $s1x1$、$e1x1$、および $e3x3$
- $s1x1$ はスクイーズレイヤーのフィルターの数、$e1x1$ はエキスパンドレイヤーの 1x1 フィルターの数、$e3x3$ はエキスパンドレイヤーの 3x3 フィルターの数
## アーキテクチャ
- アーキテクチャの図と詳細を下記に示す
<img alt="sqeeze_achi" src=./image/sqeeze_achi.png></img>
- 図左から2, 3のアーキテクチャはSqeezeNetにSkipConnectionを追加したもの。2のアーキテクチャは通常のアーキテクチャより精度が向上した
<img alt="sqeeze_achi2" src=./image/sqeeze_achi2.png></img>
- 1x1フィルタと 3x3フィルタからの出力が同じ高さと幅になるように、入力データにゼロパディングの1を追加
- ReLUをスクイーズ層とエキスパンド層のアクティベーション関数として適用
- 50%のドロップアウトをfire9 モジュールの後に適用
- 最終層において、全結合層が存在しない
## Fireモジュールのメタパラメータ
- ハイパーパラメータの影響を調査するために、実験行った。アーキテクチャの探索のために以下の高レベルパラメータを定義する
- $Base$：CNN最初のFireモジュールエキスパンドレイヤ数
- $e_i$：i番目のFireモジュールエキスパンドレイヤの数：$e_i = Base + (incre * i/div{freq})$
- $e_i = e_{i,1x1} +e_{i,3x3}：$e_i$は1x1フィルタ数と3x3フィルタ数に分割できるため、左記で定義できる
- $pct3x3, [0, 1]$：$e_i$における3x3フィルタ数の割合
- $SR, [0, 1]$：スクイーズレイヤーのフィルタ数を求めるために使用。$s_{i,1x1} = e_i * SR$で求める
- 上記図のベースアーキテクチャは、$Base = 128、incre = 128、pct3x3 = 0.5、freq = 2、および SR = 0.125$で設計
- SRを変更する実験では$SR = 0.75$で精度が頭打ちとなる
- エクスパンドレイヤーの割合を変更する実験（$Base = incre = 128、freq = 2、SR = 0.500 を使用し、pct3x3を1%から99%まで変化$）では、$pct3x3$で精度が頭打ち
## 参考
1. https://arxiv.org/pdf/1602.07360.pdf
2. https://pystyle.info/pytorch-squeezenet/
