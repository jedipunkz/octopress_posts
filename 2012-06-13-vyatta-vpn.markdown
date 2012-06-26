---
layout: post
title: "Vyatta で構築する簡単 VPN サーバ"
date: 2012-06-13 13:36
comments: true
categories: vyatta
---
Vyatta で VPN しようと思ったら信じられないくらい簡単に構築できたので共有します。

今回は PPTP (Point-to-Point Tunneling Protocol) を用いました。

さっそく手順に。

    $ configure
    # set vpn pptp remote-access authentication local-users username ${USER} password ${PASSWORD}
	# set vpn pptp remote-access authentication mode local
	# set vpn pptp remote-access client-ip-pool start ${IP_START}
	# set vpn pptp remote-access client-ip-pool stop ${IP_END}
	# set vpn pptp remote-access outside-address ${GLOBAL_IP}
    # set vpn pptp remote-access dns-servers server-1 8.8.8.8
    # set vpn pptp remote-access dns-servers server-1 8.8.4.4
    # commit
	# save

これだけです。

認証のためのユーザを1行目で作っています。client-ip-pool で VPN 接続端末に付与
する IP アドレスのレンジを指定して outside-address で Listen Port を指定します。
DNS リゾルバに指定させたいアドレスを dns-servers で指定したら終わりです。

私は手元の iPhone で自宅に VPN 接続して使っています。これで iPhone か 自宅内の
機器に直接アクセスできるので便利です。

あと、Vyatta をしばらく自宅で使ってみての所感です。

Vyatta は汎用機にインストールできるし VM としても動作させられるし便利です。が、
実際に起動しているプロセス等を見ていくと、Linux 界では古くからあるレガシなソフ
トウェアが起動しているだけに見えます。Vyatta の UI はこれらソフトウェアのコン
フィグレーションを簡易化するラッパー的なモノになっているのが分かります。

汎用性がある一方、パフォーマンスはあまり期待出来ないかもしれません。実際にリッ
チなコンテンツを閲覧した時のパフォーマンスは Buffalo 製コンシューマ向け機器に
劣ります。Linux なのでチューニングは出来ますが、予め幾つかのチューニングは入っ
ていますし、あくまでもチューニングなので劇的なパフォーマンス改善には繋がりませ
ん。

FreeBSD 系 ? で pfsense という 'open source firewall distribution' もあるよう
なので、いずれ試してみたいです。

