---
layout: post
title: "Opscode Bootstrap を使った Chef-Server 構築"
date: 2012-08-18 16:57
comments: true
categories: chef
---
chef-server の構築は少し面倒だと前回の記事
<http://jedipunkz.github.com/blog/2012/08/18/chef-solo/> に書いたのですが、
opscode が提供している bootstrap を用いると、構築作業がほぼ自動化出来ます。
今回はこの手順を書いていきます。

chef のインストール
----

前回同様に rbenv を使って ruby をインストールし chef を gem でインストールして
いきます。

    % sudo apt-get update
	% sudo apt-get install zlib1g-dev build-essential libssl-dev
	% sudo -i
	# cd ~
	# git clone git://github.com/sstephenson/rbenv.git .rbenv
	# echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
	# echo 'eval "$(rbenv init -)"' >> ~/.zshrc

ruby-build をインストールします。

    # mkdir -p ~/.rbenv/plugins
	# cd ~/.rbenv/plugins
	# git clone git://github.com/sstephenson/ruby-build.git

ruby-1.9.2 インストール

    # rbenv install 1.9.2-p290
	# rbenv global 1.9.2-p290
	# rbenv rehash

gem を使って、chef, chef-server-api をインストールします

	# gem install chef
    # gem install chef-server-api

opscode bootstrap を使った chef-server の構築
----

まずは chef-solo の環境整備。この bootstrap は chef-solo で実行出来るクックブッ
ク集になっています。中を覗くと 'apache2, chef, chef-server, couchdb, erlang,
rabgitmq' などなど必要なミドルウェア群のクックブックが入っていることが判ります。

まずは chef-solo の環境を整備します。

    # mkdir /etc/chef
	# vi /etc/chef/solo.rb
	# cat /etc/chef/solo.rb
	file_cache_path "/tmp/chef-solo"
	cookbook_path "/tmp/chef-solo/cookbooks"
    # vi /etc/chef/chef.json
	# cat /etc/chef/chef.json
	{
	  "chef_server": {
      "server_url": "http://localhost:4000"
      },
	  "run_list": [ "recipe[chef-server::rubygems-install]" ]
	}

solo.rb で指定したパスは任意のもので構いません。

Amazon S3 上に opscode bootstrap があります。いよいよ chef-solo で bootstrap
を実行します。

    # chef-solo -c /etc/chef/solo.rb -j /etc/chef/chef.json -r http://s3.amazonaws.com/chef-solo/bootstrap-latest.tar.gz

これで chef-server 構築は終わりです。実行したホストに couchDB, Erlang などが構
成されていることが分かると思います。

次に knife の環境を整備します。knife は chef の操作を行うためのコマンドライン
ツールです。

    # mkdir -p ~/.chef
	# cp /etc/chef/validation.pem /etc/chef/webui.pem ~/.chef
	# sudo chown -R root ~/.chef
	# knife configure -i
	Where should I put the config file? [~/.chef/knife.rb] 
	Please enter the chef server URL: [http://localhost:4000] 
	Please enter a clientname for the new client: [root]
	Please enter the existing admin clientname: [chef-webui] 
	Please enter the location of the existing admin client's private key:
	# [/etc/chef/webui.pem] /root/.chef/webui.pem
	Please enter the validation clientname: [chef-validator] 
	Please enter the location of the validation key:
	# [/etc/chef/validation.pem] /root/.chef/validation.pem
	Please enter the path to a chef repository (or leave blank): 
	WARN: Creating initial API user...
	INFO: Created (or updated) client[root]
	WARN: Configuration file written to /home/root/.chef/knife.rb

パラメータはほぼそのまま入力してください。pem ファイルのパス指定だけ先ほど
root ユーザの HOME ディレクトリにインストールしたモノを指定してください。

これで下記のように knife が使えるようになっています。

    # knife client list
	chef-validator
	chef-webui
	root

今回は以上です。この chef-server の構築を knife::server というツールで実現する
方法もあります。

<http://fnichol.github.com/knife-server/>

同じく bootstrap を指定して構築するツールなのですが、これを使う利点として

* chef-server 構成のバックアップが取れる
* バックアップを元に復元できる

があります。これは便利。ただし knife が最初から使える環境に限ります。

これらツールを使うことを前提にしないと、作業が複雑化するため、必須だと思います。
手作業はミスの元ですし。ただ、一度は手作業でやってみて構成を理解することはしな
くてはならないでしょう。couchDB, Rabbitmq などの新しいミドルウェアについての理
解も深めなくてはならないでしょうし、これらをスケールさせることを前提にしておか
ないとノードの数が増える度に負荷が上昇していくので将来困るでしょう。前回の記事
でも触れましたが chef-solo を用いた場合はそれらの心配が無くなります。

