---
layout: post
title: "switching screen->tmux"
date: 2012-04-01 11:40
comments: true
categories: 
---
長年 Gnu screen 愛用者だったのだけど完全に tmux に移行しました。

愛用している iterm2 との相性も良く、好都合な点が幾つかあり移行する価値がありました。

ただ、サーバサイドでの利用は諦めました。問題だったコピペ問題をクリアしている tmux のバージョンが
Debian sid から取得出来たのだけど、まだまだ完成度高くなく..。

よって、Mac に tmux をインストールして作業するようになりました。インストール方法はこれ。

予め [https://github.com/mxcl/homebrew/wiki/installation] (https://github.com/mxcl/homebrew/wiki/installation) に
したがって homebrew をインストールする必要あり。

    % brew update
    % brew install tmux

インストールしたら .tmux.conf の作成に入る。prefix キーは C-t にしたかった。screen 時代から
これを使っていて指がそう動くから。

    # prefix key
    set-option -g prefix C-t

またステータスライン周りの設定。色なども自分で選択すると良い。

    # view
    set -g status-interval 5
    set -g status-left-length 16
    set -g status-right-length 50
    # status
    set -g status-fg white
    set -g status-bg black
    set -g status-left-length 30
    set -g status-left '#[fg=white,bg=black]#H#[fg=white]:#[fg=white][#S#[fg=white]][#[default]'
    set -g status-right '#[fg=white,bg=red,bold] [%Y-%m-%d(%a) %H:%M]#[default]'
    # window-status-current
    setw -g window-status-current-fg white
    setw -g window-status-current-bg red
    setw -g window-status-current-attr bold#,underscore
    # pane-active-border
    set -g pane-active-border-fg black
    set -g pane-active-border-bg blue

UTF-8 有効化やキーバインド設定等は...

    # Option
    set-window-option -g utf8 on
    #set-window-option -g mode-keys vi

私は emacs 使いなので vi モードは無効にした。これで emacs モードが有効になる。

マウス選択を有効にすると、便利かもしれない。

    #set-option -g mouse-select-pane on
    #set-option -g mouse-resize-pane

が、PANE (ペイン) を横分割した際にこれらの設定を入れているとマウスでのコピーペーストが出来ない
ため、私は無効にした。

その他の設定。.tmux.conf を tmux 開いたまま設定リロードだったり、ペインの移動キーバインド等。
特に hjkl キーの vi キーバインドをペイン移動のためにアサインするため、その他のキーを下記のように
設定している。ウィンドウの移動は C-n, C-p にも追加でアサイン (default は n, p) し、ペインの移動
には hjkl をアサインした。

    bind C-r source-file ~/.tmux.conf
    bind C-n next-window
    bind C-p previous-window
    bind c new-window
    bind | split-window -h
    
    bind C-k kill-pane
    bind K kill-window
    bind i display-panes
    bind y copy-mode
    
    bind h select-pane -L
    bind j select-pane -D
    bind k select-pane -U
    bind l select-pane -R

ウィンドウの中にペインの概念が入ったことで、作業効率が非常に上がったし、Mac 上のアプリなので、Mac を
開いた瞬間に作業に入れる。(今までは debian 上の screen で生活していた) debian 上の tmux に移行する必要
があるかどうか、これから使い込んでみたいと思う。その時にまたレビューします。
