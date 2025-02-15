# oauth2-tutorial-funiki

雰囲気でOAuth2.0を使っているエンジニアがOAuth2.0を整理して、手を動かしながら学べる本のチュートリアル実践

# 書籍情報

* 技術書典マーケット

  https://techbookfest.org/product/5704912797171712

* 技術書同人誌博覧会（技書博）

  https://gishohaku.dev/gishohaku7/books/mZPdD26VFkAIruCrPARr

* BOOTH

  https://booth.pm/ja/items/1296585

# 第6章 チュートリアル 実践メモ

Google Cloudを利用して、クライアントがアクセストークンを取得する流れを、手を動かしながら学ぶ。

## 6.1 クライアントの登録

流れ

1. Google Cloudの利用規約の同意
2. プロジェクトの作成
3. Google Photos Library API の有効化
4. 認証画面の作成
5. OAuthクライアントIDの作成
6. テストユーザーの追加

### 1. Google Cloudの利用規約の同意

今回はSkip（初回利用時のみ実施）

https://console.cloud.google.com/

### 2. プロジェクトの作成

* プロジェクト名
```
FunikiProject
```

### 3. Google Photos Library API の有効化

特記事項無し

### 4. 認証画面の作成

* アプリ名
```
image-edit
```

### 5. OAuthクライアントIDの作成

* 承認済みのリダイレクト URI
```
http://127.0.0.1/callback
```
* 認証情報をダウンロード
```
{
  "web": {
    "client_id": "95006....n9goq.apps.googleusercontent.com",
    "project_id": "funikiproject-450608",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "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",
    "redirect_uris": [
      "http://127.0.0.1/callback"
    ]
  }
}
```
※補足：ダウンロードした情報を紛失しても以下から確認できる

https://console.cloud.google.com/auth/clients


### 6. テストユーザーの追加

特記事項無し

## 6.2 認可コードグラント

![書籍より引用](images/image-1.png)

流れ

* Step1. 認可コードの取得（認可エンドポイント）
* Step2. アクセストークンの取得（トークンエンドポイント）
* Step3. リソースへのアクセス

### Step1. 認可コードの取得（認可エンドポイント）

* 認可エンドポイント
```
https://accounts.google.com/o/oauth2/v2/auth
```
* 認可リクエスト
```
https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=95006....n9goq.apps.googleusercontent.com&state=xyz&scope=https://www.googleapis.com/auth/photoslibrary.readonly&redirect_uri=http://127.0.0.1/callback
```
* 認可レスポンス
```
http://127.0.0.1/callback?state=xyz&code=4%2F0ASVgi3JL0_XPYCSJ-BHRYFr_wPg-CiJB9mKU5At2A1SunmtsTZtCTznREA5B5Rrkh.....&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fphotoslibrary.readonly
```

### Step2. アクセストークンの取得（トークンエンドポイント）

* トークンリクエスト
```
curl \
-d "client_id=95006....n9goq.apps.googleusercontent.com" \
-d "client_secret=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" \
-d "redirect_uri=http://127.0.0.1/callback" \
-d "grant_type=authorization_code" \
-d "code=4%2F0ASVgi3JL0_XPYCSJ-BHRYFr_wPg-CiJB9mKU5At2A1SunmtsTZtCTznREA5B5Rrkh....." \
https://www.googleapis.com/oauth2/v4/token
```
* トークンレスポンス
```
{
  "access_token": "ya29.a0AXeO80SdenD2ceTbOqZ2V-_....._-KcMCSEyVg0175",
  "expires_in": 3539,
  "scope": "https://www.googleapis.com/auth/photoslibrary.readonly",
  "token_type": "Bearer"
}
```

### Step3. リソースへのアクセス

* リソースへのアクセス
```
curl \
-H 'Authorization: Bearer ya29.a0AXeO80SdenD2ceTbOqZ2V-_....._-KcMCSEyVg0175' \
https://photoslibrary.googleapis.com/v1/albums
```

