---
templateKey: 'blog-post'
title: Chatwork用のRuboty Adapterを作った
date: 2018-10-26
redirect_from: 
  - /2018/10/26/ruboty-chatwork_webhook.html
tags:
  - Ruboty
  - Ruby
  - Chatwork
---

書くことがなかったわけじゃなかったけど、すごく放置してしまった。ネタができたので書きます。

## やったこと

RubotyでChatworkを使うためのアダプタを作りました。

[honeniq/ruboty-chatwork_webhook](https://github.com/honeniq/ruboty-chatwork_webhook)

実は[mhag/ruboty-chatwork](https://github.com/mhag/ruboty-chatwork)というアダプタが既にあるのですが、こちらはREST APIを使ってチャットルームを監視するので、リクエスト回数制限が常につきまといます。

回避方法を色々考えていたときにWebフックが使えることに気づき、自分でアダプタを作ることにしました。

## 中身の解説

Rubotyアダプタとしての入力はWebフック、出力はREST APIと繋がるイメージです。Webフックを待ち受けるためにWEBrickを使いました。

### ChatworkのWebフック

Chatworkの[Webフック](http://developer.chatwork.com/ja/webhook.html)は、特定のイベントが発生したときに指定したURLに通知することができる機能です。対象にできるイベントは以下2種類で、Chatworkの設定画面で指定します。

- 指定したルームへのメッセージ送信・編集
- 自分宛の発言(メンション)

Webフック(POSTリクエスト)のリクエストボディには、こんな感じのJSONが入っています。

```
{
    "webhook_setting_id": "12345",
    "webhook_event_type": "mention_to_me",
    "webhook_event_time": 1498028130,
    "webhook_event":{
        "from_account_id": 123456,
        "to_account_id": 1484814,
        "room_id": 567890123,
        "message_id": "789012345",
        "body": "[To:1484814]おかずはなんですか？",
        "send_time": 1498028125,
        "update_time": 0
    }
}
```

``webhook_event`` -> ``body`` の中に発言の本文が入っているので、これをRubotyに引き渡せば入力のお仕事は完了です。

### To記法

あ、引き渡す前にちょっとだけ加工しています。

- To記法を使ったメンション
  - "[To:123456] Rubotyさん\nruboty help"
- Botとの1対1チャットでの発言
  - "ruboty help"  ※To記法をつけなくてもメンションになる

Chatworkでのメンションは、よくある「@honeniq」パターンではなく、独自の記法を使います。そのままだとRubotyで扱えないので、To記法を含む1行目全体を除去しています。

アダプタからの出力は普通にChatworkの[REST API](http://developer.chatwork.com/ja/endpoints.html)を叩いているだけなので、割愛。


## 使い方

他のRubotyプラグインと同様、Ruboty本体のGemfileに"ruboty-chatwork_webhook"を追加します。

環境変数は2つ。

- CHATWORK_API_TOKEN
  - REST API経由の発言に利用
- WEBHOOK_LISTEN_PORT
  - Webフックを待ち受けるWEBrickのポート番号

### インターネットへの公開

上記で設定したポートに、ChatworkサーバーからHTTPSで接続できるようにしないといけません。ChatworkのWebフック機能はSSL必須です。

とりあえず試しに使ってみるときは``ngrok``等で一時的に公開してあげると良いと思います。

Chatworkの設定画面で、公開先のURLをWebフックの送信先に指定すれば、準備完了です。


## ToDo

- アダプタ単体でSSL化できるようにしたい
- Slack等と違って、``ruboty-google_image``が返してくるURLを画像として展開してくれない どうにかしたい
- READMEをちゃんとしたい

