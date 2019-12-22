---
templateKey: 'blog-post'
title: オライリーの「Head First Rails」で躓いたところ (2章補足)
date: 2014-01-12
redirect_from: 
  - /entry/2014/01/12/215416
  - /2014/01/12/head-first-rails-2append.html
tags:
  - プログラミング
  - Ruby
  - Rails
---

明けましておめでとうございます。  
年末年始は転職やら大掃除やらで更新していませんでしたが、じわじわ進めています。

今回は、2章の記事に載せ忘れていたエラーについて書きます。


## 躓いたところ

### ActiveRecord::PendingMigrationError が出る
実は2章をやっている最中にも同じエラーが出てて、色々やってたら何故か直ったので忘れてました。3章で再発したので、今度は立ち向かおうと思います。

エラーメッセージ内に解決策として``rake db:migrate RAILS_ENV=development``をやってみろと書いてありますが、これが通りません。

```
==  CreateAds: migrating ======================================================
-- create_table(:ads)
rake aborted!
An error has occurred, this and all later migrations canceled:

SQLite3::SQLException: table "ads" already exists: CREATE TABLE "ads" ("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, "name" varchar(255), "description" text, "price" decimal, "seller_id" integer, "email" varchar(255), "img_url" varchar(255), "created_at" datetime, "updated_at" datetime) 

Tasks: TOP => db:migrate
(See full trace by running task with --trace)
```

既にadsテーブルが存在するところに、もう一回同じテーブルを作ろうとするから弾かれると。そりゃそうだ。  
問題は、何故マイグレーションができていないと認識されているか。

2章でダウンロードしたサンプルデータ入りの「developpment.sqlite3」を、元のファイル((自分の環境でdb:migrateして生成されたファイル))に切り戻すと、このエラーは出なくなります。ってことはDBの中身が原因なのかなと思い、調べてみました。

「development.sqlite3」を差し替えつつ、``rails dbconsole``→``.dump``した結果。

#### 自分の環境で、db:migrateで生成したファイル

```sql
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE "schema_migrations" ("version" varchar(255) NOT NULL);
INSERT INTO "schema_migrations" VALUES('20131215140724');
CREATE TABLE "ads" ("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, "name" varchar(255), "description" text, "price" decimal, "seller_id" integer, "email" varchar(255), "img_url" varchar(255), "created_at" datetime, "updated_at" datetime);
CREATE UNIQUE INDEX "unique_schema_migrations" ON "schema_migrations" ("version");
COMMIT;
```

#### 本のサイトからDLしてきた、サンプルデータ入りのファイル

```sql
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE "schema_migrations" ("version" varchar(255) NOT NULL);
INSERT INTO "schema_migrations" VALUES('20081127162635');
CREATE TABLE "ads" ("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, "name" varchar(255), "description" text, "price" decimal, "seller_id" integer, "email" varchar(255), "img_url" varchar(255), "created_at" datetime, "updated_at" datetime);
INSERT INTO "ads" VALUES(1,'Typewriter','Old manual typewriter. Many years useful service. Works best with a bottle next to it.',71.95,54,'dhammett@email.com','http://homepage.mac.com/david_griffiths/typewriter.png','','');
(サンプルデータいくつか)
DELETE FROM sqlite_sequence;
INSERT INTO "sqlite_sequence" VALUES('ads',37);
CREATE UNIQUE INDEX "unique_schema_migrations" ON "schema_migrations" ("version");
COMMIT;
```

「schema_migrations」の値が違いますね。もしかして、ここを見てどの時点のマイグレーションが適用されているか判断してる？ってことはここを変更すれば！

1章で作ったticketsアプリでdbconsoleして覗いてみると、新しいmigrationのバージョンをINSERTでどんどん追加していっていました。真似してINSERTして、

```sql
sqlite> INSERT INTO "schema_migrations" VALUES('20131215140724');
sqlite> .dump schema_migrations
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE "schema_migrations" ("version" varchar(255) NOT NULL);
INSERT INTO "schema_migrations" VALUES('20081127162635');
INSERT INTO "schema_migrations" VALUES('20131215140724');
CREATE UNIQUE INDEX "unique_schema_migrations" ON "schema_migrations" ("version");
COMMIT;
sqlite> .exit
```

DBのバージョンを確認してみると、

```
mebay honeniq$ rake db:version
Current version: 20131215140724
```

**上がってる！**  
エラーも無事消えました。

今回はサンプルデータとの不一致だから仕方ないけど、普段はやらない方が良さそう。DBとマイグレーションファイルの数が一致しなくなるし。
