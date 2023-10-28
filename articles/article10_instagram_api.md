---
title: "Instagram Graph API で投稿データを取得する"
emoji: "🥞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Instagram,API,Python]
published: true
---

## はじめに

Instagram Graph API で投稿を取得するスクリプトを Python で作成。
Instagram Graph API を使う機会がある方の参考になれば幸いです。
対応した時の API version は、**v18.0** です。

アクセストークンの取得については、他の記事がわかりやすく、今回は自身が残しておきたいポイントだけをまとめております。



## 事前準備

### アクセストークン生成

他に参考にしていただいた記事がわかりやすく、今回は自身がつまったポイントだけをまとめております。


1. ビジネスアカウントを作成する必要がある
    
   Instagram, facebook アカウント連携する工程が必要...

  https://tabiato.co.jp/biz/blog/instagram-graph-api-setup/

    
2. アクセス許可をアプリに付与

    **v18.0** では、以下のアクセス許可が必要. 公式にも使いたい API によって、必要なアクセス許可が記載されていました。

    | No | 名称 | Endpoints | 用途 |
    | --- | --- | --- | --- |
    | 1 | [IG Hashtag Search](https://developers.facebook.com/docs/instagram-api/reference/ig-hashtag-search) | GET /ig_hashtag_search?user_id={user-id}&q={q} | IGハッシュタグ検索<br>※ No.2 の {ig-hashtag-id} を入れるため |
    | 2 | [IG Hashtag Recent Media](https://developers.facebook.com/docs/instagram-api/reference/ig-hashtag/recent-media) | GET /{ig-hashtag-id}/recent_media?user_id={user-id}&fields={fields} | IGハッシュタグの最近のメディア取得 |
    | 3 | [Business Discovery](https://developers.facebook.com/docs/instagram-api/guides/business-discovery) | GET /{ig-user-id}/business_discovery | ビジネスアカウント取得 |

    
    ```markdown
    ## アクセス許可 8つ
    - pages_show_list
    - ads_management
    - business_management
    - instagram_basic
    - instagram_manage_comments
    - instagram_manage_insights
    - instagram_content_publish
    - instagram_manage_messages
    ```
    
3. アクセストークン延長をする

    こちらを参考にさせていただきました🙏
        
　  https://calieto.com/calietoblog/embedding-method-instagram-wordpress/


```bash: 参考_3つのアクセストークンの違い
# １つ目のアクセストークン
アプリを作成した際のデフォルト状態のアクセストークンです。
インスタグラムへのアクセス許可を追加していないので、連携に使う事はできません。

# ２つ目のアクセストークン
期間限定でインスタグラムへのアクセス許可を追加した状態のアクセストークンです。
短期的にしか使用できません。

# ３つ目のアクセストークン
期間制限のない、長期的に使用できるアクセストークンです。
これを使用します。
```
    
4. 以下の アクセストークンを使用する
    
    ![img2.png](/images/instagram/img2.png)
    
5. ビジネスアカウント取得
    
    ```bash
    me?fields=accounts{instagram_business_account}
    ```
    
    ![img3.png](/images/instagram/img3.png)
    

<br><br>ここまでやって、やっと投稿を取得できるようになります 🍵 🍵 🍵 <br><br>

気を取り直して、Python で実装した内容です !!



## 動作環境
1. Mac M1
2. 仮想環境
3. Python 3.9 系
4. Instagram Graph API v18.0

## リポジトリ

README にも記載していますが、以下のリポジトリにコードをまとめています。
clone して、興味があれば access token を発行して実行してみてください。

https://github.com/KazusaNakagawa/external-api-app


## 検証

### 1. 実行結果

1. コマンド実行
    
    `limit` パラメータがあり **Max 50 件** まで取得できることを確認. 公式ドキュメントからは読み取れなかった ...

    ```bash
    # 3件で取得
    (.venv)  ~/code/external_api_app  (🍁🍡main *=)
    $ python main.py -query bluebottle -limit 3
    2023-10-29 00:49:50,503 - INFO - instagram(49) - 200
    2023-10-29 00:49:50,505 - INFO - instagram(55) - {'query': 'bluebottle', 'id': '17843857450040591'}
    2023-10-29 00:49:51,627 - INFO - instagram(77) - {'response code': 200}
    2023-10-29 00:49:51,632 - INFO - instagram(120) - 投稿数: 3
    2023-10-29 00:49:52,346 - INFO - instagram(102) - {'response code': 200}
    2023-10-29 00:49:52,354 - INFO - instagram(120) - 投稿数: 3
    ```

2. 取得データイメージ
  JSON, CSV でファイル出力できるようにしています。

    ```json:JSONの出力結果イメージ
    {
        "data": [
            {
                "id": "xxxxxxxxxxxxxxxxx",
                "username": "bluebottle",
                "caption": "Our ...",
                "comments_count": 21,
                "like_count": 283,
                "media_product_type": "FEED",
                "media_url": "https://scon...",
                "permalink": "https://www.instagram.com/p/...",
                "timestamp": "2023-10-26T19:05:20+0000"
            },
            {
                # ...
            }
        ],
        "paging": {
            "cursors": {
                "after": "QVF..."
            }
        }
    }
   ```


### 2. API 叩きすぎるとアカウント一時停止
意図的に、レート制限を超えるまでリクエスト. アカウント一時停止!!?
異議申請をして復旧させましたが、やはりシビアですね... 🧨

![img4.png](/images/instagram/img4.png =550x) 
![img5.png](/images/instagram/img5.png =450x)

## これから

1. リクエスト制限を超えないようにする
   - ig-hashtag-search で取得した `{ig-hashtag-id}` は、一度取得したら、DB, 設定ファイルに保存するなどして再利用する
2. トークンの有効期限が切れたら、自動で更新するようにする. 多分今のままだと、無期限トークンが取得できてないので
3. ハッシュタグ, ビジネスアカウントの起動コマンドを分ける. 今は一度に取得する動きになっている
   - API 化して、フロントエンドからも使えるようにする
5. エラーハンドリングを入れる
6. 本格的に使うなら、アプリ申請も必要になりそう


## 参考
https://developers.facebook.com/docs/instagram-api?locale=en_US
https://tabiato.co.jp/biz/blog/instagram-graph-api-setup/
https://calieto.com/calietoblog/embedding-method-instagram-wordpress/