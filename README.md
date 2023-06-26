# はじめに
MQ入門の入り口になるようなコンテンツを目指して。
簡単にデモ環境つくって、メッセージキューを体験してみます。

# MQとは
メッセージキューアプリケーションです。
いわゆるシステム間連携で使用するもので、オンブレとクラウドとの間のデータ連携や
システムとシステムとのつなぐ仲立ちをするものになります。
いまや世の中にある様々なシステムの裏側で活躍する縁の下の力持ちです。

# やりたいこと
そんなメッセージキューを体感するためローカルにコンテナ環境を作り。
以下システム構成でメッセージのやりとりをしてみました。今回はcURLコマンドを使用して
REST APIで「Hello MQ!!」というメッセージを送受信してみます。

※以下図における「アプリ」はcURLコマンドを指します。
![MQコンテナ構成](https://github.com/IBMDeveloperTokyo/techdojo-mq-sample/assets/99166088/60ce8ec0-e35a-4844-9051-683c6943bc69)

# 私の環境
* Windows11 Pro
* Docker for Business

# 事前準備
* Docker（またはpodman）

# 構築
## 永続ボリュームを作って、MQコンテナ起動
```
$ docker volume create qm1v
```
```
$ docker run -d --name mymq -v qm1v:/mnt/mqm -e LICENSE=accept -e MQ_QMGR_NAME=QM1 -p 9443:9443  icr.io/ibm-messaging/mq:9.3.2.0-r2
```

## コンテナIDを確認して、動作確認
```
# CONTAINER IDを確認
$ docker ps
CONTAINER ID   IMAGE                                COMMAND   CREATED      STATUS         PORTS                NAMES
eaed19cf19a4   icr.io/ibm-messaging/mq:9.3.2.0-r2   ""        2 days ago   Up 3 seconds   1414/tcp, 9443/tcp   mymq
```
※上記コマンドから「eaed19cf19a4」とわかる

```
# コンテナIDを指定して実行
$ docker exec -it eaed19cf19a4 dspmq
QMNAME(QM1)                                               STATUS(Running)
```

## REST APIのエンドポイントを確認
```
# PC側で実行
$ docker exec -it eaed19cf19a4 /bin/bash
# コンテナ内で実行
$ dspmqweb status
MQWB1124I: Server 'mqweb' is running.
URLS:
  https://eaed19cf19a4:9443/ibmmq/console/
  https://eaed19cf19a4:9443/ibmmq/rest/
```
※上記で「https://eaed19cf19a4:9443」となっているが
コンテナで実行しているため「https://localhost:9443」となる。

# ブラウザでMQ Web Consoleの動作確認
https://localhost:9443/ibmmq/console/
（admin/passw0rdでログイン）
※警告が出るが無視してログイン

# キュー（DEV.QUEUE1）の中身を確認
## キューマネージャQM1をクリック
![image](https://github.com/IBMDeveloperTokyo/techdojo-mq-sample/assets/99166088/180d875f-9aca-4e99-827b-e4be219b1d47)

## DEV.QUEUE.1を開く
赤枠のDEV.QUEUE.1をクリック
![image](https://github.com/IBMDeveloperTokyo/techdojo-mq-sample/assets/99166088/1988528e-4820-42af-8bee-f8bcac7387a1)
※このあとのメッセージ送受信で使用するキューです。

# RESTAPI実行のために事前認証（トークン認証）
```
# app/passw0rdでログイン。ローカルPC側のCookieに認証情報を保存する
$ curl -k https://localhost:9443/ibmmq/rest/v2/login -X POST -H "Content-Type: application/json; charset=UTF-8" -d "{\"username\":\"app\",\"password\":\"passw0rd\"}" -c c:\users\hogehoge\cookiejar.txt
```

# メッセージ送信
```
# 送信
$ curl -k https://localhost:9443/ibmmq/rest/v2/messaging/qmgr/QM1/queue/DEV.QUEUE.1/message -X POST -b c:\users\hogehoge\cookiejar.txt -H "ibm-mq-rest-csrf-token: value" -H "Content-Type: text/plain;charset=utf-8" --data "Hello MQ!!"
```

# 送信メッセージ確認
WebConsoleで送信したメッセージを格納しているか確認
![image](https://github.com/IBMDeveloperTokyo/techdojo-mq-sample/assets/99166088/87505bb2-fd29-47f8-a159-7733e9c687c2)

# メッセージ受信
```
# 受信（メッセージ受信しつつ、キューからメッセージを消す）
$ curl -k https://localhost:9443/ibmmq/rest/v2/messaging/qmgr/QM1/queue/DEV.QUEUE.1/message -X DELETE -b c:\users\hogehoge\cookiejar.txt -H "ibm-mq-rest-csrf-token: value" -H "Content-Type: text/plain;charset=utf-8"
Hello MQ!!
```
コンソールに「Hello MQ!!」となればOK。メッセージ受信できています。

# メッセージ削除確認
WebConsoleで、受信したメッセージが消えているか確認
![image](https://github.com/IBMDeveloperTokyo/techdojo-mq-sample/assets/99166088/4d827b2d-a6cc-4957-9da6-f226bd88e198)

# 最後に
ということで、ここまでMQコンテナを使って、メッセージキューを体験してみましたがいかがでしたか？
今回はコマンドでメッセージの送受信をしたので、ものすごくシンプルであっさりした見た目になりましたが・・・
でもメッセージキューはシステムにおける重要なファクターであることを忘れないであげてください。。。
次回はローカルコンテナではなく、クラウド上で同じことをやってみたいと思います。お楽しみに。

# 参考資料
## MQコンテナ使用方法
https://github.com/ibm-messaging/mq-container/blob/master/docs/usage.md
## 認証
https://www.ibm.com/docs/ja/ibm-mq/9.2?topic=security-using-token-based-authentication-rest-api
## MQ REST APIドキュメント
https://www.ibm.com/docs/ja/ibm-mq/9.2?topic=resources-messagingqmgrqmgrnamequeuequeuenamemessage
