---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (5章)
date: 2014-01-21
redirect_from: 
  - /entry/2014/01/21/232520
  - /2014/01/21/head-first-rails-5.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

ここ1ヶ月ほど、MacBookAirでエディタとターミナルを開きっぱなしです。思い立ったらすぐに始められる環境って大事ですね。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

5章は入力データのバリデーションがテーマです。

ここに来て、3章でお別れした筈のMeBayが復活してくるとは。現実では、数年前に終わった案件の修正で呼び出しを食らったりもするので、これはかなり生々しい演出ですね。


## 躓いたところ

### P212 エラーメッセージのメソッド

```
undefined method `error_messages'
```

そんなメソッド知らないよ、と。  
どうも、Rails3以降ではこのメソッドは無くなっているようです。

参考: [`error_messages' メソッドが廃止されていた](http://ameblo.jp/bewasbeen/entry-11468517401.html)

#### client_workoutsアプリを参考に

Scaffoldで生成したclient_workoutsアプリではどうなっているのかとビューのソースを見てみます。  
「_form.html.erb」の序盤にエラー関係のコードがありました。

```html
  <% if @client_workout.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@client_workout.errors.count, "error") %> prohibited this client_workout from being saved:</h2>

      <ul>
      <% @client_workout.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
```

エラーが発生している場合は、エラーの件数を教えてあげて、それからエラー内容もeachで全部表示するっていう流れでしょうか。

特に問題なく使い回せそうなので、オブジェクトだけ「ad」に変えてコピペします。  
出来上がった「new.html.erb」はこんな感じ。

```html
<h1>New Ad</h1>
<%= form_for(@ad,:url=>{:action=>'create'}) do |f| %>
  <% if @ad.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@ad.errors.count, "error") %> prohibited this ad from being saved:</h2>

      <ul>
      <% @ad.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
  <p><b>Name</b><br /><%= f.text_field :name %></p>
  <p><b>Description</b><br /><%= f.text_area :description%></p>
  <p><b>Price</b><br /><%= f.text_field :price %></p>
  <p><b>Seller</b><br /><%= f.text_field :seller_id %></p>
  <p><b>Email</b><br /><%= f.text_field :email %></p>
  <p><b>Img Url</b><br /><%= f.text_field :img_url %></p>
  <p><%= f.submit "create" %></p>
<% end %>
```

これでエラー表示が出るようになりました。

旧バージョンでは1行で書けていたのに、新バージョンではしっかり書いてあげないといけないんですね。珍しいパターン。


### P216 編集ページのエラー表示

ひとつ上とやっていることは同じですが、せっかくなので「edit.html.erb」を全部載せておきます。

```html
<h1>Editing <%= @ad.name %></h1>
<%= form_for(@ad,:url=>{:action=>'update'}) do |f| %>
  <% if @ad.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@ad.errors.count, "error") %> prohibited this ad from being saved:</h2>

      <ul>
      <% @ad.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
  <p><b>Name</b><br /><%= f.text_field :name %></p>
  <p><b>Description</b><br /><%= f.text_area :description%></p>
  <p><b>Price</b><br /><%= f.text_field :price %></p>
  <p><b>Seller</b><br /><%= f.text_field :seller_id %></p>
  <p><b>Email</b><br /><%= f.text_field :email %></p>
  <p><b>Img Url</b><br /><%= f.text_field :img_url %></p>
  <p><%= f.submit "update" %></p>
<% end %>
```

5章はページ数もソースの変更箇所も少なかったので、すぐに終わってしまいました。  
