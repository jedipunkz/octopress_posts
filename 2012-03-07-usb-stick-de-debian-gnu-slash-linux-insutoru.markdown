---
layout: post
title: "USB Stick で Debian Gnu/Linux インストール"
date: 2012-03-07 10:27
comments: true
categories: debian
---
MS Windows なツールを使う方法だったり、vmlinuz, initrd をファイラーでコピーしたり、何故かいつもインターネットで調べると USB スティックを利用した debian のインストール方法が'面倒', '不確か' なので、忘れないようにメモ。

手元に linux 端末用意して、USB スティック挿す。

    % wget ftp://ftp.jp.debian.org/pub/Linux/debian-cd/6.0.3/amd64/iso-cd/debian-6.0.3-amd64-netinst.iso
    % sudo cat debian-6.0.3-amd64-netinst.iso > /dev/sdb # 挿した USB スティックのデバイス名

で終わり。

ただ弱点があって、フルイメージの iso は利用できないでの、今回みたいに netinstall だったり businesscard な iso を利用しかない。
他のディストリビューションもだけど、インストールする環境はネットに繋がっていないと不都合があるって時代だからいいかなぁ。

一方、Ubuntu Server は賢い子なので <a href="http://www.ubuntu.com/download/ubuntu/download" target="blank">公式サイト</a> に行くと USB スティック用の iso がダウンロードできたり iso を焼く環境に合わせて手順まで教えてくれる...。
