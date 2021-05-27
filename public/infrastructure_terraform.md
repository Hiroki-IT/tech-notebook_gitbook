# Terraform

## 01. コマンド

### init

#### ・-backend=false

ローカルにstateファイルを作成する．

```shell
$ terraform init -backend=false
```

```shell
# ディレクトリを指定することも可能
$ terraform init -backend=false <ルートモジュールのディレクトリへの相対パス>
```

#### ・-backend=true, -backend-config

リモートにstateファイルを作成する．代わりに，```terraform settings```ブロック内の```backend```で指定しても良い．

```shell
$ terraform init \
    -backend=true \
    -reconfigure \
    -backend-config="bucket=<バケット名>" \
    -backend-config="key=terraform.tfstate" \
    -backend-config="profile=<プロファイル名>" \
    -backend-config="encrypt=true" \
    <ルートモジュールのディレクトリへの相対パス>
```

#### ・-reconfigure

指定されたバックエンドのstateファイルがある場合，これを削除し，新しくstateファイルを作成する．

```shell
$ terraform init -reconfigure
```

#### ・-upgrade

モジュールとプラグインを更新する．

```shell
$ terraform init -upgrade
```

<br>

### validate

#### ・オプション無し

設定ファイルの検証を行う．

```shell
$ terraform validate

Success! The configuration is valid.
```

```shell
# ディレクトリを指定することも可能
$ terraform validate <ルートモジュールのディレクトリへの相対パス>
```

<br>

### fmt

#### ・-check

インデントを揃えるべき箇所が存在するかどうかを判定する．もし存在する場合「```1```」，存在しない場合は「```0```」を返却する．

```shell
$ terraform fmt -check
```

#### ・-recursive

設定ファイルのインデントを揃える．処理を行ったファイルが表示される．

```shell
# -recursive: サブディレクトリを含む全ファイルをフォーマット
$ terraform fmt -recursive

main.tf
```

<br>

### import

#### ・-var-file

terraformによる構築ではない方法で，すでにクラウド上にリソースが構築されている場合，これをterraformの管理下におく必要がある．リソースタイプとリソース名を指定し，stateファイルにリモートの状態を書き込む．現状，全てのリソースを一括して```import```する方法は無い．リソースIDは，リソースによって異なるため，リファレンスの「Import」または「Attributes Referenceの```id```」を確認すること（例えば，ACMにとってのIDはARNだが，S3バケットにとってのIDはバケット名である）．

```shell
$ terraform import \
    -var-file=config.tfvars \
    <リソースタイプ>.<リソース名> <AWS上リソースID>
```

モジュールを使用している場合，指定の方法が異なる．

```shell
$ terraform import \
    -var-file=config.tfvars \
    module.<モジュール名>.<リソースタイプ>.<リソース名> <AWS上リソースID>
```

例えば，AWS上にすでにECRが存在しているとして，これをterraformの管理下におく．

```shell
$ terraform import \
    -var-file=config.tfvars \
    module.ecr.aws_ecr_repository.www xxxxxxxxx
```

そして，ローカルのstateファイルとリモートの差分が無くなるまで，```import```を繰り返す．

````shell
$ terraform plan -var-file=config.tfvars

No changes. Infrastructure is up-to-date.
````

#### ・importを行わなかった場合のエラー

もし```import```を行わないと，すでにクラウド上にリソースが存在しているためにリソースを構築できない，というエラーになる．

（エラー例1）

```shell
Error: InvalidParameterException: Creation of service was not idempotent.
```

（エラー例2）

```shell
Error: error creating ECR repository: RepositoryAlreadyExistsException: The repository with name 'tech-notebook_www' already exists in the registry with id 'XXXXXXXXXXXX'
```

<br>

### refresh

#### ・-var-file

クラウドに対してリクエストを行い，現在のリソースの状態をtfstateファイルに反映する．

```shell
$ terraform refresh -var-file=config.tfvars
```

<br>

### plan

#### ・シンボルの見方

構築（```+```），更新（```~```），削除（```-```）で表現される．

```
+ create
~ update in-place
- destroy
```

#### ・-var-file

クラウドに対してリクエストを行い，現在のリソースの状態をtfstateファイルには反映せずに，設定ファイルの記述との差分を検証する．スクリプト実行時に，変数が定義されたファイルを実行すると，```variable```で宣言した変数に，値が格納される．

```shell
$ terraform plan -var-file=config.tfvars
```

```shell
# ディレクトリを指定することも可能
# 第一引数で変数ファイルの相対パス，第二引数でをルートモジュールの相対パス
$ terraform plan \
    -var-file=<ルートモジュールのディレクトリへの相対パス>/config.tfvars \
    <ルートモジュールのディレクトリへの相対パス>
```

差分がなければ，以下の通りになる．

```shell
No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```

#### ・-target

特定のリソースに対して，```plan```コマンドを実行する．

```shell
$ terraform plan \
    -var-file=config.tfvars \
    -target=<リソースタイプ>.<リソース名>
```

モジュールを使用している場合，指定の方法が異なる．

```shell
$ terraform plan \
    -var-file=config.tfvars \
    -target=module.<モジュール名>.<リソースタイプ>.<リソース名>
```

#### ・-refresh

このオプションをつければ，```refresh```コマンドを同時に実行してくれる．ただ，デフォルトで```true```なので，不要である．

```shell
$ terraform plan \
    -var-file=config.tfvars \
    -refresh=true
```

https://github.com/hashicorp/terraform/issues/17311

#### ・-parallelism

並列処理数を設定できる．デフォルト値は```10```である．

```shell
$ terraform plan \
    -var-file=config.tfvars \
    -parallelism=30
```

#### ・-out

実行プランファイルを生成する．```apply```コマンドのために使用できる．

```shell
$ terraform plan \
    -var-file=config.tfvars \
    -out=<実行プランファイル名>.tfplan
```

<br>

### apply

#### ・-var-file

AWS上にクラウドインフラストラクチャを構築する．

```shell
$ terraform apply -var-file config.tfvars
```

```shell
# ディレクトリを指定することも可能
# 第一引数で変数ファイルの相対パス，第二引数でをルートモジュールの相対パス
$ terraform apply \
    -var-file=<ルートモジュールのディレクトリへの相対パス>/config.tfvars \
    <ルートモジュールのディレクトリへの相対パス>
```

成功すると，以下のメッセージが表示される．

```shell
Apply complete! Resources: X added, 0 changed, 0 destroyed.
```

#### ・-target

特定のリソースに対して，```apply```コマンドを実行する．

```shell
$ terraform apply \
    -var-file=config.tfvars \
    -target=<リソースタイプ>.<リソース名>
```

モジュールを使用している場合，指定の方法が異なる．

```shell
$ terraform apply \
    -var-file=config.tfvars \
    -target=module.<モジュール名>.<リソースタイプ>.<リソース名>
```

#### ・-parallelism

並列処理数を設定できる．デフォルト値は```10```である．

```shell
$ terraform apply \
    -var-file=config.tfvars \
    -parallelism=30
```

#### ・実行プランファイル

事前に，```plan```コマンドによって生成された実行プランファイルを元に，```apply```コマンドを実行する．実行プランを渡す場合は，変数をオプションに設定する必要はない．

```shell
$ terraform apply <実行プランファイル名>.tfplan
```

<br>

### taint

#### ・-var-file <リソース>

stateファイルにおける指定されたリソースの```tainted```フラグを立てる．例えば，```apply```したが，途中でエラーが発生してしまい，リモートに中途半端はリソースが構築されてしまうことがある．ここで，```tainted```を立てておくと，リモートのリソースを削除したと想定した```plan```を実行できる．

```shell
$ terraform taint \
    -var-file=config.tfvars \
    module.<モジュール名>.<リソースタイプ>.<リソース名>
```

この後の```plan```コマンドのログからも，```-/+```で削除が行われる想定で，差分を比較していることがわかる．

```shell
$ terraform plan -var-file=config.tfvars

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

-/+ <リソースタイプ>.<リソース名> (tainted) (new resource required)
      id: '1492336661259070634' => <computed> (forces new resource)


Plan: 1 to add, 0 to change, 1 to destroy.
```

<br>

### state list

#### ・オプション無し

ファイル内で定義しているリソースの一覧を表示する．

```shell
$ terraform state list
```

以下の通り，モジュールも含めて，リソースが表示される．

```shell
aws_instance.www-1a
aws_instance.www-1c
aws_key_pair.key_pair
module.alb_module.aws_alb.alb
module.ami_module.data.aws_ami.amazon_linux_2
module.route53_module.aws_route53_record.r53_record
module.route53_module.aws_route53_zone.r53_zone
module.security_group_module.aws_security_group.security_group_alb
module.security_group_module.aws_security_group.security_group_ecs
module.security_group_module.aws_security_group.security_group_instance
module.vpc_module.aws_internet_gateway.internet_gateway
module.vpc_module.aws_route_table.route_table_public
module.vpc_module.aws_route_table_association.route_table_association_public_1a
module.vpc_module.aws_route_table_association.route_table_association_public_1c
module.vpc_module.aws_subnet.subnet_public_1a
module.vpc_module.aws_subnet.subnet_public_1c
module.vpc_module.aws_vpc.vpc
```

<br>

##  01-02. ディレクトリ構成

### ルートモジュールの構成

#### ・稼働環境別

稼働環境別に，```config.tfvars```ファイルで値を定義する．

```shell
terraform_project/
├── modules
│   ├── route53 # Route53
│   │   ├── dev # 開発
│   |   ├── prd # 本番
│   |   └── stg # ステージング
│   | 
│   ├── ssm # SSM
|   |   ├── dev
│   |   ├── prd
│   |   └── stg
│   | 
│   └── waf # WAF
|       ├── dev
│       ├── prd
│       └── stg
|
├── dev # 開発環境ルートモジュール
│   ├── config.tfvars
│   ├── main.tf
│   ├── providers.tf
│   ├── tfnotify.yml
│   └── variables.tf
│
├── prd # 本番環境ルートモジュール
│   ├── config.tfvars
│   ├── main.tf
│   ├── providers.tf
│   ├── tfnotify.yml
│   └── variables.tf
│
└── stg # ステージング環境ルートモジュール
      ├── config.tfvars
      ├── main.tf
      ├── providers.tf
      ├── tfnotify.yml
      └── variables.tf
```

