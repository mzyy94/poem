---
layout: post
title: "Honda 3DのサイトがFlashだったときの症例"
date: 2014-02-13 19:03:54 +0900
tags: html5 css3 webgl three.js
---


RSSを消化してたときのこと。
[ホンダが歴代コンセプトカーの3DデータをCCライセンスで公開、NSX Concept など全5種 (動画) - Engadget Japanese](http://japanese.engadget.com/2014/01/28/3d-nsx-concept-5/)
なる記事を見つけて、「クリエイティブコモンズで3Dモデル公開とはなかなかやるな。流行に乗っててよろしい。」と感心していました。

しかし、そんな関心も次のクリックで吹っ飛びました。

<pre>
＿人人人人人人人＿
＞　衝撃のFlash　＜
￣Y^Y^Y^Y^Y^Y￣
</pre>

流行に乗ってると微塵でも思ってしまったことを後悔。
3Dプリンタという流行を意識した3Dモデルの公開に使われたものが流行遅れのFlashであるという、なんとも言い難い悲しい状況を目の当たりにしてしまったのです。

この惨状を目にした60分後、手元にはこんなものが出来上がっていました。

![top image]({{ site.baseurl }}/images/honda-3d-diff.png)

### [Honda 3D - WebGL Demo (Beta)](http://mzyy94.com/HONDA-3D-WebGL-demo/)
{% raw %}
<span>※Chrome 33向けです。</span><br>
<span>※※RAMとCPU使用率と通信量を膨大に使うので開く際はご注意ください</span>
{% endraw %}

[GitHub Repository](https://github.com/mzyy94/HONDA-3D-WebGL-demo)

悲しみからか、手が勝手に動き__Flashなし__で模倣していました。



このれらのサイトを構成している技術に関して比較してみました。

![comparison]({{ site.baseurl }}/images/comparison.png)

せっかくつくったので解説させてください。

# トップページ

上：拙作デモサイト　下：本家サイト

![demo]({{ site.baseurl }}/images/demo-top.png)
![original]({{ site.baseurl }}/images/original-top.png)

メインページの全体像はほとんど違和感を感じさせない作りとなっています。
左右のパネルを選択すると本家と変わらないスライドアニメーションで次のパネルへ移り変わります。


このカルーセルはjQueryによるclass操作とCSS3のkey frame アニメーションを使っています。クリック時に左端のパネルのmarginを以下のように動かすことで滑らかなスライドアニメーションを疑似的に見せています。


CSS3の技術が詰まったパネルをもう少し詳しく見てみましょう。
テキストの上でもカーソル形状は本家のものと同じくデフォルトになるようにしてあります。
背景透過はbackground-colorをCSS3からのrgba値を用いており、サムネイルを円形にしてあるのもCSS3のborder-radiusを指定して実現しています。さらに、パネル上の美しいfontもCSS3によるWeb fontを利用しています。

背景動画はYouTubeのHTML5 videoでの埋め込みにしてあるというAnti-Flashの徹底ぶり。オプションを以下の通り指定してNo Flashにしました。

```
<iframe id="bgmovie" width="1920" height="1080" src="http://www.youtube.com/embed/MJWzLm91Hmg?html5=1&playlist=kdOVr4Tqdoc&autoplay=1&disablekb=1&controls=0&showinfo=0&vq=hd1080&modestbranding=1&rel=0&loop=1" style="top: -374px;"></iframe>
```


# 3D ビュー

ここのシーンを見せるためにHondaはFlashを試用したというのは安易に想定できますが、せっかくなのでHTML5の技術を使って欲しかったです。
ということで、HTML5の新技術であるWebGLを利用してモデルを表示するようにしました。

WebGLの複雑なコードを書かずとも、数十行のコードで手軽にWebGLをつかえるライブラリはいくつか存在します。
今回は[three.js](http://threejs.org/)とよばれるWebGLラッパーを利用しました。

WebGLで3Dモデルを表示するには、表示するための3Dデータが必要です。
現在、Honda-3d.comからダウンロードできるファイルは3Dプリンタ向けのSTL形式で、WebGL、加えてthree.jsでもそのままでは利用できるものではありません。
そこで、three.jsで扱える形式に変換する必要があります。
今回は3Dモデリング界では名高い[Blender](http://www.blender.org/)を利用しました。

## 3Dモデルデータを変換する

BlenderでSTL形式をthree.jsで扱える形式に変換する方法はそこまで難しいことではありません。
Blenderは標準でSTL形式を扱えるので、three.jsで使えるよう、
three.jsに含まれるBlender用のAdd-onを導入すれば、three.jsで表示できる形式に変換する環境は整います。
このAdd-onはthree.jsをgitリポジトリからCloneした際に、一緒にダウンロードされています。

Add-onを導入するのは簡単で、必要なファイル群をBlenderのアプリケーションフォルダに入れてあげるだけです。
OS X上で、Blender@2.69とthree.js@r65を利用する場合は、

	[three.jsのClone先ディレクトリ]/utils/exporters/blender/2.65/scripts/addons

にある`io_mesh_threejs`ディレクトリを

	[アプリケーションフォルダ]/Blender.app/Contents/MacOS/2.69/scripts/addons

の中にコピーします。

![blender]({{ site.baseurl }}/images/blender-1.png)

コピーし終わったらBlenderを起動し、File->User Preferences...で設定画面を開き、
Addonsタブにある検索ボックスでthreejsとして検索して出てくる項目をチェックし有効化します。
チェックし終わったら次回以降も反映されるよう、Save User Settingsをしておきます。
![blender]({{ site.baseurl }}/images/blender-2.png)
![blender]({{ site.baseurl }}/images/blender-3.png)

設定画面を閉じたらFile->Import->Stlでダウンロードしたファイルを開き、
起動時に作られていた不要なオブジェクトを削除し、File->Export->Three.jsでthree.js用の形式にして出力します。
![blender]({{ site.baseurl }}/images/blender-4.png)
![blender]({{ site.baseurl }}/images/blender-5.png)
![blender]({{ site.baseurl }}/images/blender-6.png)

## 3Dモデルデータを表示する
上：拙作デモサイト　下：本家サイト

![demo]({{ site.baseurl }}/images/demo-3dview.png)
![original]({{ site.baseurl }}/images/original-3dview.png)

表示はthree.jsの基本的な機能を使うだけで行っています。
基本的な方法は、
[Webグラフィックをハックする（5）：多彩な表現力のWebGLを扱いやすくする「Three.js」 (1/5) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1210/04/news142.html)
にて説明されています。
この説明を参考に、three.jsのファイルを読み込ませて表示させています。
ちゃんとマウスでぐりぐりできるようになっています。


各コードに関して説明したかったのですが、すでに結構な分量になっているので、気になる人は以下のGistにてご参照ください。

[https://gist.github.com/mzyy94/8974444](https://gist.github.com/mzyy94/8974444)

以上、Anti-Flashをこじらせた人の症例でした。
