---
layout: post
title: "OpenFlow Trema ハンズオン参加レポート"
date: 2012-11-21 21:46
comments: true
categories: openflow
---
InternetWeek2012 で開かれた "OpenFlow Trema ハンズオン" に参加してきました。

    講師   : Trema 開発チーム 鈴木一哉さま, 高宮安仁さま
	開催日 : 2012年11月21日

OpenStack の Quantum Plugin として Trema が扱えるという話だったので興味を持っ
たのがきっかけです。また Ruby で簡潔にネットワークをコード化出来る、という点も
個人的に非常に興味を持ちました。OpenStack, CloudStack 等のクラウド管理ソフトウェ
アが提供する API といい、Opscode Chef, Puppet 等のインフラソフトウェア構築フレー
ムワークといい、この OpenFlow もインフラを形成する技術を抽象化し、技術者がコー
ドを書くことでインフラ構築を行える、という点ではイマドキだなと思います。

Google は既にデータセンター間の通信を 100% 、OpenFlow の仕様に沿った機器・ソフ
トウェアをを独自に実装しさばいているそうですし、我々が利用する日も近いと想像し
ます。

OpenFlow のモチベーション
----

OpenFlow の登場には理由が幾つかあって、既存のネットワークの抱えている下記の幾
つかの問題を解決するためです。

* 装置仕様の肥大化
* 多様なプロトコルが標準化
* 装置のコスト増大
* ある意味、自律したシステムが招く複雑さ

一方、OpenFlow を利用すると..

* コモディティ化された HW の利用が可能
* OpenFlow はコントローラ (神) が集中管理するので楽な場合もある
* ネットワーク運用の自動化が図れる
* アプリケーションに合わせた最適化
* 柔軟な自己修復

等のメリットが。

OpenFlow と Trema とは?
----

OpenFlow は 'OpenFlow コントローラ', 'OpenFlow スイッチ' から成る。OpenFlow コ
ントローラと OpenFlow スイッチの間の通信は OpenFlow プロトコルでされる。今日の
話題 Trema はこの..

* OpenFlow コントローラのフレームワーク
* エミュレータ
* trema コマンド

のセットである。自宅の PC 一台で OpenFlow プログラムが行え、エミュレーションも
行える手軽さ、また Ruby による簡潔な記述が可能でプログラミング初心者でも扱いや
すい、という趣味ユーザにはもってのほかだ。

Hellow, Trema !
----

早速 プログラミングの初歩、Hello World から。

コード hello-trema.rb は

    class HelloTrema < Controller
	  def start
        puts "Hello, Trema!"
	  end
	end

実行...

    % trema run hello-trema.rb
	Hello, Trema!

Trema が提供する Controller クラスを '継承' し HelloTrema クラスを定義した。
Controller クラスには幾つものハンドラが用意されていて start ハンドラもその一つ。
他にも色んなメソッドが用意されている。詳しくは後ほど。

Trema によるスイッチの起動
----

次にスイッチを起動してみる。Trema は ruby の DSL でコンフィギュレーションを定
義出来る。hello-switch.conf として下記の内容...

    vswitch { dpid "0xabc" }
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }

hello-switch.rb として

    class HelloSwitch < Controller
	  def switch_ready dpid
	    puts "Hello #{ dpid.to_hex }!"
	  end
      def switch_disconnected dpid
        puts "Killed ! :D #{ dpid.to_hex }!"
	  end
	end

実行すると

    % trema run hello-switch.rb -c hello-switch.conf
	Hello 0xabc!
	Hello 0x1!
	Hello 0x2!

となる。スイッチを3つ起動したわけだ。switch_ready とは Trema が提供するコント
ローラで定義された "スイッチが稼働した時に実行されるハンドラ" だ。スイッチが起
動したため "Hello .." なるメッセージが出力された、と理解すればいい。

またこの起動中に

    % trema kill 0x1
	Killed ! :D 0x1!

と実行することでスイッチを停止出来る。この際 switch_disconnected ハンドラが実
行され上記のメッセージを出力したというわけだ。また逆に再稼働させるには trema
up コマンドを用いる。

その他のハンドラ一覧
----

紹介した start, switch_ready 等のハンドラ以外にも下記のモノがある。

    start switch_ready switch_disconnected packet_in flow_removed port_status
    openflow_error features_reply stats_reply barrier_reply get_config_reply
    queue_get_config_reply vendor

ドキュメントは

    http://rubydoc.info/github/trema/trema/master/frames

にあるので参照すると良い。

L2 スイッチの実装
----

OpenFlow はネットワーク機器を実装出来るモノなので、ここで L2 スイッチを実装し
てみたい。L2 スイッチの動作は

* 既に ARP テーブルを持っていればパケットを受け流す
* 自分の ARP テーブルで管理されていないパケットは学習してからパケットを受け流
  す

が基本だ。これを実装する。

l2-switch.conf として

    vswitch { dpid "0xabc" }
    
    vhost ("host1") {
      ip "192.168.0.1"
      netmask "255.255.0.0"
      mac "00:00:00:01:00:01"
    }
    
    vhost ("host2") {
      ip "192.168.0.2"
      netmask "255.255.0.0"
      mac "00:00:00:01:00:02"
    }
    
    link "0xabc", "host1"
    link "0xabc", "host2"

