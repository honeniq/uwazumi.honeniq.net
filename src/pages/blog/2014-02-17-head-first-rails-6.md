---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (6章)
date: 2014-02-17
redirect_from: 
  - /entry/2014/02/17/233951
  - /2014/02/17/head-first-rails-6.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

そろそろ、中断期間が長いと頭がこんがらがってくるようになりました。複雑になってきてますね。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

どこを編集したのかも分からなくなってくるので、勉強も兼ねてバージョン管理とかした方がいいのかもしんない。

## 躓いたところ

### P222 アプリケーションの作成

もはやおなじみ。

```ruby
rails new coconut
```

### P222 Scaffoldの実行

これもいつも通り。

```ruby
rails generate scaffold flight departure:datetime arrival:datetime destination:string baggage_allowance:decimal capacity:integer
```

もうひとつ。
```
rails generate scaffold seat flight_id:integer name:string baggage:decimal
```

``rake db:migrate``するのを忘れずに。


### P230 予約フォームのパーシャル

Rails4でScaffoldが作ってくれる「new.html.erb」はこんな感じです。

```html
<h1>New seat</h1>

<%= render 'form' %>

<%= link_to 'Back', seats_path %>
```

本のサンプルと違って、``<%= render 'form' %>``でフォームを埋め込んでいます。フォルダを見てみると、「_form.html.erb」っていうパーシャルも一緒に生成されていました。

これをそのまま使ってしまうと、flightsの新規作成フォームを埋め込むことになってしまいそうです。別フォルダのパーシャルを見に行くように出来れば問題なさそうですけど。

```html
<h1>New seat</h1>

<%= render '/seats/form' %>

```
こんな感じで、パス付きで書いてあげればOKでした。どうやら、``'form'``とパス無しで書くと同じフォルダから探して、パス付きだと「/app/views」の下から探すようです。便利。

これで本と同じエラーが出ました。


### P235 パーシャルに変数を渡す

ちょっと困ったことになりました。  
本では「show.html.erb」から「_new_seat.html.erb」を呼ぶだけなのですが、Rails4のScaffoldの作法に従うと、さらに「_form.html.erb」も絡んできます。

さきほど編集したところですが、「_new_seat.html.erb」をさらに変更。

```html
<h1>New seat</h1>

<%= render '/seats/form', :seat=>seat %>

```

続いて/seatsの中の「_form.html.erb」も。  
``@seat``の@を取っただけですけど。

```html
%= form_for(seat) do |f| %>
  <% if seat.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(seat.errors.count, "error") %> prohibited this seat from being saved:</h2>

      <ul>
      <% seat.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :flight_id %><br>
    <%= f.number_field :flight_id %>
  </div>
  <div class="field">
    <%= f.label :name %><br>
    <%= f.text_field :name %>
  </div>
  <div class="field">
    <%= f.label :baggage %><br>
    <%= f.text_field :baggage %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>

```

これで動きました。

renderに引数を渡すとき、本のように``:partial=>"new_seat"``って書いたほうがいいのかなあ？

