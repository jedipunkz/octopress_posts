---
layout: post
title: "Vyatta ハンズオン参加レポ #interop2012"
date: 2012-06-14 22:28
comments: true
categories: vyatta
---
interop 2012 で開催された "仮想ルータ Vyatta を使ったネットワーク構築法" に参
加してきました。簡単ですがレポートを書いておきます。

    開催日 : 2012/06/14(木)
	場所   : 幕張メッセ Interop 2012

最初に所感。反省です。サーバエンジニアの視点でしか感じられていなかった。次期バー
ジョンの vPlane 実装などエンタープライズ向けとしても利用出来る可能性を感じる
し、比較的小さなリソースでハンズオン参加者30人程度の vyatta ルータを動かしてサ
クサク動いているのを見て、簡単にパフォーマンスがどうとか
[前回の記事](http://jedipunkz.github.com/blog/2012/06/13/vyatta-vpn/)で言うべ
きじゃなかったぁ。ホント反省。

ではハンズオンの内容。

最初に、基本情報の話
----

有償版と無償版の違い

* メジャーリリースのみの無償版に対して有償版はマイナーリリースもあり
* 有償版は保守あり
* 仮想 image template 機能が有償版であり
* API, Web GUI, Config Sync, Systen Image Cloning が有償版であり

image template, config sync, cloning など、有償版ではあると嬉しい機能がモリモ
リ。

最近の利用ケース
----

* Vyatta + VM で VPN 接続環境構築
* キャンパスネットワーク

キャンパスネットワークのユースケースが一番多いそうだ。また、会場内にアンケート
をとった結果、仮想環境での構築を想定されている方が多数だった。

構成と特徴
----

* Debian Gnu/Linux ベース
* Quagga, StrongSwam が内部で動作

apt-get など馴染み深いコマンドが使えます。
[http://www.vyatta4people.org/tag/vybuddy/](http://www.vyatta4people.org/tag/vybuddy/)
ここに色々ノウハウがあるらしい。僕はまだ見ていません。まほろば工房の近藤さんに
教えて頂きました。あとでチェックします。

バージョン
----

stable リリースは 6.4 。次期バージョン 6.5 では
[vPlane](http://www.vyatta.com/technology/vplane) の実装が予定されている。これ
は IA サーバのコア毎に役割を変えるといったモノ。ノードが忙しくなった時に例えば
転送を司るコアには影響を与えない等。これは重要だ。サーバと違ってネットワーク機
器は何があっても死守しなくちゃいけない機能があるもんなぁ。6.5 に期待。

ここで、ハンズオンで課題になった IPSec の話を少しだけ説明します。

    192.168.18.0/24
    +------+       +----------+
    | vm01 |-------| vyatta01 |pppoe XXX.XXX.XXX.XXX
    +------+       +----------+            |
                   |192.168.20.0/24        | IPSec
    +------+       +----------+            |
    | vm02 |-------| vyatta02 |pppoe YYY.YYY.YYY.YYY
    +------+       +----------+
    192.168.19.0/24

上図の環境で vm01 が vm02 に IPSec を用いて接続するための方法を記しています。
vyatta 同士は 192.168.20.0/24 のネットワークで接続されていますが、互いにルーティ
ングテーブルは書いていないものとします。まずは vyatta01 に対しての設定。

あ、pppoe による NAT の設定は予めしてあるものとします。

    # set vpn ipsec ipsec-interfaces interface pppoe1
	# set vpn ipsec ike-group IKE-G proposal 1 encryption aes256
	# set vpn ipsec ike-group IKE-G proposal 1 hash sha1
	# set vpn ipsec ike-group IKE-G lifetime 3600

鍵交換の方式 IKE の設定をします。

    # set vpn ipsec esp-group ESP-G proposal 1 encryption aes256
    # set vpn ipsec esp-group ESP-G proposal 1 hash sha1
    # set vpn ipsec esp-group ESP-G lifetime 1800

ESP の設定をします。鍵の破棄・生成時間 1800 を設定しています。

    # edit vpn ipsec site-to-site peer YYY.YYY.YYY.YYY
	# set authentication mode pre-shared-secret
	# set authentication pre-shared-secret hogehoge

site-to-site peer を指定して接続先 vyatta のグローバル IP アドレスを指定します。
また pre-shared-secret でパスフレーズを指定します。

    # set ike-group IKE-G
	# set local-ip YYY.YYY.YYY.YYY
	# set tunnel 1 local subnet 192.168.18.0/24
	# set tunnel 1 remote subnet 192.168.19.0/24
	# set tunnel 1 esp-group ESP-G

自グローバル IP, 自プライベートネットワーク、リモートネットワークの情報諸々、
設定します。

    # set nat source rule 10 destination address !192.168.19.0/24
    # commit

また最後に IPsec で接続する先のネットワークへのパケットの場合、NAT しないよう
にします。

次は逆に vyatta01 に対しても設定を投入します。諸々の値は逆にする必要があります。
これを実施すると、vm01 から vm02 に対して pppoe interface 同士の IPsec を介し
て接続できるのが確認出来ました。

私の職場で開発した製品にも投入してみようかなと目論んでいます。楽しかった。

最後に、この場を提供して下さった方々に感謝します。

講演者
----

* 株式会社まほろば工房 近藤邦昭さま
* 有限会社銀座堂 浅間正和さま
* さくらインターネット 大久保修一さま
* 伊藤忠テクノソリューションズ 伊藤哲史さま
