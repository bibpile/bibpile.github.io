+++
title = "MultiBox"
date = 2021-07-26T00:47:15
pubdate = 2013-12-08T19:40:51
category = "paper"
tags = ["Object Detection", "Region Proposal", "CVPR", ]
math = true
draft = false
+++

{{< cite >}}
@InProceedings{Erhan_2014_CVPR,
author = {Erhan, Dumitru and Szegedy, Christian and Toshev, Alexander and Anguelov, Dragomir},
title = {Scalable Object Detection using Deep Neural Networks},
booktitle = {Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
month = {June},
year = {2014}
}
{{< /cite >}}

{{< links paper="https://openaccess.thecvf.com/content_cvpr_2014/html/Erhan_Scalable_Object_Detection_2014_CVPR_paper.html" video="https://www.youtube.com/watch?v=3U-bZgKFS7g" >}}

MultiBoxは、画像中に複数インスタンスが会った場合でも、Confidence付きのbboxを複数出力可能なモデルです。これも重要論文に数えられるんですが、解説がなかなかありません。おそらく単純な方法だからだとおもうんですが、一応読んでみました。軽くさらっていきます。
![](2021-07-26-00-03-19.png)

---

# 概要

一枚の画像中で複数の同一インスタンスがあった場合に、複数のbboxをconfidence付きで出力できるようにした論文です。  
従来手法だと、一枚画像を入力して、各クラスに対して単一のbboxを推論するネットワークしかなかったらしいんですが、MultiBoxでは、画像中に複数の同一インスタンスがあっても、一発で複数の領域提案が行えるようにしています。
Multiboxはクラスに依らない領域提案を学習しているので、高い汎化能力があるそうです。

# Multibox

論文を読んだ感じだと、次図のアーキテクチャになると思います。共通のAlexNetのバックボーンがあり、そこからbbox回帰と物体らしさ(Objectness)のconfidenceを出力する2つのブランチがあります。

Multiboxは、事前に一枚の画像に含まれる最大の物体数Kを決めておきます。そして、K個のbboxの位置とconfidenceを推定した後に、NMSやConfidenceを使って、不要なbboxの廃棄(supression)を行います。論文中では\\( K = 100, 200 \\)を使用しています。  

![](2021-07-26-00-03-27.png)

ほとんど解説が終わってしまったんですが、一応各ブランチでの損失関数などを解説します。

## bbox回帰ブランチ

bbox回帰ブランチでは、K個あるbbox候補のGT座標への回帰を行います。出力は、GT bboxの左上と右下のxy座標の4点です。

損失関数は次式です。ここで、\\( x_{ij} \\)は、推論bbox\\( i \\)がGT bbox \\( j \\)と対応しているとき1になりそれ以外は0になるバイナリ値です。それ以外は普通にL2ロスで最適化しています。筆者は、この\\( x_{ij} \\)の理解が怪しいです。  

$$
F_{\text {match }}(x, l)=\frac{1}{2} \sum_{i, j} x_{i j}\left\|l_{i}-g_{j}\right\|_{2}^{2}
$$

## Confidence推定ブランチ

Confidenceブランチでは、各bboxに対応したConfidence値を推定します。損失関数は、次式です。bbox回帰ブランチに入ってくる\\( x_{ij} \\)を除くと、バイナリクロスエントロピーロスと同じ形をしています。物体かどうかの判定をしています。推論bbox\\( i \\)のGT bbox \\( j \\)へのConfidenceが上昇すると損失が下がる構造になっています。  

$$
F_{\mathrm{conf}}(x, c)=-\sum_{i, j} x_{i j} \log \left(c_{i}\right)-\sum_{i}\left(1-\sum_{j} x_{i j}\right) \log \left(1-c_{i}\right)
$$

最後に上記２つのブランチの損失をマージした損失関数（次式）をまとめて最適化していきます。詳しい学習方法は割愛します。


$$
F(x, l, c)=\alpha F_{\text {match }}(x, l)+F_{\text {conf }}(x, c)
$$
