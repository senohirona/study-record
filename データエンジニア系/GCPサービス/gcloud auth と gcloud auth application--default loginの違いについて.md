# what

- gcloudコマンドである`gcloud auth` と `gcloud auth application--default login`の違いについて調べたことを記載する

# gcloud authコマンドについて

Google Cloud を操作する際に必要な認証コマンド。

【コマンド例】

```python
$ gcloud auth list

$ gcloud auth application-default login
```

# 違い

- **gcloud auth login**
    - ユーザーアカウントで認証する際に利用するコマンド
- **gcloud auth application--default login**
    - ローカルで実行するアプリケーションやスクリプトがGoogle Cloudにアクセスするための認証情報を設定するためのコマンド

### 【比較】

| 特徴 | gcloud auth login | **gcloud auth application--default login** |
| --- | --- | --- |
| 認証対象 | ユーザーアカウント | アプリケーション |
| 認証情報 | ユーザーアカウントの認証情報 | アプリケーションのデフォルト認証情報（ADC） |
| 主な用途 | gcloudコマンドラインツール（bq、gsutilなど）の操作 | ローカルでのアプリケーション開発、スクリプト実行 |
| 具体例 | VMインスタンスの作成、Cloud Storageバケットの操作 | PythonスクリプトからBigQueryにデータを書き込む |

## 使い分けの基準

- **Google Cloudの管理作業をしたいとき**: `gcloud auth login`
- **ローカルで開発しているアプリケーションからGoogle Cloudサービスを利用したいとき**: `g-cloud auth application-default login`

# 参考
- [gcloud auth login  |  Google Cloud SDK](https://cloud.google.com/sdk/gcloud/reference/auth/login)
- [gcloud auth application-default login  |  Google Cloud SDK](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)