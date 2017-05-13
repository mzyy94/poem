---
layout: post
title:  B−CASリーダをDocker内で読もうとしてlibpcscliteのバージョン違いではまった話
date:   2016-07-03 16:35:00 +0900
tags: pcscd docker libpcsclite
---

ChinachuをDockerに入れようとしたらカードリーダーがB−CASをうまく認識しなくてつらかった話。

結論から言うと、ホストの`pcscd`とゲストの`pcsc_scan`のpcscliteのライブラリのバージョン違いが問題でしたって話。
要するにこんなかんじ（表1)。

表1. 要するにこんなかんじ

 OS | libpcscliteのバージョン
:---:|:-----
 CentOS 7.2 | pcsc-lite-libs 1.8.8
 Debian jessie 8.5 | libpcsclite1 1.8.13

`pcscd`の使うライブラリのマイナーバージョンより`pcsc_scan`のライブラリのマイナーバージョンが新しいと動かないということだった。
解決する方法は２パターンあって、

- ホスト側（`pcscd`側）のライブラリのバージョンを1.8.8で保持してゲスト側(`pcsc_scan`側）を1.8.8以前にダウングレード
- ホスト側のライブラリを1.8.8以上にアップデード

今回はホスト側を汚さないように前者の解決策をとって、wheezy向けのlibpcscliteをインストールして解決した。
おしまい。



----
以下、駄文なので見なくていいです。

## そもそもなんで動かなかったの？

CentOS側で動かしてる`pcscd`にDebian jessieから`pcsc_scan`してみたときの`pcscd`のログがこちら。

```
08174657 winscard_msg_srv.c:230:ProcessEventsServer() Common channel packet arrival
00000030 winscard_msg_srv.c:242:ProcessEventsServer() ProcessCommonChannelRequest detects: 9
00000007 pcscdaemon.c:93:SVCServiceRunLoop() A new context thread creation is requested: 9
00000098 winscard_svc.c:299:ContextThread() Thread is started: dwClientID=9, threadContext @0x555636dee7f0
00000025 winscard_svc.c:317:ContextThread() Received command: CMD_VERSION from client 9
00000009 winscard_svc.c:329:ContextThread() Client is protocol version 4:3
00000005 winscard_svc.c:338:ContextThread() Client protocol is 4:3
00000004 winscard_svc.c:340:ContextThread() Server protocol is 4:2
00000007 winscard_svc.c:349:ContextThread() CMD_VERSION rv=0x8010001D for client 9
00000238 winscard_svc.c:309:ContextThread() Client die: 9
00000024 winscard_svc.c:928:MSGCleanupClient() Thread is stopping: dwClientID=9, threadContext @0x555636dee7f0
00000007 winscard_svc.c:934:MSGCleanupClient() Freeing SCONTEXT @0x555636dee7f0
```

`クライアント、死す: 9`って書いてあるように、クライアントが死んだらしい。

対して同じCentOSホスト上で`pcsc_scan`を実行した時の`pcscd`のログがこちら

```
99551719 winscard_msg_srv.c:230:ProcessEventsServer() Common channel packet arrival
00000041 winscard_msg_srv.c:242:ProcessEventsServer() ProcessCommonChannelRequest detects: 9
00000008 pcscdaemon.c:93:SVCServiceRunLoop() A new context thread creation is requested: 9
00000103 winscard_svc.c:299:ContextThread() Thread is started: dwClientID=9, threadContext @0x555636db2f80
00000027 winscard_svc.c:317:ContextThread() Received command: CMD_VERSION from client 9
00000011 winscard_svc.c:329:ContextThread() Client is protocol version 4:2
00000005 winscard_svc.c:349:ContextThread() CMD_VERSION rv=0x0 for client 9
00000123 winscard_svc.c:317:ContextThread() Received command: ESTABLISH_CONTEXT from client 9
00000026 winscard.c:193:SCardEstablishContext() Establishing Context: 0x42ADED6E
00000007 winscard_svc.c:410:ContextThread() ESTABLISH_CONTEXT rv=0x0 for client 9
00000130 winscard_svc.c:317:ContextThread() Received command: CMD_GET_READERS_STATE from client 9
00000108 winscard_svc.c:317:ContextThread() Received command: CMD_GET_READERS_STATE from client 9
00000093 winscard_svc.c:317:ContextThread() Received command: CONNECT from client 9
00000021 winscard.c:235:SCardConnect() Attempting Connect to Gemalto PC Twin Reader 00 00 using protocol: 2
00000007 readerfactory.c:739:RFReaderInfo() RefReader() count was: 1
(略)
```

クライアント、死なずにちゃんと最後まで動いた。

ログのdiffをとったら怪しい行があった。

```diff
+ Client is protocol version 4:3
- Client is protocol version 4:2
```

なにが違うのかstraceで確認してdiffしたところ、pcscdに対してpcsc_scanが送ってくるバイナリが違う部分があった。

```diff
+ read(9, "\4\0\0\0\3\0\0\0\0\0\0\0", 12) = 12
+ write(1, "00000116 winscard_svc.c:329:ContextThread() Client is protocol version 4:3\n", 75) = 75
- read(9, "\4\0\0\0\2\0\0\0\0\0\0\0", 12) = 12
- write(1, "00000116 winscard_svc.c:329:ContextThread() Client is protocol version 4:2\n", 75) = 75
```

なるほどクライアントがサーバに要求するプロトコルバージョンが違うのか。

で、libpcscliteの変更を疑って変更履歴を見てきたらビンゴ！プロトコルバージョンがちょうど1.8.8以前と1.8.9以後で変わっていることがわかった。

> pcsc-lite-1.8.9: Ludovic Rousseau
> 16 October 2013
> - SCardEndTransaction(): Return an error if is called with no
> corresponding SCardBeginTransaction()
> （中略）
> - Check the Info.plist file is (a minimum) correct
> - Update PROTOCOL_VERSION_MINOR from 2 to 3
> （後略）

https://alioth.debian.org/plugins/scmgit/cgi-bin/gitweb.cgi?p=pcsclite/PCSC.git;a=blob;f=ChangeLog;hb=HEAD

ホスト側はCentOS 7.2でlibpcscliteのバージョンが1.8.8、ゲスト側が1.8.13だったというマイナーバージョン程度の違いだけれども、後方互換を残さないような大きな変更があったとは疑いもせず１日をとかしてしまったというポエムでした。まぁ解決できたのでめでたしめでたし。