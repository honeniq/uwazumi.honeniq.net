---
templateKey: 'blog-post'
title: Wiresharkで公衆無線LANのヤバさを確認してみた
date: 2013-11-21
redirect_from: 
  - /entry/2013/11/21/001559
  - /2013/11/21/wireshark-public-wifi.html
tags:
  - ネットワーク
  - セキュリティ
---

## 前置き

ここ数年の携帯キャリアやコンビニ業界の頑張りで、町中に公衆無線LANのAPが溢れていますが、あれって安全なんでしょうか？盗聴される的な観点で。

パスワード無しのノーガードAPは論外としても、

* 契約者にだけWPAキーを教える((けど、利用者が多すぎて公開しているも同然の))タイプ
* APにはキー無しで入ることができ、Webアクセスをすると認証ページにリダイレクトするタイプ

よく見かけるこの2タイプもやヤバそう。  


### 試してみる前の認識
* 無線である以上は、自分が飛ばした電波は誰でも傍受できる。じゃあ暗号化して中身が分からないようにしましょう、ってなるけど、1つ目のタイプみたいに不特定多数の人が同じWPAキーを知っている場合、暗号化してもあんまり意味ないのでは？
* 2つ目のタイプは、APを運用している人が**「タダ乗りを防ぐ」**ためにやっているだけで、利用者の通信は保護できてないよね？

こんな感じ。  

「たぶん丸見えだろう」と思いながらも自分で試してみたことはなかったので、やってみました。1つ目のタイプ想定で。

もちろん公衆無線LAN用のAPなんて手元にないので、自宅で使ってるAPを使って「同じWPAキーを知っているけど、それ以外のことは知らない」体でやっています。



<!-- more -->


### 参考にした記事

* [無線LANのパケットを手っ取り早くキャプチャする方法メモ](http://bogus.jp/wp/?p=993)
* [Wireless Sniffing using a Mac with OS X 10.6 and above](https://supportforums.cisco.com/docs/DOC-19212)  
* [How to Decrypt 802.11](http://wiki.wireshark.org/HowToDecrypt802.11)


### 使用した環境
#### 無線LAN AP
ごく普通の無線LANルータを使用。  
暗号方式はWPAｰPSKに設定。

#### Wireshark入りのMac
盗聴する人。  
OSX用のWiresharkはX11上で動きます。が、最近のOSXにはX11が同梱されていないので、[XQuartz](http://xquartz.macosforge.org/landing/)も必要。

#### iPhone
盗聴される人。  

------

## 設定手順

### キャプチャオプションの設定
そのままキャプチャ開始すると自分の通信しか見えないので、プロミスキャスモードかつモニターモードに設定します。

Wiresharkのメニューから「Capture」→「Options」を開く。

![Wireshark Capture Options](/img/2013-11-21-capture-options.png)
Wi-Fiのインターフェイスをダブルクリック。この操作を知らなかったので若干ハマりました。

![Edit Interface Settings](/img/assets/2013-11-21-interface-settings.png)
* 「Capture packets in promiscuous mode」にチェックを入れる。
* 「Capture packets in monitor mode」にチェックを入れる。

### 復号の有効化
Wiresharkのメニューから「Edit」→「Preferences」を開く。  
![Enable decryption](/img/assets/2013-11-21-enable-decryption.png)
* 左側のリストから「Protocol」→「IEEE802.11」を選択。
* 「Enable decryption」にチェックを入れる。


### Decryption Keyの設定
WEP/WPAキーをWiresharkに教えてあげます。

Wiresharkのメニューから「View」→「Wireless Toolbar」にチェックを入れる。((初期状態だと表示されていないので。))  
新しく増えたツールバー右端の「Decryption Keys...」をクリック。

![Decryption Key Management](/img/assets/2013-11-21-decryption-key-management.png)
ここでキーとSSIDの対応を設定しておくと、キャプチャ時に復号してくれます。  
「New」をクリック。

![Add Decryption Key](/img/assets/2013-11-21-add-decryption-key.png)
* 「Type」は対応した方式を選択。普通にWPAキーを使っている環境なら、「WPA-PWD」でOK。
* 「Passphrase」はWEP/WPAキーを入力。
* 「SSID」は文字通りSSIDを入力。


### 復号できるまで待つ
上記の設定を済ませてキャプチャを開始すると、IEEE802.11のフレームがじゃんじゃか流れ始めます。まだ復号できていないので、中身はさっぱり分かりませんが。

WPAではクライアントごとに暗号化用のキー((テンポラリキー))を生成しているので、実際に復号するにはこのキーを入手する必要があります。

>[無線LANの基本とセキュリティ技術](http://www.uquest.co.jp/embedded/columns/lan2.html)  
WPA-PSKでは、APとステーションで共有するパスフレーズから、決められたアルゴリズムに従って512bitのマスターキー（PMKと呼ばれる）を事前に生成しておきます。そして、AP-ステーション間のLINK確立後、4Way-Handshakeと呼ばれるプロトコルで乱数とお互いのMACアドレスを交換し、それらとマスターキーを組み合わせて512bit（但し、AESを使用する場合は384ビット）のテンポラリキー（PTKと呼ばれる）を生成します。 

実際に暗号化につかうテンポラリキーは、「マスターキー」と「MACアドレス」と「乱数」の3つから生成されています。

今回はパスフレーズ(WPAキー)は知っている前提なので、マスターキーは既に手元にあります。残りの2つも4Way-Handshakeの最中に大っぴらに交換されるため、**APに接続する最初の通信**もしくは**テンポラリキーの更新タイミング**をキャプチャすることが出来れば、復号に必要な材料は揃ってしまうという事ですね。

とりあえず復号しているのが確認できればいいってことであれば、Wi-Fiを一旦切って繋ぎ直せばOK。((今回の例だと、iPhoneのWi-FiをOFF/ONする。))

無事にテンポラリキーを入手できれば、あとはWiresharkが勝手に復号して表示してくれます。




### キャプチャを見やすく
キャプチャフィルタを設定していない状態だと、掴んだ通信を片っ端から表示していくので、一瞬でとんでもない量になってしまいます。近所にマンションがあったりするとえらいことに。  
調査しているアクセスポイントだけに絞ってあげましょう。

1. Wiresharkのメニューから「Stastics」→「WLAN Traffic」
2. 対象のSSIDを選択して右クリック→「Apply as Filter」→「Selected」→「BSSID」

これで、関係ないSSIDは表示されないようにディスプレイフィルタがかかります。

------

## 感想
簡単にできるんだろうなーと思いつつも試したことがなかったので、やってみて良かったと思います。モニターモードに設定するところとか、今回初めて触ったし。

あとは、iPhone・Androidの通信をキャプチャしたいときに便利です。これまではAPとルータの間にバカHubを挟んで使っていたもので。

最近はSIMカードで認証するAPなんかも出てきているので、こういう危ない状態は一過性のものかもしれませんが、危ないことは意識しておきたいところ。

実は公衆無線LAN向けのAPでは何かの対策がされていたりするのかな？試してみたいけど、法に触れる予感。
