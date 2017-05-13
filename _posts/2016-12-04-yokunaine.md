---
layout: post
title:  よくないねボタン
date:   2016-12-04 19:10:00 +0900
tags: いいね よくないね 技術で歯向かえ「いいね」ボタン ポエム
---

いいね だけじゃ　だめなんだよ。

[良い記事を書くためのガイドライン - Qiita:Support](http://help.qiita.com/ja/articles/qiita-article-guideline)

こういうガイドラインがあるけども、これに沿った良い記事なんて半分もないんだよ。
質問の場所と勘違いしている記事や、READMEを貼り付けただけの記事、ブックマークサービスとしてURL貼るだけの記事ばかり。
Googleで「あれどうやるんだっけ」と調べるとトップにQiitaが出てきて開いてみると、
「このブログにやりかたが書いてありました！」なんて記事ばっか。
そんなんだったらそのブログに検索トップの座をゆずれよお前の記事が邪魔してるんだよ。

そして先日、良い記事がごく少数なのにもかかわらず「👍いいね」ボタンなる間抜けな実装が追加されてゴミ記事にうんざりしている人たちは憤怒ですよ。

[「いいね」が使えないのはどう考えてもQiitaが悪い - Qiita](http://qiita.com/bezeklik/items/e28ab36e1071e761a277)

もうだめだよね。「👍いいね」なんて。どうせなら「🍣すし」にしてほしかったくらいだよ。

[Twitterの仕様変更を受け入れられなくなったので:heart:を:sushi:に変えて精神を落ち着かせる - Qiita](http://qiita.com/mzyy94/items/54d7c55eb01af37954fe)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Qiitaの「👍いいね」がイラっとするので内容の無いポストに対して「👎よくないね」を付けるウェブサービスとChrome拡張を作りたい</p>&mdash; はいふり@3日目東v-14a (@mzyy94) <a href="https://twitter.com/mzyy94/status/801694537219444736">2016年11月24日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

------

と、愚痴を言いだしたら冬コミの薄い本以上に書き上がりそうなのでほどほどにしておいて、
おなじみの**技術で歯向かえ「いいね」ボタン**の新作、「👎よくないね」ボタンが出来上がりました。

### [mzyy94/yokunaine: よくないね。](https://github.com/mzyy94/yokunaine/)

今回はChrome拡張機能として実装しました。Firefoxは知りません。
拡張機能をインストールしてトークンを取得すると、各記事によくないねボタンが現れます。
ためしにこの記事に「👎よくないね」をつけてみてください。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">震えて眠れプロポエマー <a href="https://t.co/19gGUsZFDa">pic.twitter.com/19gGUsZFDa</a></p>&mdash; はいふり@3日目東v-14a (@mzyy94) <a href="https://twitter.com/mzyy94/status/803071379654352896">2016年11月28日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

先日Lightsailなるお手軽なインスタンスが立ち上がったのでAWSで「よくないね」の数を集計します。
使い方は上記リポジトリを確認してください。

追記：

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">動かない人、ちゃんとREADME読んでオプションページからTOKEN取得してね！！！ <a href="https://t.co/lcqSpKcA9s">pic.twitter.com/lcqSpKcA9s</a></p>&mdash; 岬明乃@3日目東v-14a (@mzyy94) <a href="https://twitter.com/mzyy94/status/805362469543649280">2016年12月4日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

------

# あとがき

~~たぶんこの記事も[利用規約 - Qiita](https://qiita.com/terms)に違反しているとかなんとか言われて消されるんだろうなぁ。~~
~~ほかにもっと消えるべき記事はたくさんあるのに。~~
~~もし消えなかったら2017年に向けたモダンな技術をふんだんに使ったこの拡張機能の技術説明記事を書いてもいいかなーって。~~
~~この拡張機能ほんとすごいんだよ。~~
~~拡張機能の方のコードはReact+ES2015だしサーバの方のコードなんて200行足らずで描き上がってるんだよ。~~
~~めちゃくちゃ機能の紹介したいよ。JavaScriptの良さみんなにいっぱい知ってほしいよ。~~