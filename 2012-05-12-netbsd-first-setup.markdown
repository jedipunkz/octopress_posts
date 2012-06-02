---
layout: post
title: "NetBSD インストール直後の初期設定まとめ"
date: 2012-05-12 14:18
comments: true
categories: netbsd
---
普段は Mac, Linux がメインなのですが、NetBSD もたまにデスクトップ機として利用
するので、初期設定手順をまとめ。

Console 設定, キーリマップ
----

Caps と Ctrl キーを入れ替えます。お約束。

    # wsconsctl -w encoding=us.swapctrlcaps
    # vi /etc/wscons.conf # 同様の設定を入れる

キーリピートを速くします。標準では遅すぎる。値は好みで。

    # wsconsctl -w repeat.del1=300 # リピートが始まるまでの時間
    # wsconsctl -w repeat.deln=40  # 反復間隔
    # vi /etc/wscons.conf          # 同様の値を入れる

pkgsrc をインストール
----

xx, y は次期に応じて変える。例 :2012Q1

    # cd /usr
    # cvs -q -z3 -d anoncvs@anoncvs.NetBSD.org:/cvsroot checkout -r pkgsrc-20xxQy -P pkgsrc

最新の状態に更新する場合。リリースタグを利用したいなら不要。

    # cd /usr/pkgsrc
    # cvs update -dP

grub のインストール
----

grub をインストール。デフォルトでも良いのだけど、安心したいから。他の OS とデュ
アル・トリプルブートに設定することが多いので必要を感じて入れている場合もある。
もちろん Linux 側からインストールしても OK。

    # cd /usr/pkgsrc/sysutils/grub
    # make install
    # /usr/pkg/sbin/grub-install '(hd0)'
    # vi /grub/menu.lst # 適当に設定


X の設定
----

    # X -configure
	# cp /root/Xorg.conf.new /etc/X11/xorg.conf

ウィンドウマネージャは普段は twm を利用しているので、何も入れない。

日本語フォントとして efont を使う。スグレモノフォント。

    # cd /usr/pkgsrc/fonts/efont-unicode
    # make install

xorg.conf に FontPath を通す。

    FontPath "/usr/pkg/lib/XZ11/fonts/efont"

urxvt と zsh
----

ターミナルエミュレータとして rxvt を利用。Unicode 対応な urxvt をインストール
する。zsh はマルチバイト対応な zsh-current をインストール。

    # cd /usr/pkgsrc/shells/zsh-curren
    # make install
    # cd /usr/pkgsrc/x11/rxvt-unicode
    # make install

日本語入力設定
----

日本語のインプットメソッド uim, anthy をインストール。

    # cd /usr/pkgsrc/inputmethod/uim
    # make install

環境変数を設定。.zshrc でも .xinitrc でも可。

    export LANG=ja_JP.UTF-8
    export LC_ALL=ja_JP.UTF-8
    export LC_CTYPE=ja_JP.UTF-8
    export XMODIFIERS=@im=uim
    export GTK_IM_MODULE=uim
    exec uim-xim &
    exec uim-toolbar-gtk *
    exec /usr/X11R7/bin/twm

所感
----

昔は FreeBSD を使っていたのだけど、Mac, Linux, NetBSD だけになってしまった。
NetBSD は実装が綺麗なのでスーッと入っていけるので好き。ただ、source からのビル
ドは時間が掛かるので、git や chef, puppet 等の仕組みを利用してビルドする方法を
確立したいなぁと最近思ってる。