<br>

### リソースのモジュールの構成

####　・対象リソース別

一つのリソースの設定が対象のリソースごとに異なる場合，冗長性よりも保守性を重視して，リソースに応じたディレクトリに分割する．

```shell
terraform_project/
└── modules
    ├── cloudwatch # CloudWatch
    │   ├── alb        # ALB
    |   ├── cloudfront # CloudFront
    |   ├── ecs        # ECS
    |   ├── lambda     # Lambda
    |   └── rds        # RDS    
    |
    └── waf # WAF
        ├── dev
        ├── prd
        └── stg
            ├── alb        # ALB
            └── cloudfront # CloudFront
```

#### ・稼働環境別

一つのリソースの設定が稼働環境ごとに異なる場合，冗長性よりも保守性を重視して，稼働環境に応じたディレクトリに分割する．

```shell
terraform_project/
└── modules
    ├── route53 # Route53
    │   ├── dev # 開発
    |   ├── prd # 本番
    |   └── stg # ステージング
    | 
    ├── ssm # SSM
    |   ├── dev
    |   ├── prd
    |   └── stg
    | 
    └── waf # WAF
        ├── dev
        ├── prd
        └── stg
```

#### ・リージョン別

一つのリソースの設定がリージョンごとに異なる場合，冗長性よりも保守性を重視して，リージョンに応じたディレクトリに分割する．

```shell
terraform_project/
└── modules
    └── acm # ACM
        ├── ap-northeast-1 # 東京リージョン
        └── us-east-1 # バージニアリージョン  
```

#### ・ファイルの切り分け

ポリシーのためにJSONを定義する場合，Terraformのソースコードにハードコーディングせずに，切り分けるようにする．また，「カスタマー管理ポリシー」「インラインポリシー」「信頼ポリシー」も区別し，ディレクトリを分割している．なお，```templatefile```メソッドでこれを読みこむ時，```json```ファイルではなく，tplファイルとして定義しておく必要あるため，注意する．

``` shell
terraform_project/
└── modules 
    ├── ecr #ECR
    │   └── ecr_lifecycle_policy.tpl # ECRライフサイクル
    │
    ├── ecs # ECS
    │   └── container_definitions.tpl # コンテナ定義
    │
    ├── iam # IAM
    │   └── policies  
    |       ├── customer_managed_policies # カスタム管理ポリシー
    |       |   ├── aws_cli_executor_access_policy.tpl
    |       |   ├── aws_cli_executor_access_address_restriction_policy.tpl
    |       |   ├── cloudwatch_logs_access_policy.tpl
    |       |   └── lambda_edge_execution_policy.tpl
    |       |     
    |       ├── inline_policies # インラインポリシー
    |       |   └── ecs_task_policy.tpl
    |       |     
    |       └── trust_policies # 信頼ポリシー
    |           ├── cloudwatch_events_policy.tpl
    |           ├── ecs_task_policy.tpl
    |           ├── lambda_policy.tpl
    |           └── rds_policy.tpl
    |
    └── s3 # S3
        └── policies # バケットポリシー
            └── alb_bucket_policy.tpl
```

<br>

### CI/CDディレクトリ

#### ・opsディレクトリ

TerraformのCI/CDで必要なシェルスクリプトは，```ops```ディレクトリで管理する．

```shell
terraform_project/
├── .circleci # CI/CDツールの設定ファイル
└── ops # TerraformのCI/CDの自動化シェルスクリプト
```

<br>

## 02. ルートモジュールにおける実装

### tfstateファイル

#### ・tfstateファイルとは

リモートのインフラの状態が定義されたjsonファイルのこと．初回時，```apply```コマンドを実行し，成功もしくは失敗したタイミングで生成される．

<br>

### terraform  settings

#### ・terraform settingsとは

terraformの実行時に，エントリポイントとして機能するファイル．

#### ・required_providers

AWSやGCPなど，使用するプロバイダを定義する．プロバイダによって，異なるリソースタイプが提供される．一番最初に読みこまれるファイルのため，変数やモジュール化などが行えない．

**＊実装例＊**

```hcl
terraform {

  required_providers {
    # awsプロバイダを定義
    aws = {
      # グローバルソースアドレスを指定
      source  = "hashicorp/aws"
      
      // プロバイダーのバージョン変更時は initを実行
      version = "3.0" 
    }
  }
}
```

#### ・backend

stateファイルを管理する場所を設定する．S3などのリモートで管理する場合，アカウント情報を設定する必要がある．代わりに，```init```コマンド実行時に指定しても良い．デフォルト値は```local```である．

**＊実装例＊**

```hcl
terraform {

  // ローカルPCで管理するように設定
  backend "local" {
    path = "${path.module}/terraform.tfstate"
  }
}
```

```hcl
terraform {

  // S3で管理するように設定
  backend "s3" {
    bucket                  = "<バケット名>"
    key                     = "<バケット内のディレクトリ>"
    region                  = "ap-northeast-1"
    profile                 = "example"
    shared_credentials_file = "$HOME/.aws/<Credentialsファイル名>"
  }
}
```

どのユーザもバケット内のオブジェクトを削除できないように，ポリシーを設定しておくとよい．

**＊実装例＊**

```json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:DeleteObject",
            "Resource": "arn:aws:s3:::<tfstateのバケット名>/*"
        }
    ]
}
```

<br>

### provider

#### ・providerとは

Terraformがリクエストを送信するプロバイダ（AWS，GCP，Azure，など）を選択し，そのプロバイダにおけるアカウント認証を行う．```terraform settings```で定義したプロバイダ名を指定する必要がある．

**＊実装例＊**

```hcl
terraform {
  required_version = "0.13.5"

  required_providers {
    # awsプロバイダを定義
    aws = {
      # 何らかの設定
    }
  }
  
  backend "s3" {
    # 何らかの設定
  }
}

# awsプロバイダを指定
provider "aws" {
  # アカウント認証の設定
}
```

<br>

### multiple providers

#### ・multiple providersとは

複数の```provider```を実装し，エイリアスを使用して，これらを動的に切り替える方法．

**＊実装例＊**

```hcl
terraform {
  required_version = "0.13.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.0"
    }
  }
}

provider "aws" {
  # デフォルト値とするリージョン
  region = "ap-northeast-1"
}

provider "aws" {
  # 別リージョン
  alias  = "ue1"
  region = "us-east-1"
}
```

#### ・子モジュールでproviderを切り替える

子モジュールで```provider```を切り替えるには，ルートモジュールで```provider```の値を明示的に渡す必要がある．

**＊実装例＊**

```hcl
module "route53" {
  source = "../modules/route53"

  providers = {
    aws = aws.ue1
  }
  
  // その他の設定値
}
```

さらに子モジュールで，```provider```の値を設定する必要がある．

**＊実装例＊**

```hcl
###############################################
# Route53
###############################################
resource "aws_acm_certificate" "example" {
  # CloudFrontの仕様のため，us-east-1リージョンでSSL証明書を作成します．
  provider = aws

  domain_name               = var.route53_domain_example
  subject_alternative_names = ["*.${var.route53_domain_example}"]
  validation_method         = "DNS"

  tags = {
    Name = "${var.environment}-${var.service}-example-cert"
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

<br>

### アカウント情報の設定方法

#### ・ハードコーディングによる設定

リージョンの他，アクセスキーとシークレットキーをハードコーディングで設定する．誤ってコミットしてしまう可能性があるため，ハードコーディングしないようにする．

**＊実装例＊**

```hcl
terraform {
  required_version = "0.13.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.0"
    }
  }
  
  backend "s3" {
    bucket     = "<バケット名>"
    key        = "<バケット内のディレクトリ>"
    region     = "ap-northeast-1"
    access_key = "<アクセスキー>"
    secret_key = "<シークレットキー>"
  }
}

provider "aws" {
  region     = "ap-northeast-1"
  access_key = "<アクセスキー>"
  secret_key = "<シークレットキー>"
}
```

#### ・credentialsファイルによる設定

　AWSアカウント情報は，```~/.aws/credentials```ファイルに記載されている．

```
[default]
aws_access_key_id=<アクセスキー>
aws_secret_access_key=<シークレットキー>

[user1]
aws_access_key_id=<アクセスキー>
aws_secret_access_key=<シークレットキー>
```

credentialsファイルを読み出し，プロファイル名を設定することにより，アカウント情報を参照できる．

**＊実装例＊**

```hcl
terraform {
  required_version = "0.13.5"

  required_providers {
  
    aws = {
      source  = "hashicorp/aws"
      version = "3.0"
    }
  }
  
  // credentialsファイルから，アクセスキー，シークレットアクセスキーを読み込む
  backend "s3" {
    bucket                  = "<バケット名>"
    key                     = "<バケット内のディレクトリ>"
    region                  = "ap-northeast-1"
    profile                 = "example"
    shared_credentials_file = "$HOME/.aws/<Credentialsファイル名>"
  }
}

// credentialsファイルから，アクセスキー，シークレットアクセスキーを読み込む
provider "aws" {
  region                  = "ap-northeast-1"
  profile                 = "example"
  shared_credentials_file = "$HOME/.aws/<Credentialsファイル名>"
}
```

#### ・環境変数による設定

Credentialsファイルではなく，```export```を使用して，必要な情報を設定しておくことも可能である．参照できる環境変数名は決まっている．

```shell
# regionの代わり
$ export AWS_DEFAULT_REGION="ap-northeast-1"

# access_keyの代わり
$ export AWS_ACCESS_KEY_ID="<アクセスキー>"

# secret_keyの代わり
$ export AWS_SECRET_ACCESS_KEY="<シークレットキー>"

# profileの代わり
$ export AWS_PROFILE="<プロファイル名>"

#tokenの代わり（AmazonSTSを使用する場合）
$ export AWS_SESSION_TOKEN="<トークン>"
```

環境変数を設定した上でteraformを実行すると，値が```provider```に自動的に出力される．CircleCIのような，一時的に環境変数が必要になるような状況では有効な方法である．

```hcl
terraform {
  required_version = "0.13.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.0"
    }
  }
  
  // リージョン，アクセスキー，シークレットアクセスキーは不要
  backend "s3" {
    bucket  = "<バケット名>"
    key     = "<バケット内のディレクトリ>"
  }
}

