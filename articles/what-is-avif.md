---
title: "次世代の画像ファイル形式、AVIF"
emoji: "📸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["web"]
published: true
---

# はじめに
2024 年に新しく Baseline の Newly available に追加された機能のうち、今回は **AVIF** について気になったのでまとめました。
専門領域ではないのでなるべく一次ソースをあたりながら書いていますが、間違いなどがあればコメントまでお願いします。

# AVIF について
AV1F は、png や jpg といった画像ファイルの形式のひとつです。

https://developer.mozilla.org/ja/docs/Web/Media/Formats/Image_types#avif_%E7%94%BB%E5%83%8F

MDN では『ウェブコンテンツで画像を共有するための「次の大きな流れ」となる可能性を秘めている』と書かれており、その期待通りのパフォーマンスを持っています。
その流れに乗るようにして、2024年の初頭に Baseline の Newly Available に追加され、主要ブラウザで問題なく動作することが保証されました。

https://caniuse.com/?search=avif

## 特徴
**AV1** という動画圧縮コーデックで圧縮された画像を、**HEIF** という画像コンテナに保存することで高圧縮を実現しています。

---

AV1 は、2018年に AOMedia によって公開された動画コーデックです。オープンソースでロイヤリティフリー（ライセンス料が無料）なことから注目を浴びました。既存よりも高い圧縮率を持ち、画像の劣化を最小限に抑えます。他方、処理速度に関する懸念がさまざまな記事で取り上げられていましたが、これを解決するための [SVT-AV1](https://gitlab.com/AOMediaCodec/SVT-AV1) プロジェクトなどが動いていたりする現状もあります。

AV1 については、総務省が公開している動向調査のスライドが参考になります。
https://www.soumu.go.jp/main_content/000683387.pdf

パフォーマンス比較については以下の記事が参考になります。
https://qiita.com/yanoshi/items/544a361baf76b8114067
https://qiita.com/daredeshow/items/a780041a0d023a12cc63

---

HEIF（High Efficiency Image File Format） は JPEG2000 の後継となる画像データのコンテナ仕様です。主に H.265/HEVC という画像圧縮コーデックを使用することを想定していますが、任意の圧縮形式のデータをいれられます。

HEIF については以下の資料スライドが参考になりました。ありがとうございます。
https://speakerdeck.com/yoya/heif-kaisetsu

:::message
📽️「コーデック（Codec）」とは
一般に、「データのエンコード（符号化）とデコード（復号）ができるもの」とされています。
画像や動画においては、ある画像を別の拡張子（フォーマット）に変換したり、動画を圧縮して容量を減らしたりするときに使われます。
:::

---

AV1, HEIF ともに動画データに対応していることもあり、アニメーションも扱えます。


以上の点を踏まえ、AVIF のメリットとして以下の3つを挙げます。
- **優れた圧縮率**
- **画像の劣化を最小限に抑える**
- **アニメーションに対応している**

# 参照
https://developer.mozilla.org/ja/docs/Web/Media/Formats/Image_types#avif_%E7%94%BB%E5%83%8F