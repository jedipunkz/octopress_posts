---
layout: post
title: "github.com で octopress 構築"
date: 2012-03-20 11:40
comments: true
categories: octopress
---
pages.github.com は github.com の WEB ホスティングサービスです。これを利用して octopress のブログを
構築する方法をメモしていきます。

まず、github.com に "${好きな名前}.github.com" という名前のレポジトリを github.com 上で作成します。
レポジトリの作成は普通のレポジトリ作成と同じ方法で行えます。しばらくすると
"${好きな名前}.github.com のページがビルド出来ました" という内容でメールが送られてきます。

pages.github.com によると、レポジトリページで "GitHub Page" にチェックを入れろと書いてありますが、情報が古いようです。
2012/03/20 現在、この操作の必要はありませんでした。

次に octopress の環境構築。

octopress は、jekyll ベースのブログツールです。markdown 形式で記事を書くのですが、emacs や vim 等
好きなエディタを使って記事を書けるので便利です。最近 "Blogging with Emacs" なんてブログをよく目にしたと
思うのですが、まさにソレですよね。エンジニアにとっては嬉しいブログ環境です。

まずは、rvm の環境構築を。octopress は ruby 1.9.2 以上が必要なので用意するのですが rvm を使うと
手軽に用意出来るので、今回はその方法を記します。

参考 URL は [http://octopress.org/docs/setup/rvm/](http://octopress.org/docs/setup/rvm/) です。

まずは準備から。私の環境は Ubuntu Server 10.04 LTS なのですが、下記のパッケージが必用になります。

    % sudo apt-get install gcc make zlib1g-dev libssl-dev

下記のコマンドを実行すると、rvm がインストールされます。

    % bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)

次に使っている shell に合わせて rc ファイルを設定します。

bash なら...

    % echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.bash_profile
    % source ~/.bash_profile

zsh なら...

    % echo '[[ -s $HOME/.rvm/scripts/rvm ]] && source $HOME/.rvm/scripts/rvm' >> ~/.zshrc
    % source ~/.zshrc

ruby 1.9.2 をインストールします。

    % rvm install 1.9.2 && rvm use 1.9.2
    % rvm rubygems latest

次に octopress のインストールと環境設定です。

    % mkdir ${好きな名前}.github.com
    % cd  ${好きな名前}.github.com
    % git clone git://github.com/imathis/octopress.git octopress
    % cd octopress
    % gem install bundler
    % bundle install
    % rake install # classic テーマのインストール

ここまで出来たら github.com へデプロイするだけです。

    % rake setup_github_pages
    Enter the read/write url for your repository:

と表示されるので、作成した github.com ページの情報を入力します。私の場合は

    git@github.com:chobiwan/chobiwan.github.com.git

この操作で、.git 内の情報諸々を更新してくれます。実行した後に覗いて見て下さい。

です。最後にブログ生成とデプロイを実行します。この作業で ${好きな名前}.github.com へのデプロイが
実行されます。

    % rake gen_deploy

暫く時間がかかりますが (数分) 、${好きな名前}.github.com のサイトにアクセスできるようになっている
はずです。

次に、必要最低限の設定を行います。_config.yml ファイル内の下記の情報を満たしていきましょう。

    url:                # For rewriting urls for RSS, etc
    title:              # Used in the header and title tags
    subtitle:           # A description used in the header
    author:             # Your name, for RSS, Copyright, Metadata
    simple_search:      # Search engine for simple site search
    description:        # A default meta description for your site
    subscribe_rss:      # Url for your blog's feed, defauts to /atom.xml
    subscribe_email:    # Url to subscribe by email (service required)
    email:              # Email address for the RSS feed if you want it.

ではいよいよ、最初の投稿を。

    % rake new_post["title"]
    % rake new_post\["title"\] # zsh の場合

source/_posts/2012-03-20-test-post.markdown という日付付きファイルが生成されるので、このファイルを
好きなエディタで編集します。

markdown 形式で記すことが出来ます。上記の操作でテンプレートらしきファイルになっているので
"---" の配下から記事を書いていきます。また categories 等の情報は自分で修正出来ます。

書き終わったら、ブログを生成してデプロイです。

    % rake gen_deploy

先ほど作った ${好きな名前}.github.com へアクセスしてみましょう。octopress のブログが完成している
はずです。