---
layout: post
title: "OpenStack API を理解しインフラエンジニアの仕事の変化を感じる"
date: 2012-12-08 07:50
comments: true
categories: openstack
---
今日は "OpenStack Advent Calendar 2012 JP" というイベントのために記事を書きた
いと思います。Advent Calendar とはキリスト生誕を祝うため 12/25 まで毎日誰かがブログ
等で特定の話題について述べるもの、らしいです。CloudStack さん, Eucalyptus さん
も今年はやっているそうですね。

イベントサイト : <http://atnd.org/events/34389>

では早速！(ただ..CloudStack の Advent Calendar とネタがかぶり気味です..。)

御存知の通り OpenStack は API を提供していてユーザがコードを書くことで
OpenStack のコマンド・Horizon で出来ることは全て可能です。API を叩くのに幾つか
フレームワークが存在します。

* fog
* libcloud
* deltacloud

などです。

ここでは内部で fog を使っている knife-openstack を利用して API に触れてみよう
かと思います。API を叩くことを想像してもらって、インフラエンジニアの仕事内容の
変化まで述べられたらいいなぁと思っています。

OpenStack 環境の用意
----

予め OpenStack 環境は揃っているものとしますです。お持ちでなければ

<http://jedipunkz.github.com/blog/2012/11/10/openstack-folsom-install/>

この記事を参考に環境を作ってみて下さい。あ、devstack でも大丈夫です。

chef, knife-openstack の用意
----

chef, knife-openstack を入れるのは OpenStack 環境でも、別のノードでも構いません。

chef が確か 1.9.2 ベースが推奨だったので今回は 1.9.2-p320 使います。
ruby は rbenv で入れるのがオススメです。knife-openstack, chef のインストールは...

    % sudo apt-get install libreadline-dev libxslt1-dev libxml2-dev
	% gem install chef --no-rdoc --no-ri
	% gem install knife-openstack --no-rdoc --no-ri
	% rbenv rehash # rbenv を使っている際に実行..

次に knife.rb を用意します。情報として下記を

* OS_USERNAME : :openstack_username
* OS_PASSWORD : :openstack_password
* OS_AUTH_URL : :openstack_auth_url
* OS_TENANT_NAME : :openstack_tenant

追加します。例として下記を参考にしてください。

    % mkdir .chef
    % ${EDITOR} .chef/knife.rb
	
    knife[:openstack_username] = "demo"
    knife[:openstack_password] = "demo"
    knife[:openstack_auth_url] = "http://172.16.1.11:5000/v2.0/tokens"
    knife[:openstack_tenant] = "service"

ここで注意なのが OS_AUTH_URL がいつもコマンドラインで扱うものと違い /tokens が
付いています。fog を直に扱う時も同じですがこれが必要です。

ssh keypair の用意
----

ssh keypair が必要になってくるので用意します。

    % nova keypair-add testkey01 > testkey01

knife-openstack の操作方法
----

いよいよ knife-openstack を使って OpenStack を操作してみましょう。

まずは flavor のリストを取得します。

    % knife openstack flavor list
	ID  Name       Virtual CPUs  RAM       Disk
	1   m1.tiny    1             512 MB    0 GB
	2   m1.small   1             2048 MB   20 GB
	3   m1.medium  2             4096 MB   40 GB
	4   m1.large   4             8192 MB   80 GB
	5   m1.xlarge  8             16384 MB  160 GB

image リストを取得します。id が必要になります。

    % knife openstack image list
	ID                                    Name
	436deba5-8fab-4bb7-9205-41e33fe22744  Cirros 0.3.0 x86_64

VM を生成してみましょー。

    % knife openstack server create -f 1 -I 436deba5-8fab-4bb7-9205-41e33fe22744 -S testkey01 -N knifetest01
	Instance Name: knifetest01
	Instance ID: 1e20850d-9572-46c8-a41e-fdf56f4f65a7
	SSH Keypair: testkey
	
	Waiting for server............
	Flavor: 1
	Image: 436deba5-8fab-4bb7-9205-41e33fe22744

出来たかどうか、チェック。

    % knife openstack server list
	Instance ID                           Name         Public IP  Private IP  Flavor  Image                                 Keypair   State
	1e20850d-9572-46c8-a41e-fdf56f4f65a7  knifetest01                         1       436deba5-8fab-4bb7-9205-41e33fe22744  hogehoge  active

できました。逆に VM を削除するには

    % knife openstack server delete 1e20850d-9572-46c8-a41e-fdf56f4f65a7
	Instance ID: 1e20850d-9572-46c8-a41e-fdf56f4f65a7
	Instance Name: knifetest01
	Flavor: 1
	Image: 436deba5-8fab-4bb7-9205-41e33fe22744
	
	Do you really want to delete this server? (Y/N) y
	WARNING: Deleted server 1e20850d-9572-46c8-a41e-fdf56f4f65a7
	WARNING: Corresponding node and client for the 1e20850d-9572-46c8-a41e-fdf56f4f65a7 server were not deleted and remain registered with the Chef Server