// リージョン，アクセスキー，シークレットアクセスキーは不要
provider "aws" {}
```

<br>

### module

#### ・moduleとは

ルートモジュールで子モジュール読み込み，子モジュールに対して変数を渡す．

#### ・実装方法

**＊実装例＊**

```hcl
###############################
# ALB
###############################
module "alb" {
  // モジュールのResourceを参照
  source = "../modules/alb"
  
  // モジュールに他のモジュールのアウトプット値を渡す．
  acm_certificate_api_arn = module.acm.acm_certificate_api_arn
}
```

<br>

## 03. 変数

### tfvarsファイル

#### ・tfvarsファイルの用途

実行ファイルに入力したい値を定義する．各サービスの間で実装方法が同じため，VPCのみ例を示す．

**＊実装例＊**

```hcl
###############################
# VPC
###############################
vpc_cidr_block = "n.n.n.n/n" // IPv4アドレス範囲
```

<br>

### variable 

#### ・variableとは

リソースで使用する変数を定義する．

**＊実装例＊**

```hcl
###############################
# Input Value
###############################
// AWSCredentials
variable "credential" {
  type = map(string)
}
```

<br>

## 04. リソースの実装

### resource

#### ・resourceとは

AWSのAPIに対してリクエストを送信し，クラウドインフラの構築を行う．

#### ・実装方法

**＊実装例＊**

```hcl
###############################################
# ALB
###############################################
resource "aws_lb" "this" {
  name               = "${var.app_name}-alb"
  load_balancer_type = "application"
  security_groups    = [var.sg_alb_id]
  subnets            = var.subnet_public_ids
}
```

<br>

### data

#### ・dataとは

AWSのAPIに対してリクエストを送信し，クラウドインフラに関するデータを取得する．ルートモジュールに実装することも可能であるが，各モジュールに実装した方が分かりやすい．

#### ・実装方法

**＊実装例＊**

例として，タスク定義名を指定して，AWSから

```hcl
###############################################
# ECS task definition
###############################################
data "aws_ecs_task_definition" this {
  task_definition = aws_ecs_task_definition.this.family
}
```

**＊実装例＊**

例として，AMIをフィルタリングした上で，AWSから特定のAMIの値を取得する．

```hcl
###############################################
# AMI
###############################################
data "aws_ami" "bastion" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "name"
    values = ["amzn-ami-hvm-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "block-device-mapping.volume-type"
    values = ["gp2"]
  }
}
```

<br>

### output

#### ・outputとは

モジュールで構築されたリソースがもつ特定の値を出力する．

#### ・実装方法

**＊実装例＊**

例として，ALBを示す．```resource```ブロックと```data```ブロックでアウトプットの方法が異なる．

```hcl
###############################################
# ALB
###############################################
output "alb_zone_id" {
  value = aws_lb.this.zone_id
}

output "elb_service_account_arn" {
  value = data.aws_elb_service_account.this.arn
}
```
#### ・map型でアウトプット

```hcl
###############################################
# Output VPC
###############################################
output "public_subnet_ids" {
  value = {
    a = aws_subnet.public[var.vpc_availability_zones.a].id,
    c = aws_subnet.public[var.vpc_availability_zones.c].id
  }
}

output "private_app_subnet_ids" {
  value = {
    a = aws_subnet.private_app[var.vpc_availability_zones.a].id,
    c = aws_subnet.private_app[var.vpc_availability_zones.c].id
  }
}

output "private_datastore_subnet_ids" {
  value = {
    a = aws_subnet.private_datastore[var.vpc_availability_zones.a].id,
    c = aws_subnet.private_datastore[var.vpc_availability_zones.c].id
  }
}
```

```hcl
###############################################
# ALB
###############################################
resource "aws_lb" "this" {
  name                       = "${var.environment}-${var.service}-alb"
  subnets                    = values(private_app_subnet_ids)
  security_groups            = [var.alb_security_group_id]
  internal                   = false
  idle_timeout               = 120
  enable_deletion_protection = true

  access_logs {
    enabled = true
    bucket  = var.alb_s3_bucket_id
  }
}
```

<br>

## 05. メタ引数

### メタ引数とは

全てのリソースで使用できるオプションのこと．

<br>

### depends_on

#### ・depends_onとは

リソース間の依存関係を明示的に定義する．Terraformでは，基本的にリソース間の依存関係が暗黙的に定義されている．しかし，複数のリソースが関わると，リソースを適切な順番で構築できない場合があるため，そういったときに使用する．

#### ・ALB target group vs. ALB，ECS

例として，ALB target groupを示す．ALB Target groupとALBのリソースを適切な順番で構築できないため，ECSの構築時にエラーが起こる．ALBの後にALB target groupを構築する必要がある．

**＊実装例＊**

```hcl
###############################################
# ALB target group
###############################################
resource "aws_lb_target_group" "this" {
  name                 = "${var.environment}-${var.service}-alb-tg"
  port                 = var.ecs_nginx_port_http
  protocol             = "HTTP"
  vpc_id               = var.vpc_id
  deregistration_delay = "60"
  target_type          = "ip"
  slow_start           = "60"

  health_check {
    interval            = 30
    path                = "/healthcheck"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
    matcher             = 200
  }

  depends_on = [aws_lb.this]
}
```

#### ・Internet Gateway vs. EC2，Elastic IP，NAT Gateway

例として，NAT Gatewayを示す．NAT Gateway，Internet Gateway，のリソースを適切な順番で構築できないため，Internet Gatewayの構築後に，NAT Gatewayを構築するように定義する必要がある．

```hcl
###############################################
# EC2
###############################################
resource "aws_instance" "bastion" {
  ami                         = var.bastion_ami_amazon_id
  instance_type               = "t2.micro"
  vpc_security_group_ids      = [var.ec2_bastion_security_group_id]
  subnet_id                   = var.public_a_subnet_id
  key_name                    = "${var.environment}-${var.service}-bastion"
  associate_public_ip_address = true
  disable_api_termination     = true

  tags = {
    Name = "${var.environment}-${var.service}-bastion"
  }

  depends_on = [var.internet_gateway]
}
```

```hcl
###############################################
# Elastic IP
###############################################
resource "aws_eip" "nat_gateway" {
  for_each = var.vpc_availability_zones

  vpc = true

  tags = {
    Name = format(
      "${var.environment}-${var.service}-ngw-%s-eip",
      each.value
    )
  }

  depends_on = [aws_internet_gateway.this]
}
```

```hcl
###############################################
# NAT Gateway
###############################################
resource "aws_nat_gateway" "this" {
  for_each = var.vpc_availability_zones

  subnet_id     = aws_subnet.public[each.key].id
  allocation_id = aws_eip.nat_gateway[each.key].id

  tags = {
    Name = format(
      "${var.environment}-${var.service}-%s-ngw",
      each.value
    )
  }

  depends_on = [aws_internet_gateway.this]
}
```

#### ・S3バケットポリシー vs. パブリックアクセスブロックポリシー

例として，S3を示す．バケットポリシーとパブリックアクセスブロックポリシーを同時に構築できないため，構築のタイミングが重ならないようにする必要がある．

```hcl
###############################################
# S3
###############################################

# Example bucket
resource "aws_s3_bucket" "example" {
  bucket = "${var.environment}-${var.service}-example-bucket"
  acl    = "private"
}

