---
templateKey: 'blog-post'
title: GitHub PagesをCloudflareでSSL化する
date: 2018-01-16
redirect_from: 
  - /2018/01/16/github-pages-ssl.html
tags:
  - GitHubPages
  - Jekyll
  - Tips
---

## 今回のシチュエーション
[前回の記事]({{ site.url }}/2018/01/14/github-pages-custom-domain.html)でカスタムドメインでのアクセスができるようになったサイトを、今度はHTTPS化します。

* GitHub Pages : honeniq.github.io
  * リポジトリ : [honeniq/honeniq.github.io](https://github.com/honeniq/honeniq.github.io)
* カスタムドメイン : uwazumi.honeniq.net
  * ドメイン管理 : [VALUE-DOMAIN](https://www.value-domain.com/)
* SSL終端/CDN : [Cloudflare](https://www.cloudflare.com/)

GitHub PagesはカスタムドメインのHTTPS化に対応していないので、SSL終端ができるCDNサービスのCloudflareを利用します。

[ブラウザ] - [Cloudflare] - [GitHub Pages]

こんな感じに、ブラウザとGitHub Pagesの間に挟まる構成です。


## DNSレコード設定

CloudflareにDNSサーバーの役割もお願いすることになるので、DNSレコードを設定していきます。 

### 自分のドメインをスキャン

ユーザー登録を済ませてログインすると、まずは管理しているドメイン名を入力するよう促されます。 
入力したドメインのDNSレコードをCloudflareがスキャンして、推奨設定的なものを出してくれるようです。

### DNS設定を作成

先ほど出てきた推奨設定は既に追加されているので、既存の設定(今回であればVALUE-DOMANのDNS設定)と見比べて、足りない部分を追加しておきます。

![DNS設定]({{ site.url }}/assets/2018-01-16-dns-settings.png)

こんな感じに、[GitHub Pagesのカスタムドメイン化]({{ site.url }}/2018/01/14/github-pages-custom-domain.html)で設定したDNSレコードを移植しました。


## ネームサーバーの変更

使いたいドメイン(今回はhoneniq.net)のネームサーバーをCloudflareが用意したものに変更する必要があります。

### ネームサーバーの確認

Overviewの画面でのステータスが、「not Activate」になっていて、指定するネームサーバーの情報が出ていると思います。

![Overview - not Active]({{ site.url }}/assets/2018-01-16-status-not-active.png)


### レジストラのネームサーバーを変更

レジストラ(今回はVALUE-DOMAIN)の管理画面に設定項目があるはずなので、Cloudflareに指定されたサーバーに変更します。

![ネームサーバー変更]({{ site.url }}/assets/2018-01-16-value-domain-ns.png)


### Active化を確認

not Active画面の「Recheck Nameservers」を押して、Activeに変更されることを確認します。

![Overview - Active]({{ site.url }}/assets/2018-01-16-status-active.png)

変わらない場合はまだ設定が反映されていないので、しばらく待って再試行。


## ``_config.yml``の修正

ここまでの設定でサイト自体はHTTPS化されていますが、記事内の画像等のURLがHTTPのままになっているので、ブラウザによっては警告が出てしまいます。

```yaml
url: "https://uwazumi.honeniq.net" # the base hostname & protocol for your site, e.g. http://example.com
```

ベースとなるURLもHTTPSに変更しておきます。

## その他

### HTTPアクセスを転送

HTTPでアクセスしてきたユーザーを全てHTTPSに転送する設定です。

![HTTPS転送設定]({{ site.url }}/assets/2018-01-16-always-use-https.png)

設定後、すぐに反映されました。 
HTTPでアクセスすると即座にリダイレクトされるようになります。
