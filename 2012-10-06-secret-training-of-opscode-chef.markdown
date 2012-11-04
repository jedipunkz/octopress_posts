---
layout: post
title: "Secret Training of Opscode Chef"
date: 2012-10-06 16:53
comments: true
categories: devops
---
昨日、開かれた "Opscode Chef のシークレットトレーニング" に参加してきました。

場所はうちの会社で KDDI ウェブコミュニケーションズ。主催はクリエーションオンラ
インさんでした。講師は Sean OMeara (@someara) さん。今後 Chef のトレーニングを
日本で開くため、事前に内容についてフィードバックが欲しかったそうで、オープンな
レッスンではありませんでしたが、次回以降、日本でも期待できそうです。

内容は chef の基本・メリット・考え方などを網羅した資料で1時間程進められ、その
後はハンズオンがメインでした。今日は実際にハンズオンの内容を書いていこうかと思
います。

chef workstation 環境は揃っている前提にします。また chef server として opscode
の hosted chef (opscode が提供している chef のホスティングサービス,
chef-server として動作します) を使います。またターゲットホストは当日は ec2 イ
ンスタンスを使いましたが、chef ワークステーションから到達できるホストであれば
何でも良いでしょう。

まずは chef-repo のクローン。講習会で使われたものです。

    git clone https://github.com/opscode/chef-repo-workshop-sysadmin.git chef-repo

予め cookbook が入っています。

次に、manage.opscode.com へアクセスしアカウントを作ります。Free アカウントが誰
でも作れるようになっています。

https://manage.opscode.com へアクセス -> Sign Up をクリック -> アカウント情報
を入力 -> submit -> メールにて verify -> 自分のアカウント名をクリック -> Get a
new key をクリックし <アカウント名>.pem をダウンロード -> create a
organization をクリックし Free を選択し、適当な名前で organization を作成。
validation key と knife.rb をダウンロード

これらで得た3つのファイル (2つの pem とknife.rb) を chef-repo/.chef/ 配下に置
きます。これで準備 OK。knife.rb を見ると、opscode の chef ホスティングにアクセ
スするよう記述があります。

	% cp <somewhere>/knife.rb <somewhere>/<account>.pem <somewhere>/<account>-validation.pem chef-repo/.chef/

トレーニング用 chef-repo には予め幾つかの cookbook と role, data_bag が入って
います。これらを knife を使ってアップロードします。

    % cd chef-repo
    % knife cookbook upload -a
    % knife role from file role/*.rb
	% knife data bag create users
	% knife data bag from file users nagiosadmin.json

そして bootstrap を実行。.chef/bootstrap ディレクトリに chef-full.erb ファイル
が入っていて bash スクリプトになっている。knife bootstrap でこの bash スクリプ
トをターゲットホスト上で実行することになる。中身はと言うと chef 環境をインストー
ルしているようだ。

chef 環境をどうやってノードに入れているかなぁ。preseed 使うかな。手作業じゃ意
味ないしなぁと考えていたのですが、こうやればいいのですね。参考になります。これ
は簡単。

ではいよいよ bootstrap を実行。

    % knife bootstrap <IPADDRESS> -r 'role[base],role[monitoring]' --sudo -x <USER> -P <PASSWD>

IP アドレス、ssh ユーザ名・パスワードは適宜入れてください。これで cookbooks ディ
レクトリ配下の各種 cookbook が実行されました。中身は nagios とそれに依存する
apache2, openssl, mysql, php 等。

次に cookbook を新たにダウンロードし knife upload してみます。

    % knife cookbook site download chef-client
	% tar zxvf chef-client-1.2.0.tar.gz -C cookbooks/
	% knife cookbook upload chef-client

「アカウントの pem がない場合 validation が用いられるが、一旦サーバと通信出来た
ら validation は必要なくなる。それどころか wi-fi パスワードのようなものなので
validation は削除することを強くすすめる」と Sean が言っていました。chef 環境は
すでに bootstrap で入っているものの、validation を削除するための recipe を
base という role に追加し、実行してみます。

Sean の言っていたこと、間違っていたら指摘してください( >< ) 英語だったので自信
ないです。

    % vim role/base.rb
	name "base"
	description "Base role applied to all nodes."
	run_list(
	  "recipe[apt]",
      "recipe[nagios::client]"
      "recipe[chef-client::delete_validation]"
    )
	
	default_attributes(
	  "nagios" => {
	  "server_role" => "monitoring"
	  }
	)

role をアップロード。

    % knife role from file roles/base.rb

ターゲットホストで chef-client を実行。ここで バージョン 10.14 から登場した
why-run を試してから実行。

    target# chef-client -Fdoc -lfatal --color --why-run
	target# chef-client -Fdoc -lfatal --color

もしくはワークステーションから knife ssh を使っても良い。

    % knife search node "role:base" -a cloud.public_ipv4
	% knife ssh "role:base" "sudo chef-client -Fmin" -x ubuntu -P opscodechef -a cloud.public_ipv4

-a cloud.public_ipv4 とは、AWS EC2 や OpenStack, CloudStack 環境ではインスタン
スの eth0 インターフェースにプライベート IP アドレスが付与されているケースが
殆どなため、グローバル IP アドレスを検索し実行している。

大きな流れはこれでおしまい。

トレーニングでは bento, minitest, cucumber-chef 等の紹介があったが、詳細な内容
については見送られた。昼休み1時間をはさんで計7時間の長丁場だったが、chef 初級
を脱するのには最適なトレーニングだった。これからトレーニング内容や種別を組んで
いくそうなので、内容は変わってくるかもしれない。あとでフィードバックをしなく
ちゃ。

個人的には OpenStack に興味を持っているので chef を使って OpenStack をデプロイ
したい。今年6月には 'Chef for OpenStack' というアナウンスがあり、DELL,
RackSpace, HP 等の大企業がパートナーとして参加することになったと発表があった。
今まで openstack cookbook は長くメンテナンスされてこなかったので、これには期待
している。

最後にこの機会を作って下さった クリエーションオンラインさん、opscode の Sean
さん、mr.devops さん、ありがとうございましたー。貴重な体験でした。
