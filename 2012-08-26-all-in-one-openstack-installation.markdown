---
layout: post
title: "OpenStack ESSEX オールインワン インストール"
date: 2012-08-26 00:20
comments: true
categories: openstack
---
OpenStack のインストールってしんどいなぁ、って感じて devstack
<http://devstack.org/> とかで構築して中を覗いていたのですが、そもそも devstack
って再起動してしまえば何も起動してこないし、swift がインストールされないしで。
やっぱり公式のマニュアル見ながらインストールするしかないかぁって...。感じてい
たのですが...。

<http://docs.openstack.org/essex/openstack-compute/starter/os-compute-starterguide-trunk.pdf>

このマニュアルの前提は、ネットワーク2セグメント・server1, server2 の計2台が前
提なのですが、環境作るのがしんどいので、オールインワンな構築がしたい！サーバ1台で OpenStack ESSEX を
インストールしたい！で、シェルスクリプトを作ったのでそれを使ったインストール方法を紹介します。

<img src="http://jedipunkz.github.com/pix/openstack_thinkpad.jpg">

ぼくの Thinkpad に OpenStack ESSEX をインストールしてブラウザで localhost に接続して
いる画面です。ちゃんと KVM VM が起動して noVNC で接続できています。自己満足やぁ。

前提条件
----

* Ubuntu Server 12.04 LTS amd64 がインストールされていること
* Intel-VT もしくは AMD−Vなマシン
* NIC が一つ以上ついているマシン
* /dev/sda6, /dev/sda7 (デバイス名は何でもいい) の2つが未使用で空いていること

です。

構成
----

1 NIC を前提に eth0 と eth0:0 の2つを想定すると、こんな構成になります。eth0:0
は完全にダミーで IP アドレスは何でもいいです。br100 ブリッジデバイス上で VM が
NW I/F を持ちます。floating range ってのは OpenStack で言うグローバル IP レン
ジ。グローバルである必要は無いですが eth0 と同じレンジの IP アドレスを VM に付
与出来ます。/dev/sda6 が nova-volumes で /dev/sda7 が swift 。なので OS インス
トール時に2つのデバイスを未使用で空けておいてください。

    +--+--+--+
    |VM|VM|VM|  192.168.4.32/27
    +--+--+--+..
    +----------+ +--------+
    |          | | br100  | 192.168.4.33/27 -> floating range : 10.200.8.32/27
    |          | +--------+
    |          | | eth0:0 | 192.168.3.1       disk devices
    |   Host   | +--------+   (dummy)  +------------------------+
    |          |                       | /dev/sda6 nova-volumes |
    |          | +--------+            +------------------------+
    |          | |  eth0  | ${HOST_IP} | /dev/sda7 swift        |
    +----------+ +--------+            +------------------------+
    |              nw I/Fs
    +----------+
    |   CPE    |
    +----------+

インストール手順
-----

インストール手順は簡単です。

    % sudo -i
    # git clone git://github.com/jedipunkz/openstack_install.git

して取得したスクリプトを環境に合わせて環境変数設定します。スクリプト上部のこれ
らの内容を、環境に合わせて設定。${HOST_IP} と ${NOVA_VOLUMES_DEV},
${SWIFT_DEV} だけ気をつければ OK です。その他は内部ネットワークの設定なので何
でもつながります。

    # -----------------------------------------------------------------
	# Environment Parameter
	# -----------------------------------------------------------------
	HOST_IP='10.200.8.15'
	HOST_MASK='255.255.255.0'
	HOST_NETWORK='10.200.8.0'
	HOST_BROADCAST='10.200.8.255'
	GATEWAY='10.200.8.1'
	MYSQL_PASS='secret'
	FIXED_RANGE='192.168.4.1/27'
	FLOATING_RANGE='10.200.8.32/27'
	FLAT_NETWORK_DHCP_START='192.168.4.33'
	ISCSI_IP_PREFIX='192.168.4'
	NOVA_VOLUMES_DEV='/dev/sda6'
	SWIFT_DEV='/dev/sda7'

で実行。

    # chmod +x openstack_install/openstack_install.sh
    # ./openstak_install/openstack_install.sh allinone
    ( wait some minutes...)

マシンによりますが、10分弱すると OpenStack が構築されているはずです。

OS イメージと SSH キーペアのインストール
----

上記の手順で OpenStack は構築されるのですが、VM を起動するには OS イメージが必
要ですよね。これは自分で用意するしかないです。ただ、これは簡単で下記の手順で出
来ます。

#### OS イメージ作成

サンプルで Ubuntu Server 12.04 LTS amd64 なイメージをここで作ってみます。

    # kvm-image create -f qcow2 server.img 5G
    # wget http://gb.releases.ubuntu.com//precise/ubuntu-12.04-server-amd64.iso
    # kvm -m 256 -cdrom ubuntu-12.04-server-amd64.iso -drive file=server.img,if=virtio,index=0 -boot d -net nic -net user -nographic -vnc :0

手元の端末の VNC ツールで ${HOST_IP}:0 に接続し OS のインストールを済ませます。
その後、下記のコマンドで HDD から起動してあげて...

    # kvm -m 256 -drive file=server.img,if=virtio,index=0 -boot c -net nic -net user -nographic -vnc :0

再度、VNC で VM に接続して..

    # sudo rm -rf /etc/udev/rules.d/70-persistent-net.rules
    # shutdown -h now

上記の操作をしたら OS イメージ作成は終わり。その他のディストリビューションでの
イメージ作成については公式マニュアルに書いてあります。

#### Glance に OS イメージをインストール

作成した OS イメージを Glance に追加します。先ほどのスクリプトで生成された
/root/.openstack が Glance に接続するために必要なので zsh の場合 source してから...

    # source /root/.openstack
	# glance add name="Ubuntu Server 12.04LTS" is_public=true container_format=ovf disk_format=qcow2 < server.img

で追加出来ます。.openstack は bash でも取得できるのでその際は

    # . /root/.openstack

してください。root ユーザ以外でも操作出来ます。

#### SSH キーペアの生成とインストール

VM に割り当てる SSH キーペアを作ってインストールする手順です。

    # ssh-keygen
	# nova keypair-add --pub_key .ssh/id_rsa.pub mykey
	# nova keypair-list

#### Horizon へ接続

いよいよ Horizon へ接続です。Horizon は OpenStack ESSEX から取り込まれた Web
UI です。VM の作成・削除、ネットワークの設定等がブラウザで操作出来ます。

    http://${HOST_IP

に接続してユーザ : 'admin', パスワード 'admin' でログインしてください。

まとめ
----

OpenStack はしんどいｗ ですが来月 2012/09 リリース予定の Folsom は 'Easy
Setup' がフューチャされてるそうです。期待。手動で構築していると glance のとこ
ろで ID 地獄にハマりますｗ 今回の手順で all in one な環境ができたら、色々覗い
てみてコンポーネント毎に Node を切り出すってことも考えないといけないと思います。
それぞれは HTTP ベースの API で接続できれば OK なので切り出すこと自体は簡単。
冗長を組む方法は.. これから調べます。keystone, glance を拡張・冗長させるって出
来るのか？難しそう。CloudStack と違って rabbitmq-server でキューイングしてくれ
るので、Node が増えた時の対処は考えれれているよう。

あと、OS イメージではなくて AMI で VM を作る方法もあるのですが AMI の作成方法は Web
を見ていると沢山載っていますので参考にして作ってみてください。

