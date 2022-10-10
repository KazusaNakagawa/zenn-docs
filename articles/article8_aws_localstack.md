---
title: "【AWS】localstack で AWS SQS を local 操作する"
emoji: "🤧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws,localstack,docker,]
published: true
---


## はじめに

localstack で AWS SQS を local 操作できる方法を書いていこうかと
構築は、`docker-compose`でやってます。

`docker-compose.yml`, `Dockerfile` は以下を参考にさせて頂きました🙏
[docker-composeとlocalstackを使ってローカルでAWS SDK for JavaScript V3を動かしてみる](https://dev.classmethod.jp/articles/docker-compose-localstack-aws-sdk-for-javascript-v3-local/)

後、**SQS** にリクエストを飛ばすのに開発中やと何回も試すので、Postman で実行できる方法もメモしました!

curlコマンドや **aws CLI** コマンドを連打しなくていいので楽です!!

## 作成したリポジトリを clone して docker build コマンドを実行

現状作成しました [リポジトリ: aws-localstack](https://github.com/KazusaNakagawa/aws-localstack) を cloen します。
:::message
1発目の実行の際は、build 失敗します

localstack 情報が格納される `share/` ディレクトリか参照されないため.もう一回 `docker-compose up -d` を実行するとコンテナ立ち上がりますんで、ご了承ください😓
:::


1. 作成された localstack のコンテナにアクセス
    ```bash
    docker exec -it localstack_main bash
    ```

2. queue を作成
    ```bash
    $ awslocal sqs  create-queue --queue-name test-queue
    {
        "QueueUrl": "http://localhost:4566/000000000000/test-queue"
    }
    ```

3. queue message を送信
    ```bash
    $ awslocal sqs send-message --queue-url http://localhost:4566/00000000000/test-queue --message-body test
    {
        "MD5OfMessageBody": "098f6bcd4621d373cade4e832627b4f6",
        "MessageId": "9000f162-54b9-4b66-af7f-c69c51db959b"
    }
    ```

4. queue message の受信を確認
    ```bash
    $ awslocal sqs receive-message --queue-url http://localhost:4566/00000000000/test-queue
    {
        "Messages": [
            {
                "MessageId": "9000f162-54b9-4b66-af7f-c69c51db959b",
                "ReceiptHandle": "NjZjMjI4NDUtZjg5NC00ZmMxLTlhNTEtZGIzODQzYzE0ZTIyIGFybjphd3M6c3FzOmFwLW5vcnRoZWFzdC0xOjAwMDAwMDAwMDAwMDp0ZXN0LXF1ZXVlIDkwMDBmMTYyLTU0YjktNGI2Ni1hZjdmLWM2OWM1MWRiOTU5YiAxNjY1Mjg0Mjc1LjYyODk2NzU=",
                "MD5OfBody": "098f6bcd4621d373cade4e832627b4f6",
                "Body": "test"
            }
        ]
    }
    ```

## Postman でやるときは

### 1. [Postman AWS SQS](https://documenter.getpostman.com/view/2631434/SWLh56pX) から Postman DeskTop版をインポートする。
  1. `▶︎ Run in Postman`をクリック
  2. `Postman desktop App to Import` を選択 (アプリ入れてる前提)
  ![postman_import](/images/aws/1_import_aws_postman_sqs.png)

### 2. Import された **AWS SQS** プロジェクトの`queue-url`を反映し、Bodyの中身を確認する。
   POST: `http://localhost:4566/test-queue` の部分です! Body の使いかも参考になりました。
  
   ![body](/images/aws/2_postman_body.png)

### 3. `Pre-request Script` にサンプルが含まれていることを確認し、`Send`をクリックします。
   ここも import したときに反映されてる内容で、そんな使い方があるんや!! って参考になりました。
  ![request_script](/images/aws/3_postman_request_script.png)

### 4. `queue-url` を反映させて受信処理を行う
  **Send Message** の内容が受信されたことが確認できます。
  ![recive_message](/images/aws/4_postman_recive_message.png)


## これから
localstack サービスをつかうと、AWS サービスをローカルで試すことでき、課金もある程度気にしなくて開発ができるので便利そうです!!
あとは、やりたいことに合わせて調べながら試せると良さそうです!

以上になります。ありがとうございました🙏


## 参考

https://documenter.getpostman.com/view/2631434/SWLh56pX

https://dev.classmethod.jp/articles/docker-compose-localstack-aws-sdk-for-javascript-v3-local/