# Public access block
resource "aws_s3_bucket_public_access_block" "example" {
  bucket                  = aws_s3_bucket.example.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Bucket policy attachment
resource "aws_s3_bucket_policy" "example" {
  bucket = aws_s3_bucket.example.id
  policy = templatefile(
    "${path.module}/policies/example_bucket_policy.tpl",
    {
      example_s3_bucket_arn                        = aws_s3_bucket.example.arn
      s3_cloudfront_origin_access_identity_iam_arn = var.s3_cloudfront_origin_access_identity_iam_arn
    }
  )

  depends_on = [aws_s3_bucket_public_access_block.example]
}
```

<br>

### count

#### ・countとは

指定した数だけ，リソースの構築を繰り返す．```count.index```でインデックス数を出力する．

**＊実装例＊**

```hcl
resource "aws_instance" "server" {
  count = 4
  
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"

  tags = {
    Name = "ec2-${count.index}"
  }
}
```

<br>

### for_each

#### ・for_eachとは

事前に```for_each```に格納したmap型の```key```の数だけ，リソースを繰り返し実行する．繰り返し処理を行う時に，```count```とは違い，要素名を指定して出力することができる．

**＊実装例＊**

例として，subnetを繰り返し構築する．

```hcl
###############################################
# Variables
###############################################
vpc_availability_zones             = { a = "a", c = "c" }
vpc_cidr                           = "n.n.n.n/23"
vpc_subnet_private_datastore_cidrs = { a = "n.n.n.n/27", c = "n.n.n.n/27" }
vpc_subnet_private_app_cidrs       = { a = "n.n.n.n/25", c = "n.n.n.n/25" }
vpc_subnet_public_cidrs            = { a = "n.n.n.n/27", c = "n.n.n.n/27" }
```

```hcl
###############################################
# Public subnet
###############################################
resource "aws_subnet" "public" {
  for_each = var.vpc_availability_zones

  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.vpc_subnet_public_cidrs[each.key]
  availability_zone       = "${var.region}${each.value}"
  map_public_ip_on_launch = true

  tags = {
    Name = format(
      "${var.environment}-${var.service}-pub-%s-subnet",
      each.value
    )
  }
}
```

<br>

### dynamic

#### ・dynamicとは

指定したブロックを繰り返し構築する．

**＊実装例＊**

例として，RDSパラメータグループの```parameter```ブロックを，map型変数を使用して繰り返し構築する．

```hcl
###############################################
# Variables
###############################################
rds_parameter_group_values = {
  time_zone                = "asia/tokyo"
  character_set_client     = "utf8mb4"
  character_set_connection = "utf8mb4"
  character_set_database   = "utf8mb4"
  character_set_results    = "utf8mb4"
  character_set_server     = "utf8mb4"
  server_audit_events      = "connect,query,query_dcl,query_ddl,query_dml,table"
  server_audit_logging     = 1
  server_audit_logs_upload = 1
  general_log              = 1
  slow_query_log           = 1
  long_query_time          = 3
}

###############################################
# RDS Cluster Parameter Group
###############################################
resource "aws_rds_cluster_parameter_group" "this" {
  name        = "${var.environment}-${var.service}-cluster-pg"
  description = "The cluster parameter group for ${var.environment}-${var.service}-rds"
  family      = "aurora-mysql5.7"

  dynamic "parameter" {
    for_each = var.rds_parameter_group_values

    content {
      name  = parameter.key
      value = parameter.value
    }
  }
}
```

**＊実装例＊**

例として，WAFの正規表現パターンセットの```regular_expression```ブロックを，list型変数を使用して繰り返し構築する．

```hcl
###############################################
# Variables
###############################################
waf_blocked_user_agents = [
  "ExampleCrawler",
  "EXampleSpider",
  "ExampleBot",
]

###############################################
# WAF Regex Pattern Sets
###############################################
resource "aws_wafv2_regex_pattern_set" "cloudfront" {
  name        = "blocked-user-agents"
  description = "Blocked user agents"
  scope       = "CLOUDFRONT"

  dynamic "regular_expression" {
    for_each = var.waf_blocked_user_agents

    content {
      regex_string = regular_expression.value
    }
  }
}
```

<br>

### lifecycle

#### ・lifecycleとは

リソースの構築，更新，そして削除のプロセスをカスタマイズする．

#### ・create_before_destroy

リソースを新しく構築した後に削除するように，変更できる．通常時，Terraformの処理順序として，リソースの削除後に構築が行われる．しかし，他のリソースと依存関係が存在する場合，先に削除が行われることによって，他のリソースに影響が出てしまう．これに対処するために，先に新しいリソースを構築し，紐づけし直してから，削除する必要がある．

**＊実装例＊**

例として，ACM証明書を示す．ACM証明書は，ALBやCloudFrontに関連付いており，新しい証明書に関連付け直した後に，既存のものを削除する必要がある．

```hcl
###############################################
# For example domain
###############################################
resource "aws_acm_certificate" "example" {
  domain_name               = var.route53_domain_example
  subject_alternative_names = ["*.${var.route53_domain_example}"]
  validation_method         = "DNS"

  tags = {
    Name = "${var.environment}-${var.service}-example-cert"
  }

  # 新しい証明書を構築した後に削除する．
  lifecycle {
    create_before_destroy = true
  }
}
```

**＊実装例＊**

例として，RDSのクラスターパラメータグループとサブネットグループを示す．クラスターパラメータグループとサブネットグループは，RDSに関連付いており，新しいクラスターパラメータグループに関連付け直した後に，既存のものを削除する必要がある．

```hcl
###############################################
# RDS Cluster Parameter Group
###############################################
resource "aws_rds_cluster_parameter_group" "this" {
  name        = "${var.environment}-${var.service}-rds-cluster-param-gp"
  description = "The cluster parameter group for ${var.environment}-${var.service}-rds"
  family      = "aurora-mysql5.7"

  dynamic "parameter" {
    for_each = var.rds_parameter_group_values

    content {
      name  = parameter.key
      value = parameter.value
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

###############################################
# RDS Subnet Group
###############################################
resource "aws_db_subnet_group" "this" {
  name        = "${var.service}-${var.environment}-rds-subnet-gp"
  description = "The subnet group for ${var.environment}-${var.service}-rds"
  subnet_ids  = [var.private_a_datastore_subnet_id, var.private_c_datastore_subnet_id]

  lifecycle {
    create_before_destroy = true
  }
```

**＊実装例＊**

例として，Redisのパラメータグループとサブネットグループを示す．ラメータグループとサブネットグループは，RDSに関連付いており，新しいパラメータグループとサブネットグループに関連付け直した後に，既存のものを削除する必要がある．

```hcl
###############################################
# Redis Parameter Group
###############################################
resource "aws_elasticache_parameter_group" "redis" {
  name        = "${var.environment}-${var.service}-redis-v5-param-gp"
  description = "The parameter group for ${var.environment}-${var.service}-redis 5.0"
  family      = "redis5.0"

  lifecycle {
    create_before_destroy = true
  }
}

###############################################
# Redis Subnet Group
###############################################
resource "aws_elasticache_subnet_group" "redis" {
  name        = "${var.environment}-${var.service}-redis-subnet-gp"
  description = "The redis subnet group for ${var.environment}-${var.service}-rds"
  subnet_ids  = [var.private_a_app_subnet_id, var.private_c_app_subnet_id]

  lifecycle {
    create_before_destroy = true
  }
}
```

#### ・ignore_changes

リモートのみで起こったリソースの構築・更新・削除を無視し，```hclstate```ファイルに反映しないようにする．基本的に使用することはないが，リモート側のリソースが動的に変更される可能性があるリソースでは，設定が必要である．

**＊実装例＊**

例として，ECSを示す．ECSでは，AutoScalingによってタスク数が増減し，またアプリケーションのデプロイでリビジョン番号が増加する．そのため，これらを無視する必要がある．

```hcl
###############################################
# ECS Service
###############################################
resource "aws_ecs_service" "this" {
  name                               = "${var.environment}-${var.service}-ecs-service"
  cluster                            = aws_ecs_cluster.this.id
  launch_type                        = "Fargate"
  platform_version                   = "1.4.0"
  task_definition                    = "${aws_ecs_task_definition.this.family}:${max(aws_ecs_task_definition.this.revision, data.aws_ecs_task_definition.this.revision)}"
  desired_count                      = var.ecs_service_desired_count
  deployment_maximum_percent         = 200
  deployment_minimum_healthy_percent = 100
  health_check_grace_period_seconds  = 300

  network_configuration {
    security_groups  = [var.aws_security_group_ecs_id]
    subnets          = var.subnet_private_app_ids
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = var.aws_lb_target_group_arn
    container_name   = "${var.environment}-${var.service}-nginx"
    container_port   = var.ecs_container_nginx_port_http
  }

  lifecycle {
    ignore_changes = [
      # AutoScalingによるタスク数の増減を無視．
      desired_count,
      # アプリケーションのデプロイによるリビジョン番号の増加を無視．
      task_definition,
    ]
  }
}
```

**＊実装例＊**

例として，Redisを示す．Redisでは，AutoScalingによってプライマリ数とレプリカ数が増減する．そのため，これらを無視する必要がある．


```hcl
###############################################
# Redis Cluster
###############################################
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id          = "${var.environment}-${var.service}-redis-cluster"
  replication_group_description = "The cluster of ${var.environment}-${var.service}-redis"
  engine_version                = "5.0.6"
  port                          = var.redis_port_ssm_parameter_value
  parameter_group_name          = aws_elasticache_parameter_group.redis.name
  node_type                     = var.redis_node_type
  number_cache_clusters         = 2
  availability_zones            = ["${var.region}${var.vpc_availability_zones.a}", "${var.region}${var.vpc_availability_zones.c}"]
  subnet_group_name             = aws_elasticache_subnet_group.redis.id
  security_group_ids            = [var.redis_security_group_id]
  automatic_failover_enabled    = true
  maintenance_window            = "sun:17:00-sun:18:00"
  snapshot_retention_limit      = 0
  snapshot_window               = "19:00-20:00"
  apply_immediately             = true

  lifecycle {
    ignore_changes = [
      # プライマリ数とレプリカ数の増減を無視します．
      number_cache_clusters
    ]
  }
}
}
```

**＊実装例＊**

使用例はすくないが，ちなみにリソース全体を無視する場合は```all```を設定する．

```hcl
resource "aws_example" "example" {

  // 何らかの設定

  lifecycle {
    ignore_changes = all
  }
}
```

<br>

## 06. tpl形式の切り出しと読み出し

### templatefile関数

#### ・templatefile関数とは

第一引数でポリシーが定義されたファイルを読み出し，第二引数でファイルに変数を渡す．ファイルの拡張子はtplとするのがよい．

**＊実装例＊**

例として，S3を示す．

```hcl
###############################################
# S3 bucket policy
###############################################
resource "aws_s3_bucket_policy" "alb" {
  bucket = aws_s3_bucket.alb_logs.id
  policy = templatefile(
    "${path.module}/policies/alb_bucket_policy.tpl",
    {
      aws_elb_service_account_arn = var.aws_elb_service_account_arn
      aws_s3_bucket_alb_logs_arn  = aws_s3_bucket.alb_logs.arn
    }
  )
}
```

バケットポリシーを定義するtpl形式ファイルでは，string型で出力する場合は```"${}"```で，int型で出力する場合は```${}```で出力する．ここで拡張子をjsonにしてしまうと，int型の出力をjsonの構文エラーとして扱われてしまう．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "${aws_elb_service_account_arn}/*"
      },
      "Action": "s3:PutObject",
      "Resource": "${aws_s3_bucket_alb_logs_arn}/*"
    }
  ]
}
```

<br>

### ポリシーのアタッチ

<br>

### containerDefinitionsの設定

#### ・containerDefinitionsとは

タスク定義のうち，コンテナを定義する部分のこと．

**＊実装例＊**

```json
{
  "ipcMode": null,
  "executionRoleArn": "<ecsTaskExecutionRoleのARN>",
  "containerDefinitions": [
    
  ],

   ~ ~ ~ その他の設定 ~ ~ ~

}
```

#### ・設定方法

**＊実装例＊**

例として，SSMのパラメータストアの値を参照できるように，```secrets```を設定している．int型を変数として渡せるように，拡張子をjsonではなくtplとするのが良い．

```hcl
###############################################
# ECS Task Definition
###############################################
resource "aws_ecs_task_definition" "this" {
  family                   = "${var.environment}-${var.service}-ecs-task-definition"
  task_role_arn            = var.ecs_task_iam_role_arn
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  execution_role_arn       = var.ecs_task_execution_iam_role_arn
  memory                   = var.ecs_task_memory
  cpu                      = var.ecs_task_cpu
  container_definitions = templatefile(
    "${path.module}/container_definitions.tpl",
    {
      environment                                     = var.environment
      region                                          = var.region
      service                                         = var.service
      ecs_container_laravel_cloudwatch_log_group_name = var.ecs_container_laravel_cloudwatch_log_group_name
      ecs_container_nginx_cloudwatch_log_group_name   = var.ecs_container_nginx_cloudwatch_log_group_name
      laravel_ecr_repository_url                      = var.laravel_ecr_repository_url
      nginx_ecr_repository_url                        = var.nginx_ecr_repository_url
      ecs_container_laravel_port_http                 = var.ecs_container_laravel_port_http
      ecs_container_nginx_port_http                   = var.ecs_container_nginx_port_http
    }
  )
}
```

ログ分割の目印を設定する```awslogs-datetime-format```キーでは，タイムスタンプを表す```\\[%Y-%m-%d %H:%M:%S\\]```を設定すると良い．

```json
[
  {
    "name": "<コンテナ名>",
    "image": "<ECRリポジトリのURL>",
    "essential": true,
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 80,
        "protocol": "tcp"
      }
    ],
    "secrets": [
      {
        "name": "<アプリケーションの環境変数名>",
        "valueFrom": "<SSMのパラメータ名>"
      },
      {
        "name": "DB_HOST",
        "valueFrom": "/ecs/DB_HOST"
      },
      {
        "name": "DB_DATABASE",
        "valueFrom": "/ecs/DB_DATABASE"
      },
      {
        "name": "DB_PASSWORD",
        "valueFrom": "/ecs/DB_PASSWORD"
      },
      {
        "name": "DB_USERNAME",
        "valueFrom": "/ecs/DB_USERNAME"
      },
      {
        "name": "REDIS_HOST",
        "valueFrom": "/ecs/REDIS_HOST"
      },
      {
        "name": "REDIS_PASSWORD",
        "valueFrom": "/ecs/REDIS_PASSWORD"
      },
      {
        "name": "REDIS_PORT",
        "valueFrom": "/ecs/REDIS_PORT"
      }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "<ロググループ名>",
        "awslogs-datetime-format": "\\[%Y-%m-%d %H:%M:%S\\]",
        "awslogs-region": "<リージョン>",
        "awslogs-stream-prefix": "<ログストリーム名のプレフィクス>"
      }
    }
  }
]
```

<br>

## 07. 命名規則

### 変数の命名

#### ・単数形と複数形の命名分け

複数の値をもつlist型の変数であれば複数形で命名する．一方で，string型など値が一つしかなければ単数形とする．

**＊実装例＊**

例として，VPCを示す．

```hcl
###############################################
# VPC variables
###############################################
vpc_availability_zones             = { a = "a", c = "c" }
vpc_cidr                           = "n.n.n.n/23"
vpc_subnet_private_datastore_cidrs = { a = "n.n.n.n/27", c = "n.n.n.n/27" }
vpc_subnet_private_app_cidrs       = { a = "n.n.n.n/25", c = "n.n.n.n/25" }
vpc_subnet_public_cidrs            = { a = "n.n.n.n/27", c = "n.n.n.n/27" }
```

<br>

### リソースとデータリソースの命名

#### ・リソース名で種類を表現

リソース名において，リソースタイプを繰り返さないようにする．もし種類がある場合，リソース名でその種類を表現する．

**＊実装例＊**

例として，VPCを示す．

```hcl
###############################################
# VPC route table
###############################################

# 良い例
resource "aws_route_table" "public" {

}

resource "aws_route_table" "private" {

}
```

```hcl
###############################################
# VPC route table
###############################################

# 悪い例
resource "aws_route_table" "route_table_public" {

}

resource "aws_route_table" "route_table_private" {

}
```

#### ・this

一つのリソースタイプに，一つのリソースしか種類が存在しない場合，```this```で命名する．

**＊実装例＊**

```hcl
resource "aws_internet_gateway" "this" {

}
```

#### ・AWSリソース名

1. `<接頭辞>-<種類>-<接尾辞>`とする．
2. 接頭辞は， `<稼働環境>-<サービス名>`とする．
3. 接尾辞は，AWSリソース名とする．

**＊実装例＊**

例として，CloudWatchを示す．この時，他のresourceと比較して，種類はALBのHTTPCode_TARGET_4XX_Countメトリクスに関するアラームと見なせる．そのため，`alb_httpcode_4xx_count`と名付けている．

```hcl
resource "aws_cloudwatch_metric_alarm" "alb_httpcode_target_4xx_count" {

  alarm_name = "${var.environment}-${var.service}-alb-httpcode-target-4xx-count-alarm"
  
}
```

#### ・設定の順序，行間

最初に`count`や`for_each`を設定し改行する．その後，各リソース別の設定を行間を空けずに記述する（この順番にルールはなし）．最後に共通の設定として，`tags`，`depends_on`，`lifecycle`，の順で配置する．ただし実際，これらの全ての設定が必要なリソースはない．

**＊実装例＊**

```hcl
###############################################
# EXAMPLE
###############################################
resource "aws_example" "this" {
  // 最初にfor_each
  for_each = var.vpc_availability_zones

  // 各設定
  subnet_id = aws_subnet.public[*].id

  tags = {
    Name = format(
      "${var.environment}-${var.service}-%d-example",
      each.value
    )
  }
  
  depends_on = []

  lifecycle {
    create_before_destroy = true
  }
}
```

<br>

### アウトプット値の命名

#### ・基本ルール

アウトプット値の名前は，```<リソース名>_<リソースタイプ>_<attribute名>```で命名する．

**＊実装例＊**

例として，CloudWatchを示す．リソース名は`ecs_container_nginx`，リソースタイプは`aws_cloudwatch_log_group`，attributeは`name`オプションである．

```hcl
output "ecs_container_nginx_cloudwatch_log_group_name" {
  value = aws_cloudwatch_log_group.ecs_container_nginx.name
}
```



**＊実装例＊**

例として，IAM Roleを示す．

```hcl
###############################################
# Output IAM Role
###############################################
output "ecs_task_execution_iam_role_arn" {
  value = aws_iam_role.ecs_task_execution.arn
}

output "lambda_execute_iam_role_arn" {
  value = aws_iam_role.lambda_execute.arn
}

output "rds_enhanced_monitoring_iam_role_arn" {
  value = aws_iam_role.rds_enhanced_monitoring.arn
}
```

#### ・list型アウトプット値は複数形

countでループで構築したリソースは，list型でアウトプットすることができる．この時，アウトプットの変数名は複数形にする．ちなみに，for_eachで構築したリソースはアスタリスクでインデックス名を指定できないので，注意．

**＊実装例＊**

例として，VPCを示す．

```hcl
###############################################
# Output VPC
###############################################
output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_app_subnetids" {
  value = aws_subnet.private_app[*].id
}

output "private_datastore_subnet_ids" {
  value = aws_subnet.private_datastore[*].id
}
```

#### ・thisは省略

リソース名が```this```である場合，アウトプット値名ではこれを省略してもよい．

**＊実装例＊**

例として，ALBを示す．

```hcl
###############################################
# Output ALB
###############################################
output "alb_zone_id" {
  value = aws_lb.this.zone_id
}

output "alb_dns_name" {
  value = aws_lb.this.dns_name
}
```

#### ・冗長なattribute名は省略

**＊実装例＊**

例として，ECRを示す．

```hcl
###############################################
# Output ECR
###############################################
output "laravel_ecr_repository_url" {
  value = aws_ecr_repository.laravel.repository_url
}

output "nginx_ecr_repository_url" {
  value = aws_ecr_repository.nginx.repository_url
}
```

<br>

## 08. 各リソースタイプ独自の仕様

### API Gateway

#### ・OpenAPI仕様のインポートと差分認識

あらかじめ用意したOpenAPI仕様のYAMLファイルを```body```オプションのパラメータとし，これをインポートすることにより，APIを定義できる．YAMLファイルに変数を渡すこともできる．

```hcl
###############################################
# REST API
###############################################
resource "aws_api_gateway_rest_api" "example" {
  name        = "${var.environment}-${var.service}-api-for-example"
  description = "The API that enables two-way communication with ${var.environment}-example"
  # VPCリンクのプロキシ統合のAPI
  body = templatefile(
    "${path.module}/open_api.yaml",
    {
      api_gateway_vpc_link_example_id = aws_api_gateway_vpc_link.example.id
      nlb_dns_name                          = var.nlb_dns_name
    }
  )

  endpoint_configuration {
    types = ["REGIONAL"]
  }

  lifecycle {
    ignore_changes = [
      policy
    ]
  }
}
```

APIの再デプロイのトリガーとして，```redeployment```パラメータに```body```パラメータのハッシュ値を渡すようにする．これにより，インポート元のYAMLファイルに差分があった場合に，Terraformが```redeployment```パラメータの値の変化を認識できるようになり，再デプロイを実行できる．

```hcl
###############################################
# Deployment
###############################################
resource "aws_api_gateway_deployment" "example" {
  rest_api_id = aws_api_gateway_rest_api.example.id

  triggers = {
    redeployment = sha1(aws_api_gateway_rest_api.example.body)
  }

  lifecycle {
    create_before_destroy = true
  }
}

###############################################
# Stage
###############################################
resource "aws_api_gateway_stage" "example" {
  deployment_id = aws_api_gateway_deployment.example.id
  rest_api_id   = aws_api_gateway_rest_api.example.id
  stage_name    = var.environment
}
```

<br>

### AMI

#### ・取得するAMIのバージョンを固定

取得するAMIのバージョンを常に最新にしておく，EC2が再構築されなねない．そこで，特定のAMIを取得できるようにしておく．

```hcl
###############################################
# For bastion
###############################################
data "aws_ami" "bastion" {
  # EC2が，意図せず再構築されないように，特定のAMIを取得します．
  most_recent = false
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn-ami-hvm-2018.03.0.20201028.0-x86_64-gp2"]
  }

  filter {
    name   = "image-id"
    values = ["ami-040c9333a9c90b2b6"]
  }
}
```

<br>

### CloudFront

#### ・全体の実装例

```hcl
resource "aws_cloudfront_distribution" "this" {

  price_class      = "PriceClass_200"
  web_acl_id       = var.cloudfront_wafv2_web_acl_arn
  aliases          = [var.route53_domain_example]
  comment          = "${var.environment}-${var.service}-cf-distribution"
  enabled          = true
  retain_on_delete = true

  viewer_certificate {
    acm_certificate_arn      = var.example_acm_certificate_arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2019"
  }

  logging_config {
    bucket          = var.cloudfront_s3_bucket_regional_domain_name
    include_cookies = true
  }

  restrictions {

    geo_restriction {
      restriction_type = "none"
    }
  }

  # S3をオリジンに設定します．
  origin {
    domain_name = var.s3_bucket_regional_domain_name
    origin_id   = "S3-${var.s3_bucket_id}"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.s3_example.cloudfront_access_identity_path
    }
  }

  # ALBをオリジンに設定します．
  origin {
    domain_name = var.alb_dns_name
    origin_id   = "ELB-${var.alb_name}"

    custom_origin_config {
      origin_ssl_protocols     = ["TLSv1.2"]
      origin_protocol_policy   = "match-viewer"
      origin_read_timeout      = 30
      origin_keepalive_timeout = 5
      http_port                = var.alb_listener_port_http
      https_port               = var.alb_listener_port_https
    }
  }

  ordered_cache_behavior {
    path_pattern           = "/images/*"
    target_origin_id       = "S3-${var.s3_bucket_id}"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods         = ["GET", "HEAD"]
    min_ttl                = 0
    max_ttl                = 31536000
    default_ttl            = 86400
    compress               = true

    forwarded_values {
      query_string = true

      cookies {
        forward = "none"
      }
    }
  }

  default_cache_behavior {
    target_origin_id       = "ELB-${var.alb_name}"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods         = ["GET", "HEAD"]
    min_ttl                = 0
    max_ttl                = 31536000
    default_ttl            = 86400
    compress               = true

    forwarded_values {
      query_string = true
      headers      = ["*"]

      cookies {
        forward = "all"
      }
    }
  }
}
```

#### ・削除保持機能

Terraformでは，```retain_on_delete```で設定できる．固有の設定で，AWSに対応するものは無い．

<br>

### ECR

#### ・ライフサイクルポリシー

ECRにアタッチされる，イメージの有効期間を定義するポリシー．コンソール画面から入力できるため，基本的にポリシーの実装は不要であるが，TerraformなどのIaCツールでは必要になる．

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images untagged",
      "selection": {
        "tagStatus": "untagged",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    },
    {
      "rulePriority": 2,
      "description": "Keep last 10 images any",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

<br>

### ECS

#### ・リモートのリビジョン番号の追跡

```hcl
###############################################
# ECS Service
###############################################
resource "aws_ecs_service" "this" {
  name                               = "${var.environment}-${var.service}-ecs-service"
  cluster                            = aws_ecs_cluster.this.id
  launch_type                        = "FARGATE"
  platform_version                   = "1.4.0"
  desired_count                      = var.ecs_service_desired_count
  deployment_maximum_percent         = 200
  deployment_minimum_healthy_percent = 100
  health_check_grace_period_seconds  = 300

  # アプリケーションのデプロイによって，リモートのタスク定義のリビジョン番号が増加するため，これを追跡できるようにします．
  task_definition = "${aws_ecs_task_definition.this.family}:${max(aws_ecs_task_definition.this.revision, data.aws_ecs_task_definition.this.revision)}"

  network_configuration {
    security_groups  = [var.ecs_security_group_id]
    subnets          = [var.private_a_app_subnet_id, var.private_c_app_subnet_id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = var.alb_target_group_arn
    container_name   = "nginx"
    container_port   = var.ecs_container_nginx_port_http
  }

  lifecycle {
    ignore_changes = [
      desired_count,
      task_definition,
    ]
  }
}
```

#### ・タスク定義の更新

Terraformでタスク定義を更新すると，現在動いているECSで稼働しているタスクはそのままに，新しいリビジョン番号のタスク定義が作成される．コンソール画面の「新しいリビジョンの作成」と同じ挙動である．実際にタスクが増えていることは，サービスに紐づくタスク定義一覧から確認できる．次のデプロイ時に，このタスクが用いられる．

<br>

### EC2

#### ・全体の実装例

```hcl
###############################################
# For bastion
###############################################
resource "aws_instance" "bastion" {
  ami                         = var.bastion_ami_amazon_id
  instance_type               = "t2.micro"
  vpc_security_group_ids      = [var.ec2_bastion_security_group_id]
  subnet_id                   = var.public_a_subnet_id
  key_name                    = "${var.environment}-${var.service}-bastion"
  associate_public_ip_address = true
  disable_api_termination     = true

  tags = {
    Name  = "${var.environment}-${var.service}-bastion"
  }

  depends_on = [var.internet_gateway]
}

```

#### ・キーペアはコンソール上で設定

誤って削除しないように，またソースコードに機密情報をハードコーディングしないように，キーペアはコンソール画面で作成した後，```key_name```でキー名を指定するようにする．

<br>

### IAMユーザ

#### ・カスタマー管理ポリシーを持つロール

事前に，tpl形式のカスタマー管理ポリシーを定義しておく．構築済みのIAMロールに，```aws_iam_policy```リソースを使用して，AWS管理ポリシーをIAMユーザにアタッチする．

**＊実装例＊**

ローカルからAWS CLIコマンドを実行する必要がある場合に，コマンドを特定の送信元IPアドレスを特定のものに限定する．事前に，list型でIPアドレスを定義する．

```hcl
###############################################
# IP addresses
###############################################
global_ip_addresses = [
  "nn.nnn.nnn.nnn/32",
  "nn.nnn.nnn.nnn/32"
]
```

また事前に，指定した送信元IPアドレス以外を拒否するカスタマー管理ポリシーを定義する．

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "NotIpAddress": {
        "aws:SourceIp": ${global_ip_addresses}
      }
    }
  }
}
```


コンソール画面で作成済みのIAMユーザの名前を取得する．tpl形式のポリシーにlist型の値を渡す時，```jsonencode```関数を使用する必要がある．

```hcl
###############################################
# For IAM User
###############################################
data "aws_iam_user" "aws_cli_command_executor" {
  user_name = "aws_cli_command_executor"
}

resource "aws_iam_policy" "aws_cli_command_executor_ip_address_restriction" {
  name        = "${var.environment}-aws-cli-command-executor-ip-address-restriction-policy"
  description = "Allow global IP addresses"
  policy = templatefile(
    "${path.module}/policies/customer_managed_policies/aws_cli_command_executor_ip_address_restriction_policy.tpl",
    {
      global_ip_addresses = jsonencode(var.global_ip_addresses)
    }
  )
}
```

#### ・AWS管理ポリシー

IAMユーザにAWS管理ポリシーをアタッチする．

**＊実装例＊**

```hcl
###############################################
# For IAM User
###############################################
resource "aws_iam_user_policy_attachment" "aws_cli_command_executor_s3_read_only_access" {
  user       = data.aws_iam_user.aws_cli_command_executor.user_name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}
```

<br>

### IAMロール

#### ・信頼ポリシーを持つロール

コンソール画面でロールを作成する場合は意識することはないが，特定のリソースにロールをアタッチするためには，ロールに信頼ポリシーを組み込む必要がある．事前に，tpl形式の信頼ポリシーを定義しておく．```aws_iam_role```リソースを使用して，IAMロールを構築すると同時に，これに信頼ポリシーをアタッチする．

**＊実装例＊**

事前に，ECSタスクのための信頼ポリシーを定義する．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

ECSタスクロールとECSタスク実行ロールに信頼ポリシーアタッチする．

```hcl
###############################################
# IAM Role For ECS Task Execution
###############################################
resource "aws_iam_role" "ecs_task_execution" {
  name        = "${var.environment}-${var.service}-ecs-task-execution-role"
  description = "The role for ${var.environment}-${var.service}-ecs-task"
  assume_role_policy = templatefile(
    "${path.module}/policies/trust_policies/ecs_task_policy.tpl",
    {}
  )
}

###############################################
# IAM Role For ECS Task
###############################################
resource "aws_iam_role" "ecs_task" {
  name        = "${var.environment}-${var.service}-ecs-task-role"
  description = "The role for ${var.environment}-${var.service}-ecs-task"
  assume_role_policy = templatefile(
    "${path.module}/policies/trust_policies/ecs_task_policy.tpl",
    {}
  )
}
```

**＊実装例＊**

事前に，Lambda@Edgeのための信頼ポリシーを定義する．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "lambda.amazonaws.com",
          "edgelambda.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Lambda実行ロールに信頼ポリシーアタッチする．

```hcl
###############################################
# IAM Role For Lambda@Edge
###############################################

# ロールに信頼ポリシーをアタッチします．
resource "aws_iam_role" "lambda_execute" {
  name = "${var.environment}-${var.service}-lambda-execute-role"
  assume_role_policy = templatefile(
    "${path.module}/policies/lambda_execute_role_trust_policy.tpl",
    {}
  )
}
```

#### ・インラインポリシーを持つロール

事前に，tpl形式のインラインポリシーを定義しておく．```aws_iam_role_policy```リソースを使用して，インラインポリシーを構築すると同時に，これにインラインポリシーをアタッチする．

**＊実装例＊**

事前に，ECSタスクに必要最低限の権限を与えるインラインポリシーを定義する．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameters"
      ],
      "Resource": "*"
    }
  ]
}
```

ECSタスクロールとECSタスク実行ロールにインラインポリシーアタッチする．

```hcl
###############################################
# IAM Role For ECS Task
###############################################
resource "aws_iam_role_policy" "ecs_task" {
  name = "${var.environment}-${var.service}-ssm-read-only-access-policy"
  role = aws_iam_role.ecs_task_execution.id
  policy = templatefile(
    "${path.module}/policies/inline_policies/ecs_task_policy.tpl",
    {}
  )
}
```

#### ・AWS管理ポリシーを持つロール

事前に，tpl形式のAWS管理ポリシーを定義しておく．```aws_iam_role_policy_attachment```リソースを使用して，リモートにあるAWS管理ポリシーを構築済みのIAMロールにアタッチする．ポリシーのARNは，AWSのコンソール画面を確認する．

**＊実装例＊**

```hcl
###############################################
# IAM Role For ECS Task Execution
###############################################
resource "aws_iam_role_policy_attachment" "ecs_task_execution" {
  role       = aws_iam_role.ecs_task_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}
```

#### ・カスタマー管理ポリシーを持つロール

事前に，tpl形式のインラインポリシーを定義しておく．```aws_iam_role_policy```リソースを使用して，カスタマー管理ポリシーを構築する．```aws_iam_role_policy_attachment```リソースを使用して，カスタマー管理ポリシーを構築済みのIAMロールにアタッチする．

**＊実装例＊**

事前に，ECSタスクに必要最低限の権限を与えるカスタマー管理ポリシーを定義する．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": [
        "arn:aws:logs:*:*:*"
      ]
    }
  ]
}
```

ECSタスクロールにカスタマー管理ポリシーアタッチする．

```hcl
###############################################
# IAM Role For ECS Task
###############################################
resource "aws_iam_policy" "ecs_task" {
  name        = "${var.environment}-${var.service}-cloudwatch-logs-access-policy"
  description = "Provides access to CloudWatch Logs"
  policy = templatefile(
    "${path.module}/policies/customer_managed_policies/cloudwatch_logs_access_policy.tpl",
    {}
  )
}

