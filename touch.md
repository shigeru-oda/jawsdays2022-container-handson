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
```
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
vpcid=`aws ec2 describe-vpcs \
    --query 'Vpcs[*].VpcId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`

echo $vpcid
```

#### result
```
vpc-08a77289b9b351429
```


### Subnetの作成
#### cmd1
```
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --cidr-block 10.0.0.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### cmd2
```
aws ec2 create-subnet \
    --vpc-id $vpcid \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result1
```
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1d",
        "AvailabilityZoneId": "apne1-az2",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.0.0/24",
        "DefaultForAz": false,
:...skipping...
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1d",
        "AvailabilityZoneId": "apne1-az2",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.0.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-09a994448454b186e",
        "VpcId": "vpc-08a77289b9b351429",
        "OwnerId": "378647896848",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOn"
            }
        ],
        "SubnetArn": "arn:aws:ec2:ap-northeast-1:378647896848:subnet/subnet-09a994448454b186e"
    }
}
```

#### result2
```
```


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