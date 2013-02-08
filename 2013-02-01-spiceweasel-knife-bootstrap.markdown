---
layout: post
title: "Spiceweasel で knife バッチ処理"
date: 2013-02-01 15:48
comments: true
categories: chef
---
Spiceweasel <https://github.com/mattray/spiceweasel#cookbooks> を使ってみた。

Spiceweasel は Chef の cookbook のダウンロード, role/cookbook の chef server
へのアップロード, ブートストラップ等をバッチ処理的に行なってくれる(もしくはコ
マンドラインを出力してくれる)ツールで、自分的にイケてるなと感じたのでブログに
書いておきます。

クラウドフェデレーション的サービスというかフロントエンドサービスというか、複数
のクラウドを扱えるサービスは増えてきているけど、chef を扱えるエンジニアであれ
ば、この Spiceweasel で簡単・一括デプロイ出来るので良いのではないかと。

早速だけど chef-repo にこんな yamp ファイルを用意します。

    cookbooks:
    - apt:
    - nginx:
    
    roles:
    - base:
    
    nodes:
    - 172.24.17.3:
        run_list: role[base]
        options: -i ~/.ssh/testkey01 -x root -N webset01
    - 172.24.17.4:
        run_list: role[base]
        options: -i ~/.ssh/testkey01 -x root -N webset02

上から説明すると...

* 'apt', 'nginx' の cookbook を opscode レポジトリからダウンロード
* 'apt', 'nginx' の cookbook を chef-server へアップロード
* roles/base.rb を chef-server へアップロード
* 2つのノードに対して bootstrap 仕掛ける

ってことをやるためのファイルです。予め chef-repo と roles は用意してあげる必要
があります。この辺りは knife の操作のための準備と全く同じ。また Spiceweasel は、
この yaml フィアル内の各パラメータや指定した role の内容の依存関係をチェックし
てくれます。

では、このファイルに対して

    % spiceweasel <yaml ファイル名>

すると、結果として下記のようなバッチが取得出来る。

    knife cookbook site download apt  --file cookbooks/apt.tgz
    tar -C cookbooks/ -xf cookbooks/apt.tgz
    rm -f cookbooks/apt.tgz
    knife cookbook upload apt
    knife cookbook site download test  --file cookbooks/test.tgz
    tar -C cookbooks/ -xf cookbooks/test.tgz
    rm -f cookbooks/test.tgz
    knife cookbook upload test
    knife role from file base.rb
    knife bootstrap 172.24.17.3 -i ~/.ssh/testkey01 -x root -N webset01 -r 'role[base]'
    knife bootstrap 172.24.17.4 -i ~/.ssh/testkey01 -x root -N webset02 -r 'role[base]'

尚且つ -e オプションを指定すると、実際にこれらのバッチを実行出来る。cookbooks
ディレクトリに予めレシピが存在すればダウンロードのバッチは省略されるぽいし、-d
を指定すると逆にノード削除バッチ処理、-r でリビルドのためのバッチ処理が得られ
る。

削除系・リビルド系は現時点では不具合が見られました。実行すると他の環境にも影響
出るので注意が必要。

個人的にはプライベートレポジトリの cookbooks も取ってこれるようになると嬉しい。
まぁ、Berkshelf 使えばいいのだけど。<http://berkshelf.com/>

開発者の Matt Ray さんの資料によれば Chef for OpenStack もこの Spiceweasel を
使う方向で修正が掛かったらしい。個人的には一番興味あるところ。ちなみに Chef
for OpenStack は folsom ベースが現在 'active development' 状態らしい。

Opscode のサイトで紹介されているサンプルは古いバージョンでの指定方法らしく、注
意が必要です。

<http://wiki.opscode.com/display/chef/Spiceweasel>

#### 所感

chef, knife 周りはいろんな関連技術があるので技術を選定する上で迷ってしまうこと
が多いのだけど、この Spiceweasel には可能性を感じました。って言うのは、Chef が
実装出来ていないインテグレーションやオーケストレーションっていう所まで踏み込め
る可能性があるから。ノード間の関連付けが出来るんです。Swift-Storage と
Swift-Proxy の関連付け、または Load-Balancer と HTTP-Server の関連付け等。継続
的デリバリなインテグレーションって Chef を使ってどう実現するんだ？って思ってい
た時があったのですが、こういったラッパーツールの登場で解決されそうな気がします。

