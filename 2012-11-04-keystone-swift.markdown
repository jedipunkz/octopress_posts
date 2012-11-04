---
layout: post
title: "Swift で簡単に分散オブジェクトストレージ"
date: 2012-11-04 13:17
comments: true
categories: openstack
---
最近、OpenStack にどっぷり浸かってる <a href="https://twitter.com/jedipunkz">@jedipunkz</a> です。

Folsom がリリースされて Quantum を理解するのにめちゃ苦労して楽しい真っ最中なのだけど、
今日は OpenStack の中でも最も枯れているコンポーネント Swift を使ったオブジェクトストレー
ジ構築について少し書こうかなぁと思ってます。

最近は OpenStack を構築・デプロイするのに皆、Swift 入れてないのね。仲間はずれ
感たっぷりだけど、一番安定して動くと思ってる。

これを読んで、自宅にオブジェクトストレージを置いちゃおぅ。

構成は ?...
----

                            +--------+
                            | client |
                            +--------+
                                 |
                          +-------------+
                          | swift-proxy |
                          +-------------+
                                 |
             +-------------------+-------------------+
             |                   |                   |
    +-----------------+ +-----------------+ +-----------------+
    | swift-storage01 | | swift-storage02 | | swift-storage03 |
    +-----------------+ +-----------------+ +-----------------+

となる。IP アドレスは...

* client : 172.16.0.0/24 のどこか
* swift-proxy : 172.16.0.10
* swift-storage01 : 172.16.0.11
* swift-storage02 : 172.16.0.12
* swift-storage03 : 172.16.0.13

これはサンプル。自宅の環境に合わせて読み替えてください。

全て同じネットワークセグメントに。なので上の図は概念的な図です。通信の流れだけ
把握出来ればいいなぁと。向き書いてないけど..。四角で囲まれているのがノード (サー
バ) です。物理サーバでも仮想マシンでも大丈夫！

マシンの準備
----

Ubuntu Server 12.04 もしくは 12.10 を用意。swift-storage のマシン3台だけは
/dev/sda6 など Disk デバイスを swift 用に用意してあげてください。普通にハード
ディスクのパーティションを切ってあげるだけでいいです。高価な Disk を使うまでも
ないので。それが分散ストレージの良いところ！ Disk ・ノードが壊れてもデータが失
われないんです！


swift-proxy の構築
-----

今回は簡易認証機能の tempauth を使います。Keystone を使った構成を説明しようか
迷ったのだけど、Keystone の構築だけで一つ記事が書けるくらいになるので..、諦め
ました。tempauth は Swift 本体に実装されていますよ。

構築は簡単、まず下記をコピペしてください。

    apt-get update
    apt-get install swift python-swift
    apt-get install python-keystoneclient python-keystone 
    mkdir /etc/swift
    chown -R swift:swift /etc/swift
    export PROXY_LOCAL_NET_IP=10.200.4.133
    export POUND_NET=10.200.4.138
    apt-get install swift-proxy memcached
    perl -pi -e "s/-l 127.0.0.1/-l $PROXY_LOCAL_NET_IP/" /etc/memcached.conf
    service memcached restart
    cat >/etc/swift/proxy-server.conf <<EOF
    [DEFAULT]
    #cert_file = /etc/swift/cert.crt
    #key_file = /etc/swift/cert.key
    bind_port = 8080
    workers = 8
    user = swift
    
    [pipeline:main]
    pipeline = healthcheck cache tempauth proxy-server
    
    [app:proxy-server]
    use = egg:swift#proxy
    allow_account_management = true
    account_autocreate = true
    
    [filter:tempauth]
    use = egg:swift#tempauth
    user_system_root = testpass .admin
    https://$PROXY_LOCAL_NET_IP:8080/v1/AUTH_system
    
    [filter:healthcheck]
    use = egg:swift#healthcheck
    
    [filter:cache]
    use = egg:swift#memcache
    memcache_servers = $PROXY_LOCAL_NET_IP01:11211
    EOF

