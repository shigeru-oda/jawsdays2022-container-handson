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

### VPC?

xx

### CloudShell?

xx

## VPC作成

Duration: 0:05:00

### CloudShellの起動

### VPCの作成

東京リージョンにVPCを新規に作成します。

#### cmd

``` cmd
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specification "ResourceType=vpc,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result

```result
{
    "Vpc": {
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-d50f23b2",
        "State": "pending",
        "VpcId": "vpc-00ee8367984814805",
        "OwnerId": "378647896848",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-08913abf335aba6a4",
                "CidrBlock": "10.0.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false,
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOn"
            }
        ]
    }
}
```

### 変数設定

作成したVPC IDを取得します。

#### cmd

```
VpcId=`aws ec2 describe-vpcs \
    --query 'Vpcs[*].VpcId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`

cat << EOF
VpcId : ${VpcId}
EOF
```

#### result

IDが取得されていることを確認。以下は一例。

```con
VpcId : vpc-08a77289b9b351429
```

### Subnetの作成

#### cmd1

```
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --availability-zone ap-northeast-1a \
    --cidr-block 10.0.0.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### cmd2

```
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --availability-zone ap-northeast-1c \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### 変数取得

```cmd1
SubnetId1a=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    "Name=availabilityZone,Values=ap-northeast-1a" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```cmd2
SubnetId1c=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    "Name=availabilityZone,Values=ap-northeast-1c" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```
cat << EOF
VpcId : ${VpcId}
InternetGatewayId : ${InternetGatewayId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
EOF
```

#### result

```
```

### InternetGatewayの作成

#### cmd1

```
aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result1

```
```

### 変数設定

作成したInternet GatewayのIDを取得します。

``` cmd1
InternetGatewayId=`aws ec2 describe-internet-gateways \
    --query 'InternetGateways[*].InternetGatewayId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`

cat << EOF
VpcId : ${VpcId}
InternetGatewayId : ${InternetGatewayId}
EOF
```

変数が取得されていること、以下は一例。

``` result1
VpcId : vpc-08a77289b9b351429
InternetGatewayId : igw-0db61da9fcd82b6eb
```

### InternetGatewayをVPCにAttach

```cmd1
aws ec2 attach-internet-gateway \
    --internet-gateway-id ${InternetGatewayId} \
    --vpc-id ${VpcId}
```

```result1
何もなし
```

### InternetGatewayをVPCにAttachされていることを確認

```cmd1
aws ec2 describe-internet-gateways \
    --internet-gateway-ids ${InternetGatewayId} \
    --query 'InternetGateways[*].Attachments' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn"
```

Stateがavailableであること

```result1
[
    [
        {
            "State": "available",
            "VpcId": "vpc-08a77289b9b351429"
        }
    ]
]
```

### RouteTableにSubnetを関連付け

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
