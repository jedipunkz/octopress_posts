---
layout: post
title: "powerline.el で emacs モードラインを派手に"
date: 2012-05-04 06:24
comments: true
categories: emacs
---
[http://www.emacswiki.org/emacs-en/PowerLine](http://www.emacswiki.org/emacs-en/PowerLine)

vim の powerline に似たモードを emacs で実現できる powerline.el を試してみた。
見た目が派手でかわいくなるので、気に入った。w すでに下記のようなサイトで紹介さ
れつつある。

[http://d.hatena.ne.jp/kenjiskywalker/20120502/1335922233](http://d.hatena.ne.jp/kenjiskywalker/20120502/1335922233)
[http://n8.hatenablog.com/entry/2012/03/21/172928](http://n8.hatenablog.com/entry/2012/03/21/172928)

インストールすると見た目はこんな感じになる。

<img src="http://jedipunkz.github.com/pix/powerline.png">

うん、かわいい。

インストール方法は簡単。auto-install 環境が構築されているとして

    auto-install-from-emacs-wiki powerline.el

する。auto-install が入っていなければ
[http://www.emacswiki.org/emacs/powerline.el](http://www.emacswiki.org/emacs/powerline.el)
ここにある powerline.el をダウンロードして ~/.emacs.d/lisp/ 配下など path の通っ
ている所に入れれば OK。

あとは .emacs 内には下記を追記するだけだ。

    ;; powerline.el
    (defun arrow-right-xpm (color1 color2)
      "Return an XPM right arrow string representing."
      (format "/* XPM */
    static char * arrow_right[] = {
    \"12 18 2 1\",
    \". c %s\",
    \"  c %s\",
    \".           \",
    \"..          \",
    \"...         \",
    \"....        \",
    \".....       \",
    \"......      \",
    \".......     \",
    \"........    \",
    \".........   \",
    \".........   \",
    \"........    \",
    \".......     \",
    \"......      \",
    \".....       \",
    \"....        \",
    \"...         \",
    \"..          \",
    \".           \"};"  color1 color2))
    
    (defun arrow-left-xpm (color1 color2)
      "Return an XPM right arrow string representing."
      (format "/* XPM */
    static char * arrow_right[] = {
    \"12 18 2 1\",
    \". c %s\",
    \"  c %s\",
    \"           .\",
    \"          ..\",
    \"         ...\",
    \"        ....\",
    \"       .....\",
    \"      ......\",
    \"     .......\",
    \"    ........\",
    \"   .........\",
    \"   .........\",
    \"    ........\",
    \"     .......\",
    \"      ......\",
    \"       .....\",
    \"        ....\",
    \"         ...\",
    \"          ..\",
    \"           .\"};"  color2 color1))
    
    
    (defconst color1 "#FF6699")
    (defconst color3 "#CDC0B0")
    (defconst color2 "#FF0066")
    (defconst color4 "#CDC0B0")
    
    (defvar arrow-right-1 (create-image (arrow-right-xpm color1 color2) 'xpm t :ascent 'center))
    (defvar arrow-right-2 (create-image (arrow-right-xpm color2 "None") 'xpm t :ascent 'center))
    (defvar arrow-left-1  (create-image (arrow-left-xpm color2 color1) 'xpm t :ascent 'center))
    (defvar arrow-left-2  (create-image (arrow-left-xpm "None" color2) 'xpm t :ascent 'center))
    
    (setq-default mode-line-format
     (list  '(:eval (concat (propertize " %b " 'face 'mode-line-color-1)
                            (propertize " " 'display arrow-right-1)))
            '(:eval (concat (propertize " %m " 'face 'mode-line-color-2)
                            (propertize " " 'display arrow-right-2)))
    
            ;; Justify right by filling with spaces to right fringe - 16
            ;; (16 should be computed rahter than hardcoded)
            '(:eval (propertize " " 'display '((space :align-to (- right-fringe 17)))))
    
            '(:eval (concat (propertize " " 'display arrow-left-2)
                            (propertize " %p " 'face 'mode-line-color-2)))
            '(:eval (concat (propertize " " 'display arrow-left-1)
                            (propertize "%4l:%2c  " 'face 'mode-line-color-1)))
    )) 
    
    (make-face 'mode-line-color-1)
    (set-face-attribute 'mode-line-color-1 nil
                        :foreground "#fff"
                        :background color1)
    
    (make-face 'mode-line-color-2)
    (set-face-attribute 'mode-line-color-2 nil
                        :foreground "#fff"
                        :background color2)
    
    (set-face-attribute 'mode-line nil
                        :foreground "#fff"
                        :background color3
                        :box nil)
    (set-face-attribute 'mode-line-inactive nil
                        :foreground "#fff"
                        :background color4)

色を少し変えただけで、ほぼ emacswiki に載っているコードそのままだ。no window
だと寂しいことになるが、まぁ仕方ない。vim でも powerline を使って気に入ってい
たので emacs でも使えるとあって嬉しい限りだ。これから少しずつカスタマイズして
いこうと思う。
