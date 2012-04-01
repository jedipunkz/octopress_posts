---
layout: post
title: "cocoa な emacs インストール"
date: 2012-03-07 10:26
comments: true
categories: emacs
---
Carbon な API は排除していくべきと Apple も言っているようですし、自宅も会社も Cocoa な emacs を使うようになりました。
                                                                                                                          
その手順を書いていきます。
                                                                                                                          
ソースとパッチでビルドも出来るのですが、<a href="http://mxcl.github.com/homebrew/" target="_blank">homebrew</a> 使うとメチャ楽なので今回はそれを使います。
homebrew は公式サイトに詳しいことが書いてありますけどインストールがワンラインで済みます。
あと、事前に AppStore で Xcode を入れてください。                                                                                                              
                                                                                                                          
    % /usr/bin/ruby -e "$(curl -fsSL https://raw.github.com/gist/323731)"                                                     
                                                                                                                          
だけです。                                                                                                                
                                                                                                                          
そして Cocoa な emacs インストール。                                                                                      
                                                                                                                          
    % brew install --cocoa emacs 
                                                                                                                          
以上です..。簡単すぎる。先人たちのおかげですね。                                                                          
                                                                                                                          
次は時間見つけて anything.el のことを書こうかなぁと思ってます。

参考 URL : <a href="http://mxcl.github.com/homebrew/" title="homebrew" target="_blank">http://mxcl.github.com/homebrew/</a>
