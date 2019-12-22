---
templateKey: 'blog-post'
title: はてなブログの記事下のAdsenseをPC・スマホ版で切り替える方法
date: 2013-09-13
redirect_from: 
  - /entry/2013/09/13/003407
  - /2013/09/13/hatenablog-adsense.html
tags:
  - Tips
---

はてなブログにAdsense等を埋め込む際、挿入場所として使えるのはヘッダ・フッタ・サイドバー、そして記事ページ下部です。
記事ページの場合、「スマートフォン版にも表示する」という設定項目があるのですが、PC版向けに埋め込んだコードをそのまま表示するだけなので、色々と不都合があったりします。(PC向けの広告サイズをスマートフォンで表示すると、画面からはみ出てしまったり。)

なので、UserAgentで機種判別をして表示を切り替えるようにしてみました。

------

参考にしたのはこちらの記事。

[ユーザーエージェントによってPCとスマートフォン（iPhone / Android）を振り分ける方法いろいろ（PHP / JavaScript / .htaccess等）](http://html5-css3.jp/smartphone/pc-iphone-android-php-javascript-htaccess.html)


## 埋め込み用コード例

サイト運営者IDとかは適当です。

```javascript
<script type="text/javascript"><!--
google_ad_client = "ca-pub-0000000000000000）";    //Adsenseのサイト運営者ID
if ((navigator.userAgent.indexOf('iPhone') > 0 && 
     navigator.userAgent.indexOf('iPad') == -1) || 
     navigator.userAgent.indexOf('iPod') > 0 || 
     navigator.userAgent.indexOf('Android') > 0) {
    /* スマートフォン向けはこっち */
    google_ad_slot = "0000000000";    //広告ユニットのID
    google_ad_width = 000;    //横幅
    google_ad_height = 000;    //高さ
}
else {
    /* それ以外はこっち */
    google_ad_slot = "0000000000";
    google_ad_width = 000;
    google_ad_height = 000;
}
//-->
</script>
<script type="text/javascript"
src="http://pagead2.googlesyndication.com/pagead/show_ads.js">
</script>
```

WindowsPhoneとかTizenがシェアを伸ばしてきたら、適宜条件を増やしましょう。



## つかいかた

PC版とスマートフォン版で広告を分けたいので、Adsenseのサイトで広告ユニットを別個に作成します。
サイズはこのページを参考に。
[モバイル広告の例 - AdSense ヘルプ](https://support.google.com/adsense/answer/185668?hl=ja&ref_topic=29561)

それぞれのユニットの広告コードを取得すると、こんな感じに出てくると思います。

```javascript
<script type="text/javascript"><!--
google_ad_client = "ca-pub-1708252390535922";
/* うわずみ記事下モバイル */
google_ad_slot = "9971835900";
google_ad_width = 320;
google_ad_height = 50;
//-->
</script>
<script type="text/javascript"
src="http://pagead2.googlesyndication.com/pagead/show_ads.js">
</script>
```

この中から、
-google_ad_client
-google_ad_slot
-google_ad_width
-google_ad_height
の値だけ貰ってきて、上のコード例の同じ箇所を置き換えればOKです。


コードを貼り付けたら、「スマートフォン版にも表示する」にチェックを入れるのを忘れずに。

![スマートフォン版にも表示する](/img/2013-09-13-hatenablog-checkbox.jpg)
