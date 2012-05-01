---
layout: post
title: "vyatta で UPnP 接続"
date: 2012-04-29 11:45
comments: true
categories: vyatta
---
Vyatta を自宅ルータで使い始めて感じたのは、PS3 などのゲーム機や IP 電話など
UPnP 接続が必要なことがあるってこと。ただ Vyatta は UPnP に対応していないので、
どうしようかと思っていたら、有志の方が作ってくれたソフトウェアがあり、うちでも
これを使うことにした。今回はその方法を記していきます。

[https://github.com/kiall/vyatta-upnp](https://github.com/kiall/vyatta-upnp)

上記のソースを取得して生成するのだが、vyatta 上で構築する環境を作りたくないの
で、私は Debian Gnu/Linux マシン上で行いました。Ubuntu でも大丈夫だと思います。

    debian% sudo apt-get && sudo apt-get install build-essential
    debian% git clone https://github.com/kiall/vyatta-upnp.git
    debian% cd vyatta-upnp
	debian% dpkg-buildpackage -us -uc -d

一つ上のディレクトリに vyatta-upnp_0.2_all.deb という .deb ファイルができあがっ
ているはずで、これが UPnP パッケージファイル vyatta-upnp_0.2_all.deb です。

次に vyatta 上での作業。packages.vyatta.com から libupnp4 と linux-igd を取得、
その後先ほど生成した vyatta-upnp_0.2_all.deb を vyatta 上に持ってきてからイン
ストールします。

    vyatta# cd /tmp/
    vyatta# wget http://packages.vyatta.com/debian/pool/main/libu/libupnp4/libupnp4_1.8.0~svn20100507-1_amd64.deb
    vyatta# wget http://packages.vyatta.com/debian/pool/main/l/linux-igd/linux-igd_1.0+cvs20070630-3_amd64.deb
    vyatta# scp ${DEBIAN}:/${SOMEWHERE}/vyatta-upnp_0.2_all.deb . # 先ほど生成したファイル
    vyatta# dpkg -i libupnp4_1.8.0~svn20100507-1_amd64.deb linux-igd_1.0+cvs20070630-3_amd64.deb
    vyatta# dpkg -i vyatta-upnp_0.2_all.deb

これで設定が可能になりました。設定してみます。

    vyatta# configure
	vyatta# set service upnp listen-on eth1
    vyatta# commit
	vyatta# save

これで完了です。eth1 は 自宅環境に合わせて下さい。ルータのプライベート側インター
フェース名です。私の家は wlan0 と eth1 をブリッジしているので listen-on br0 に
しました。

私の環境では PS3 の "アンチャーテッド3" で動作確認しています。
