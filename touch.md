author: Shigeru Oda
summary: JAWS DAYS 2022 コンテナ ハンズオン
id:docs
categories: codelab,markdown
environments: HandsOn
status: Draft
feedback link: <https://github.com/shigeru-oda/jawsdays2022-container-handson>
analytics account: XXXXXXXX

# [JAWS DAYS 2022] ハンズオン～コンテナサービスをCI/CDパイプラインでデプロイしよう～

## はじめに

Duration: 0:05:00

## VPC作成

Duration: 0:05:00

### CloudShellの起動

### VPCの作成

VPCを新規に作成します。

#### cmd

CloudShellに以下cmdをCopy & Paste

``` CloudShell
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specification "ResourceType=vpc,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result

CloudShellに以下のような結果が返却されていることを確認下さい。ID等は個人個人異なります。

```CloudShell
xx
```

#### 変数設定

VPC IDを取得します、CloudShellに以下cmdをCopy & Paste。

```CloudShell
VpcId=`aws ec2 describe-vpcs \
    --query 'Vpcs[*].VpcId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`


cat << EOF
VpcId : ${VpcId}
EOF
```

#### 変数設定確認

IDが取得されていることを確認。ID等は個人個人異なります。

```CloudShell
VpcId : vpc-08a77289b9b351429
```

### Subnetの作成

作成したVPCの中にSubnetを2つ作成します。

#### cmd1

```CloudShell
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --availability-zone ap-northeast-1a \
    --cidr-block 10.0.0.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result1

CloudShellに以下のような結果が返却されていることを確認下さい。ID等は個人個人異なります。

```CloudShell
xxx
```

#### cmd2

```CloudShell
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --availability-zone ap-northeast-1c \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result2

CloudShellに以下のような結果が返却されていることを確認下さい。ID等は個人個人異なります。

```CloudShell
xxx
```

#### 変数取得

IDが取得されていることを確認。ID等は個人個人異なります。

```CloudShell
SubnetId1a=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    "Name=availabilityZone,Values=ap-northeast-1a" \
    --query "Subnets[*].SubnetId" \
    --output text`



SubnetId1c=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    "Name=availabilityZone,Values=ap-northeast-1c" \
    --query "Subnets[*].SubnetId" \
    --output text`


cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
EOF
```

#### 変数設定確認

IDが取得されていることを確認。ID等は個人個人異なります。

```CloudShell
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
```

### InternetGatewayの作成

Internetに繋がるInternetGatewayを作成します。

#### cmd

```CloudShell
aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result1

```CloudShell
xxx
```

### 変数設定

作成したInternet GatewayのIDを取得します。

``` CloudShell
InternetGatewayId=`aws ec2 describe-internet-gateways \
    --query 'InternetGateways[*].InternetGatewayId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`



cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
InternetGatewayId : ${InternetGatewayId}
EOF
```

### 変数設定確認

変数が取得されていることを確認します。

``` CloudShell
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
InternetGatewayId : igw-0db61da9fcd82b6eb
```

### InternetGatewayをVPCにAttach

作成したInternetGatewayをVPCに紐付けします。

#### cmd

```CloudShell
aws ec2 attach-internet-gateway \
    --internet-gateway-id ${InternetGatewayId} \
    --vpc-id ${VpcId}
```

#### result

```CloudShell
何もなし
```

### InternetGatewayをVPCにAttachされていることを確認

紐付けが正しく行われたことを確認します。

#### cmd

```CloudShell
aws ec2 describe-internet-gateways \
    --internet-gateway-ids ${InternetGatewayId} \
    --query 'InternetGateways[*].Attachments[*].State' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text
```

#### result

結果がavailableであること

```CloudShell
available
```

### RouteTableの確認

VPC作成時にデフォルトのRouteTableがあるので、このIDを取得します。

#### 変数設定

```CloudShell
```

#### 変数設定確認

#### cmd

```cmd1
aws ec2 associate-route-table \
    --route-table-id rtb-22574640 \
    --subnet-id ${SubnetId1a}
```

```cmd2
aws ec2 associate-route-table \
    --route-table-id rtb-22574640 \
    --subnet-id ${SubnetId1c}
```

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
