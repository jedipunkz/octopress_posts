---
layout: post
title: "OpenStack Folsom on Thinkpad"
date: 2013-01-12 16:46
comments: true
categories: openstack
---
以前紹介した OpenStack Folsom 構築 bash スクリプトなのだけど quantum の代わり
に nova-network も使えるようにしておいた。

構築 bash スクリプトは、

<https://github.com/jedipunkz/openstack_folsom_deploy/blob/master/README_jp.md>

に詳しい使い方を書いておきました。またパラメータを修正して実行するのだけどパラ
メータについては、

<https://github.com/jedipunkz/openstack_folsom_deploy/blob/master/README_parameters_jp.md>

に書いておきました。

<img src="http://jedipunkz.github.com/pix/openstack_folsom_thinkpad.jpg">

手持ちの Thinkpad に OpenStack folsom 入れた写真。この写真の OpenStack Folsom
を構築した時の手順を書いておくよ。

#### OS をインストール

OS をインストールします。12.10 を使いました。(12.04 LTS でも可)。/dev/sda6 等、
Cinder 用に一つパーティションを作ってマウントしないでおきます。また固定 IP ア
ドレスを NIC に付与しておきます。

#### スクリプト取得

スクリプトを取得する。

    % sudo apt-get update; sudo apt-get install git-core
	% git clone https://github.com/jedipunkz/openstack_folsom_deploy.git
	% cd openstack_folsom_deploy

#### パラメータ修正

deploy_with_nova-network.conf 内のパラメータを修正します。オールインワン構成な
ので、ほぼほぼ修正せずに実行しますが

    HOST_IP='<Thinkpad の IP アドレス>'

だけ修正。

#### 実行...

実行する。

    % sudo ./deploy.sh allinone nova-network

... 最近 ./deploy.sh create_network nova-network を実行しなくて済むようにしました。

#### Horizon にアクセスする

ブラウザで http://localhost/horizon にアクセスすれば horizon にアクセス出来る。
ユーザ情報は...

    user : demo
	pass : demo

です。Horizon から後でユーザ情報変更してもらって構わないです。

#### コマンドラインからアクセスする。

実行したユーザのホームディレクトリに ~/openstackrc がある。

* ~/openstackrc     # admin アカウント用
* ~openstackrc-demo # demo ユーザ用

読み込んで OpenStack のコマンドを実行する。下記は例です。

    % source ~/openstackrc-demo
	% nova list
	+--------------------------------------+----------+--------+---------------------------------+
	| ID                                   | Name     | Status | Networks                        |
	+--------------------------------------+----------+--------+---------------------------------+
	| 81730be5-f2b3-411c-bfef-9a879e3d7d56 | grievous | ACTIVE | private=10.0.0.5, 192.168.1.195 |
	| 0ba8416a-58bd-476d-96a8-2ecc37ae53e6 | luke     | ACTIVE | private=10.0.0.2, 192.168.1.194 |
	| 5b1f23b3-c15e-4260-bf6c-f3e965bcd546 | r2d2     | ACTIVE | private=10.0.0.4, 192.168.1.193 |
	+--------------------------------------+----------+--------+---------------------------------+

#### 次のリリース版は...

bash で書いても...満足感得られないしコード汚くなるし、人に読んでもらえないし。
ruby で書いても perl で書いても同じ。コマンドをバシバシ打つインフラ系のコード
は汚くなるしまともにテストも出来ない。これからはインフラ系もコードを書く時代だっ
て言っておいてこれじゃアカンぉ。

ならば情報が整理されるって意味だけでも chef のようなフレームワークを使う意味は
大きい。資源を他の人に有効活用してもらえるチャンスも増えるし。chef のクックブッ
クなんてコードじゃないってアプリエンジニアに言われようが、やっぱりフレームワー
ク使うべきだし使いたい。次の OpenStack のリリースの時は Chef のクックブック作
るって決めたよぉー。why-run 出来たり、テストも出来るしね。


