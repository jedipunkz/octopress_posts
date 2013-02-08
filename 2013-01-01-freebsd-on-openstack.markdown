---
layout: post
title: "FreeBSD on OpenStack"
date: 2013-01-01 17:35
comments: true
categories: openstack
---
FreeBSD を OpenStack で管理したいなぁと思って自宅に OpenStack 環境作ってました。

お正月なのに...

使ったのは folsom ベースの OpenStack (nova-network) と FreeBSD 9.1 です。8 系
の FreeBSD でも大体同じ作業で実現出来るぽいです。あと nova-network でって書い
たのは自宅に quantum だと少し厳しいからです。FlatDHCPManager が調度良かった。

今回のポイントは FreeBSD の HDD, NIC のドライバに virtio を使うように修正する
ところです。OpenStack (KVM) は virtio 前提なので、そうせざるを得なかったです。

今回使ったソフトウェア
----

* OpenStack Folsom (nova-network)
* Ubuntu Server 12.10
* FreeBSD 9.1 amd64

作業方法
----

準備としてこれらが必要になります。事前に行なってください。

* FreeBSD-9.1-RELEASE-amd64-disc1.iso ダウンロード
* 作業ホスト (Ubuntu Server 12.10) に qemu-kvm をインストール

freebsd9.img として qcow2 イメージを作成します。

    % kvm-img create -f qcow2 freebsd9.img 8G

作成したイメージファイルに FreeBSD 9.1 をインストールします。

    % kvm -m 256 -cdrom ./FreeBSD-9.1-RELEASE-amd64-disc1.iso \
    -drive file=./freebsd9.img -boot d -net nic -net user -nographic -vnc :10

VNC のディスプレイ番号は空いているものを使ってください。空いていれば他でも構い
ません。

VNC viewer をインストールし localhost:10 に接続します。ここでは作業ホストに X
Window System が入っていることを前提に書いていますが、Non-X な方は他のホストか
らその他の VNC ソフトウェアを使ってアクセスしてもらっても構わないです。

    % sudo apt-get update
	% sudo apt-get gvncviewer
    %gvncviewer localhost:10

インストーラが起動しているのでインストール行なってください。気をつける点として
は一つ。src をインストールしてください。後に virtio をインストールするのに必要
になってくるからです。kmod ファイルをコンパイルしてインストールすることになります。

インストールが終わったら今度は仮想マシンを HDD から起動します。

    % kvm -m 256 -drive file=./freebsd9.img -boot c -net nic -net user \
      -nographic -vnc :10

VNC で再度接続し仮想マシン上で emulators/virtio-kmod をインストールします。

    freebsd9% cd /usr/ports/emulators/virtio-kmod/
	freebsd9% su
	freebsd9# make install clean

次に virtio ドライバを扱うように HDD, NIC ドライバ周りの設定を変更します。
virtio のインストールが終わった後に「こうしろ」とメッセージが出てきますので
それを参考に行います。一部、僕の環境ではそのままではダメだったので修正して
使いました。

インストールした virtio を起動時に読み込むために下記を追記します。

    freebsd9# vi /boot/loader.conf
	virtio_load="YES"
	virtio_pci_load="YES"
	virtio_blk_load="YES"
	if_vtnet_load="YES"
	virtio_balloon_load="YES"

HDD ドライバを virtio を使うように変更します。これによってデバイス名が変わって
くるので /etc/fstab を編集します。

    freebsd9# sed -i.bak -Ee 's|/dev/ada?|/dev/vtbd|' /etc/fstab

NIC も virtio を使います。僕の環境では ifconfig_re0 でした。これを vtnet0 に変
更します。

    freebsd9# cat /etc/rc.conf
    ifconfig_vtnet0="DHCP"
    ..<snip>..

qemu の起動方法、もしくはデフォルトのハードウェア定義によって re0 は変わってく
るかもしれません。適宜変更します。

仮想マシンをシャットダウンします。

    freebsd9# shutdown -p now

完成したイメージファイル freebsd9.img を openstack 環境に転送し (openstack 環
境で作業している方は必要無いです) glance に登録します。

環境変数諸々を揃えて...

    % glance add name="FreeBSD 9" is_public=true container_format=ovf disk_format=qcow2 < freebsd9.img
    % glance image-list
    +--------------------------------------+------------------------+-------------+------------------+-------------+--------+
	| ID                                   | Name                   | Disk Format | Container Format | Size        | Status |
	+--------------------------------------+------------------------+-------------+------------------+-------------+--------+
	| 1af6f41b-1048-4d78-9715-87935c0bc6ae | FreeBSD 9              | qcow2       | ovf              | 4305584128  | active |
	+--------------------------------------+------------------------+-------------+------------------+-------------+--------+

以上です。

まとめ
----

FreeBSD は仕事場でもプライベートでもまだまだ現役だし OpenStack で扱えるように
することは僕にとってとても重要でした。なので満足ｗ FreeBSD 8 系では同じ手順で
扱えるらしいのだけど、それ以前のバージョンになるとどうか... virtio がポイント
なのと、KVM とゲスト OS バージョンって相性がめちゃ有るのでここもポイントになり
そう。ハイパーバイザに VMWare も使えるらしいので今度やってみるかな。VMWare で
も相性はあるけど、KVM ほどシビアにならなくていい印象がある。FreeBSD 3系でも頑
張れば動いたし。

あとは cloud-init 。まだ開発途中だそうです。なので metadata サーバにアクセスし
て、色んな事しようと思ってもまだ難しい。

この年末に Windows も OpenStack に乗せてみたので、そちらの記事も時間があったら
載せようっと。

#### 2013/01/02 追記

FreeBSD 8.3 で試してみましたが、全く同じ手順で起動してくれました。ただ、
emulators/virtio-kmod が 8.2 or 9 に対応しているとあったので Makefile を修正し
て virtio-kmod をインストールしています。今のところ何も問題出ていません。

``` bash diff output of Makefiles
# diff -u /usr/ports/emulators/virtio-kmod/Makefile.org  /usr/ports/emulators/virtio-kmod/Makefile
--- /usr/ports/emulators/virtio-kmod/Makefile.org   2013-01-02 06:02:48.000000000 +0900
+++ /usr/ports/emulators/virtio-kmod/Makefile   2013-01-02 06:02:59.000000000 +0900
@@ -28,7 +28,7 @@
 
 .include <bsd.port.pre.mk>
 
-.if ${OSREL} != "8.2" && ${OSREL} != "9.0"
+.if ${OSREL} != "8.3" && ${OSREL} != "9.0"
 IGNORE=not supported $${OSREL} (${OSREL})
 .endif
```