次に swift.conf とリング情報 (バランシングのための情報) を swift-proxy 上に用意します。これらは、
あとで各すべてのノードに配置するので重要です。これもコピペしてください。

    cat > /etc/swift/swift.conf << EOF
    [swift-hash]
    # random unique string that can never change (DO NOT LOSE)
    swift_hash_path_suffix = `od -t x8 -N 8 -A n </dev/random`
    EOF
    cd /etc/swift
    swift-ring-builder account.builder create 18 3 1
    swift-ring-builder container.builder create 18 3 1
    swift-ring-builder object.builder create 18 3 1
    export STORAGE_LOCAL_NET_IP01=172.16.0.11
    export STORAGE_LOCAL_NET_IP02=172.16.0.12
    export STORAGE_LOCAL_NET_IP03=172.16.0.13
    export ZONE01=1
    export ZONE02=2
    export ZONE03=3
    export WEIGHT=100
    export DEVICE=sda6
    swift-ring-builder account.builder add z$ZONE01-$STORAGE_LOCAL_NET_IP01:6002/$DEVICE $WEIGHT
    swift-ring-builder container.builder add z$ZONE01-$STORAGE_LOCAL_NET_IP01:6001/$DEVICE $WEIGHT
    swift-ring-builder object.builder add z$ZONE01-$STORAGE_LOCAL_NET_IP01:6000/$DEVICE $WEIGHT
    swift-ring-builder account.builder add z$ZONE02-$STORAGE_LOCAL_NET_IP02:6002/$DEVICE $WEIGHT
    swift-ring-builder container.builder add z$ZONE02-$STORAGE_LOCAL_NET_IP02:6001/$DEVICE $WEIGHT
    swift-ring-builder object.builder add z$ZONE02-$STORAGE_LOCAL_NET_IP02:6000/$DEVICE $WEIGHT
    swift-ring-builder account.builder add z$ZONE03-$STORAGE_LOCAL_NET_IP03:6002/$DEVICE $WEIGHT
    swift-ring-builder container.builder add z$ZONE03-$STORAGE_LOCAL_NET_IP03:6001/$DEVICE $WEIGHT
    swift-ring-builder object.builder add z$ZONE03-$STORAGE_LOCAL_NET_IP03:6000/$DEVICE $WEIGHT
    swift-ring-builder account.builder
    swift-ring-builder container.builder
    swift-ring-builder object.builder
    swift-ring-builder account.builder rebalance
    swift-ring-builder container.builder rebalance
    swift-ring-builder object.builder rebalance

で、起動。

    swift-proxy# chown -R swift:swift /etc/swift
    swift-proxy# swift-init proxy start

おしまい。

swift-storage 構築
----

swift-storage の構築は各ノードで下記の内容をコピペしてください。今回は3台分だ
けど、マシンが余ってたら4台でも5台でも OK ！

    apt-get update
    apt-get install swift python-swift
    mkdir /etc/swift
    chown -R swift:swift /etc/swift
    
    export STORAGE_LOCAL_NET_IP=172.16.0.11
    
    apt-get install swift-account swift-container swift-object xfsprogs
    mkfs.xfs -i size=1024 /dev/sda6
    echo "/dev/sda6 /srv/node/sda6 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
    mkdir -p /srv/node/sda6
    mount /srv/node/sda6
    chown -R swift:swift /srv/node
    cat >/etc/rsyncd.conf <<EOF
    uid = swift
    gid = swift
    log file = /var/log/rsyncd.log
    pid file = /var/run/rsyncd.pid
    address = $STORAGE_LOCAL_NET_IP
    
    [account]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/account.lock
    
    [container]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/container.lock
    
    [object]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/object.lock
    EOF
    perl -pi -e 's/RSYNC_ENABLE=false/RSYNC_ENABLE=true/' /etc/default/rsync
    service rsync start
    cat >/etc/swift/account-server.conf <<EOF
    [DEFAULT]
    bind_ip = $STORAGE_LOCAL_NET_IP
    workers = 2
    
    [pipeline:main]
    pipeline = account-server
    
    [app:account-server]
    use = egg:swift#account
    
    [account-replicator]
    
    [account-auditor]
    
    [account-reaper]
    EOF
    cat >/etc/swift/container-server.conf <<EOF
    [DEFAULT]
    bind_ip = $STORAGE_LOCAL_NET_IP
    workers = 2
    
    [pipeline:main]
    pipeline = container-server
    
    [app:container-server]
    use = egg:swift#container
    
    [container-replicator]
    
    [container-updater]
    
    [container-auditor]
    
    [container-sync]
    EOF
    cat >/etc/swift/object-server.conf <<EOF
    [DEFAULT]
    bind_ip = $STORAGE_LOCAL_NET_IP
    workers = 2
    
    [pipeline:main]
    pipeline = object-server
    
    [app:object-server]
    use = egg:swift#object
    
    [object-replicator]
    
    [object-updater]
    
    [object-auditor]
    EOF

