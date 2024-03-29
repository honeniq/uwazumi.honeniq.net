---
templateKey: 'blog-post'
title: ローミングを使わずに海外で着信を受ける方法
date: 2012-10-15
redirect_from: 
  - /entry/20121015/1377302838
  - /2012/10/15/no-roaming.html
tags:
  - 旅行
---

先日、海外旅行に行ったんですが、週末利用だったこともあって家族だけに伝えておきました。仕事関係の緊急連絡先を携帯電話にしているので、<span style="color: #800000;">一応出られるようにはしとかなきゃな〜</span>と思い、工夫してみました。

&nbsp;

ローミングを使わずに、
<ul>
	<li>着信転送機能</li>
	<li>IP電話サービス</li>
	<li>現地のインターネット回線</li>
</ul>
この三つを使って実現します。

<span style="color: #003300;">携帯キャリアの着信転送機能でIP電話の番号に飛ばして、渡航先のiPhoneで受ける</span>という寸法です。

iPhone向けに書いていますが、おそらくAndroid端末でも同じことが出来ると思います。

&nbsp;
<h2>事前準備</h2>
まず、050の番号がもらえて、iPhoneに対応しているIP電話サービスの契約が必要です。
・<a href="http://050plus.com/">050plus</a>
・<a href="http://www.fusioncom.co.jp/kojin/smart/">FUSION IP Phone Smart</a>
今のところ、この二つが有名どころでしょうか。私は050plusを使用しました。後述の不在着信メール通知が使えるので、<span style="color: #800000;">取りこぼしが無いという点ではこちらがオススメ</span>です。

両者の違いは、<span style="color: #800000;">毎月の<strong>基本料金の有無</strong>と、<strong>通話料の違い</strong>・<strong>国際電話対応の有無</strong></span>です。FUSIONは基本料金無料で通話料も一律に揃えていて、分かりやすくしている印象で、対して050plusは固定電話やPHS宛てが安くなっています。音質や遅延にも差が出てくると思いますが、実運用で試してみないと判らない部分です。

FUSIONは国際電話非対応なので、<span style="color: #800000;">渡航先で現地の番号に発信したいのであれば050plus</span>を選ぶことになります。ただ、ローミングよりは安くてもそれなりに通話料がかかりますので、多用するのであれば<span style="color: #003366;">現地で携帯をレンタルする方が安上がりになるかも</span>。

&nbsp;

&nbsp;
<h2>出国前にしておくこと</h2>
飛行機に乗ってしまうと出来ないこともあるので、チェックイン待ちの行列や搭乗前の暇な時間にやってしまいましょう。

&nbsp;
<h3>転送電話の設定</h3>
出国前に転送電話の設定をしておきます。
各キャリアごとの設定方法は下リンクの通り。

<a href="http://www.au.kddi.com/tensou/index.html">au　着信転送サービス
</a><a href="http://faq.mb.softbank.jp/detail.aspx?id=498&a=102">Softbank　[転送電話]設定方法を教えてください</a> ((何故かQ＆Aが一番詳しい))
<a href="http://www.nttdocomo.co.jp/service/communication/transfer/">ドコモ　転送でんわサービス&#160;</a>

もちろん、転送先はIP電話の番号です。

出国前に転送したい携帯電話から直接設定するのが一番簡単ですが、特定の電話番号にかけて遠隔操作をすることも出来るので、渡航先からでも間に合います。 ((もちろん設定用の番号に海外からかけると国際電話になってしまうので、その部分は割高です。))

&nbsp;
<h3>通信の制限</h3>
海外でローミング・海外パケット定額系サービスを利用しないために、設定状態をしっかり確認しておきます。本来どちらも便利なサービスですが、今回の記事はそこの部分のコストを気にする内容ですので、断じて使うわけにはいきません。

まず、誤って機内モードを解除してしまったとき ((設定画面の一番上にあるので、意外と誤操作しやすい)) のために各種通信を切っておきます。これは<span style="color: #800000;">機内モード中は出来ない</span>ので注意。

&nbsp;

設定→一般→モバイルデータ通信
「モバイルデータ通信」「ローミング」ともにオフにします。

&nbsp;

そして海外滞在中は機内モード+Wi-Fiのみ使用にすること。設定は簡単です。機内モードにした状態で、Wi-Fiをオンにするだけ。

&nbsp;

&nbsp;
<h2>実際の使用感</h2>
先日の旅行で実際に使ってみた使用感を書いておきます。渡航先はマレーシア。

&nbsp;

&nbsp;
<h3>利用環境</h3>
現地キャリアの3Gネットワークが使えるWi-Fiルータと、ホテルのWi-Fiアクセスポイントで使用。

&nbsp;
<h3>音質</h3>
意外とかなり良い。
調子が良いときは、日本で050plusを使っているときと遜色ないです。携帯電話での通話と比べて、周囲の雑音が全然聞こえないのと、カーテンの向こうから話しているような音声になるので、違和感はあります。
人間すぐに慣れるもので、しばらく通話していると、自然とハッキリ発音するようになりました。

&nbsp;
<h3>遅延</h3>
日本〜マレーシア間なら、ほとんど判らないくらい。これは環境に大きく影響されるので一概には言えませんが、通話に影響が出るほどではありません。
問題になるのは速度や遅延よりも回線の安定性で、不安定な回線だと音声が途切れたり片通話になったりします。3GでもWi-Fiでも、電波状況が悪いと途切れがちでした。

&nbsp;
<h3>不在着信の扱い</h3>
050plusの場合、不在着信をメールで通知することが出来ます。<strong>着信の取りこぼしを無くす</strong>ためには大事な機能ですね。

FUSIONではどういう扱いになるのか分かりません。公式サイトで特に言及されていないので、通知はしないんでしょうか。

&nbsp;

&nbsp;
<h2>まとめ</h2>
「海外で日本の携帯宛ての着信を逃さない」という目的は十分達成出来ていました。

ネット環境によっては音質に問題が出てくるかもしれませんが、<span style="color: #800000;">大事な電話であれば固定電話から折り返す</span>ことも出来ます。

あとは、携帯のMMS宛てメールも転送出来ればもっと便利になりますね。

<!--
*1377302557*[]フライトモード


-->

<!--
*1377302552*[]ローミング


-->

<!--
*1377302546*[]モバイルデータ通信


-->
