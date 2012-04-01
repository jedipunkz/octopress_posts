---
layout: post
title: "gitosis ssh+git サーバ"
date: 2012-03-07 10:30
comments: true
categories: git
---
github.com は便利なのだけどプライベートなレポジトリを作るのにお金払うのはもったいないので自宅サーバに SSH 経由の Git サーバを構築した。その時の手順をメモしておきます。

gitosis という便利なツールがあって、これを使うとあっという間に環境構築できます。私の環境は debian Gnu/Linux Squeeze なのですが apt-get で必要なモノを入れました。gitosis は git で持ってきます。

    remote% sudo apt-get update
    remote% sudo apt-get install git git-core python python-setuptools
    remote% cd $HOME/usr/src
    remote% git clone git://eagain.net/gitosis.git
    remote% cd gitosis
    remote% sudo python setup.py install

SSH でアクセスする先のユーザを作ります。

    remote% sudo adduser --shell /bin/sh -gecos --group \
            --disable-password --home /home/git git

作業端末で rsa な SSH 公開鍵を生成して ${remote} サーバは転送する。

    local% ssh-keygen -t rsa
    ... インタラクティブに答える
    local% scp .ssh/id_dsa.pub ${remote}:/tmp/

転送した鍵を元に ${remote} サーバ上で git レポジトリを初期化する。

    remote% sudo -H -u git gitosis-init < /tmp/id_rsa.pub

もし実行権が付いていなかったら

    remote% sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update

これで環境構築完了。ただ、このままだとローカルの作業端末からレポジトリの生成が出来ない。

ローカルの作業端末でレポジトリを生成する手順は、
gitosis-admin.git を clone してきて、gitosis.conf を修正し commit/push する。

    local% git clone git@${remote}:gitosis-admin.git
    local% vi gitosis.conf

今回はテストで test グループに test レポジトリを作るための config を書いてみる。members は複数人書ける。

    [gitosis]
    
    [group gitosis-admin]
    writable = gitosis-admin
    members = chobiwan@${local}
    
    [group test]
    writable = test
    members = chobiwan@${local}

修正した内容を ${remote} サーバへ git push して反映させる。

    local% git add .
    local% git commit -m "added test repo"
    local% git push origin master

これで 'test' レポジトリをローカルの作業端末から作ってコミットする準備完了。
試しにファイルを add, commit, push してみる。

    local% mkdir ~/gitwork
    local% cd ~/gitwork
    local% touch README
    local% git add README
    local% git commit -m "my first commi"
    local% git remote add origin git@obi.chobiwan.me:test.git
    local% git push origin master

公開鍵のアップやレポジトリ・グループの生成の仕方を覚えれば、OK なのかなぁと。
次はレポジトリのバックアップとレカバリについてまとめていきたい。リカバリできないと死ねるから。
