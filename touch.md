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

### ■CloudShellの起動

### ■VPCの作成

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
```

```CloudShell
clear; cat << EOF
VpcId : ${VpcId}
EOF
```

#### 変数設定確認

IDが取得されていることを確認。ID等は個人個人異なります。

```CloudShell
VpcId : vpc-08a77289b9b351429
```

### ■Subnetの作成

作成したVPCの中にSubnetを2つ作成します。

#### cmd1

Subnetの1つ目を作成します。

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

Subnetの2つ目を作成します。

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
```

```CloudShell
SubnetId1c=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    "Name=availabilityZone,Values=ap-northeast-1c" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```CloudShell
clear; cat << EOF
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

### ■InternetGatewayの作成

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

#### 変数設定

作成したInternet GatewayのIDを取得します。

``` CloudShell
InternetGatewayId=`aws ec2 describe-internet-gateways \
    --query 'InternetGateways[*].InternetGatewayId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`
```

``` CloudShell
cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
InternetGatewayId : ${InternetGatewayId}
EOF
```

#### 変数設定確認

変数が取得されていることを確認します。

``` CloudShell
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
InternetGatewayId : igw-0db61da9fcd82b6eb
```

### ■InternetGatewayをVPCにAttach

作成したInternetGatewayをVPCに紐付けします。

#### cmd

```CloudShell
aws ec2 attach-internet-gateway \
    --internet-gateway-id ${InternetGatewayId} \
    --vpc-id ${VpcId}
```

#### result

```CloudShell
（何もなし）
```

### ■InternetGatewayをVPCにAttachされていることを確認

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

### ■RouteTableの確認

VPC作成時にデフォルトのRouteTableがあるので、このIDを取得します。

#### 変数設定

```CloudShell
RouteTableId=`aws ec2 describe-route-tables \
  --query "RouteTables[*].RouteTableId" \
  --filters "Name=vpc-id,Values=${VpcId}" \
  --output text`
```

```CloudShell
clear; cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
InternetGatewayId : ${InternetGatewayId}
RouteTableId : ${RouteTableId}
EOF
```

#### 変数設定確認

```CloudShell
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
InternetGatewayId : igw-0db61da9fcd82b6eb
RouteTableId : rtb-01b343a22f94f5031
```

### ■RouteTableにSubnetを紐付け

#### cmd1

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableId} \
  --subnet-id ${SubnetId1a}
```

#### result1

```CloudShell
{
    "AssociationId": "rtbassoc-07e4b9476d5384320",
    "AssociationState": {
        "State": "associated"
    }
}
```

#### cmd2

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableId} \
  --subnet-id ${SubnetId1c}
```

#### result2

```CloudShell
{
    "AssociationId": "rtbassoc-001ad16bd1720e77c",
    "AssociationState": {
        "State": "associated"
    }
}
```

### ■RouteTableにInternetGatewayを紐付け

#### cmd

```CloudShell
aws ec2 create-route \
  --route-table-id ${RouteTableId} \
  --destination-cidr-block "0.0.0.0/0" \
  --gateway-id ${InternetGatewayId}
```

#### result

```CloudShell
{
    "Return": true
}
```

### ■環境変数をメモ

Cloud9で使うため、取得した変数をエディターに残して下さい。

#### cmd

```CloudShell
clear; cat << EOF
export VpcId="${VpcId}"
export SubnetId1a="${SubnetId1a}"
export SubnetId1c="${SubnetId1c}"
export InternetGatewayId="${InternetGatewayId}"
export RouteTableId="${RouteTableId}"
EOF
```

#### result

```CloudShell
export VpcId="vpc-08a77289b9b351429"
export SubnetId1a="subnet-0ae475cbd47289960"
export SubnetId1c="subnet-051a32873cc5c562b"
export InternetGatewayId="igw-0db61da9fcd82b6eb"
export RouteTableId="rtb-01b343a22f94f5031"
```

## Cloud9作成

Duration: 0:05:00

### ■Cloud9の作成

#### cmd

```CloudShell
aws cloud9 create-environment-ec2 \
  --name ContainerHandsOn \
  --description "ContainerHandsOn" \
  --instance-type t3.small  \
  --subnet-id ${SubnetId1a}  \
  --automatic-stop-time-minutes 60  \
  --tags "Key=Name,Value=ContainerHandsOn"
