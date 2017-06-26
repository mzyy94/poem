---
layout: post
title: "OS Xで再現されるBus error: 10の原因探ってみた"
date: 2013-12-18 12:48:12 +0900
tags: c bus_error segmentation_fault osx
---


Cです。

アプリケーションの例外でおこるSegmetion fault: 11 はよく目にするのですが、
OS Xで時々現れるBus error: 10 が現れる条件が気になったのでごちゃごちゃいじって探してみました。

こういう情報はどこかインターネットにあるのだと思ったのですが、
[Segmentation FaultとBus Errorの違いとMac - Togetterまとめ](http://togetter.com/li/253717)
などには納得のいく答えがなかったのが今回探してみた経緯となります。

結論からいうと、原因の特定はできませんでした。
様子を伺うまでの手順など、やったことの記録になります。

![top image]({{ site.baseurl }}/images/buserr-segfault-osx.png)


早速ですが、Segmentation faultとBus errorがどういう基準で区別されているか、明確ではありません。
たとえば、`char a[1]`としたときに`int i = 512341232; a[i] = '\0';`とするとSegmentation faultになったり、
`int i = 512341285; a[i] = '\0';`とするとBus errorになったりします。
さらに、この条件は起動時に割り当てられるメモリによって変わったりするので、原因を探るのはすこし大変です。

しかし、記憶クラスを変えると出される例外がある程度固定されるようです。

まず、以下のようなコードを書いて例外が発生した状態を調べました。
<script async src="https://gist.github.com/mzyy94/8016375.js"></script>

結果は以下の通りとなりました。
<script async src="https://gist.github.com/mzyy94/8017126.js"></script>

staticで宣言した変数に対してのアクセスでおこる例外はBus errorとなっています。

そこで、次のようなコードを利用し、デバッガで探ってみました。
<script async src="https://gist.github.com/mzyy94/8017103.js"></script>

OS X Mavericsなので、デバッガはgdbではなくlldbを用いました。
0,1,2,3の引数を与えてデバッグした結果は以下の通りになりました。
<script async src="https://gist.github.com/mzyy94/8017373.js"></script>

この結果から、引数をそれぞれ与えたときのエラーメッセージはどれもEXC_BAD_ACCESSで、
このメッセージの中のエラーコードが1もしくは2ということで分かれています。例外は、2がSIGBUSで1がSIGSEGVであるようです。

disassembleした結果が面白いことになっています。
処理が違うのがわかりますが、ここでアセンブラをみてみます。
<script async src="https://gist.github.com/mzyy94/8017712.js"></script>

アセンブラのコードでは、switch文中のcaseにあたるの部分が、上から順に

Cソース |アセンブラ
--------|---------
case '0'|LBB0_5:
case '1'|LBB0_6:
case '2'|LBB0_7:
default |LBB0_8:

と、対応しています。

このアセンブラを追いかけてみていると、値は次のように対応付けしているようです。

Cソース |アセンブラ
--------|---------
c[]     | -17(%rbp)
s[]     | _main.s(%rip)
i       | -24(%rbp)


LBB0_5と、LBB0_6をみてみると、
leaq(64bit Load Effective Address)で実効アドレスから変数s[]を%raxレジスタに格納しています。
その次にはmovslq(64bit Move Signed Long)で変数iを%rcxに格納しています。
この読み出した%raxレジスタの%rcx番目、すなわち、s[i]にアクセス際に例外が発生しています。

LBB0_7と、LBB0_8をでは、
leaqはせず、
movslqでレジスタ%raxに格納した変数iを-17(%rbp,%rax)としてアクセス、すなわちc[i]としてアクセスしたところで例外が発生しています。

さて、この二つ組の違いはleaqにありますが、leaq命令事態はforループ中のレジスタ操作効率化のためのものなので、特段問題を抱えているようすはありません。
また、アセンブラコードの下部にある.zerofillはstatic変数に対するものですが、この命令はゼロ埋めするというもので、static変数の初期値に0を代入するものなので、これも影響は与えていません。[1][1]

ほかの部分に関してもアクセス例外が明確に違うところは見あたらず、根本的原因はわかりませんでした。

おしまい

[1]: https://developer.apple.com/library/mac/documentation/DeveloperTools/Reference/Assembler/040-Assembler_Directives/asm_directives.html "OS X "
