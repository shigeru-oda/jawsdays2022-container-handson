author: Shigeru Oda
summary: JAWS DAYS 2022 コンテナ ハンズオン
id:docs
categories: codelab,markdown
environments: HandsOn
status: Draft
feedback link: https://github.com/shigeru-oda/jawsdays2022-container-handson
analytics account: XXXXXXXX
# [JAWS DAYS 2022] ハンズオン～コンテナサービスをCI/CDパイプラインでデプロイしよう～

## はじめに
Duration: 0:05:00




## VPC作成
Duration: 0:05:00
### CloudShellの起動
### VPCの作成
```
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specification ResourceType=vpc,Tags=[{Key=Name,Value=MyVpc}]
```

```go
This block will be highlighted as Go source code.
```

```console
This block will not be syntax highlighted.
```

### Subnetの作成
### RouteTableの作成
### RouteTableの作成






## Cloud9作成
Duration: 0:05:00
## ECR作成
Duration: 0:05:00

## VPCエンドポイント作成
Duration: 0:05:00
## ALB作成
Duration: 0:05:00
## Fargate作成
Duration: 0:05:00
## 動作確認
Duration: 0:05:00

## CodeBuild作成
Duration: 0:05:00
## CodeDeploy作成
Duration: 0:05:00
## CodePipeline作成
Duration: 0:05:00
## Dockerコンテナ再ビルド(Codeシリーズを利用)
Duration: 0:05:00
## 動作確認
Duration: 0:05:00
## 片付け
Duration: 0:05:00
### 途中の見出し
Positive
: 何かお知らせを書きたい時のボックス

Negative
: 何か注意点などを書きたい時のボックス