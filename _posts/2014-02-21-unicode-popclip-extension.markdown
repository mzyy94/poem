---
layout: post
title: "Unicode形式に変換するPopClipのExtensionを作りたかっただけだったのに"
date: 2014-02-21 17:46:27 +0900
tags: bash popclip bash unicode utf-8
---


どうやら[PopClip](https://itunes.apple.com/jp/app/popclip/id445189367?mt=12&uo=4&at=1l3v4mQ)が便利らしいことを[発見](http://hitoriblog.com/?p=22987)したので、だいぶ昔に買ったっきりで使っていなかったPopClipを活用してみることにしました。

![top image]({{ site.baseurl }}/images/UnicodeEncode_popclipext.png)

# PopClipとは
PopClipに関しては、[ひとりぶろぐさんの記事](http://hitoriblog.com/?p=22987)にて紹介されているので省略します。知っているという前提で以下、話を進めます。

# Unicodeエンコード
最近なにかとマイブームなUnicode。CSSのcontentプロパティで記号を表示したり、JavaScriptで難読化したりなど、よく使う割には毎回ブラウザ上でコンソールを拡げていました。この手間を解決する手段として、せっかくなのでPopClipの拡張機能を利用しようと思います。

<!-- more -->

# PopClip拡張機能
PopClipが人気なのは、iOSライクなコピー＆ペーストができるからではなく、PopClip拡張機能で選択テキストなどに対する便利な機能が実現できるかららしい[_要出典_]。
PopClip拡張機能を作るための手順に関しては、さっきからよく登場する[ひとりぶろぐさんの記事](http://hitoriblog.com/?p=22987)にて説明があるので、読んで作っていきます。

## UnicodeEncode.popclipext

![UnicodeEncode_popclipext]({{ site.baseurl }}/images/UnicodeEncode_popclipext.gif)

そこまで難しく考えることなく、サンプルのURLエンコードの拡張機能をちょちょっと編集して、\uXXXXの形のUnicode形式を出力するプログラムをかきました。UTF-8→Unicodeの変換は、[第4回 UTF-8の冗長なエンコード：本当は怖い文字コードの話｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/charcode/0004)を参考にしました（はせがわさんの警告無視）。


こちらになります。[mzyy94/UnicodeEncode.popclipext](https://github.com/mzyy94/UnicodeEncode.popclipext)

__作るの意外と大変だった__

さらっとサンプルのソースコードを読んだ限り、簡単にできるものだとおもったのですが、いろいろと問題がありました。

### LANG環境変数問題
一つ目の問題は、LANG環境変数。
おなじみシステムのローケルを設定する環境変数ですが、日本語を扱うときのこのロケールがいろいろと厄介なものでした。
OS X Mavericksで日本語環境でターミナルを起動する際は、標準でLANG=ja_JP.UTF-8に設定されています。この状態で、日本語を扱うときには特に問題ないのですが、LANG=Cのときだと厄介なことになることがあるのです。
その例が以下の画像。

![Problem1]({{ site.baseurl }}/images/problem1.gif)

"あ"をUnicode表記にすると\u0342になるはずですが、この例では出力がUTF-8バイトコードの"あ"であるe3 81 82がそれぞれ個々の文字としてUnicodeにエンコードされています。
PopClip拡張機能は、起動時にLANG環境変数を引き継ぐことなく、未指定の状態でスクリプトが実行されているようで、デフォルトであるLANG=Cでの処理となってしまっているためです。bashで実装したのが大きな原因であるようですが、どうも文字列を文字ごとに分割するときにLANG=Cではマルチバイト文字をASCII文字として扱ってしまっているのが原因らしいです。

解決策として、`export LANG=en_US.UTF-8`と、UTF-8ロケールに設定しました。

### 濁点文字分離問題
まず、以下の例をみてください。

![Problem2]({{ site.baseurl }}/images/problem2.gif)


三人ともおなじ５文字の名前なのに、鹿目さんだけがUnicode化した際に６文字になってしまっています。
これは、鹿目さんが悪いのではなく、OS Xの挙動が悪いのです。

この問題は、OS X特有のもので、OS X上でのUTF-8の文字コードの取り扱いに起因しています。
OS XではUTF-8を扱う文字コードが２種存在します。`iconv -l | grep UTF-8`とすれば確認できますが、一つは純粋なUTF-8で、もう一つはUTF-8-MACなるものです。このUTF-8-MACは、OS XのファイルシステムによってUTF-8を扱うためのものであるらしく[1](http://macwiki.sourceforge.jp/wiki/index.php/UTF-8-MAC)、濁点のある仮名文字などで正規分解が行われ、今回の例では「鹿」「目」「ま」「と」「 ゙」「か」といった具合にUTF-8の文字列がPopClip拡張機能にわたされてしまったようです。この問題は、PopClip拡張機能にせずにシェルスクリプトをそのままのかたちで実行し、テストしたときには再現されなかったので、発見が遅れ、解決に時間がかかってしまいました。

この問題は`iconv -f UTF-8-MAC -t UTF-8`とすることで解決しました。

# PopClip拡張機能を作りたかっただけだったのに
PopClipがロケール環境変数を引き継いでいなかったおかげでLANG=CでのUTF-8の文字の扱いがわかり、PopClipがUTF-8-MACとしてテキストを渡してくれていたおかげでOS XでのUTF-8の取り扱いに関して知識を得ることができて、結果的にいい経験になりました。ありがとう、PopClip。
