---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (4章)
date: 2014-01-14
redirect_from: 
  - /entry/2014/01/14/235721
  - /2014/01/14/head-first-rails-4.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

[3章](http://uwazumi.honeniq.net/entry/2014/01/12/232605)に引き続き4章。

<a href="https://www.amazon.co.jp/First-Rails-%E2%80%95%E9%A0%AD%E3%81%A8%E3%81%8B%E3%82%89%E3%81%A0%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8BRails%E3%81%AE%E5%9F%BA%E6%9C%AC-David-Griffiths/dp/4873114381/ref=as_li_ss_il?ie=UTF8&qid=1515597257&sr=8-1&keywords=Head+First+Rails&linkCode=li2&tag=honeniq0b-22&linkId=0d1e55c54cbc1109c0515719c8f0bb33" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4873114381&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=honeniq0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=honeniq0b-22&l=li2&o=9&a=4873114381" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

4章で新しく使うのは検索機能だけなので、ページ数としては少なめ。  
その代わり、Scaffoldでの自動生成とかルートの記述に関して細かい説明はもうしてくれないので、ここまでの内容をしっかり理解していないと大変。

リファレンス的に使える本ではないので、ちょっと構文を忘れて調べたいときなんかは困りますね。

## 躓いたところ

### P156 Scaffoldの実行
もう4章なので、アプリケーションの作成に関する部分はさらっと流されてしまっています。  
Scaffold作成の前にアプリケーションのディレクトリに移動しておくことを忘れずに。

```sh
rails new client_workouts
cd client_workouts
```

ここでちょっと疑問が。アプリケーション名って、どうつけるのが正しいんでしょうか？この本の例だと、1章がtickets、2〜3章がmebayだったわけですが、これどういうポリシーなんだろう。

client_workoutsアプリだと1章っぽいですが、2〜3章のようにRubyvilleアプリの中にclient_workoutモデルを作るっていうやり方もアリなように思います。どっちでもいいのかもしれないけど。

#### Scaffoldコマンド
いつものように、Rails4では書き方が違います。

```ruby
rails generate scaffold client_workout client_name:string trainer:string duration_mins:integer date_of_workout:date paid_amount:decimal
```

### P164 formタグ

ここでも``<% form_tag``なのか``<%= form_tag``なのか問題。  
Rails3以降は``<%=``が正解。

以下、コードの太字部分のみ。

```ruby
<span style="text-align: right">
  <%= form_tag "/client_workouts/find" do %>
    <%= text_field_tag :search_string %>
    <%= submit_tag "Search" %>
  <% end %>
</span>
```

### P168 実はルートが必要

本には思いっきり「ルートは作成しなくてもよい」とありますが、必要です。

```ruby
ClientWorkouts::Application.routes.draw do
  post 'client_workouts/find' => 'client_workouts#find'
  resources :client_workouts
end
```

試運転時のエラーの出方が結構変わっているので、ちゃんと動いているか心配になるかも。以下に抜粋で載せておきます。

```
Started POST "/client_workouts/find" for 127.0.0.1 at 2014-01-13 23:45:38 +0900
Processing by ClientWorkoutsController#find as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"K7nQYt/TC1TlfeeQ/055HOJuhW7fXsX38MYDWGYLQ1w=", "search_string"=>"HONENIQ", "commit"=>"Search"}
HONENIQ
Completed 500 Internal Server Error in 2ms

ActionView::MissingTemplate (Missing template client_workouts/find, application/find with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder, :raw, :ruby, :jbuilder, :coffee]}. Searched in:
  * "/Users/honeniq/client_workouts/app/views"
):
```

検索窓に「HONENIQ」って入れたときのログ。  
たぶん、4行目がputsで出力された文字列です。


### P176 find.html.erbの中身
index.html.erbから「client_name」に関する列を削除してあげれば良いわけですが、本と若干違います。

違いは本当にちょっとだけで、``<thead>``と``<tbody>``タグがあるかどうか。本の例ではついていないのですが、Rails4のScaffoldで生成したindex.html.erbにはついているので、そちらに合わせることにします。

```ruby
<h1>Listing client_workouts for <%= params[:search_string] %></h1>

<table>
  <thead>
    <tr>
      <th>Trainer</th>
      <th>Duration mins</th>
      <th>Date of workout</th>
      <th>Paid amount</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <% @client_workouts.each do |client_workout| %>
      <tr>
        <td><%= client_workout.trainer %></td>
        <td><%= client_workout.duration_mins %></td>
        <td><%= client_workout.date_of_workout %></td>
        <td><%= client_workout.paid_amount %></td>
        <td><%= link_to 'Show', client_workout %></td>
        <td><%= link_to 'Edit', edit_client_workout_path(client_workout) %></td>
        <td><%= link_to 'Destroy', client_workout, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Client workout', new_client_workout_path %>
```