```

#### result

```CloudShell
{
    "environmentId": "96614b2a3f434be7a83b5dffb22a1f0a"
}
```

### ■AWS コンソールでCloud9を起動

- 上部の検索バーで`Cloud9`と検索
- `AWS Cloud9 > Your environments`に`ContainerHandsOn`が作成されているので`Open IDE`ボタン押下
- Cloud9の画面が表示される。

## ECR作成

Duration: 0:05:00

### ■環境変数を貼り付け

#### cmd

```Cloud9
export VpcId="vpc-08a77289b9b351429"
export SubnetId1a="subnet-0ae475cbd47289960"
export SubnetId1c="subnet-051a32873cc5c562b"
export InternetGatewayId="igw-0db61da9fcd82b6eb"
export RouteTableId="rtb-01b343a22f94f5031"
```

``` Cloud9
clear; cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
InternetGatewayId : ${InternetGatewayId}
RouteTableId : ${RouteTableId}
EOF
```

#### result

``` Cloud9
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
InternetGatewayId : igw-0db61da9fcd82b6eb
RouteTableId : rtb-01b343a22f94f5031
```

### ■ECRの作成

#### cmd

```Cloud9
aws ecr create-repository \
    --repository-name jaws-days-2022/container-hands-on \
    --tags "Key=Name,Value=ContainerHandsOn"
```

#### result

```Cloud9
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:378647896848:repository/jaws-days-2022/container-hands-on",
        "registryId": "378647896848",
        "repositoryName": "jaws-days-2022/container-hands-on",
        "repositoryUri": "378647896848.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on",
        "createdAt": 1662173179.0,
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

## DockerImage作成

Duration: 0:05:00

### ■Cloud9上にdockerがあることを確認

#### cmd

```Cloud9
docker -v
```

#### result

```Cloud9
Docker version 20.10.13, build a224086
```

### ■Cloud9上にDockerfileを作成

[参考元](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/create-container-image.html)

#### cmd

```Cloud9
cat << EOF > Dockerfile
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello! Jaws Days 2022!!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
EOF
```

```Cloud9
cat Dockerfile;
```

#### result

```Cloud9
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update &&  apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello! Jaws Days 2022!!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh &&  echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh &&  echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```

### ■Cloud9上でDocker イメージを構築

#### cmd

```Cloud9
docker build -t jaws-days-2022/container-hands-on .
```

#### result

```Cloud9
（略）
Successfully built fed9645afaae
Successfully tagged jaws-days-2022/container-hands-on:latest
```

### ■Cloud9上でDocker イメージを構築されたことを確認

#### cmd

```cloud9
docker images --filter reference= jaws-days-2022/container-hands-on:latest
```

#### result

```cloud9
REPOSITORY                          TAG       IMAGE ID       CREATED         SIZE
jaws-days-2022/container-hands-on   latest    fed9645afaae   8 minutes ago   202MB
```

### ■Cloud9上でDocker イメージを起動

#### cmd

```Cloud9
docker run --name container-hands-on -d -p 8080:80 jaws-days-2022/container-hands-on:latest
```

#### result

```Cloud9
dcaf3423f6abeea3a67bab0c01a33e7b9d2c97131c8b304900f0455ee73da7b7
```

#### 画面

xxx

### ■（ご参考）Cloud9上でコンテナを停止・削除する方法

#### cmd

```Cloud9

docker stop $(docker ps -q)
docker rm $(docker ps -q -a)
```

### ■AWS Account IDの取得

#### cmd

```Cloud9
AccoutID=`aws sts get-caller-identity --query Account --output text`
```

```Cloud9
clear; cat << EOF
VpcId : ${VpcId}
SubnetId1a : ${SubnetId1a}
SubnetId1c : ${SubnetId1c}
InternetGatewayId : ${InternetGatewayId}
RouteTableId : ${RouteTableId}
AccoutID : ${AccoutID}
EOF
```

#### result

```Cloud9
VpcId : vpc-08a77289b9b351429
SubnetId1a : subnet-0ae475cbd47289960
SubnetId1c : subnet-051a32873cc5c562b
InternetGatewayId : igw-0db61da9fcd82b6eb
RouteTableId : rtb-01b343a22f94f5031
AccoutID : 378647896848
```

aws sts get-caller-identity --query Account --output text

### ■DockerImageにTag付けを行う

#### cmd

```Cloud9
docker tag jaws-days-2022/container-hands-on:latest 378647896848.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest
```

### ■DockerImageをECRにPush

#### 認証トークンを取得し、レジストリに対して Docker クライアントを認証します

aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 378647896848.dkr.ecr.ap-northeast-1.amazonaws.com

## VPCエンドポイント作成

Duration: 0:05:00

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

### ■VPCからECRに繋ぐVPCエンドポイントを作成

#### cmd

```CloudShell
```

#### result

```CloudShell
```

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
