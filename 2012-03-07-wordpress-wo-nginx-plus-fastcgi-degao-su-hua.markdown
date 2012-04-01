---
layout: post
title: "WordPress を nginx + fastcgi で高速化"
date: 2012-03-07 10:21
comments: true
categories: nginx, wordpress
---
ブログを始めるにあたり、wordpress 環境を構築する必要が出てきました。いつもの apache2 + mysql5 + PHP じゃつまらないので、nginx と fastcgi を使って少しだけ高速化してみました。メモですけど、ここに手順を記していきます。

※ wordpress から octopress に移行しました... (2012/03/07)

ただ、今回は nginx や mysql の基本的なオペレーション手順は割愛させてもらいます。

私の環境について...


    % lsb_release -a
    No LSB modules are available.
    Distributor ID: Debian
    Description:    Debian GNU/Linux 6.0.3 (squeeze)
    Release:        6.0.3
    Codename:       squeeze

インストールしたもの...
メタパッケージを指定したのでその他必要なモノはインストールされます。


    % sudo apt-get update
    % sudo apt-get install spawn-fcgi php5 php5-mysql php5-cgi mysql-server nginx 


まずはお決まりの gzip 圧縮転送。IE の古いモノ以外は対応しているので心配なし。今回のテーマと関係無いですけど、一応入れておきます。


    % diff -u /etc/nginx/nginx.conf.org /etc/nginx/nginx.conf
    --- /etc/nginx/nginx.conf.org   2012-01-14 15:27:45.000000000 +0900
    +++ /etc/nginx/nginx.conf       2012-01-14 15:28:58.000000000 +0900
    @@ -22,6 +22,10 @@
         tcp_nodelay        on;
      
         gzip  on;
    +    gzip_http_version 1.0;
    +    gzip_vary         on;
    +    gzip_comp_level   6;
    +    gzip_types        text/html text/xml text/css application/xhtml+xml application/xml application/rss+xml application/atom_xml application/x-javascript application/x-httpd-php;
         gzip_disable "MSIE [1-6]\.(?!.*SV1)";
     
         include /etc/nginx/conf.d/*.conf;


spawn-fcgi を稼働させるスクリプトを生成する。/usr/bin/php-fastcgi として下記の内容で保存する。

    #! /bin/sh
    /usr/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -C 6 -u www-data -f /usr/bin/php5-cgi


    % sudo chmod 755 /usr/bin/php-fastcgi


次にこれを実行する起動スクリプトの用意と実行。/etc/init.d/php-fastcgi


    #!/bin/bash
    
    ### BEGIN INIT INFO
    # Required-Start:    $local_fs $remote_fs $network $syslog
    # Required-Stop:     $local_fs $remote_fs $network $syslog
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: php-fastcgi script
    # Description:       php-fastcgi script
    ### END INIT INFO
    
    # env
    SCRIPT=/usr/bin/php-fastcgi
    USER=www-data
    RETVAL=0
    PIDFILE=/var/run/php5-cgi.pid
    
    # start or stop
    case "$1" in
      start)
        su - $USER -c $SCRIPT
        pidof php5-cgi > $PIDFILE
        RETVAL=$?
      ;;
      stop)
        killall -9 php5-cgi
        echo '' > $PIDFILE
        RETVAL=$?
      ;;
      restart)
        killall -9 php5-cgi
        su - $USER -c $SCRIPT
        pidof php5-cgi > $PIDFILE
        RETVAL=$?
      ;;
      *)
        echo "Usage: php-fastcgi {start|stop|restart}"
        exit 1
      ;;
    esac


起動すると php-cgi のプロセスが立ち上がり localhost:9000 で LISTEN された状態になっているはずです。nginx はここへのプロキシのような動作をすることになります。
下記の手順で起動と起動スクリプトへの組み込みを行なってください。

    % sudo chmod 755 /etc/init.d/php-fastcgi
    % sudo update-rc.d php-fastcgi defaults
    % sudo service php-fastcgi start


次に nginx の virtualhost を掘ります。変数 ${FQDN_HOSTNAME}, ${DOCUMENT_ROOT}は自分
の環境情報に読み替えてください。

    server {
            listen  80;
            server_name     ${FQDN_HOSTNAME}
            access_log      /var/log/nginx/${FQDN_HOSTNAME}.access.log;
            error_log       /var/log/nginx/${FQDN_HOSTNAME}.error.log;
            location / {
                    root   ${DOCUMENT_ROOT};
                    index  index.html index.php;
            } 
            location ~ \.php$ {
                    include /etc/nginx/fastcgi_params;
                    fastcgi_pass 127.0.0.1:9000;
                    fastcgi_index index.php;
                    fastcgi_param SCRIPT_FILENAME ${DOCUMENT_ROOT}$fastcgi_script_name;
            }
    }

nginx を stop/start してこれらの設定を有効にします。


    % sudo service nginx stop
    % sudo service nginx start


以上です。

その他にも proxy cache を有効にして wordpress の静的な出力をキャッシュするチューニング方法もあるそうなので、次回試してみます。 