resource "aws_iam_role_policy_attachment" "ecs_task" {
  role       = aws_iam_role.ecs_task.name
  policy_arn = aws_iam_policy.ecs_task.arn
}
```

#### ・サービスリンクロール

サービスリンクロールは，AWSリソースの構築時に自動的に作成され，アタッチされる．そのため，Terraformの管理外である．```aws_iam_service_linked_role```リソースを使用して，手動で構築することが可能であるが，数が多く実装の負担にもなるため，あえて管理外としても問題ない．

**＊実装例＊**

サービス名を指定して，Application Auto Scalingのサービスリンクロールを構築する．

```hcl
###############################################
# IAM Role For ECS Service
###############################################
# Service Linked Role
resource "aws_iam_service_linked_role" "ecs_service_auto_scaling" {
  aws_service_name = "ecs.application-autoscaling.amazonaws.com"
}
```

```hcl
###############################################
# Output IAM Role
###############################################
output "ecs_service_auto_scaling_iam_service_linked_role_arn" {
  value = aws_iam_service_linked_role.ecs_service_auto_scaling.arn
}
```

Application Auto Scalingにサービスリンクロールをアタッチする．

```hcl
#########################################
# Application Auto Scaling For ECS
#########################################
resource "aws_appautoscaling_target" "ecs" {
  service_namespace  = "ecs"
  resource_id        = "service/${var.ecs_cluster_name}/${var.ecs_service_name}"
  scalable_dimension = "ecs:service:DesiredCount"
  max_capacity       = var.auto_scaling_ecs_task_max_capacity
  min_capacity       = var.auto_scaling_ecs_task_min_capacity
  # この設定がなくとも，サービスリンクロールが自動的に構築され，AutoScalingにアタッチされる．
  role_arn           = var.ecs_service_auto_scaling_iam_service_linked_role_arn
}
```

<br>

### Network Interface

#### ・Network Interfaceをデタッチできない

Network Interfaceは特定のリソースの構築時に，自動で構築されるため，Terraformの管理外にある．また，このリソースを削除しない限り，デタッチできない．Network Interfaceをデタッチできないと，セキュリティグループを削除できないため，Terraformは永遠にリクエストを繰り返すことになる．

| 関連付くリソース            | 備考                                          |
| --------------------------- | --------------------------------------------- |
| GlobalAccelerator           |                                               |
| EC2                         | EC2のパブリックIPアドレスを決定する．         |
| ECSタスク定義（Active状態） |                                               |
| ALB                         | ALBのパブリックIPアドレスを決定する．         |
| NAT Gateway                 | NAT GatewayのパブリックIPアドレスを決定する． |
| RDS                         |                                               |
| VPC Endpoint                |                                               |

<br>

### Route53

#### ・全体の実装例

```hcl
###############################################
# For api domain
###############################################
resource "aws_route53_zone" "example" {
  name = var.route53_domain_example
}

