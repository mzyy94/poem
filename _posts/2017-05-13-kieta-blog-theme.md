---
layout: post
title:  "プログラミングに限定しない知識でも記録・共有したくなるようなブログテーマを作った"
date:   2017-05-13 22:00:00 +0900
tags: Bootstrap4 Kieta 技術で歯向かえTOKYO2020
---

Markdownで気軽にポエムが書けるサイトがあったような前世の記憶があるのですが、みつからなかったのでGitHub Pagesで色々な知識をひけらかすためのJekyllテーマを作りました。ライセンスはMITです。

**[mzyy94/jekyll-kieta-theme: Clean knowledge sharing jekyll theme](https://github.com/mzyy94/jekyll-kieta-theme)**

その名もKieta。最初のKはKnowledgeのKということだけ決めていますが、名前の由来は特にありません。

リポジトリからファイル一式をダウンロードしてきて、Jekyllの作法に従って設定してMarkdownで記事を書くとこのように仕上がります。

# 特徴

## 自由な設計

オープンソースなので好きなようにカスタマイズができます。たとえばサイドバーにAmazonの広告を表示して、みんなに充電池をオススメすることができます。

## モダンなCSSフレームワーク

![preview 1]({{ site.baseurl }}/images/preview1.png)

みんな忘れかけていたであろうTwitter Bootstrap 4を採用しています。
ReactだVueだの騒がれているこのご時世、コンポーネントとしてではなくCSS単体として提供されているBootstrapを使っている人は少ないかと思います。
俗にいうBootstrap臭がツンと来る人も多く、あまり好まれて使われてこなかったBootstrapですが、基本テーマをフラットデザインに近づける努力を続けています。
そして間も無くベータ版が登場するTwitter Bootstrap 4では、Flexboxなどの機能を引っさげて大幅な更新を仕掛けて来る予定です。

そんなTwitter Bootstrap 4を一足先に導入しました。今回は執筆時点の最新版alpha-6を利用しています。

### 基本テーマの調整

Twitter Bootstrapの最も自己主張の強いコンポーネントにJumbotronがあります。このKietaテーマでも強い存在感を放つ記事のタイトル部分にそれを用いています。
Jumbotronの協調される色は記事の印象も左右するため、Jekyllの設定ファイルにて変更ができるようにしてあります。

```yaml
color: deepskyblue
```

![preview 2]({{ site.baseurl }}/images/preview2.png)

## タグ

![preview 3]({{ site.baseurl }}/images/preview3.png)

記事にはタグがついていますが、それをタグであるとわかりやすく表示するために工夫しました。タグ自体は1つのａタグですが、これを「タグ」っぽく表示させるために擬似要素を用いてデザインしています。ついでにJekyllのsite変数から全投稿の中のタグ利用数を表示してあります。

```scss
.tag {
    background: lightgrey;
    color: black;
    outline: none;
    position: relative;
    font-size: 0.9rem;
    height: 1rem;
    line-height: 1rem;
    padding: 0 8px;
    display: inline-block;
    margin: auto 20px;
    &::before {
        content: '';
        height: 0;
        position: absolute;
        width: 0;
        border: 0.5rem solid transparent;
        border-right-color: lightgrey;
        left: -1rem;
        top: 0;
    }
    &::after {
        content: attr(data-count);
        display: inline-block;
        padding: 0 0.5em;
        text-align: center;
        color: white;
        background: darkslategrey;
        position: absolute;
        margin-left: 8px;
    }
}
```

## 追従する目次

技術系記事に限らず、文書の全体のどの位置を読んでいるかが把握できないとモヤモヤするので、常に目次が見れるようにしました。
この記事をデスクトップPCやタブレットで読んでいる方は、右側に目次が追従してきているのがわかると思います。これは[tscanlin/tocbot](https://github.com/tscanlin/tocbot)を利用して実現しています。

## 新鮮なポエムをあなたに

古いポエムには警告が表示されるようにしました。Jekyllは素のLiquidとは違って、数値の比較が動かなかったのでがんばりました。

![preview 4]({{ site.baseurl }}/images/preview4.png)

{% raw %}
```liquid
{% capture one_year %}{{ 60 | times: 60 | times: 24 | times: 365 }}{% endcapture %}
{% capture current_time %}{{ site.time | date: "%s" }}{% endcapture %}
{% capture year_old %}{{ page.date | date: "%s" | minus: current_time | divided_by: one_year | plus: 1 }}{% endcapture %}
{% if year_old contains '-' %}
<div class="alert alert-warning" role="alert">
    <i class="fa fa-clock-o" aria-hidden="true"></i> This article is more than 1 year old. Please read this page keeping its age in your mind.
</div>
{% endif %}
```
{% endraw %}

## 数々のシェアボタン

ポエムはいろいろな人に共有していきたい気持ちが高いと思うので、思いつく限りのシェアボタンを設置しました。
参考にした公式ドキュメントは以下の通りです。

- [Twitterボタン \| About](https://about.twitter.com/ja/resources/buttons)
- [いいね！ボタン - ソーシャルプラグイン](https://developers.facebook.com/docs/plugins/like-button?locale=ja_JP)
- [はてなブックマークボタンの作成・設置について - はてなブックマーク](http://b.hatena.ne.jp/guide/bbutton)
- [Pocket for Publishers: Pocket Button](https://getpocket.com/publisher/button)
- [+1 Button  \|  Google+ Platform for Web  \|  Google Developers](https://developers.google.com/+/web/+1button/)

## コメント機能とコメント数の表示

コメント機能はGitHub Pages利用者にはおなじみの[Disqus](https://disqus.com/)を利用しています。また、Jumbotronにタイトルとタグだけを表示して置くだけではあっさりしすぎていたので、いっしょにコメント数とツイートボタンを表示しています。
このコメント数は、DisqusのAPIで取得できるものを整形して出しています。Disqus側で出力を変えられるのですが、ちょっと利用環境の都合でこんな細々とした作業を入れてあります。

```html
<script>
var MutationObserver = window.MutationObserver || window.WebKitMutationObserver;
if (MutationObserver) {
  var obs = new MutationObserver(function(mutations, observer){
    var mutation = mutations[0];
    var count = mutation.addedNodes[0].textContent.replace(/[^0-9]/g, '');
    document.getElementById('comment_count').textContent = count;
    mutation.target.parentNode.removeChild(mutation.target);
    obs.disconnect();
  });
  obs.observe(document.getElementById('disqus_comment_count'), { childList:true, subtree:true, characterData: true });
}
</script>
```

## 404ページ

うっかり遭遇すると悲しいページである404 Not Found。悲しさを紛らわすための工夫として、新鮮なポエムが流れるようにしておきました。

![preview 5]({{ site.baseurl }}/images/preview5.png)

# まとめ

検閲のない自由なインターネットを謳歌しよう。