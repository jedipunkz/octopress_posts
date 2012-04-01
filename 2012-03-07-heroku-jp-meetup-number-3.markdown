---
layout: post
title: "Heroku JP Meetup #3"
date: 2012-03-07 10:35
comments: true
categories: heroku, lokka
---
白金台のクックパッドさんで行われた "heroku jp meet up #3" に参加してきました。

東京マラソン参加のため来日されていた <a href="http://twitter.com/stolt45">Christopher Stolt</a> さんや Ruby コミッタの<a href="http://twitter.com/ayumin">相澤</a>さんなどの話を聞けました。

Christopher さんからは、基本的な使い方や heroku で動作させたアプリケーションをローカル環境で動作させる foreman、また皆が意外と気にするアプリのログを tail する方法などの説明がありました。PaaS での皆の懸念点が結構解決されたんじゃないかなぁ。

相澤さんからは NY マラソンでの実績など、比較的エンタープライズな使われ方もされ初めていると説明がありました。あと、呼び名なのですが heroku は "へろく" と発音するそうです。確かに about.heroku.com には "Heroku (pronounced her-OH-koo) is a cloud application platform" と書いてあるのだが、"へろく" が正しいそうです。w

そのた LT が幾つあって、ちょうど気になっていた <a href="http://lokka.org/">Lokka</a> の話があったので、自宅に帰ってから自分の heroku アカウントで lokka を動かしてみました。lokka の公式サイトに手順が書いてあって、そのままなのですが行ったのは、

    % gem install heroku bundler
    % git clone git://github.com/komagata/lokka.git
    % cd lokka
    % heroku create
    % git push heroku master
    % heroku rake db:setup
    % heroku open

すれば OK。

私の環境では公開鍵認証がうまくいかなかったので、下記の対処をしました。

    % heroku keys:add

試しに wordpress のデータを移行してみようと思ったのですが、"Application Error" が発生。今のところうまくいっていません。コードとブログコンテンツを git で管理出来るので、今時ですよね。初めてみたい。

また、今回会場を提供してくださったクックパッドさんが、バレンタインデーということもあり皆に料理を提供して下さいました。感謝ー。

本当は heroku システムがどう構成されているか？が知りたかったのですが、ユーザ視点で使われ方が分からないと何も出来ないと感じていたため、今回はとても良い機会でした。個人的にも heroku は使い続けたい、と感じたサービスでした。みなさん、当日はありがとうございました。