resource "aws_route53_record" "example" {
  zone_id = aws_route53_zone.example.id
  name    = var.route53_domain_example
  type    = "A"

  alias {
    name                   = var.alb_dns_name
    zone_id                = var.alb_zone_id
    evaluate_target_health = true
  }
}
```

#### ・ネームサーバレコードは管理外

ホストゾーンを作成すると，レコードとして，ネームサーバの情報が自動的に設定される．これは，Terraformの管理外である．

<br>

### RDS

#### ・全体の実装例

```hcl
#########################################
# RDS Cluster
#########################################
resource "aws_rds_cluster" "rds_cluster" {
  engine                          = "aurora-mysql"
  engine_version                  = "5.7.mysql_aurora.2.08.3"
  cluster_identifier              = "${var.environment}-${var.service}-rds-cluster"
  master_username                 = var.rds_db_username_ssm_parameter_value
  master_password                 = var.rds_db_password_ssm_parameter_value
  availability_zones              = ["${var.region}${var.vpc_availability_zones.a}", "${var.region}${var.vpc_availability_zones.c}"]
  vpc_security_group_ids          = [var.rds_security_group_id]
  db_subnet_group_name            = aws_db_subnet_group.this.name
  port                            = var.rds_db_port_ssm_parameter_value
  database_name                   = var.rds_db_name_ssm_parameter_value
  db_cluster_parameter_group_name = aws_rds_cluster_parameter_group.this.id
  storage_encrypted               = true
  backup_retention_period         = 7
  preferred_backup_window         = "19:00-19:30"
  copy_tags_to_snapshot           = true
  final_snapshot_identifier       = "final-db-snapshot"
  skip_final_snapshot             = false
  enabled_cloudwatch_logs_exports = ["audit", "error", "general", "slowquery"]
  preferred_maintenance_window    = "sun:17:30-sun:18:00"
  apply_immediately               = true
  deletion_protection             = true

  lifecycle {
    ignore_changes = [
      availability_zones
    ]
  }
}
```

```hcl
###############################################
# RDS Cluster Instance
###############################################
resource "aws_rds_cluster_instance" "this" {
  for_each = var.vpc_availability_zones

  engine                       = "aurora-mysql"
  engine_version               = "5.7.mysql_aurora.2.08.3"
  identifier                   = "${var.environment}-${var.service}-rds-instance-${each.key}"
  cluster_identifier           = aws_rds_cluster.rds_cluster.id
  instance_class               = var.rds_instance_class
  db_subnet_group_name         = aws_db_subnet_group.this.id
  db_parameter_group_name      = aws_db_parameter_group.this.id
  monitoring_interval          = 60
  monitoring_role_arn          = var.rds_iam_role_arn
  auto_minor_version_upgrade   = false
  preferred_maintenance_window = "sun:17:00-sun:17:30"
  apply_immediately            = true
}
```

#### ・クラスターにはAZが３つ必要

クラスターでは，レプリケーションのために，３つのAZが必要である．そのため，指定したAZが２つであっても，３つのAZが設定される．```ignore_changes```でAZを指定しておく必要がある．

https://github.com/hashicorp/terraform-provider-aws/issues/7307#issuecomment-457441633

#### ・インスタンスを配置するAZは選べない

事前にインスタンスにAZを表す識別子を入れたとしても，Terraformはインスタンスを配置するAZを選べない．そのため，AZと識別子の関係が逆になってしまうことがある．その場合は，デプロイ後に手動で名前を変更すればよい．この変更は，Terraformが差分として認識しないので問題ない．

#### ・インスタンスにバックアップウインドウは設定しない

クラスターとインスタンスの両方に，```preferred_backup_window```を設定できるが，RDSインスタンスに設定してはいけない．

<br>

### S3

#### ・バケットポリシー

S3アタッチされる，自身へのアクセスを制御するためにインラインポリシーのこと．詳しくは，AWSのノートを参照せよ．定義したバケットポリシーは，```aws_s3_bucket_policy```でロールにアタッチできる．

#### ・ALBアクセスログ

ALBがバケットにログを書き込めるように，『ELBのサービスアカウントID』を許可する必要がある．

**＊実装例＊**

```hcl
###############################################
# S3 bucket policy
###############################################

