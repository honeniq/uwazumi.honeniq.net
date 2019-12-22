---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (3章)
date: 2014-01-12
redirect_from: 
  - /entry/2014/01/12/232605
  - /2014/01/12/head-first-rails-3.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

[2章の補足](http://uwazumi.honeniq.net/entry/2014/01/12/215416)に引き続き連投。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

3章はScaffoldがやってくれていた部分を、1つずつ作りながら追いかける、という構成になっています。楽しくなってきたぞ！

そして、エラーが出たときの苦労も増してきたように思います。


## 躓いたところ

### P108 追加するルート
2章で書いたときと同様、Rails4向けの書き方で書きます。

```ruby
Mebay::Application.routes.draw do
  get 'ads/new' => 'ads#new'
  get 'ads/create' => 'ads#create'
  get 'ads/' => 'ads#index'
  get 'ads/:id' => 'ads#show'
end
```

### P112 new.html.erb

``form_for``の呼び方が変わっています。

```ruby
<%= form_for(@ad,:url=>{:action=>'create'}) do |f| %>
```
Rails3以降は``<%=``で始めるように変わったとの事。ややこしい。

[form_for - リファレンス -  - Railsドキュメント](http://railsdoc.com/references/form_for:title)


### P123 試運転の結果
本ではテンプレートが見つからない旨のエラーが出ることになっていますが、それとはまた違ったエラー「ActiveModel::ForbiddenAttributesError」が出ました。

調べてみたところ、これはRails4からコア機能として導入された**Strong Parameters**って奴らしいです。本の例のように```@ad=Ad.new(params[:ad])```と一括代入する場合、HTTPリクエストを少しいじると「id」や「updated_at」の値を改変できてしまうという脆弱性((Mass Assignment脆弱性と言うらしい。))が生まれてしまいます。そこで、コントローラ側でホワイトリスト形式でパラメータを検証するようになってるみたい。

ホワイトリストなので、初期状態では何も通してくれません。そのせいでエラーになっていたようです。

[Rails4 の Strong Parameters でリクエストパラメータを検証する](http://www.techscore.com/blog/2013/01/29/rails4-%E3%81%AE-strong-parameters-%E3%81%A7%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%82%92%E6%A4%9C%E8%A8%BC%E3%81%99%E3%82%8B/)  
[Rails4のMass Assignment脆弱性対策のStrong Parametersについて](http://wada811.blogspot.com/2013/08/strong-parameters-mass-assignment-vulnerability-countermeasure-in-rails4.html#strong-parameters:mass-assignment-vulnerability-countermeasure-in-rails4)

こちらの記事を参考に、変更。

```ruby
  def create
    @ad = Ad.new(ad_params)
    @ad.save
  end
  
  private
    # Never trust parameters from the scary internet, only allow the white list through.
    def ad_params
      params.require(:ad).permit(:name, :description, :price, :seller_id, :email, :img_url)
    end
```

これで、本と同じ「Template is missing」に辿り着けました。


### P138 route.rb
たびたび出てきましたが、書き方が変わっています。  
今まで通りに``map.connect``を``get``や``post``に置き換えたらOKかと思いきや、ここでエラー。

```
No route matches [PATCH] "/ads/37/update"

```

「PATCH」？？  
どうやら[PATCHリクエスト](http://railsdoc.com/routes#PATCH%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88(patch))というものが存在するようです。

編集画面のフォームからPATCHリクエストが届いているのに、対応したルートが無いからエラーになったわけですね。

#### PATCHリクエストって？

ブラウザで開いている編集画面のソースコードを見てみました。以下抜粋。

```html
<form accept-charset="UTF-8" action="/ads/37/update" class="edit_ad" id="edit_ad_37" method="post"><div style="margin:0;padding:0;display:inline"><input name="utf8" type="hidden" value="&#x2713;" /><input name="_method" type="hidden" value="patch" />
```

formのmethod属性では「post」となっていますが、それとは別に「_method」に「patch」が渡っています。何でこんな回りくどいことを？と思って調べたら、納得。

> Rails では PUT や DELETE メソッドなどに対応していない Web サーバに対応するために、hidden フィールド（）で本来のメソッドを送信しているので、その部分で patch という値が送信されるようになります。アプリケーションコードでは、コントローラで request.patch? で判定することになります。  
>[Rails 4.0 では HTTP の PATCH メソッドで更新する](http://d.hatena.ne.jp/benikujyaku/20120226/1330236067)

#### routes.rbに追加

というわけで、patchリクエストにも対応できるようなルートにします。

```ruby
  get 'ads/:id/edit' => 'ads#edit'
  patch 'ads/:id/update' => 'ads#update'  
```

この2つを一番上に追加。これでこのエラーは消えました。

・・・代わりに別のが出ましたが。


### ActiveModel::ForbiddenAttributesError が出る

またお前か。

とりあえずcreateのときと同じく、``@ad = Ad.find(params[:id])``を``@ad = Ad.find(ad_params)``と置き換えてみたところ、「Unknown key: name」というエラーに変わりました。えー。

>Strong Parameters are only used when creating/updating database records. They effectively replace the Rails 3 attr_accessible calls in your models.  
>[Rails 4 Strong Parameters Unknown Key](http://stackoverflow.com/questions/20804838/rails-4-strong-parameters-unknown-key)

Strong ParametersはDBへの書き込みが発生する場合（=creating/updating）のみ効くってことですね。なるほど。

その辺りを踏まえて、ads_controller.rbのupdateメソッドはこんな感じに。
```ruby
  def update
    @ad = Ad.find(params[:id])
    @ad.update_attributes(ad_params)
    redirect_to "/ads/#{@ad.id}"
  end
```
無事、エラーも消えてくれました。よかった。

### P148 追加するルート

いつものやつです。  
個人的には、ここの命名が気になります。パスが/deleteなのに、メソッド名はdestroy。なぜ？

```ruby
Mebay::Application.routes.draw do
  get 'ads/:id/edit' => 'ads#edit'
  patch 'ads/:id/update' => 'ads#update'  
  get 'ads/:id/delete' => 'ads#destroy'
  get 'ads/new' => 'ads#new'
  post 'ads/create' => 'ads#create'
  get 'ads/' => 'ads#index'
  get 'ads/:id' => 'ads#show'
end
```

ルートもだいぶ長くなってきたところで、3章が終わりました。とうとうMeBayアプリケーションともお別れですね。