環境変数 ${STORAGE_LOCAL_NET_IP} を3台毎に変えて、各台で流し込んであげたら、
swift-proxy で生成した /etc/swift.conf とリング情報達を各 swift-storage に
配置します。

    swift-proxy     # scp /etc/swift/swift.conf /etc/swift/*ring.gz 172.16.0.11:/tmp/
    swift-proxy     # scp /etc/swift/swift.conf /etc/swift/*ring.gz 172.16.0.12:/tmp/
    swift-proxy     # scp /etc/swift/swift.conf /etc/swift/*ring.gz 172.16.0.13:/tmp/
    swift-storage01 # mv /tmp/swift.conf /tmp/*ring.gz /etc/swift/
    swift-storage01 # chown -R swift:swift /etc/swift
    swift-storage02 # mv /tmp/swift.conf /tmp/*ring.gz /etc/swift/
    swift-storage02 # chown -R swift:swift /etc/swift
    swift-storage03 # mv /tmp/swift.conf /tmp/*ring.gz /etc/swift/
    swift-storage03 # chown -R swift:swift /etc/swift

swift-storage を各台で起動します。

    swift-storage01 # swift-init all start
    swift-storage02 # swift-init all start
    swift-storage03 # swift-init all start

アクセスしてみる
----

完成したので、swift クライアントでアクセスしてみる。

    % swift -A http://172.16.0.10:8080/auth/v1.0 -U system:root -K testpass stat
    Account: AUTH_system
    Containers: 4
    Objects: 15
    Bytes: 23866252
    Connection: keep-alive
    Accept-Ranges: bytes

ファイルをアップロード・ダウンロードしてみる。

    % swift -A http://172.16.0.10:8080/auth/v1.0 -U system:root -K testpass upload test /etc/hosts
    % swift -A http://172.16.0.10:8080/auth/v1.0 -U system:root -K testpass list
    % test
    % cd /tmp/
    % swift -A http://172.16.0.10:8080/auth/v1.0 -U system:root -K testpass download test
    % ls /tmp/etc/hosts

もちろん、HTTP な API なので curl 等の HTTP ブラウザを使ってもアクセスできる！

    % curl -k -v -H 'X-Storage-User: system:root' -H 'X-Storage-Pass: testpass' http://172.16.0.10:8080/auth/v1.0
    < HTTP/1.1 200 OK
    < Server: nginx/1.1.19
    < Date: Mon, 03 Sep 2012 04:58:30 GMT
    < Content-Length: 0
    < Connection: keep-alive
    < X-Storage-Url: https://172.16.0.10:8080/v1/AUTH_system
    < X-Storage-Token: AUTH_tk8a19f76c9bce4077aee02aef76257020
    < X-Auth-Token: AUTH_tk8a19f96c9bce4077aee02aef76257020
    <
    * Connection #0 to host 172.16.0.10 left intact
    * Closing connection #0
    * SSLv3, TLS alert, Client hello (1):

得られた X-Auth-Token, X-Storage-Url を使ってアクセスする。

    % curl -k -v -H 'X-Auth-Token: <token-from-x-auth-token-above>' <url-from-x-storage-url-above>
    < HTTP/1.1 200 OK
    < Server: nginx/1.1.19
    < Date: Mon, 03 Sep 2012 04:59:47 GMT
    < Content-Type: text/plain; charset=utf-8
    < Content-Length: 34
    < Connection: keep-alive
    < X-Account-Object-Count: 15
    < X-Account-Bytes-Used: 23866252
    < X-Account-Container-Count: 4
    < Accept-Ranges: bytes
    <
    test
    * Connection #0 to host 172.16.0.10 left intact
    * Closing connection #0
    * SSLv3, TLS alert, Client hello (1):

さっき放り込んだ 'test' が swift 上にあることが確認できる。

<http://cyberduck.ch/> ここにある CyberDuck という GUI なツールを使ってもアクセス出来るよ！
HTTPS が必須になるので一工夫する必要があるのだけど、そのあたりは頑張ってみてください。

オンラインでのノードの追加・削除なんてことも出来ます。次回時間があったら解説しますね。

今回は swift-ring-builder コマンドでレプリカ数 '3' を指定したので、アップロー
ドしたファイル(オブジェクトと言う) は必ず 3 個配置される。なので 1 台の
swift-storage が故障しても大丈夫。ノードを増やせばストレージ全体の容量も増やせ
る。また、swift-proxy は単純な HTTP なので負荷分散機・もしくはソフトウェアのロー
ドバランサを入れれば swift-proxy 自体の冗長も組めるほか、ストレージ I/O の拡張
にもつながる。pound, nginx などを使って冗長組んでみてください。その時に
memcached の内容は共有させてあげる必要があるので、これまた一工夫が必要なのだけ
ど。時間があったら今度解説します。


自宅でも簡単に分散オブジェクトストレージが組める swift。使わない手は無いですよぉ。
