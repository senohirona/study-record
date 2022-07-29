# what
- AWS Secrets Managerについて記載する



# AWS Secrets Managerとは

ref: [公式](https://aws.amazon.com/jp/secrets-manager/?nc2=h_mo)
- アプリケーション、サービス等へのアクセスに必要なシークレット情報(ユーザーidやパスワード)を一元管理するシステム
- RDSやPostgreSQL、Amazon Aurora への統合を組み込むことでシークレット情報の自動変更をすることが可能


# terraform

該当するような記事
 - https://www.terraform.io/docs/providers/aws/r/secretsmanager_secret.html
 - https://www.terraform.io/docs/providers/aws/r/secretsmanager_secret_version.html
