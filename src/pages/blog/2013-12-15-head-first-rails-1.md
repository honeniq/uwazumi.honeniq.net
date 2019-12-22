---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (1章)
date: 2013-12-15
redirect_from: 
  - /entry/2013/12/15/152243
  - /2013/12/15/head-first-rails-1.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

オライリーの「[Head First Rails ―頭とからだで覚えるRailsの基本](http://www.oreilly.co.jp/books/9784873114385/)」でお勉強中です。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

3年前くらいの本です。その間にRailsはメジャーバージョンで2つも上がってしまっているので、本の通りには進めない部分が沢山。  
第1章の頭から何カ所か躓いているので、対策を含めてメモ。


## 環境
* MacBook Air late2012
* OSX 10.9
* rails 4.0.0
* ruby 1.9.3p125

------

## 躓いたところ

### P7 ```rails ticket```が通らない。
ticketプロジェクトを作成しようとしているところ。  
新しいバージョンだと、```rails new ticket```とするのが正解。


### P7 プロジェクト作成中にエラー((これはRailsのバージョンの問題ではなく、OSX側が原因ぽいです。))
newをつけて実行すると、ファイルの作成とか色々やってくれた後に、エラーでこける。

```
Installing atomic (1.1.14) 
Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

(中略)

An error occurred while installing atomic (1.1.14), and Bundler cannot continue.
Make sure that `gem install atomic -v '1.1.14'` succeeds before bundling.
```

指示通りにバージョン指定してatomicのみをインストールしてみるも、またもやエラー。

```
/Users/honeniq/.rbenv/versions/1.9.3-p125/lib/ruby/1.9.1/mkmf.rb:381:in `try_do': The compiler failed to generate an executable file. (RuntimeError)
You have to install development tools first.
```
ん？？ develipment toolsって何？？

#### gccのリンクを作ってあげる
どうやら、gccが見つからなくてエラーになっていたようです。  
見つけられるようにリンクを作ってあげれば通りました。

``sudo ln -s /usr/bin/gcc /usr/bin/gcc-4.2``

参考: [XCodeを4.5にバージョンアップした後に、gem(native extensions)のインストールで「You have to install development tools first.」と言われてしまった時の対処法](http://qiita.com/mah_lab/items/e3493a99af31d61c81be)

#### 再度rails new
atomicの1.1.4も無事入ったので続きを。  
と思ったんですが、rails newが途中でこけた場合、どうやって再開すれば良いんだろう？？

とりあえずもう1回やってみました。
``rails new ticket``

既に作成されているファイルとかディレクトリは無視されますが、secret_token.rbだけ上書きするか訊いてきます。「Y」で上書き。
先ほどコケたbundle installの部分も無事完了できたようです。とりあえずこれで。


### P8 ``ruby script/server``が通らない
そんなディレクトリ自体ない。
ticketディレクトリに移動して、``rails server``で立ち上がりました。


### P12 ``ruby script/generate scaffold``が通らない
先ほどと同様、そんなディレクトリはない。  
ticketディレクトリで下記を実行。
```ruby
rails generate scaffold ticket name:string seat_id_seq:string address:string price_paid:decimal email_address:string
```


### P27 4つの.html.erbを編集
本に書かれている4つのファイルに加えて「_form.html.erb」も編集が必要。  
逆に、「edit.html.erb」と「new.html.erb」は編集箇所がない。



### P34 ``ruby script/generate migration``
これも。  
 ``rails generate migration AddPhoneToTickets phone:string``



### P36 .html.erbファイルの編集
先ほどと同じく、「_form.html.erb」も編集する。editとnewは触らない。((多分、editとnewで共通して使うフォーム部分を切り出してるんだな。))

```html
    <%= f.text_field :price_paid %>
  </div>
  <div class="field">
    <%= f.label :phone %><br>
    <%= f.text_field :phone %>
  </div>
  <div class="field">
    <%= f.label :email_address %><br>
```
div要素で囲われていたりして本と少し違うけど、他を参考に電話番号用のフィールドを追加するというのは同じ。

ここまで触って再度試してみると、エラーは出ないけどphoneフィールドへの変更が効かない。

#### コントローラーも変更
だいぶ手こずりました。  
本に書かれているビュー周りのファイル以外にも、コントローラーも変更しないといけないようです。

* /tickets/controllers/concerns/tickets_controller.rb

このファイルの一番下あたり。

```ruby
    def ticket_params
      #params.require(:ticket).permit(:name, :seat_id_seq, :address, :price_paid, :email_address)
      params.require(:ticket).permit(:name, :seat_id_seq, :address, :price_paid, :phone, :email_address)
    end
```
引数に:phoneも足してあげましょう。

#### .json.jbuilder は？
他に、

* /app/views/tickets/index.json.jbuilder
* /app/views/tickets/show.json.jbuilder

この2つのファイルにも同様にカラム名が書かれている部分があったのですが、ここにphoneを足しても足さなくても変化が無いように見えたので、とりあえず外しました。

後々困ることになったら、そのとき対応しましょう。

