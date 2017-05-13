---
layout: post
title:  "koaでJSON返させるシンプルで唯一の記述"
date:   2014-04-21 19:30:14 +0900
tags: Koa Node.js JavaScript
---

koaでRESTなものを作ろうと思ったときにJSONでオブジェクトを返す必要がでてきました。
いろいろなことが便利になったkoa、JSONを返すコードも素敵に書くことができます。
## コード (sample.js)

```js
var koa = require('koa');
var route = require('koa-route');
var app = koa();
// respond JSON
app.use(route.get('/hoge.json', function *() {
	// set Object you want to respond
	this.body = { foo: 'bar' };
}));
app.listen(3000);
```
## 実行
`node --harmony sample.js`
## 結果
`curl -v localhost:3000/hoge.json`
もしくは
`open http://localhost:3000/hoge.json` (OS X限定)
![スクリーンショット 2014-04-21 19.13.36.png]({{ site.baseurl }}/images/4cee5b47-a75f-9878-2758-35cd43c30c56.png)
## まとめ
何もせずに、無心にObjectを突っ込めばいいのです。
もうJSON.stringify()とか不要なのです。`JSON.stringify()`なしで動くのです。
Content-Typeもいじらなくていいんです。全部やってくれるんです。
koa、使わない手はないでしょう。
※動かない場合は、`app.use(json());`を指定すると動く。（バージョン次第？）