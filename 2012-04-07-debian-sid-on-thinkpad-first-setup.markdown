---
layout: post
title: "debian sid on thinkpad : first setup"
date: 2012-04-07 12:26
comments: true
categories: debian
---
ノート PC を購入するといつも Debian Gnu/Linux sid をインストールするのだけれど
も、X Window System や InputMethod をインストールして利用し始められるところま
での手順っていつも忘れる。メモとしてブログに載せておきます。

console 上での ctrl:caps swap 設定
----

取りあえずこの設定をしないと、何も操作出来ない。Caps Lock と Control キーを入
れ替える設定です。

/etc/default/keyboard を下記のように修正

    XKBOPTIONS="ctrl:swapcaps"

下記のコマンドで設定を反映。

    % sudo /etc/init.d/console-setup restart

sid の sources.list 設定
----

Debian Gnu/Linux をノート PC にインストールする時は必ず sid を入れます。新し目
のソフトウェアを使いたいから。

    deb http://ftp.riken.jp/Linux/debian/debian/ unstable main contrib non-free
    deb-src http://ftp.riken.jp/Linux/debian/debian/ unstable main contrib non-free

下記のコマンドで dist-upgrade

    % sudo apt-get update
    % sudo apt-get dist-upgrade

iwlwifi のインストールとネットワーク設定
----

買うノート PC はいつも ThinkPad。大体 intel チップな Wi-Fi モジュールが搭載さ
れているので、iwlwifi を使う。

    % sudo apt-get install firmware-iwlwifi
	% sudo modprobe iwl4965 # 各々の端末に合わせる。lspci コマンドで確認
	% sudo modprobe iwlagn
	% sudo iwconfig
	% sudo ifconfig wlan0 up

次に /etc/network/interfaces の設定

    auto wlan0
	iface wlan0 inet dhcp
         pre-up /sbin/wpa_supplicant -B -Dwext -c/etc/wpa_supplicant/wpa_supplicant.conf -iwlan0
         pre-up iwconfig wlan0 essid ${SSID}
         pre-up ip link set wlan0 up
         pre-up iwconfig wlan0 ap any

/etc/wpa_supplicant/wpa_supplicant.conf の設定

    % sudo wpa_passphrase ${SSID} ${PASSPHRASE} > /etc/wpa_supplicant/wpa_supplicant.conf

生成した上記のファイルを修正・加筆

    network={
	    proto=WPA WPA2
        key_mgmt=WPA-PSK
        pairwise=CCMP TKIP
        ssid="${SSD}"
        psk=.....
	}

ネットワークインターフェースの再起動で接続

    % sudo service networking restart
	
X Window 周りの設定
----

必要なパッケージをインストールする。ビデオカードドライバは端末に合わせてインス
トールする。thinkpad は Intel なビデオカードの場合が多いので下記のように指定。

また、昔はよく enlightenment を使っていたのだけど、e17 が何時まで経っても完成
度上がらないので諦めて openbox を使うことに。

    % sudo apt-get insatll openbox obmenu xorg xserver-xorg-video-intel

console 同様に ctrl:caps を swap する。xorg.conf が無い(2012/04時点)ので、sid
では /etc/X11/Xsession.d/10setxkbmap を下記のように生成して対応。

    #!/bin/sh
	setxkbmap -option ctl:nocaps

日本語入力 input method のインストール
----

日本語入力に uim-mozc を利用することに。今のところこれが一番快適。

    % sudo apt-get install uim-mozc uim-xim uim-utils mozc-utils-gui mozc-server uim-qt

/etc/X11/openbox/autostart に下記の行を追記

    if type uim-xim &> /dev/null ; then
        uim-xim &
	    uim-toolbar-qt4 &
	fi

uim-toolbar-qt4 の preference で "オン・オフキー" 等もろもろを好みに設定。

alsa によるサウンドの出力
----

昔は苦労したサウンド出力も、今ではこんなに簡単な操作で出来るようになりましたｗ

    % sudo apt-get install alsa-utils alsa-base
	% sudo modprobe snd-pcm-oss
	% sudo alsactl init
	% sudo alsactl store

以上です。ここまで来れば、あとは好みで設定していけばいい。もちろん Ubuntu 等の
dist を使えば、こんな操作は必要無いのだけど、自分でやらないとどうも気持ちが悪
いし、万が一トラブルが起きても自分で設定していれば治せるし。

まぁ、最近では MacBook を使うことがほとんどなのですが..。or2
