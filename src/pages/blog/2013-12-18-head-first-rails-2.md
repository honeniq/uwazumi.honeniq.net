---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (2章)
date: 2013-12-18
redirect_from: 
  - /entry/2013/12/18/161535
  - /2013/12/18/head-first-rails-2.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

[前回から](http://uwazumi.honeniq.net/entry/2013/12/15/152243)引き続き、「Head First Rails」でのお勉強中に躓いたところメモ。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

内容がどうこうというよりも、Rails2とRails4の違いを解消するのがほとんどのような気も。より深く知るきっかけにもなるので、これはこれでアリだと思って進めます。

## 躓いたところ

### P50 ruby script/generate model
他と同様、呼び出し方が変わっています。
```
rails generate model ad name:string description:text price:decimal seller_id:integer email:string img_url:string

```

### P51 サンプルデータのダウンロード
本の通りで問題ないのですが、せっかくなのでリンク。

[Head First Labs from O'Reilly](http://www.headfirstlabs.com/books/hfrails/)

ページ下部に「[development.sqlite3](http://www.headfirstlabs.com/books/hfrails/code/development.sqlite3)」があります。

### P52 ruby script/generate controller
他と同じ。
```
rails generate controller ads
```

### P53 注釈のruby script/dbconsole
これも。
```
rails dbconsole
```
使い方はよく分かっていませんが、とりあえず``.dump ads``とすればテキストでDBの中身を表示してくれました。``.exit``で終了。

### P57 config/routes.rb
中身がぜんぜんちがう。

用意されているファイルはほぼコメントで、ルートは一つも入っていない状態。コメントを参考に、こう書いてみました。

```ruby
Mebay::Application.routes.draw do
 get 'ads/:id' => 'ads#show'
end
```

これがややこしくて、本での書き方とぐぐって出てくる書き方とRails4の書き方が結構違う。

* 本での書き方(Rails2系？) : ``map.connect '/ads/:id', :controller=>'ads', :action=>'show'``
* よく見る書き方(Rails3系？) : ``match 'ads/:id' => 'ads#show'``
* Rails4での書き方 : ``get 'ads/:id' => 'ads#show'``

[ルーティング(routes) -  - Railsドキュメント](http://railsdoc.com/routes)

Rails4からはガラっと変わってしまった模様。**ややこしいな！！**  
Rails4からは、HTTPリクエストとメソッド名を一致させるようにしたのかな？


### P74 自分で考えてみようの答え
ルート : ``get 'ads/' => 'ads#index'``  

コントローラーのコードの回答が「index」だけってのがちょっと意味不明で納得いきませんが、後でindexページの実装を行う箇所があるので、ここはとりあえず空っぽのindexメソッドを作っておけばいいのかなあ？

### P76 routes.rb
それぞれのルートは既出ですけど、優先度順に並べ替えたらこうなります。

```ruby
Mebay::Application.routes.draw do
  get 'ads/' => 'ads#index'
  get 'ads/:id' => 'ads#show'
end
```

コメント部分は全部カットしてます。

### P97 静的コンテンツの置き場所
本では/publicフォルダ配下に置くように書いていますが、Rails4では/app/assetsの下に置くようです。

* /app/assets/images : 画像
* /app/assets/javascripts : JavaScript
* /app/assets/stylesheets : スタイルシート

/publicフォルダが無くなったわけではなく、404・502等のエラーページやrobots.txtが置かれています。また、本と同様に/public/imagesに画像を置いても、使えない訳ではなさそう。

[Railsの基礎知識 -  - Railsドキュメント](http://railsdoc.com/rails_base#フォルダ構造:title)

### P98 試運転時に画像が見えない
静的コンテンツの置き場所が変わった関係で、パスの書き方も変わっている様子。  
/app/assets/stylesheets/default.cssを編集します。

```css
html, body { height: 100%; }

body { background-color: #e6dacf; background-image: url('bg.png'); background-position: top center; background-repeat: repeat-y; font-family: "Trebuchet MS", Arial, san-serif; font-size: 14px; margin: 0; padding: 0; text-align: center; }

a { color: #000; }
a:hover { background: #000; color: #e6dacf; }

div#wrapper { min-height: 100%; }
* html div#wrapper { height: 100%; }

div#header { background: url('bgHeader.png'); height: 100px; padding: 8px 0 0 0; }
div#header div { margin: 0 auto; position: relative; width: 800px; }
div#header h1 { left: 0; margin: 0; position: absolute; }
div#header h1 { background: url('logo.png') no-repeat; display: block; height: 0; margin: 0 auto; padding: 92px 0 0 0; overflow: hidden; width: 319px; }

ul#nav { float: right; list-style: none; margin: 0; padding: 69px 0 0 0; }
ul#nav li { background: #e6dacf; border: solid #000; border-width: 2px 2px 0 2px; float: left; margin: 0 0 0 2px; }
ul#nav a { display: block; padding: 3px 6px 0 6px; text-decoration: none; }
ul#nav a:hover { display: block; padding: 3px 6px 7px 6px; }
ul#nav a.current { padding-bottom: 8px; }
ul#nav a.current:hover { background: #e6dacf; color: #000; cursor: default; }

div#content{ margin: 0 auto; padding: 20px 0; text-align: left; width: 720px; height: 300px}
div#content img { border: 10px solid #000; float: right; margin: 18px 0 0 20px; }
div#content h2 { border: solid #999; border-width: 0 0 5px 0; font-size: 28px; margin: 0; padding: 0 0 10px 0; }
div#content p { margin: 0; padding: 4px 6px 10px 0; }


div#clearfooter { height: 110px; }

div#footer { background: url('bgFooter.png'); height: 110px; margin: -110px auto 0 auto; }

```
画像ファイルのパスを「xxx.png」に変更します。/app/assets/imagesに入れておけば、ファイル名だけで見えるみたい。

/public/imagesに置いた場合は、本と同様に「/images/xxx.png」としてあげれば良いです。

- [image_tagメソッドを使ったイメージタグの作成 - Ruby on Rails入門](http://www.rubylife.jp/rails/template/index11.html) 
- [画像の置き場所によるパスの指定方法の違い - kobaken75の日記](http://d.hatena.ne.jp/kobaken75/20120410/1334038386)
