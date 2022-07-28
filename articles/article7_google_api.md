---
title: "【GCP】 Python + Gmail API + Sheets API で 「Gmail 送信」を試してみた"
emoji: "✉️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GCP, GmailAPI, SeetsAPI, Python]
published: true
---

## はじめに
今回は、Google Cloud を使って、Gmail 送信を試してみました。
選定理由は、Webサービスにするほど規模もでかくないからです。また、スクリプトを exe化して Windows 環境で実行する背景があります。

メンテナンスするにも、お客様はプログラムの手直しができないので、**アカウント管理**など、スプレッドシートに持たせることで、ある程度自由に変更することも可能になることを想定してます。運用コストも**DB・サーバ**を持たないので、**APIリクエスト数**を気にしておけばそれほどかからないかと。


## 前提条件

以下の設定はできてる状態から書いていこうかと、結構最初は分けわからず調べなからすすめました。

- [Google Cloud](https://console.cloud.google.com/) で プロジェクト作成し、OAuth 2.0 クライアントの JSONファイルを用意
- Google Sheet 操作するために、[サービス アカウント キー](https://00m.in/miTfQ) を作成
- [Gmail API](https://console.cloud.google.com/marketplace/product/google/gmail.googleapis.com) の有効化
- [Google Sheets API](https://console.cloud.google.com/marketplace/product/google/sheets.googleapis.com) の有効化


## </>リポジトリ

https://github.com/KazusaNakagawa/try-docs/tree/develop/src/google_api/

## イメージ

**ザックリ!!** ... 実装したイメージになります. こういうのサクッと作れるようにお試し中です!!
最近、実務でも求められて作成するんですが、4,5 時間かかってるイメージ 。。
慣れれば、使い回しとかできて工数も削減かと。[drawio](https://drawio-app.com/) で書いてみてます。

っと、実際の流れは、以下のように、

```bash: ながれ
1. スクリプト実行
2. スプレッドシート読み込む(アカウント一覧)
3. 指定アカウント選択
4. 指定アカウント・メールテンプレの情報を取得
5. Gmail API で送信
6. 結果確認
```

のような感じです!


![](/images/google/google_api_send_mail.png)


## 手順

ここからは、実装したときの手順を書いていきたいと思います。前提条件で、必要な登録は済ませていることとしておきます。他の記事を参考にしたらたくさん情報あるかとです! 

では、以下の流れから書いていきます.


### 1. サービスアカウントのメールで、使いたいスプレッドシートを共有する
まず、GCP で作成したプロジェクトから入って、作成したサービスアカウントページの詳細から、**メール**を確認して、紐付けしたいスプレッドシートに**共有**で追加しておきます.

![](/images/google/sercice_account_key_dashborad.png)
*サービスアカウント詳細ページ*

![](/images/google/target_spread_sheet_share.png)
*使いたいスプレッドシート*

### 2. スプレッドシートの中身を入力

アカウント一覧シートには、以下の情報を入れました。用途に合わせて、**メール CC・BCC** の項目などあった方が良さそうですね.
zip パスワードは、添付ファイルにつけるパスワードを想定しています。

```bash:アカウント一覧
# セル: 項目
A: ID
B: アカウント名
C: メールアドレス(TO)
D: zipパスワード
```

![](/images/google/acounts.png)
*アカウント一覧*

メール_テンプレのシートには以下を持たせました。

```bash:メール_テンプレ
# セル: 項目
A: ID
B: 件名
C: 本文
```

![](/images/google/mail_tmp.png)
*メールテンプレート*


今回は、アカウント情報と、メールテンプレートにシート分けましたが ... 。1シートに全ての情報を持たせたほうが、API リクエスト数を減らすことも可能かと。

意図としては、アカウントにより参照するメールテンプレートを選択できる仕様にもカスタマイズしやすいかなっと思った次第です。後、メールテンプレート全共通なら、スクリプト側に持たせる方が単純に書けて良さそうでもあります。件名・お客様名を変数に持たせるなどすれば。


### 3. Python スクリプト作成
ここは、実際書いたスクリプトをリポジトリとして共有したので、気になる方はロジックみていただけると幸いです🙏

ポイントとしては、

:::message
1. 認証ファイル・サービスアカウントキーの json ファイル を配置
2. [Gmail API Python Quickstart](https://developers.google.com/gmail/api/quickstart/python)を参考にリクエストを実装
3. [Sheets API Python Quickstart](https://developers.google.com/sheets/api/quickstart/python)を参考にリクエストを実装
:::


ですかね。

あとは、やりたいことに合わせて、実装していけば良いかと✋

---

1. json ファイルの配置
   こんな感じで、参照できるように配置しておくと**OK**です❗️

    |格納パス|詳細|
    |:-|:-|
    |`./token/credentials.json`|OAuth 2.0 クライアントの JSONファイル: サンプルコードに合わせた名称|
    |`./token/service-account-key.json`|サービスアカウントキー: 自分で管理しやすい名称でOKです|

2. Gmail API リクエスト部分

  https://github.com/KazusaNakagawa/try-docs/blob/081d55c2aa57bc2302845e334e9d76b1a1d08f57/src/google_api/models/client_service.py#L23-L53

3. Sheets API リクエスト部分

  https://github.com/KazusaNakagawa/try-docs/blob/081d55c2aa57bc2302845e334e9d76b1a1d08f57/src/google_api/models/sheets_api.py#L21-L41


### 4. スクリプト実行してみる

```bash:ながれ
1. main.py を実行
2. アカウント一覧から、No.1 のアカウント名を選択
3. 指定アドレスに mail 送信されていることが確認できた
```
ちょっと、アドレス隠したかったので、無理やり隠してますが、、まッ! 実装できていることが伝われば幸いです 🍊

![](/images/google/send_gmail_run.gif)


## 今後の展開として
- セキュア面を考えて、サービスアカウントキーは json ファイルより [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity?hl=ja#enable_on_cluster)を試してみたいかなっと

- ハードコートな箇所のリファクタリング

- GUI ライブラリをつかってデスクトップアプリにする。今はコンソール上でアカウント選択してるので

- Slack アラート通知も入れると Error 吐いたとき監視できたり

とかですかね。

以上です。ありがとうございました 🙏


## 参考
https://developers.google.com/gmail/api/quickstart/python

https://developers.google.com/sheets/api/quickstart/python

https://cloud.google.com/docs/authentication/production?hl=ja#create-service-account-console:~:text=%E3%81%A6%E3%81%8F%E3%81%A0%E3%81%95%E3%81%84%E3%80%82-,%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%20%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B,-%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%20%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%81%8C
