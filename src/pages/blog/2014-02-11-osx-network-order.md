---
templateKey: 'blog-post'
title: OSXでWi-FiをEthernetよりも優先させる
date: 2014-02-11
redirect_from: 
  - /entry/2014/02/11/233550
  - /2014/02/11/osx-network-order.html
tags:
  - ネットワーク
  - Mac
---

自宅のMacは普段はWi-Fi運用で、ネットワーク機器の設定等で必要なときだけEthernetポートを使います。

このときに困るのが、デフォルトルートをEthernet側に持っていかれてしまうこと。インターネットに向けての通信が全てEthertnetポートに抜けていってしまうので、実質どこにも行けない状態です。

OS標準の動作としては、Wi-FiよりもEthernetを信頼するってのは理解できるのですが、このままだと困るので変更してみます。


## サービスの順序を変更
やり方は意外と簡単でした。さすがに想定されていたみたいです。

[Mac OS X 10.6: ネットワーク接続の優先順位を変更する](http://support.apple.com/kb/PH7119?viewlocale=ja_JP&locale=ja_JP)


### 設定方法
環境はOSX 10.9.1 Mavericks。  
「システム環境設定」→「ネットワーク」を開く。

![OSX Network Setting](/img/2014-02-11-network-setting.png)

左下の歯車アイコンを押して「サービスの順序を設定…」を選択。


![OSX Network Service Order](/img/2014-02-11-service-order.png)

項目をドラッグして、Wi-FiをEthernetよりも上に持ってくる。「適用」。以上。


## ルーティングテーブルの変化
一応、ルーティングテーブルでもどういう動作をしているのか見てみます。

OSXでは``netstat -r``で確認できます。


### 初期状態
```
Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
default            192.168.1.1        UGSc           40        0     en0
default            192.168.10.254     UGScI           8        0     en1
127                localhost          UCS             0        0     lo0
localhost          localhost          UH              4    29510     lo0
```
初期状態はこう。（一部抜粋。）  
「en0」がEthernetで、「en1」がWi-Fiです。



### 変更後
```
Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
default            airport.airport    UGSc           34        0     en1
default            192.168.1.1        UGScI          13        0     en0
127                localhost          UCS             0        0     lo0
localhost          localhost          UH              4    29873     lo0
```
Wi-FiをEthernetよりも上にするとこう。  
「en1」が「en0」よりも上に来ました。これでデフォルトゲートウェイとしてWi-Fiの方が優先されます。  
こっちは何故かGatewayがホスト名表記になってますね。

これで、Wi-FiとEthernet併用時にもWi-Fiからインターネットに抜けられるようになりました。
