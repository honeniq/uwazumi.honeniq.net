---
templateKey: 'blog-post'
title: Pro MicroでバーチャスティックをUSB化してみた
date: 2018-02-04
redirect_from: 
  - /2018/02/04/virtuastick.html
tags:
  - Gadget
  - 電子工作
---

## きっかけ

Steamのゲームを普通のゲームパッドで遊んでいて、ふとアーケードスティックが使いたくなりました。ところが、値段を調べてみると本格的なものは軒並み1万円越えで、思いつきで手を出せる感じではなさげです。

実は、10年前くらいにジャンク品のバーチャスティックを使ってUSB版バーチャスティックを作ったことがあったので、大した手間じゃないなら作ってしまおう、となりました。

最終的にはゲーセンで使われているレバーとボタンを調達して本物っぽいのを作りたいところですが、まずは自分でできるかどうかの手ごたえを得るために、上記のバーチャスティックを流用してやってみました。

## 概要

USBのHID(Human Interface Device)として認識させることができる[Pro Micro](https://www.sparkfun.com/products/12640)(の、互換品)を使って、バーチャスティックのレバーとボタンの入力を受け付けられるようにします。

Pro MicroはArduinoのファミリーに入っている製品ではないので微妙な呼ばれ方をしていますが、中身はArduino互換なのでArduino Studioとかで普通に開発できます。

バーチャスティックには連射切替のような機能もついていますが、まずはスティック+6ボタンが使えればいいやと思ってその範囲だけ作っています。

### 依存ライブラリ

* [MHeironimus/ArduinoJoystickLibrary](https://github.com/MHeironimus/ArduinoJoystickLibrary)

まだデジタルゲームパッド向けの機能しか使ってませんが、ハットスイッチやアクセル・ブレーキみたいな出力もできるみたいなので、マニアックな作りこみもできそうです。

------

## 作成工程

ざっくり三段階で進めていきました。

1. 最小限の構成で動作確認する
2. とりあえずコードを書き切る
3. レバー・ボタンとハンダ付けして実際に使えるようにする

### 1. 最小限の構成で動作確認する

いきなり完成形を目指して作り始めると、ハマったときの調査範囲が広くなってしまうので、まずはPro Microのみで動作確認できるところまでやります。

ライブラリの[サンプルコード](https://github.com/MHeironimus/ArduinoJoystickLibrary/blob/master/Joystick/examples/GamepadExample/GamepadExample.ino)に、方向ボタン+1ボタンのみですが目的に近いものがあったので、まずこれを使いました。

```cpp
  (略)
void loop() {

  // Read pin values
  for (int index = 0; index < 5; index++)
  {
    int currentButtonState = !digitalRead(index + 2);
    if (currentButtonState != lastButtonState[index])
    {
      switch (index) {
        case 0: // UP
          if (currentButtonState == 1) {
            Joystick.setYAxis(-1);
          } else {
            Joystick.setYAxis(0);
          }
          break;
        case 1: // RIGHT
          if (currentButtonState == 1) {
            Joystick.setXAxis(1);
          } else {
            Joystick.setXAxis(0);
          }
          break;
  (略)
        case 4: // FIRE
          Joystick.setButton(0, currentButtonState);
          break;
      }
      lastButtonState[index] = currentButtonState;
    }
  }

  delay(10);
}
```

方向ボタンと普通のボタンは内部的に別々の扱いになっています。方向は``Joystick.setYAxis``と``Joystick.setXAxis``で出力して、ボタンは``Joystick.setButton``を使う模様。

とりあえずこれが動いてしまえば、あとはボタンを6個に増やすだけです。

#### 書き込み&動作確認

サンプルコードをそのままPro Microに書き込んで、動作確認します

![Pro Micro最小限]({{site.baseurl}}/assets/2018-02-04-promicro.jpg)

緑と黒のジャンパでひとつボタンを付けていますが、これはPro Microのリセット用です。ボタンの入力は黄色いジャンパでGNDとD-Inをショートさせて行いました。

書き込んだPro MicroをWindowsのパソコンにUSB接続して、実際に入力を受け取れているか確認していきます。コントロールパネルの「デバイスとプリンター」に「Sparkfun Pro Micro」という名前で認識されるので、こいつの設定画面で入力確認ができます。

サンプルコードだとピン2〜6を使っているので、GNDと各ピンを順番にショートさせていって、PC上でも対応したボタン入力が認識できていればOK。


### 2． とりあえずコードを書き切る

上でも書いていましたが、今回やりたい内容はサンプルコードのボタンが6個に増えただけのものなので、ここで全部書いてしまいました。

ここでも動作確認方法は1のときと同じで、ピンを順番にショートさせてWindows側の入力を確認します。

#### コード

[honeniq/promicro-gamepad](https://github.com/honeniq/promicro-gamepad)

方向ボタンと普通のボタンの関数を分けて、それぞれ内部でループ処理。

```cpp
  (略)
void loop() {
  updateDpad();
  updateButton();

  delay(10);
}

void updateDpad() {
  for (int i = 0; i <= 3; i++) {
    int currentDpadState = !digitalRead(i + OFFSET_DPAD);
    if (currentDpadState != lastDpadState[i]) {
      switch (i) {
        case 0:
          Joystick.setYAxis(-currentDpadState);
          break;
        case 1:
          Joystick.setXAxis(currentDpadState);
          break;
        case 2:
          Joystick.setYAxis(currentDpadState);
          break;
        case 3:
          Joystick.setXAxis(-currentDpadState);
          break;
      }
      lastDpadState[i] = currentDpadState;
    }
  }
}

void updateButton() {
  for (int i = 0; i <= 5; i++) {
    int currentButtonState = !digitalRead(i + OFFSET_BUTTON);
    if (currentButtonState != lastButtonState[i]) {
      Joystick.setButton(i, currentButtonState);
      lastButtonState[i] = currentButtonState;
    }
  }
}
```

#### ピン配置

| ボードの表記 | Digital in  | 役割        |
|:-------------|:------------|:------------|
| A0           | 18          | D-pad Up    |
| A1           | 19          | D-pad Right |
| A2           | 20          | D-pad Down  |
| A3           | 21          | D-pad Left  |
| 2            | 2           | Button 1    |
| 3            | 3           | Button 2    |
| 4            | 4           | Button 3    |
| 5            | 5           | Button 4    |
| 6            | 6           | Button 5    |
| 7            | 7           | Button 6    |

コードの都合上、ボタンのカテゴリ単位で連番になってほしかったので、ちょっと変則的な配置になっています。

Digital-Inの``18``から``21``はD-pad(方向ボタン)用です。時計回りに↑→↓←の順に割り振っています。(Direction-Padの略でD-padらしい。)

``2``から``7``には普通のボタンを割り当てています。サターン的な呼び方だと``2``がAボタンで``7``がZボタンです。OSからは1～6番のボタンとして認識されるはず。


### 3． レバー・ボタンとハンダ付けして実際に使えるようにする

あとはPro Microの各ピンの入力をレバー・ボタンと繋げていくだけです。

![soldering1]({{site.baseurl}}/assets/2018-02-04-soldering1.jpg)

バーチャスティックの内部基盤が結構しっかりついていたので、そのまま流用しました。基盤の端にボタンからの配線が延びているので、ここにビニール線を付けてPro Microと繋ぎます。

レバーはケースの中に小さなボタンが4つ入っていて、上下左右に動かすと対応したボタンが押される構造になっています。これも基盤の端に出力用のコネクタがあったので、そのまま流用してビニール線で延長しました。

![soldering2]({{site.baseurl}}/assets/2018-02-04-soldering2.jpg)

ユニバーサル基板にPro Microと各ボタンからのビニール線を付けて、ハンダブリッジで強引に繋いでハンダ付け完了。

あとは無理やりバーチャスティックの筐体に収めれば完成です。とりあえず動いた！


## 感想

色々と不細工な部分はあるものの、自分で書いたコードで動いているのは楽しいです。

長年放置されていたからかレバーの動きが良くないので、いずれ新品のレバーに交換とかしてみたい。