です。

残念なところとしては knife-openstack 自体はまだまだ機能が充実していません。VM
の基本的な操作くらいしか出来ないので、追加実装したいという方がいらっしゃいまし
たら Pull リクエスト送ると良いのではないでしょうか。

knife-openstack 公式サイト : <https://github.com/opscode/knife-openstack>

まぁ、ここまで書いてアレですが.. 気がついた方もいらっしゃると思います。
knife-openstack で出来ることは openstack コマンド群で全て出来るのであまり意味
はないですよね。ただ fog を使って API を叩くことを想像して欲しくて..( -_- )

Fog 単体で操作してみる
----

今日はまだまだ書ける！

knife-openstack では基本的な操作しか出来ませんでしたが fog はより多くの機能を実装しています。(2012/12/08 現在 quantum 周りの開発は未
完成らしいです。floating-ip 周りがぁ。誰かコミットして。)

fog 単体で OpenStack API を操作するには、下記の通り実行します。fog をインストー
ルし...

    % gem install fog --no-rdoc --no-ri
	% rbenv rehash

環境変数を入力し... (情報は例です)

    % cat env
	export OS_TENANT_NAME=service
	export OS_USERNAME=demo
	export OS_PASSWORD=demo
	export OS_AUTH_URL="http://172.16.1.11:5000/v2.0/"
	export OS_AUTH_URL_FOG="http://172.16.1.11:5000/v2.0/tokens"
	% source env

コードを下記のように記述すると VM の生成が行えます。
``` ruby
    #!/usr/bin/env ruby
    
    require 'fog'
    require 'pp'
    
    conn = Fog::Compute.new({
      :provider => 'OpenStack',
      :openstack_api_key => ENV['OS_PASSWORD'],
      :openstack_username => ENV["OS_USERNAME"],
      :openstack_auth_url => ENV["OS_AUTH_URL_FOG"],
      :openstack_tenant => ENV["OS_TENANT_NAME"]
    })
    
    flavor = conn.flavors.find { |f| f.name == 'm1.tiny' }
    
    image_name = 'Cirros 0.3.0 x86_64'
    image = conn.images.find { |i| i.name == image_name }
    
    puts "#{'Creating server'} from image #{image.name}..."
    server = conn.servers.create :name => "fogvm-#{Time.now.strftime '%Y%m%d-%H%M%S'}",
                                 :image_ref => image.id,
                                 :flavor_ref => flavor.id,
                                 :key_name => 'testkey01'
    server.wait_for { ready? }
````

実行 !

    % ruby <CODENAME>.rb
	Creating server from image Cirros 0.3.0 x86_64...
    %

出来ました。VM が生成されたか先ほどの knife-openstack で確認してみましょう。

    % knife openstack server list
	Instance ID                           Name         Public IP  Private IP  Flavor  Image                                 Keypair   State
	1e20850d-9572-46c8-a41e-fdf56f4f65a7  knifetest01                         1       436deba5-8fab-4bb7-9205-41e33fe22744  hogehoge  active
    f5c314a3-e32f-498f-984f-b79078d76a5a  fogvm-20121207-104743                         1       aedef2a1-f820-43a6-96cd-f5361d27df3f  testkey01  active


まとめと 考察
----

今回は API をみんな叩いてるねん！ってことに気がついて欲しくて、こんな記事を書
いてみました。実は OpenStack のコマンドも --debug を付けて (Quantum だけは -v)
実行すると OpenStack の API を叩いているメッセージが出力されると OpenStack ユー
ザ会の方から聞きました。是非やっていてください。

    % nova --debug list
	% quantum -v net-list

API を実装するのか、ツールを実装するのか？といった話題が以前の OpenStack
Summit で常に話題になっていたらしいですが、最近は API で決まり、といったと
ころでしょうか。コードを書いてインラフを定義する時代に突入です。ちなみに fog
は AWS も扱えます。また Opscode Chef や Puppet, JuJu 等のデプロイフレームワー
クを使えば VM 上のサービス構築もコードを書くことで出来ます。また、以前ブログに
書いたのですが OpenFlow といった技術を使うとネットワークをコードを書くことで設
計出来る、かもしれない。

<http://jedipunkz.github.com/blog/2012/11/21/openflow-trema-handson-report/>

つまり、コードを書くことで

* サーバ構築
* ネットワーク構築
* サービス構築

を一貫して行えることになります。

インフラエンジニアの僕としては時代の変化に着いて行かねば！という焦りで一杯です。
もちろんレガシなインフラエンジニアも生き残るのでしょうが、働く場所が限られてき
そう。より高度な(低レイヤからの深い？)技術を有している人が限定された場所で成果
を上げていく印象。僕らの様な一般的なインフラエンジニアは OpenStack, AWS, HP
Cloud, CloudStack の様なクラウドインフラを相手にコードを書く、もしくは抽象化さ
れた技術をより扱いやすいソフトウェアを介して構築・管理していくことになるのでしょ
うか。これから覚えることは山積みですが、楽しい時代です。

