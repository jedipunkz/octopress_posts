---
layout: post
title: "OpenFlow 勉強会"
date: 2012-03-07 10:33
comments: true
categories: openflow, openvsiwtch
---
2012年2月6日、西新宿にある株式会社ニフティさんで行われた "OpenFlow 勉強会" に参加したので簡単なレポメモを書いておきます。

まずは OpenFlow の基本動作。
* メッセージの切り出し
* ハンドシェイク
* コネクションの維持
* スイッチから送られてくるメッセージへの応答

構成は
* OpenFlow コントローラ, スイッチから成る
* コントローラは通信制御
* スイッチはフロールールをコントローラに問い合わせ通信を受け流し(packet/frame 転送)
* コントローラは L2 - L4 フィールドを見て制御する

そしてコントローラ、スイッチは各社・団体から提供されている。今月も Nicira Networks さんが自社システムの構成を抽象的ではありますが公開され、HP さんも OpenFlow 対応スイッチを12製品ほど発表されました。

コントローラの種別、
* Beacon (Java)
* NOX (Python)
* Ryu
* NodeFlow
* Trema (C/ruby)
* Nicira Networks
* Big Switch Networks
* Midokura
* NTT Data

スイッチの種別、
* cisco nexus 3000
* IBM BNT rackSwitch G8264
* NEC Univerge PF5240/PF5820
* Pronto Systems 3240/3290
* HP 3500/5400
* HP OpenFlow 化 firmware
* Reference Implementation (Software)
* Open vSwitch (Software)

NEC さんの PF ほにゃららは、GUI なインターフェースと API を持った製品で OpenFlow 1.0.0 仕様に準拠。スイッチ25台までを管理するコントローラ。価格は、コントローラ : 1000万, スイッチ : 250万 だそうです。スイッチ25台はあくまでもソフトリミットらしいです。

Trema は開発者が日本人で OpenFlow 1.1.0 に準拠。Ruby / C を使ってフロールールが掛けるフレームワーク型のソフトウェアで、github にある Wiki を見ても分かりますが、とても簡潔なルール記述が可能です。あと、特徴なのがテストフレームワーク(シュミレータ)が付いているので、OpenFlow 対応スイッチを購入しなくても自分で書いたルールがテスト出来ます。これはとても大きな意味がある。自宅でもあれこれルールを書けて動作を知るにはとても良い機能です。

Nicira Networks の NVP (Nacira Virtual Platform) はハイパーバイザ上の OpenvSwitch やアプライアンスなスイッチを管理するコントローラから成ると今月公開されました。ただ抽象的な言葉ばかりが並んでいて ( web, datasheet 共に ) まだまだ、仕組みを理解するには情報が足らないのかなぁと感じています。特に distributed controller はどう冗長取れている？など。software designed network の ML が日本でも管理され始めて議論されていますが、ハイパーバイザ上の OpenvSwitch で、カプセル化時のオーバーヘッドをどう攻略するか？が問題じゃないかと話し合われています。

Midokura さんも会場にいらっしゃったのですが、印象的な言葉は "OpenFlow は夢の技術ではないし、既存のネットワーク技術を置き換えるものでもない" でした。サービスを形成するサーバファーム周りで OpenFlow を使ったシステムを組むには良いですけどね。

最後に参考 URL を記しておきます。

* <a href="https://groups.google.com/group/sdnstudy">sdnstudy google group</a>
* <a href="https://github.com/trema/trema/wiki/Trema-%E3%83%81%E3%83%A5%E3%83%BC%E3%83%88%E3%83%AA%E3%82%A2%E3%83%AB">trema</a>
* <a href="http://nicira.com/en/network-virtualization-platform">Nicira Virtual Platform</a>
