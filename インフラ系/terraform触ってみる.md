# what
- terraformのチュートリアル(GCP版/AWS版)をやって、わかったこと等を記述する


# terraformとは
- HashiCorp社が提供するインフラ構成管理ツール
    - [introduction](https://www.terraform.io/intro/index.html)
    - `.tf` 拡張子のファイルにインフラ構成を記述し、Terraformのコマンドでそのファイルを実行するとその構成通りのインフラが作成 できる
- 構築内容はコードなのでをgithub上で管理できるといった利点がある

# 構成ファイルについて
- 公式ドキュメントを参考に記述していく
    - [Configuration Language](https://www.terraform.io/docs/configuration/index.html)
- terraformは主に`.tf`ファイルと`.tfstate`というものが存在している(もっとあるが今回は記載しない)
    - `.tf`...terraformのコードが記述された設定ファイル
    - `.tfstate`...terraformで管理してるインフラの状態が記述されたファイル

# 主要コマンド(一部)
- `terraform init`
    - terraformの設定ファイルを含む作業ディレクトリの初期化を行う
- `terraform apply`
    - 設定ファイルに従ってインフラストラクチャを作成
- `terraform show`
    - tfstate内の各リソースの状態を確認する
- `terraform destroy`
    - インフラストラクチャを破壊する
- `terraform plan`
    - 変更される内容を表示する。本番への変更は行わない

# tutorialやってみる
- 下記を参考にGCP上でやってみる
    - [GCP版公式チュートリアル](https://learn.hashicorp.com/terraform?track=gcp#gcp)

### 動作環境
- GCPプロジェクト: 個人で用意
- terraform ver : 0.12.9

## インストール
- brewコマンドでも入れられるようだったので試してみた
    - `brew install terraform`
- 過去にインストール済みだったようで、「アップデートしろ」と言われた
    - `brew update terraform`

## tfファイル記述
- チュートリアルを参考に記述
    - [Build Infrastructure](https://learn.hashicorp.com/terraform/gcp/build)

## インフラストラクチャを立てる
- 今回はgcpのcomputeエンジンに環境を追加したりすることをおこなった

## インフラストラクチャの変更
- tfファイルに追加や変更をおこない、applyした際の結果を見てみる

### プレフィックス
- `+`... 追加されたリソースを表す
- `-`...変更または更新されたリソースを表す
- `+/-`...リソースを破壊して再構成された事を表す(大幅な変更を加えた際に表示される)

## インフラストラクチャの削除
- `terraform destroy`コマンドで作成したインフラストラクチャを削除する

# 12/18追記
- 変数の扱いについてあまりわかっていない部分があるのでtutorialの続きを読んでみる
    - https://learn.hashicorp.com/terraform?track=gcp#gcp

# (2021/03/04)AWS版やってみる
以下を参考にaws版のterraformを触ってみる
[Get Started - AWS](https://learn.hashicorp.com/collections/terraform/aws-get-started)
今回はterraformを利用してEC2インスタンスを作成するまでがゴール

※ terraformのインストールやAWSのアカウント作成については既に行っているので省略

**動作環境**
- AWSアカウント: 個人のアカウント

##  インフラストラクチャを立てる
参考:[Build Infrastructure](https://learn.hashicorp.com/tutorials/terraform/aws-build?in=terraform/aws-get-started) 通りに作業してみる

`terraform init`する
terraformで新しい構成を作る際はディレクトリを初期化する必要があり、`init`はそれを行うためのコマンド

```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 3.27"...
- Installing hashicorp/aws v3.31.0...
- Installed hashicorp/aws v3.31.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## 構成をフォーマットして検証する
書いた内容をlintのような形式で整理する
書き方に問題がなければ特にメッセージは表示されない
```
terraform fmt
```

## インフラストラクチャを作成する
main.tfで作成していた設定をAWSに反映させる
`terrafrom apply`を利用する

```
$ envchain aws-5335-5708-6642 terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                          = "ami-830c94e3"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t2.micro"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
      + key_name                     = (known after apply)
      + outpost_arn                  = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      + primary_network_interface_id = (known after apply)
      + private_dns                  = (known after apply)
      + private_ip                   = (known after apply)
      + public_dns                   = (known after apply)
      + public_ip                    = (known after apply)
      + secondary_private_ips        = (known after apply)
      + security_groups              = (known after apply)
      + source_dest_check            = true
      + subnet_id                    = (known after apply)
      + tags                         = {
          + "Name" = "test-ExampleInstance"
        }
      + tenancy                      = (known after apply)
      + vpc_security_group_ids       = (known after apply)

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
  (ここで`yes`と入力する)

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Still creating... [40s elapsed]
aws_instance.example: Still creating... [50s elapsed]
aws_instance.example: Still creating... [1m0s elapsed]
aws_instance.example: Creation complete after 1m5s [id=i-0dcfa2a5851b7e0bf]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

**`Do you want to perform these actions?`について**
terraformは適用内容をapplyする際に確認として「本当に承認するか?」と質問する。
内容が問題なければ`yes`を入力して適用する


## 状態を検証する
applyすると、反映された内容が`terraform.tfstate`に書き込まれる。
このファイル上で設定内容を確認することが可能。
もしくは`terraform show`コマンドでも確認が可能
```
$ terraform show
# aws_instance.example:
resource "aws_instance" "example" {
    ami                          = "ami-830c94e3"
    arn                          = "arn:aws:ec2:us-west-2:533557086642:instance/i-0dcfa2a5851b7e0bf"
    associate_public_ip_address  = true
    availability_zone            = "us-west-2b"
    cpu_core_count               = 1
    cpu_threads_per_core         = 1
    disable_api_termination      = false
    ebs_optimized                = false
    get_password_data            = false
    hibernation                  = false
    id                           = "i-0dcfa2a5851b7e0bf"
    instance_state               = "running"
    instance_type                = "t2.micro"
    ipv6_address_count           = 0
    ipv6_addresses               = []
    monitoring                   = false
    primary_network_interface_id = "eni-0f18baf402cf9cec6"
    private_dns                  = "ip-172-31-38-28.us-west-2.compute.internal"
    private_ip                   = "172.31.38.28"
    public_dns                   = "ec2-54-184-207-49.us-west-2.compute.amazonaws.com"
    public_ip                    = "54.184.207.49"
    secondary_private_ips        = []
    security_groups              = [
        "default",
    ]
    source_dest_check            = true
    subnet_id                    = "subnet-88bd14c3"
    tags                         = {
        "Name" = "test-ExampleInstance"
    }
    tenancy                      = "default"
    vpc_security_group_ids       = [
        "sg-37f3d249",
    ]

    credit_specification {
        cpu_credits = "standard"
    }

    enclave_options {
        enabled = false
    }

    metadata_options {
        http_endpoint               = "enabled"
        http_put_response_hop_limit = 1
        http_tokens                 = "optional"
    }

    root_block_device {
        delete_on_termination = true
        device_name           = "/dev/sda1"
        encrypted             = false
        iops                  = 0
        tags                  = {}
        throughput            = 0
        volume_id             = "vol-051fdfb3a346645ec"
        volume_size           = 8
        volume_type           = "standard"
    }
}
```

## 実際に見に行く
実際にインスタンスが作られたかを確認しに行く
今回の検証ではインスタンスが作成されたのを確認できた
