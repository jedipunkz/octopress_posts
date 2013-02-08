---
layout: post
title: "Arch Linux セットアップまとめ"
date: 2013-01-14 16:24
comments: true
categories: Linux
---
自宅のノート PC にいつも Debian Gnu/Linux unstable を入れて作業してたのだけど、
Arch Linux が試したくなって入れてみた。すごくイイ。ミニマル思考で常に最新。端
末に入れる OS としては最適かも!と思えてきた。Ubuntu はデスクトップ環境で扱う
にはチト大きすぎるし。FreeBSD のコンパイル待ち時間が最近耐えられないし...。

前リリースの Arch Linux には /arch/setup という簡易インストーラがあったのだけ
ど、それすら最近無くなった。環境作る方法を自分のためにもメモしておきます。

#### OS イメージ iso 取得とインストール用 USB スティック作成

Linux, Windows, Mac で作り方が変わるようだけど、自分は Mac OSX を使ってインス
トール USB スティックを作成した。

diskutil で USB スティック装着前後の disk デバイス番号を覚える

    % diskutil list

(ここでは /dev/rdisk4 として進める。)

アンマウントする。

    % sudo diskutil unmountDisk /dev/rdisk4

ダウンロードした iso を USB スティックに書き込む。

    % sudo dd if=/path/to/downloaded/iso of=/dev/rdisk4 bs=8192
	% sudo diskutil eject /dev/rdisk4

#### USB スティック装着しインストール開始

起動するとメニューが表示されるので x86_64 を選んで起動。プロンプトが表示される。

まず disk のパーティション作成。

    # fdisk /dev/sda2 # 環境に合わせてデバイス名変更
	# fdisk -l /dev/sda2
    デバイス ブート      始点        終点     ブロック   Id  システム
	/dev/sda1            2048     4196351     2097152   82  Linux スワップ / Solaris
	/dev/sda2   *     4196352   156301487    76052568   83  Linux

上記のように切りました。fdisk の使い方は...ぐぐってください。また /boot にあたるパーティション
にブートフラグを必ず付けること。

ファイルシステムを作ってマウント。

    # mkfs.ext4 /dev/sda2
	# mount -t ext4 /dev/sda2 /mnt

コアシステムのインストール

    # pacstrap /mnt base base-devel

fstab の配置。必要に応じて /dev/sda1 (上記で作った) をスワップとして加筆。

    # genfstab -p /mnt >> /mnt/etc/fstab

#### chroot して /dev/sda2 内コアシステムで作業

chroot する。

    # arch-chroot /mnt

キーマップを us 配列で CTRL と Caps をスワップしたいので下記の作業を実施。

    # cd /usr/share/xkb/keymaps/i386/qwerty
	# cp us.map.gz usx.map.gz
	# gunzip usx.map.gz
	# vi usx.map
	keycode  58 = Control # Caps_Lock を Control に置き換え
	# gzip usx.map
	# loadkeys usx # キーマップをロード
	# vi /etc/vconsole.conf
	KEYMAP=usx # 追記

ロケールの生成。

    # vi /etc/locale.gen # ja_JP.UTF-8 のコメントアウトを削除
	# locale-gen

タイムゾーンの設定。

    # ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

ホスト名修正。

    # echo "<ホスト名>" > /etc/hostname
	# vi /etc/hosts # 適宜修正

ネットワークの設定。まずは有線。

    # systemctl enable dhcpcd@eth0.service

ブートローダとして syslinux を使う。ディスクデバイス名だけ修正。syslinux のインストーラが
うまくデバイス名を拾ってくれない。

    # pacman -S syslinux
	# vi /boot/syslinux/syslinux.cfg
	LABEL arch
        MENU LABEL Arch Linux
        LINUX ../vmlinuz-linux
        APPEND root=/dev/sda2 ro
        INITRD ../initramfs-linux.img

パスワード設定。

    # passwd

#### 再起動し システムをインストールした /dev/sda2 で起動する。

再起動し USB スティックを抜く。起動したら root ユーザでログイン。

