---
layout: post
title: "conky statusbar"
date: 2012-03-07 10:29
comments: true
categories: conky, linux, desktop
---
<a href="http://files.chobiwan.me/pix/conky_capture.png"><img src="http://files.chobiwan.me/pix/conky_capture.png" alt="" title="conky_capture" width="721" height="274" class="alignnone size-full wp-image-60" /></a>

上の画像は conky というツールのキャプチャです。

<a href="http://conky.sourceforge.net/" target="_blank">conky</a> は x window で使える linux マシンのステータスを文字・グラフ描画で表現してくれるツールです。

透明にしたりグラフ表示を派手にすることも出来るのだけど、わたしは上図のようにステータスバーとして使ってます。Window Manager に openbox という素っ気ないものを使うようにしてるので、これ自体がファイラーもステータスバーも無いんです。なので conky を利用して '時間', 'バッテリ残量', 'AC アダプタ有無', 'ネットワーク使用量' 等を表示してます。

deviantart に <a href="http://rent0n86.deviantart.com/art/My-horizontal-conkyrc-122604863" target="_blank">rent0n86</a> さんという方が投稿した作品があって、それをこちょこちょ自分用にいじって使ってます。

debian gnu/linux な GUI 環境があれば...

    % sudo apt-get conky-all
    % cd $HOME
    % wget https://raw.github.com/chobiwan/dotfiles/master/.conkyrc

で、この環境を作れます。

表示する内容は環境に合わせて修正すると楽しいです。幅は minimum_size パラメータで合わせてください。

パラメータ一覧は、<a href="http://wiki.conky.be/index.php?title=Configuration_Settings" target="_blank">公式 Wiki サイト</a> に正しい情報が載っています。

話変わるけど、enlightenment 17 が完成度高くならない理由ってなんなのでしょうかね？ 16 を愛用していただけに残念。