# S3にバケットポリシーをアタッチします．
resource "aws_s3_bucket_policy" "alb" {
  bucket = aws_s3_bucket.alb_logs.id
  policy = templatefile(
    "${path.module}/policies/alb_bucket_policy.tpl",
    {}
  )
}
```

ALBのアクセスログを送信するバケット内には，自動的に『/AWSLogs/<アカウントID>』の名前でディレクトリが生成される．そのため，『```arn:aws:s3:::<バケット名>/*```』の部分を最小権限として，『```arn:aws:s3:::<バケット名>/AWSLogs/<アカウントID>/;*```』にしてもよい．東京リージョンのELBサービスアカウントIDは，『582318560864』である．

参考：https://docs.aws.amazon.com/ja_jp/elasticloadbalancing/latest/application/load-balancer-access-logs.html#access-logging-bucket-permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::582318560864:root"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::<バケット名>/*"
    }
  ]
}
```

#### ・NLBアクセスログ

ALBがバケットにログを書き込めるように，『```delivery.logs.amazonaws.com```』からのアクセスを許可する必要がある．

**＊実装例＊**

```hcl
###############################################
# S3 bucket policy
###############################################

# S3にバケットポリシーをアタッチします．
resource "aws_s3_bucket_policy" "nlb" {
  bucket = aws_s3_bucket.nlb_logs.id
  policy = templatefile(
    "${path.module}/policies/nlb_bucket_policy.tpl",
    {}
  )
}
```

NLBのアクセスログを送信するバケット内には，自動的に『/AWSLogs/<アカウントID>』の名前でディレクトリが生成される．そのため，『```arn:aws:s3:::<バケット名>/*```』の部分を最小権限として，『```arn:aws:s3:::<バケット名>/AWSLogs/<アカウントID>/;*```』にしてもよい．

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSLogDeliveryWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "delivery.logs.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::<バケット名>/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    },
    {
      "Sid": "AWSLogDeliveryAclCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": "delivery.logs.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::<バケット名>"
    }
  ]
}
```

<br>

### VPC  ルートテーブル

#### ・全体の実装例

```hcl
###############################################
# Route table (public)
###############################################
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this.id
  }

  tags = {
    Name = "${var.environment}-${var.service}-pub-rtb"
  }
}

###############################################
# Route table (private)
###############################################
resource "aws_route_table" "private_app" {
  for_each = var.vpc_availability_zones

  vpc_id = aws_vpc.this.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.this[each.key].id
  }

  tags = {
    Name = format(
      "${var.environment}-${var.service}-pvt-%s-app-rtb",
      each.value
    )
  }
}
```

#### ・メインルートテーブルは自動構築

Terraformを用いてVPCを構築した時，メインルートテーブルが自動的に構築される．そのため，これはTerraformの管理外である．

<br>

### WAF

#### ・IPセットの依存関係

WAFのIPセットと他設定の依存関係に癖がある．新しいIPセットへの付け換えと古いIPセットの削除を同時にデプロイしないようにする．もし同時に行った場合，Terraformは削除処理を先に実行するが，WAFは古いIPセットに紐づいているため，ここで異常が起こってしまう．ちなみに，IPセットの名前変更は，削除を含む再構築処理が実行されるため注意する．そのため，IPセットを新しく設定し直す場合は，以下の通り二つの段階に分けてデプロイするようにする．

1. 新しいIPセットのresourceを実装し，ACLに関連付け，デプロイする．
2. 古いIPセットのresourceを削除し，デプロイする．

もし，これを忘れてしまった場合は，画面上で適当なIPセットに付け換えて，削除処理を実行できるようにする．

<br>

### 共通の設定

#### ・Terraform管理外のAWSリソース

以下のAWSリソースはTerraformで管理しない方が便利である．Terraformの管理外のリソースには，コンソール画面上から，「```Not managed by = Terraform```」というタグをつけた方が良い．

| 種類                                     | 管理外の理由                                                 |
| ---------------------------------------- | ------------------------------------------------------------ |
| SSMパラメータストア                      |                                                              |
| データベースのadmin以外のユーザ          |                                                              |
| Chatbot                                  |                                                              |
| EC2の秘密鍵                              |                                                              |
| Global Acceleratorのセキュリティグループ |                                                              |
| S3のtfstate-bucket                       | リソースを構築するとセキュリティグループが自動生成されるため，セキュリティグループのみTerraformで管理できない． |

#### ・削除保護機能のあるAWSリソース

削除保護設定のあるAWSリソースに癖がある．削除保護の無効化とリソースを削除を同時にデプロイしないようにする．もし同時に行った場合，削除処理を先に実行するため，エラーになる．そのため，このAWSリソースを削除する時は，以下の通り二つの段階に分けてデプロイするようにする．

1. 削除保護を無効化（`false`）に変更し，デプロイする．
2. ソースコードを削除し，デプロイする．

もし，これを忘れてしまった場合は，画面上で削除処理を無効化し，削除処理を実行できるようにする．

| AWSリソース名 | Terraform上での設定名            |
| ------------- | -------------------------------- |
| ALB           | ```enable_deletion_protection``` |
| EC2           | ```disable_api_termination```    |
| RDS           | ```deletion_protection```        |

<br>

## 09. CircleCIとの組み合わせ

### circleci

#### ・設定ファイル

| jobs                   |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| plan                   | aws-cliのインストールから```terraform plan -out```コマンドまでの一連の処理を実行する． |
| 承認ジョブ             |                                                              |
| apply                  | developブランチからステージング環境にデプロイ                |
| terraform_destroy_test | mainブランチから本番環境にデプロイ                           |

| workflows |                                               |
| --------- | --------------------------------------------- |
| feature   | featureブランチから開発環境にデプロイ         |
| develop   | developブランチからステージング環境にデプロイ |
| main      | mainブランチから本番環境にデプロイ            |

```yaml
version: 2.1

executors:
  primary_container:
    parameters:
      env:
        type: enum
        enum: [ "dev", "stg", "prd" ]
    docker:
      - image: hashicorp/terraform:x.xx.x
    working_directory: ~/example_infrastructure
    environment:
      ENV: << parameters.env >>

commands:
  # AWSにデプロイするための環境を構築します．
  aws_setup:
    steps:
      - run:
          name: Install jq
          command: |
            apk add curl
            curl -o /usr/bin/jq -L https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
            chmod +x /usr/bin/jq
      - run:
          name: Install aws-cli
          command: |
            apk add python3
            apk add py-pip
            pip3 install awscli
            aws --version
      - run:
          name: Assume role
          command: |
            set -x
            source ./ops/assume.sh

  # terraform initを行います．
  terraform_init:
    steps:
      - run:
          name: Terraform init
          command: |
            set -x
            source ./ops/terraform_init.sh

  # terraform fmtを行います．
  terraform_fmt:
    steps:
      - run:
          name: Terraform fmt
          command: |
            set -x
            source ./ops/terraform_fmt.sh
            
  # terraform validateを行います．
  terraform_validate:
    steps:
      - run:
          name: Terraform validate
          command: |
            set -x
            source ./ops/terraform_validate.sh

  # terraform planを行います．
  terraform_plan:
    steps:
      - run:
          name: Terraform plan
          command: |
            set -x
            source ./ops/terraform_plan.sh
            ls -la

  # terraform applyを行います．
  terraform_apply:
    steps:
      - run:
          name: Terraform apply
          command: |
            set -x
            ls -la
            source ./ops/terraform_apply.sh

  # test環境に対して，terraform destroyを行います．
  terraform_destroy_test:
    steps:
      - run:
          name: Terraform destroy test
          command: |
            set -x
            source ./ops/terraform_destroy_test.sh

jobs:
  plan:
    parameters:
      exr:
        type: executor
    executor: << parameters.exr >>
    steps:
      - checkout
      - aws_setup
      - terraform_init
      - terraform_fmt
      - terraform_validate
      - terraform_plan
      - persist_to_workspace:
          root: .
          paths:
            - .

  apply:
    parameters:
      exr:
        type: executor
    executor: << parameters.exr >>
    steps:
      - attach_workspace:
          at: .
      - terraform_apply

  destroy_test:
    parameters:
      exr:
        type: executor
    executor: << parameters.exr >>
    steps:
      - checkout
      - aws_setup
      - terraform_init
      - terraform_destroy_test

workflows:
  # Test env
  feature:
    jobs:
      - plan:
          name: plan_test
          exr:
            name: primary_container
            env: test
          filters:
            branches:
              only:
                - /feature.*/
      - apply:
          name: apply_test
          exr:
            name: primary_container
            env: test
          requires:
            - plan_test

  # Staging env
  develop:
    jobs:
      - plan:
          name: plan_stg
          exr:
            name: primary_container
            env: stg
          filters:
            branches:
              only:
                - develop
      - hold_apply:
          name: hold_apply_stg
          type: approval
          requires:
            - plan_stg
      - apply:
          name: apply_stg
          exr:
            name: primary_container
            env: stg
          requires:
            - hold_apply_stg
      - hold_destroy_test:
          type: approval
          requires:
            - apply_stg
      - destroy_test:
          exr:
            name: primary_container
            env: test
          requires:
            - hold_destroy_test

  # Production env
  main:
    jobs:
      - plan:
          name: plan_prd
          exr:
            name: primary_container
            env: prd
          filters:
            branches:
              only:
                - main
      - hold_apply:
          name: hold_apply_prd
          type: approval
          requires:
            - plan_prd
      - apply:
          name: apply_prd
          exr:
            name: primary_container
            env: prd
          requires:
            - hold_apply_prd