無線デバイスの設定。

    # pacman -S dialog wpa_supplicant
	# wifi-menu

検知されたアクセスポイント名が表示されるので希望のモノを選択しパスフレーズを入力。

一般ユーザの作成と sudo 設定。

    # adduser <ユーザ名>
	# pacman -S sudo
	# vigr # wheel グループに自分を追記
	# visudo # wheel のところのコメントアウトを削除
	%wheel ALL=(ALL) ALL

#### X 周りの設定

X とウィンドウマネージャのインストール。ウィンドウマネージャは好きなものを。

    # pacman -S xorg xorg-xinit xterm rxvt-unicode xf86-video-intel enlightenment

日本語フォントのインストールと日本語インプットメソッドの設定。ibus と mozc を選んだ。
結構賢い。とりあえずのフォントとして sazanami を。

    # vi /etc/pacman.conf # 下記のレポジトリを追記
	[pnsft-pur]
	Server = http://downloads.sourceforge.net/project/pnsft-aur/pur/$arch
	# pacman -Syy
    # pacman -S ttf-sazanami ibus-mozc mozc

一般ユーザの ~/.xinitrc に下記を追記。

    % vi ~/.xinitrc
	xrdb -merge $HOME/.Xresources
	
	export LANG=ja_JP.UTF-8
	export LANGUAGE=ja_JP.UTF-8
	export LC_ALL=ja_JP.UTF-8
	export LC_CTYPE=ja_JP.UTF-8
	
	export GTK_IM_MODULE=ibus
	export QT_IM_MODULE=xim
	export XMODIFIERS=@im=ibus
	ibus-daemon --daemonize --xim &
	
	setxkbmap -layout us -option ctrl:nocaps
	exec e16

X を起動。

    % startx

ここで enlightenment のメニューなど、日本語化されているはずだが、厳しければ

<https://wiki.archlinux.org/index.php/Input_Japanese_using_uim_(%E6%97%A5%E6%9C%AC%E8%AA%9E)>

にある IPA フォント、VL ゴシックなどを入れる。こちらのほうが数段綺麗。

IPA フォントの入れ方だけメモっておく。

    % wget https://aur.archlinux.org/packages/ot/otf-ipafont/otf-ipafont.tar.gz
	% tar zxvf otf-ipafont.tar.gz
	% cd otf-ipafont
	% makepkg -s
	% sudo pacman -U <生成された pkg.tar.xz ファイル>

Thinkpad の真ん中ボタンでスクロールする。

    # vi /etc/X11/xorg.conf.d/10-thinkpad.conf # 新規生成
    Section "InputClass"
        Identifier "Trackpoint Wheel Emulation"
        MatchProduct "TPPS/2 IBM TrackPoint|DualPoint Stick|Synaptics Inc. Composite TouchPad / TrackPoint|ThinkPad USB Keyboard with TrackPoint|USB Trackpoint pointing device|Composite TouchPad / TrackPoint"
        MatchDevicePath "/dev/input/event*"
        Option  "EmulateWheel"  "true"
        Option  "EmulateWheelButton" "2"
        Option  "Emulate3Buttons" "false"
        Option  "XAxisMapping"  "6 7"
        Option  "YAxisMapping"  "4 5"
    EndSection

X の再起動をして終了。

#### まとめ

今回はブログというより雑多なまとめになったけど、メモするだけでも意味があるので
残しておいた。Debian / Ubuntu より手間は掛かるけど、エンジニアが何をやっている
か掴めるのでいいカンジ。無駄なプロセスいないので起動もめちゃ速いし。何より最新
のソフトウェア (安定したもの) が簡単に使えるのは嬉しい。ここに書いた手順は時間
がすぎるに連れて意味がないものになっていくだろう。開発が活発でここ数年でも結構
劇的にシステムが変更になってるみたい。systemd .. とか。理解するのに苦労する所
はないので、エンジニアであれば誰でも扱えそうなところも魅力。

OS なんて何でもいい時代 (抽象化されつつあるから) だけど、手元の端末に入れる OS
だけは Arch Linux がいいなぁ。