ここでは detapath_id "0xabc" なる仮想スイッチを一つ定義し、サーバホスト host1,
host2 を定義した。それぞれで IP アドレス・MAC アドレスを定義している。また
link により仮想スイッチとサーバホストの I/F を接続している。

いよいよ L2 スイッチのコード。l2-switch.rb として、下記を記述する。

    require "fdb"
    
    class LearningSwitch < Controller
      def start
        @fdb = FDB.new
      end
	  
      def packet_in dpid, message
        @fdb.learn message.macsa, message.in_port
        port_no = @fdb.lookup( message.macda )
        if port_no
          flow_mod dpid, message, port_no
          packet_out dpid, message, port_no
        else
          flood dpid, message
        end
      end
    
      def flow_mod dpid, message, port_no
        send_flow_mod_add(
          dpid,
          :match => ExactMatch.from( message ),
          :actions => ActionOutput.new( port_no )
        )
      end
	  
      def flow_mod dpid, message, port_no
        send_flow_mod_add(
          dpid,
          :match => ExactMatch.from( message ),
          :actions => ActionOutput.new( port_no )
        )
      end
	  
      def packet_out dpid, message, port_no
        send_packet_out(
          dpid,
          :packet_in => message,
          :actions => ActionOutput.new( port_no )
        )
      end
    
      def flood dpid, message
        packet_out dpid, message, OFPP_FLOOD
      end
    end

まず実行してみる。

    % trema run l2-switch.rb -c l2-switch.conf

異なる shell でパケットの送信と状態表示を行う。

    % trema send_packet --source host1 --dest host2
	% trema show_stats host1
	ip_dst,tp_dst,ip_src,tp_src,n_pkts,n_octets
	192.168.0.2,1,192.168.0.1,1,1,50
    % trema send_packet --source host1 --dest host2
    % trema send_packet --source host2 --dest host1
	% trema dump_flows 0xabc
    NXST_FLOW reply (xid=0x4):
     cookie=0x2, duration=67.343s, table=0, n_packets=0, n_bytes=0, priority=65535,udp,in_port=1,vlan_tci=0x0000,dl_src=00:00:00:01:00:02,dl_dst=00:00:00:01:00:01,nw_src=192.168.0.2,nw_dst=192.168.0.1,nw_tos=0,tp_src=1,tp_dst=1 actions=output:2
     cookie=0x1, duration=70.339s, table=0, n_packets=0, n_bytes=0,	priority=65535,udp,in_port=2,vlan_tci=0x0000,dl_src=00:00:00:01:00:01,dl_dst=00:00:00:01:00:01,nw_src=192.168.0.1,nw_dst=192.168.0.1,nw_tos=0,tp_src=1,tp_dst=1 actions=output:2

host1 から host2 に対して trema send_packet で通信を行い、host1 の状態を表示し
た。また交互に通信をさせフローをダンプしたのが上記だ。

一番基本なところらしいので、コードを詳しく解説。l2-switch.rb の下記の部分。

      def packet_in dpid, message                  # ---(1)
        @fdb.learn message.macsa, message.in_port  # ---(2)
        port_no = @fdb.lookup( message.macda )     # ---(3)
        if port_no                                 # ---(4)
          flow_mod dpid, message, port_no
          packet_out dpid, message, port_no
        else                                       # ---(5)
          flood dpid, message
        end
      end

(1) では packet_in ハンドラを利用した。(2) で fdb (floating DB) の learn メソッ
ドで port と mac アドレスの学習を行った。(3) で @fdb にすでに mac アドレスの記
述があれば port_no に値が入る。値が入っていれば (4) を。スイッチのフローテーブ

ルを更新しパケットを packet_out する。入っていなければ (5)を実行しパケットを
flood する。

flow_mod, packet_out, flood はこのプログラム内で定義しているプライベートなメソッ
ドだ。

この時のソフトウェア構成
----

このコードを動作させた際に実行したホスト上のプロセスを見てみた。

* ovs-openflowd
* phost
* switch_manager

が居た。siwtch_manager が 6633 番ポート待ち受けているコントローラ自身だろう。
phost は仮想サーバで ovs-openflowd (OpenvSwitch) が仮想スイッチとして動作して
いると想像出来る。またこの時にネットワークインターフェースが

* trema0-0
* trema0-1
* trema1-0
* trema1-1

と起動していた。これは 0xabc スイッチと host1, host2 との接続で使われている
I/F だろう。

まとめ
----

Ruby で記述出来るので技術者にとって Trema は優しい。また簡潔な記述が行えるとい
うのも Ruby ならではだろう。github.com には高度な利用サンプルが幾つか掲載され
ている。

    https://github.com/trema/apps

また簡単なサンプル集ということであれば

    https://github.com/trema/trema/tree/develop/src/examples

がうってつけのリファレンスになる。

冒頭でも書いたがスイッチのエミュレータが同封されているので、技術者はすぐに開発
に入ることが出来る。

OpenStack は仮想マシンを管理・構成するため API を提供し、そしてネットワークを管理・
構成するため OpenFlow がある。OpenStack の API を叩くコードを書くのと同様に
OpenFlow を Trema という OpenFlow コントローラフレームワークを用いてコードを書
くことが出来る。インフラエンジニアの仕事の範囲は確実にここ数年で変化し、その変
化に追いつくには "コードを書く" ことを念頭に置かなくてはならないだろう。

最後に、この機会を与えてくださった 鈴木様・高宮様にお礼を申し上げます。


