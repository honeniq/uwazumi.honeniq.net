---
templateKey: 'blog-post'
title: GitHub Pagesにカスタムドメインを設定する
date: 2018-01-14
redirect_from: 
  - /2018/01/14/github-pages-custom-domain.html
tags:
  - GitHubPages
  - Jekyll
  - Tips
---

といっても、大してやることはありません。

* [Quick start: Setting up a custom domain](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/)
* [Adding or removing a custom domain for your GitHub Pages site](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/)

このあたりにやり方はほぼ書いてあります。


## 今回のシチュエーション

GitHub Pages上に作ったサイトを、自分が保有しているドメインのサブドメインから見られるようにします。

* GitHub Pages : honeniq.github.io
  * リポジトリ : [honeniq/honeniq.github.io](https://github.com/honeniq/honeniq.github.io)
* カスタムドメイン : uwazumi.honeniq.net

独自ドメイン「honeniq.net」は[VALUE-DOMAIN](https://www.value-domain.com/)で取得・管理しています。


## DNS設定

VALUE-DOMAINの設定値を一部抜粋。 
他のサービスだと表記が違ったりすると思いますので、適宜置き換えて下さい。

```
a honeniq.github.io. 192.30.252.153
a honeniq.github.io. 192.30.252.154
cname uwazumi honeniq.github.io.
```

「uwazumi.honeniq.net」のCNAMEレコードと、「honeniq.github.io」のAレコードを追加しています。 
Aレコードに設定しているIPアドレスは、GitHubの解説ページ([Setting up an apex domain](https://help.github.com/articles/setting-up-an-apex-domain/))を参照して設定しました。GitHub Pagesのホストのアドレスらしいです。

### 注意点

DNS設定の更新後、設定した内容が実際に使えるようになるまで数時間〜丸一日くらいかかる場合があります。 
TTL値の設定次第だったりしますが、それなりに時間がかかることは考慮に入れておいてください。

### 確認

```
~$ nslookup uwazumi.honeniq.net
  (略)

Non-authoritative answer:
uwazumi.honeniq.net	canonical name = honeniq.github.io.
honeniq.github.io	canonical name = sni.github.map.fastly.net.
Name:	sni.github.map.fastly.net
Address: 151.101.73.147
```


## GitHub Pagesサイトのリポジトリ設定を変更

今回の場合だと、[honeniq/honeniq.github.io](https://github.com/honeniq/honeniq.github.io)のSettingsを編集します。

![カスタムドメイン設定]({{ site.url }}/assets/2018-01-14-custom-domain.png)

[Options] - [Custom domain]に、GitHub Pagesの別名として使いたいドメインを設定します。 
今回は「uwazumi.honeniq.net」と入力しました。


## ``_config.yml`` の編集

```yaml
url: "http://uwazumi.honeniq.net" # the base hostname & protocol for your site, e.g. http://example.com
```

ベースURLの設定を変えておきます。

最後にちゃんとPushするのを忘れずに。