* リソースの提供
```
{
  "nextPageToken": "CkAKPnR5cGUuZ29vZ2xlY.....ZZanJqczlHcWlibzhxdk9XM3ExeTRoRnY3TkNzdHV5T2xFNGxRSS1XSDFpGgA"
}
```

## 6.2 認可コードグラント + [PKCE(Proof Key for Code Exchange)](https://www.authlete.com/developers/pkce/)

流れ

* Step1. 認可コードの取得（認可エンドポイント）
* Step2. アクセストークンの取得
* Step3. リソースへのアクセス

### Step1. 認可コードの取得（認可エンドポイント）

* 認可リクエスト
```
https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=95006....n9goq.apps.googleusercontent.com&state=xyz&scope=https://www.googleapis.com/auth/photoslibrary.readonly&redirect_uri=http://127.0.0.1/callback&code_challenge_method=S256&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
```
* 認可レスポンス
```
http://127.0.0.1/callback?state=xyz&code=4%2F0ASVgi3KEl3n8wIzskLqsVToePtlgzCm40JAksO4WCbAVtMyWscAuR5_JR3AEJ5xJi.....&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fphotoslibrary.readonly
```

### Step2. アクセストークンの取得

* トークンリクエスト
```
curl \
-d "client_id=95006....n9goq.apps.googleusercontent.com" \
-d "client_secret=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" \
-d "redirect_uri=http://127.0.0.1/callback" \
-d "grant_type=authorization_code" \
-d "code=4%2F0ASVgi3KEl3n8wIzskLqsVToePtlgzCm40JAksO4WCbAVtMyWscAuR5_JR3AEJ5xJi....." \
-d "code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk" \
https://www.googleapis.com/oauth2/v4/token
```
* トークンレスポンス
```
{
  "access_token": "ya29.a0AXeO80QM9QYNkAaI3FcmXFx.....pnsxb2Aq73g0177",
  "expires_in": 3365,
  "scope": "https://www.googleapis.com/auth/photoslibrary.readonly",
  "token_type": "Bearer"
}
```

### Step3. リソースへのアクセス

* リソースへのアクセス
```
curl \
-H 'Authorization: Bearer ya29.a0AXeO80QM9QYNkAaI3FcmXFx.....pnsxb2Aq73g0177' \
https://photoslibrary.googleapis.com/v1/albums
```

* リソースの提供
```
{
  "nextPageToken": "CkAKPnR5cGUuZ29vZ2xlY.....ZZanJqczlHcWlibzhxdk9XM3ExeTRoRnY3TkNzdHV5T2xFNGxRSS1XSDFpGgA"
}
```

## 6.4 インプリシットグラント

![書籍より引用](images/image-2.png)

流れ

* Step1. 認可コードの取得（認可エンドポイント）
* Step2. リソースへのアクセス

### Step1. 認可コードの取得（認可エンドポイント）

* 認可エンドポイント
```
https://accounts.google.com/o/oauth2/v2/auth
```
* 認可リクエスト
```
https://accounts.google.com/o/oauth2/v2/auth?scope=https://www.googleapis.com/auth/photoslibrary.readonly&redirect_uri=http://127.0.0.1/callback&response_type=token&client_id=95006....n9goq.apps.googleusercontent.com
```
* 認可レスポンス
```
http://127.0.0.1/callback#access_token=ya29.a0AXeO80QM9QYNkAaI3FcmXFx.....pnsxb2Aq73g0177&token_type=Bearer&expires_in=3599&scope=https://www.googleapis.com/auth/photoslibrary.readonly
```

### Step2. リソースへのアクセス

* リソースへのアクセス
```
curl \
-H 'Authorization: Bearer ya29.a0AXeO80QM9QYNkAaI3FcmXFx.....pnsxb2Aq73g0177' \
https://photoslibrary.googleapis.com/v1/albums
```

* リソースの提供
```
{
  "nextPageToken": "CkAKPnR5cGUuZ29vZ2xlY.....ZZanJqczlHcWlibzhxdk9XM3ExeTRoRnY3TkNzdHV5T2xFNGxRSS1XSDFpGgA"
}
```
