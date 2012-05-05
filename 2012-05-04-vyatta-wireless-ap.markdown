---
layout: post
title: "Vyatta で無線アクセスポイント"
date: 2012-05-04 22:40
comments: true
categories: vyatta
---
自宅ルータを Vyatta で運用開始したのだけど、無線ルータ化ができたのでメモしておき
ます。

まずはアクセスポイントとして稼働する無線カードの選定。下記の URL に Linux 系で
動作する無線カードチップ名の一覧が載っている。Vyatta は Linux 系ドライバを利用
しているので、この一覧が有効なはずだ。

[http://linuxwireless.org/en/users/Drivers](http://linuxwireless.org/en/users/Drivers)

Intel 系も結構アクセスポイントとしては動作しないことが判ったので Atheros の
AR9k のカードを購入した。私が買ったのはAR9280 チップの mini PCI-E 無線カード。

早速装着してみると、認識した！

    % dmesg | grep ath
	[   11.528390] ath9k 0000:01:00.0: PCI INT A -> GSI 16 (level, low) -> IRQ
	16
	[   11.528398] ath9k 0000:01:00.0: setting latency timer to 64
	[   11.961619] ath: EEPROM regdomain: 0x37
	[   11.961620] ath: EEPROM indicates we should expect a direct regpair map
	[   11.961622] ath: Country alpha2 being used: AW
	[   11.961623] ath: Regpair used: 0x37
	[   12.031676] ieee80211 phy0: Selected rate control algorithm 'ath9k_rate_control'

早速設定に入る。有線側有線 NIC と無線 NIC をブリッジ接続する。br0 デバイスにの
み IP アドレスを振り、eth1, wlan0 は IP アドレスを振らない構成にする。

まずはブリッジインターフェースを作る。

    set interfaces bridge br0 address ${IP_ADDRESS}
    set interfaces bridge br0 description LOCAL_NET

eth1 (ローカル側有線 NIC) をブリッジ br0 に含める。

    set interfaces ethernet eth1 bridge-group bridge br0
	set interfaces ethernet eth1 description LOCAL_NET

wlan0 の設定。同じく br0 に含めて、諸々の設定を投入。

    set interfaces wireless wlan0 bridge-group bridge br0
	set interfaces wireless wlan0 channel 2
	set interfaces wireless wlan0 description LOCAL_NET
	set interfaces wireless wlan0 mode n
	set interfaces wireless wlan0 security wpa mode wpa2
	set interfaces wireless wlan0 security wpa passphrase ${PASS}
	set interfaces wireless wlan0 ssid ${SSID}
	set interfaces wireless wlan0 type access-point
    commit
	save

${IP_ADDRESS}, ${SSID}, ${PASS} は任意です。読み替えてください。

ここでコツ。これらの設定をして commit してもうまく無線接続出来なかった。vyatta
を再起動させる (予め save すること) と、無線接続ができた。

購入した無線モジュールは mini PCE-I Atheros　AR5BXB92 (AR9280 チップ)、
PCI-E に変換するカード 、あとバッファローのアンテナだ。

今のところ利用できているが、無線の強度が少し弱い気がする。これから少し調べてみ
ます。
