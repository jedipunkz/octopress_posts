---
layout: post
title: "#DevLOVE に参加してきました。Chef De DevOps"
date: 2012-07-22 20:05
comments: true
categories: DevOps
---
2012年07月21日に大崎のフィーチャーアーキテクトさんで行われた #DevLOVE (Chef De
DevOps) に参加してきました。

    開催日 : 2012年07月21日(土曜日) 15:00 - 20:40
    場所   : 大崎 フューチャーアーキテクトさま
	URL    : http://www.zusaar.com/event/314003

仕事場でも Chef の利用を考え始めていて今回いい機会でした。半年前と比べるとだい
ぶ揃って来ましたが、まだまだ資料の少ない Chef。貴重な機会でした。

プログラムは下の通り。

    *『Chefの下準備』by Ryutaro YOSHIBA [ @ryuzee ]
    *『Chef自慢のレシピ披露』 by 中島弘貴さん [ @nakashii_ ]
    * ワークショップ『みんなでCooking』
    * dialog『試食会』
    * Niftyさんのスーパー宣伝タイム!!!
    *『渾身会』

@ryuzee さんの "Chef の下準備" は SlideShare に使った資料が公開されています。

<iframe src="http://www.slideshare.net/slideshow/embed_code/13712176"
width="427" height="356" frameborder="0" marginwidth="0" marginheight="0"
scrolling="no" style="border:1px solid #CCC;border-width:1px 1px
0;margin-bottom:5px" allowfullscreen> </iframe> <div
style="margin-bottom:5px"> <strong> <a
href="http://www.slideshare.net/Ryuzee/20120721-chef-devlove" title="20120721
chefの下準備 #devlove" target="_blank">20120721 chefの下準備 #devlove</a>
</strong> from <strong><a href="http://www.slideshare.net/Ryuzee"
target="_blank">Ryuzee YOSHIBA</a></strong> </div>

印象的だったのが、VirtualBox + Vagrant という環境。実際に使っていらっしゃいま
した。MacBook の中に仮想環境とそのインターフェースである Vegrant を使って、デ
プロイのテスト等が実施できるそうです。また、Vegrant 設定ファイルは Chef のレシ
ピを自動読込して、常に本番環境と同じ状態にしているそうです。CloudFormation と
いうキーワードや Capistrano というキーワードが出てきました。最近よく耳にするワー
ドです。また CI は Jenkins だよね、だったり。

3番目のプログラム "ワークショップ、みんなで Cooking" は 4-6 人の組みになり、実
際に Chef のレシピを書いてみようというもの。今回は wordpress の構築のためのレ
シピを書いていきました。僕らの組みも時間ギリギリでレシピの投入を終えました。
wordpress の設定ファイル wp-config.php と mysql への wordpress 用データベース
の作成を実際に attribute, template, script ら Resources を使ってレシピを書きま
した。簡単な例でしたが、みなとコミュニケーションがとれたのでいい機会でした。

dialog "試食会" では、グループ毎、またグループを入れ糧のディスカッション。使っ
てみてどうだったか？を話し合ってそこから議論を伸ばしていきました。

やっぱり僕も参加する前からそうだったのですが、一番皆が気になっているのが、

    "chef-solo + capistrano ? それとも chef-server がいいの？"

でした。実際に使っていらっしゃる CA さんの話によると、20台までは chef-solo +
capistrano の構成が良いと。そして100台規模になると chef-server が良いそうです。
具体的に何が chef-solo で問題になるのかが聞けなかったのですが、ちょっと僕らの
課題にしてみようかと思います。また、CA さんでは puppet を使っていらっしゃる部
署もあるそうで、chef-server の動作が不安定？とかで400台規模になると puppet が
良くなると。そして chef-server はいまは rabbitmq, couchDB, erlang などの構成で
出来ているのですが、そのうちどれかが (erlang の solr ?) が mysql に置き換わる
と噂を聞きました。

また、nifty さんが現在 opscode chef wiki を翻訳されているのですが、僕も非常に
興味があります。参加させてもらおうかな。また nifty さんが nifty cloud 用
knife-ec2 を公開しているそうで説明がありました。自社にクラウドシステムがあると、
色々楽しめるよなぁ...。いいな。

[2012/07/23 15:00 追記]
nifty の @tily さんが当日使った資料を公開なさったので貼り付けておきます。

<iframe src="http://www.slideshare.net/slideshow/embed_code/13721102"
width="427" height="356" frameborder="0" marginwidth="0" marginheight="0"
scrolling="no" style="border:1px solid #CCC;border-width:1px 1px
0;margin-bottom:5px" allowfullscreen> </iframe> <div
style="margin-bottom:5px"> <strong> <a
href="http://www.slideshare.net/tidnlyam/chef-13721102" title="ニフティ社内の
Chef 利用について" target="_blank">ニフティ社内の Chef 利用について</a>
</strong> from <strong><a href="http://www.slideshare.net/tidnlyam"
target="_blank">tidnlyam</a></strong> </div>

以上が "Chef De DevOps" の参加レポートで、ここからは僕らの課題でありここ1週間
くらいで取り組もうとしている点。

chef-server の構築ですが、ubuntu + opscode の deb package を使う方法もあるので
すが、これだと opscode が推奨している ruby のバージョン 1.9.2 ではなく 1.8.7
になってしまいます。これは ubuntu, debian の stable release もしくは
development release が frozen しているからです。なので、今僕が考えているのは、
chef-solo と opscode bootstrap の組み合わせで構築する方法です。

[http://wiki.opscode.com/display/chef/Installing+Chef+Server+using+Chef+Solo](http://wiki.opscode.com/display/chef/Installing+Chef+Server+using+Chef+Solo)

この bootstrap は opscode が chef-server 構築用レシピとして公開しているもので
す。これを使えば chef client さえ入っていれば構築できてしまうという。この手順
は一度手元でも確認してみました。(Ubuntu Server 12.04 LTS amd64).

また Chef のメーリングリストで聞いてみたところ、やはり推奨の ruby バージョンは
1.9.2 であって、この bootstrap の方法と、あとは knife::server というものです。

まだ僕は手元で確認できていないのですが、こちらにあるようです。

[http://fnichol.github.com/knife-server/](http://fnichol.github.com/knife-server/)

chef-server 構成だと rabbitmq に queue が溜まったり、couchDB のディスクを溢れ
させたりとトラブル起きた時の切り分けだったり対処方法だったり、まだまだ情報少な
いので苦労するだろうなぁと個人的には感じました。chef-solo + capistrano の構成
も考慮しつつ、このあたり調べて行こうと思います。
