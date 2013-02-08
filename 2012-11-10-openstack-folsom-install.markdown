---
layout: post
title: "OpenStack Folsom 構築スクリプト"
date: 2012-11-10 13:15
comments: true
categories: openstack
---
※2012/12/04 に内容を修正しました。Network Node を切り出すよう修正。
※213/01/09 に内容を修正しました。パラメータ修正です。

OpenStack の Folsom リリースからメインコンポーネントの仲間入りした Quantum を
理解するのに時間を要してしまったのだけど、もう数十回とインストールを繰り返して
だいぶ理解出来てきました。手作業でインストールしてると日が暮れてしまうのでと思っ
て自分用に bash で構築スクリプトを作ったのだけど、それを公開しようと思います。

OpenStack Folsom の構築に四苦八苦している方に使ってもらえたらと思ってます。

<http://jedipunkz.github.com/openstack_folsom_deploy/>

chef や puppet, juju などデプロイのフレームワークは今流行です。ただどれも環境
を予め構築しなくてはいけないので、誰でもすぐに使える環境ってことで bash スクリ
プトで書いています。時間があれば是非 chef の cookbook を書いていきたいです。と
いうか予定です。でも、もうすでに opscode 等は書き始めています。(汗

ではでは、紹介を始めます。

前提の構成
----

    management segment 172.16.1.0/24
    +--------------------------------------------+------------------+-----------------
    |                                            |                  |
    |                                            |                  |
    | eth2 172.16.1.13                           | eth2 172.16.1.12 | eth2 172.24.1.11
    +------------+                               +-----------+      +------------+
    |            | eth1 ------------------- eth1 |           |      |            |
    |  network   | vlan/gre seg = 172.24.17.0/24 |  compute  |      | controller |
    |    node    | data segment = 172.16.2.0/24  |   node    |      |    node    |
    +------------+ 172.16.2.13       172.16.2.12 +-----------+      +------------+      
    | eth0 10.200.8.13                                              | eth0 10.200.8.11
    |                                                               |
    |                                                               |
    +--------------------------------------------+------------------+-----------------
    |       public segment 10.200.8.0/24
    |
    | 10.200.8.1
    +-----------+
    | GW Router |-> The Internet
    +-----------+

Quantum は、

* API network
* Public network
* Data network
* Management Network

の4つのネットワークセグメントを前提に設計されています。4つ用意するのが大変なの
で今回は

* API / Management network
* Public Network
* Data Network

と API と Management を兼務させた3つのネットワークセグメントを前提に話続けます。
もちろん Management を追加で切り出しても構わないです。NIC を追加するだけで OK
。

詳しくは、

<http://docs.openstack.org/trunk/openstack-network/admin/content/connectivity.html>

に掲載されています。

また今回は public network に 10.200.8/24 を使ってます。サービス環境ではここが
グローバルセグメントになります。

インストール対象
----

OS は Ubuntu Server 12.04 LTS もしくは 12.10 で動作します。

3つの NIC があるマシンを1台、NIC 2つのマシンを2台を用意します。
controller node x 1台 network node x 1台 + comupte node x n台 の構成で
す。

* controller : glance, keystone, mysql, horizon, quantum server が稼働する Node
* network : quantum dhcp agent, quantum l3 agent, quantum openvswitch agent が稼働する Node
* compute : nova, quntum openvswitch agent が稼働する Node

上記の構成では下記の通り /etc/network/interface を設定します。controller 側の
設定です。

    auto lo
    iface lo inet loopback
    
    auto eth0
    iface eth0 inet static
        address 10.200.8.11
        netmask 255.255.255.0
        dns-nameservers 8.8.8.8 8.8.4.4
        dns-search cpi.ad.jp
    
    auto eth2
    iface eth2 inet static
        address 172.16.1.11
        netmask 255.255.255.0
        gateway 172.16.1.1

Network Node は...

    auto lo
    iface lo inet loopback
    
    auto eth0
    iface eth0 inet static
        up ifconfig $IFACE 0.0.0.0 up
        up ip link set $IFACE promisc on
        down ip link set $IFACE promisc off
        down ifconfig $IFACE down
        address 10.200.8.21
        netmask 255.255.255.0
        #gateway 10.200.8.1
        # dns-* options are implemented by the resolvconf package, if installed
        dns-nameservers 8.8.8.8 8.8.4.4
        dns-search cpi.ad.jp
    
    auto eth1
        iface eth1 inet static
        address 172.16.2.13
        netmask 255.255.255.0
    
    auto eth2
    iface eth2 inet static
        address 172.16.1.13
        netmask 255.255.255.0
        gateway 172.16.1.1
        dns-nameservers 8.8.8.8 8.8.4.4

compute Node は...

    auto eth1
    iface eth1 inet static
        address 172.16.2.12
        netmask 255.255.255.0
    
    auto eth2
    iface eth2 inet static
        address 172.16.1.12
        netmask 255.255.255.0
        gateway 172.16.1.1
        dns-nameservers 8.8.8.8 8.8.4.4

です。下記のコマンドでネットワークインターフェースを再起動してください。
ここまで用意出来たらいよいよ実行するのみです。

    controller% sudo /etc/init.d/networking restart
    network   % sudo /etc/init.d/networking restart
    compute   % sudo /etc/init.d/networking restart

スクリプト実行
----

下記の通りスクリプトを取得して...

    controller% git clone https://github.com/jedipunkz/openstack_folsom_deploy.git
    controller% cd openstack_folsom_deploy

それぞれの構成に合わせて deploy.conf を修正します。上記の構成の場合下記のようになります。

    BASE_DIR=`pwd`
    CONTROLLER_NODE_IP='172.16.1.11'
    CONTROLLER_NODE_PUB_IP='10.200.8.11'
    NETWORK_NODE_IP='172.16.1.12'
    COMPUTE_NODE_IP='172.16.1.13'
    DATA_NIC_COMPUTE='eth1'
    
    MYSQL_PASS='secret'
    CINDER_VOLUME='/dev/sda6'
    DATA_NIC='eth1'
    PUBLIC_NIC='eth0'
    
    NETWORK_TYPE='gre'
    INT_NET_GATEWAY='172.24.17.254'
    INT_NET_RANGE='172.24.17.0/24'
    EXT_NET_GATEWAY='10.200.8.1'
    EXT_NET_START='10.200.8.36'
    EXT_NET_END='10.200.8.40'
    EXT_NET_RANGE='10.200.8.0/24'
    
    OS_IMAGE_URL="https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img"
    OS_IMAGE_NAME="Cirros 0.3.0 x86_64"

環境に合わせて設定すれば OK です。上記の前提の構成の場合このような値を入れてい
きます。重要なところだけ説明すると..


* CONTROLLER_NODE_IP : Controller Node の IP アドレス
* NETWROK_NODE_IP : Network Node の IP アドレス
* COMPUTE_NODE_IP : Compute Node の IP アドレス
* INT_NET_.. : Quantum で管理する内部ネットワーク
* EXT_NET-.. : Quantum で管理する外部ネットワーク

です。

deploy.conf を修正したら、各 Node にディレクトリをコピーします。

    
    network% scp -r <CONTROLLER_NODE_IP>:~/openstack_folsom_deploy .
    compute% scp -r <CONTROLLER_NODE_IP>:~/openstack_folsom_deploy .

いよいよ実行。それぞれの Node で順にスクリプトを実行します。

    controller% sudo ./deploy.sh controller quantum
    network   % sudo ./deploy.sh network quantum
    compute   % sudo ./deploy.sh compute quantum

また構築が終了したら quantum 上にネットワークを作成します。

    controller% sudo ./deploy.sh create_network qunatum

完成です。http://${CONTROLLER_NODE_IP}/horizon/ にアクセスすれば管
理画面が表示されるはずです。


更に compute node を追加したければ...

    compute02% scp -r <CONTROLLER_NODE_IP>:~/openstack_folsom_deploy .
    compute02% cd openstack_folsom_deploy
    compute02% vim deploy.conf # $COMPUTE_NODE_IP を修正する。その他はそのまま。
    compute02% sudo ./deploy.sh compute quantum

と deploy.conf 内 $COMPUTE_NODE_IP を更新して実行すれば OK です。

Floating IP の利用
----

現在、folsom リリース版には floating ip が horizon 経由で利用できない問題があ
ります。コマンドラインでは利用できるのでその方法を。

    % source $HOME/openstackrc
    % quantum net-list
    % quantun floatingip-create <ext_net_id>
    % quantun floatingip-list
    % quantum port-list
    % quantum floatingip-associate <floatingip_id> <vm_port_id>

Quantum と I/O と..所感
----

<http://www.readability.com/read?url=http://docs.openstack.org/trunk/openstack-network/admin/content/services.html>

最近、メーリングリストで挙がっている話題。今回の構成だと Quantum は controller
上で稼働し全ての VM がこの Quantum を利用することになります。つまり単一障害点っ
ていうだけではなく I/O が集中するので負荷も上昇する。前リリース版 ESSEX の
nova-network の時は追加する compute node 上全てで nova-network を稼働させ、
node が増えるにつれ I/O も拡張出来るシンプルな構成が組めるモノだったのですが、
Quantum の構成になって、そう簡単にいかなくなった。

オールインワン構成等、ほかの構成について
----

2013/01/09 に nova-network にも対応しました。

オールインワン構成や quantum に代わって nova-network を使う構成等、その他の構成構築方法
については下記のドキュメントを参考にしてください。

<https://github.com/jedipunkz/openstack_folsom_deploy/blob/master/README_jp.md>

