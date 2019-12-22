---
templateKey: 'blog-post'
title: TeraTermProのマクロでよく忘れることメモ
date: 2013-11-09
redirect_from: 
  - /entry/2013/11/09/155039
  - /2013/11/09/teratermpro-macro.html
tags:
  - ネットワーク
---

最近NW機器の一括キッティングでTeraTermProのマクロを書くことが多かったのですが、数ヶ月おきにしか触らないため、毎回忘れて困ることがあります。

今後も時々使うことになるので、メモっておきます。


## :computer:プロンプトを待つ
ログイン処理や、コマンド実行完了の判定に使えます。

### 単純にログインするだけ
```
wait 'Login:'
sendln 'admin'

wait 'Password:'
sendln 'abcdefg'
```

### ホスト名等が変わる場合
``Router_osaka_1>``みたいなプロンプトを待つ場合。
waitと違ってwaitregexは正規表現が使えます。

```
waitregex 'Router.*>'
```

------

## マクロへの値の渡し方
``ttpmacro.exe``の実行時にコマンドラインオプションを付けると、マクロ中で変数として受け取れます。

```
 > ttpmacro.exe test.ttl 192.168.0.1 23
```

* param1 : マクロファイル名  
* param2 : 2つめのオプション  
* param3 : 3つめのオプション  
* param4~9 : (略)  
* paramcnt : 値の個数  

```
param1 -> 'test.ttl'
param2 -> '192.168.0.1'
param3 -> '23'
paramcnt -> 3
```

``paramcnt``の数で分岐させることで、引数が無い場合にも対応できます。

### マクロにIPアドレスを渡す

```
DEFAULT_IP = '192.168.0.11'
if paramcnt > 1 then
  IPADDRESS = param2
else
  IPADDRESS = DEFAULT_IP
endif
```

IPアドレスを渡さなかった場合はデフォルト値を使います。

------

## TeraTerm呼び出し時のコマンドラインオプション
``connect``の引数に渡した文字列は、TeraTerm本体へのコマンドラインオプションになります。
参照: [Tera Term Pro コマンドライン](http://ttssh2.sourceforge.jp/manual/ja/commandline/teraterm.html)

### TeraTerm本体に渡すオプションを作る

```
COM_PORT = '5'
BAUDRATE = '9600'
COMMAND = '/C='
strconcat COMMAND COM_PORT 
strconcat COMMAND ' /BAUD=' 
strconcat COMMAND BAUDRATE
connect COMMAND
```

``strconcat``で1個ずつ繋げるのが面倒。  

------

## 文字列配列

文字列配列の初期値は空文字列なので、``.+``と比較してやれば空かどうか判断できます。

```
strdim ARRAY 5
ARRAY[0] = 'A'
ARRAY[1] = 'B'
ARRAY[2] = 'C'
ARRAY[3] = 'D'
;ARRAY[4] = 

for i 0 4
  strmatch ARRAY[i] '.+'
  if result = 0 then 
    break
  endif
  sendln ARRAY[i]
next
```

```
->A
->B
->C
->D
```

------

## waitで分岐
waitに複数の引数を渡すと、何番目の文字列を受け取ったのか教えてくれます。

```
wait 'A' 'B'
if result = 1 then
  messagebox 'A' '結果'
else
  messagebox 'B' '結果'
endif
```

------

## その他
ほぼ毎回書くこと。

### ログの取得

```
; ログ取得
LOG_FILE = 'C:\Temp\teraterm.log'
logopen LOG_FILE 0 0

; ログ閉じる
logclose
```

ログは不要っぽくてもとりあえず取る。


```
; ログファイル名をクリップボードにコピー
var2clipb LOG_FILE
```

戻り値の返し方が分かってないので、クリップボード経由で。
いつもVBAからTeraTermを呼んで使っているので、結果としてのログをVBA側で拾えるように。

```
; シリアル切断・ポート解放
disconnect
; TeraTerm終了
; /Vオプションを付けている場合は不要
closett
```
後始末。

