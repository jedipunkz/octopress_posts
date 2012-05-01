---
layout: post
title: "vyatta で自宅ルータ構築"
date: 2012-04-28 16:03
comments: true
categories: vyatta
---
自宅ルータを Vyatta で構築してみたくなり、秋葉原の ark でマシンを調達しました。
Broadcom の BCM57780 チップが搭載された NIC がマザーボード J&W MINIX™
H61M-USB3 だったのですが、Vyatta.org によると Broadcom の NIC が Certificated
Hardware に載っていなくて心配でした。まぁ定評のある NIC メーカだから動くだろう
と楽観視していたのですけど、案の定動きました。vyatta.org の Certificated
Hardware にコミットしたら "user tested" として掲載してもらえました。

[http://www.vyatta.org/hardware/interfaces](http://www.vyatta.org/hardware/interfaces)

こんな感じに見えています。

    # dmesg | grep Broadcom
    [    3.284646] tg3 0000:03:00.0: eth0: attached PHY driver [Broadcom BCM57780](mii_bus:phy_addr=300:01)
    [    3.524122] tg3 0000:05:00.0: eth1: attached PHY driver [Broadcom BCM57780](mii_bus:phy_addr=500:01)

今回は、基本的な設定 (PPPoE, NAT, DHCP) 周りを記していきます。

環境は...

    +--------+
    |  Modem |
    +--------+
    |
    | pppoe0
    +--eth0--+
    | vyatta |
    +--eth1--+
    |           192.168.1.0/24
    +----------+
    |          |192.168.1.10
    +--------+ +--------+
    |  CPE   | |  DNS   |
    +--------+ +--------+

として記します。

まずインストール。http://www.vyatta.org/downloads から 64bit VC6.3 Live CD iso
をダウンロードしてきます。(2012/04/28現在最新). インストール対象のマシンに挿入
して CDROM ブートすると Vyatta が立ち上がるので、ユーザ vyatta, パスワード
vyatta でログインし

    # install-system

します。インタラクティブに問い合わせられるので答えていってください。インストールが
終わったらマシンを再起動します。

eth1 にプライベート IP アドレスを振ります。IP アドレスは適当に読み替えてくださ
い。また基本的な設定も行います。

    # set service ssh port 10022 # firewall 設定するまではこうしたほうが安心です
	# set system host-name ${HOSTNAME}
    # set system time-zone Asia/Tokyo
	# set system name-server 8.8.8.8
    # set interface ethernet eth1 address 192.168.1.254

eth0 を PPPoE デバイスとして利用して PPPoE 接続を実際にします。

    # set interface ethernet eth0 pppoe 0
	# set interface ethernet eth0 pppoe 0 user-id ${PPPoE_username}
	# set interface ethernet eth0 pppoe 0 password ${PPPoE_password}
    # set interface ethernet eth0 pppoe 0 name-server auto
    # set interface ethernet eth0 pppoe 0 defaultroute auto
	# set interface ethernet eth0 pppoe 0 local-address XXX.XXX.XXX.XXX # 固定IPがある場合
    # commit

ここまでで vyatta ノードからインターネットに接続出来るようになります。

次に、ローカルネットワーク上の CPE だったりサーバノードからのインターネット接
続のために NAT 設定をします。

    # set service nat rule 1
	# set service nat rule 1 type masquerade
	# set service nat rule 1 source address 192.168.1.0/24
	# set service nat rule 1 outbound-interface pppoe0
    # commit

CPE からのインターネットへの接続が出来るようになりました。

あとは自宅ルータなので DHCP サービスがあったほうが便利だよね、とうことで。

    # set service dhcp-server shared-network-name HOME subnet 198.168.1.0/24 start 192.168.1.100 stop 192.168.1.150
    # set service dhcp-server shared-network-name HOME subnet 192.168.1.0/24 default-router 192.168.1.254
    # set service dhcp-server shared-network-name HOME subnet 198.168.1.0/24 dns-server 8.8.8.8
    # set service dhcp-server shared-network-name HOME subnet 198.168.1.0/24 dns-server 8.8.4.4
    # commit

ローカルネットワーク上の CPE から DHCP リクエストを出してみてください。IP が取得できると思います。

ここまでで基本的な有線自宅ルータとしての構築はほぼ完了ですが、最後にローカルネットワーク上のサーバ
を DNS サーバとして稼働させるための NAT 設定方法を記しておきます。

    # set service nat rule 2
    # set service nat rule 2 destination
    # set service nat rule 2 type destination
	# set service nat rule 2 type destination
    # set service nat rule 2 inbound-interface pppoe0
	# set service nat rule 2 protocol udp
	# set service nat rule 2 destination port 53
	# set service nat rule 2 inside-address address 192.168.1.10
    # commit

DNS は稀に TCP にフォールバックするので同様に TCP ようの NAT ルールも追記します。

    # set service nat rule 3
    # set service nat rule 3 destination
    # set service nat rule 3 type destination
	# set service nat rule 3 type destination
    # set service nat rule 3 inbound-interface pppoe0
	# set service nat rule 3 protocol tcp
	# set service nat rule 3 destination port 53
	# set service nat rule 3 inside-address address 192.168.1.10
set service nat rule 3 destinationset service nat rule 3 destination    # commit

最後に設定を保存するために...

    # save

して終わりです。次回は UPnP な設定方法を書いて行きたいと思います。

ちなみに、私の自宅ルータはこんなモノを使いました。

<img src="http://jedipunkz.github.com/pix/vyatta.jpg" width="200">