```

<br>

### シェルスクリプト

#### ・assume_role.sh

AWSのノートを参照せよ．

#### ・terraform_apply.sh

```shell
#!/bin/bash

set -xeuo pipefail

# credentialsの情報を出力します．
source ./aws_envs.sh

terraform apply \
  -parallelism=30 \
  ${ENV}.tfplan | ./ops/tfnotify --config ./${ENV}/tfnotify.yml apply
```

#### ・terraform_destroy_test.sh

```shell
#!/bin/bash

set -xeuo pipefail

if [ $ENV = "test" ]; then
    # credentialsの情報を出力します．
    source ./aws_envs.sh
    terraform destroy -var-file=./test/config.tfvars ./test
else
    echo "The parameter ${ENV} is invalid."
    exit 1
fi

```

#### ・terraform_fmt.sh

```shell
#!/bin/bash

set -xeuo pipefail

terraform fmt \
  -recursive \
  -check
```

#### ・terraform_init.sh

```shell
#!/bin/bash

set -xeuo pipefail

# credentialsの情報を出力します．
source ./aws_envs.sh

terraform init \
  -upgrade \
  -reconfigure \
  -backend=true \
  -backend-config="bucket=${ENV}-tfstate-bucket" \
  -backend-config="key=terraform.tfstate" \
  -backend-config="encrypt=true" \
  ./${ENV}
```

#### ・terraform_plan.sh

```shell
#!/bin/bash

set -xeuo pipefail

# credentialsの情報を出力します．
source ./aws_envs.sh

terraform plan \
  -var-file=./${ENV}/config.tfvars \
  -out=${ENV}.tfplan \
  -parallelism=30 \
  ./${ENV} | ./ops/tfnotify --config ./${ENV}/tfnotify.yml plan
```

#### ・terraform_validate.sh

```shell
#!/bin/bash

set -xeuo pipefail

terraform validate ./${ENV}

```

<br>

### tfnotify

#### ・tfnotifyとは

terraformの```plan```または```apply```の処理結果を，POSTで送信するバイナリファイルのこと．URLや送信内容を設定ファイルで定義する．

#### ・コマンド

CircleCIで利用する場合は，commandの中で，以下からダウンロードしたtfnotifyのバイナリファイルを実行する．環境別にtfnotifyを配置しておくとよい．

https://github.com/mercari/tfnotify/releases/tag/v0.7.0

```shell
#!/bin/bash

set -xeuo pipefail

terraform plan | ./bin/tfnotify --config ./${ENV}/tfnotify.yml plan
```

#### ・設定ファイル

**＊実装例＊**

例として，GitHubの特定のリポジトリのプルリクエストにPOSTで送信する．

```yaml
# https://github.com/mercari/tfnotify
---
ci: circleci

notifier:
  github:
    token: <環境変数に登録したGitHubToken>
    repository:
      owner: "<送信先のユーザ名もしくは組織名>"
      name: "<送信先のリポジトリ名>"

terraform:
  plan:
    template: |
      {{ .Title }} for staging <sup>[CI link]( {{ .Link }} )</sup>
      {{ .Message }}
      {{if .Result}}
      <pre><code> {{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code> {{ .Body }}
      </pre></code></details>
  apply:
    template: |
      {{ .Title }}
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code>{{ .Body }}
      </pre></code></details>
```

