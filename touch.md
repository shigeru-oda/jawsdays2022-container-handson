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

#### AWS コンソールにログイン

![img](./image/img2-1.png)

#### CloudShellボタン押下

画面右上のCloudShellボタンを押下  
![img](./image/img2-2.png)

#### CloudShellを起動

今後は以下の画面にコマンド（以下 cmd）と結果（以下 result）を確認し、進める。
![img](./image/img2-3.png)

### ■AWS Account IDの取得

#### cmd

```CloudShell
AccoutID=`aws sts get-caller-identity --query Account --output text`
```

```CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
EOF
```

#### result

```CloudShell
AccoutID : 152767562250
```

### ■VPCの作成

VPCを新規に作成します。

#### cmd

``` CloudShell
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specification "ResourceType=vpc,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result

```CloudShell
{
    "Vpc": {
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-c3de3ba5",
        "State": "pending",
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-098ed848444db197b",
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

#### 変数設定

```CloudShell
VpcId=`aws ec2 describe-vpcs \
    --query 'Vpcs[*].VpcId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`
```

```CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
EOF
```

#### 変数設定確認

```CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
```

#### ■DNS名前解決をONにする

### cmd

```CloudShell
 aws ec2 modify-vpc-attribute \
  --vpc-id ${VpcId}  \
  --enable-dns-support  '{"Value":true}' 
```

### result

```CloudShell
（なし）
```

#### ■DNS名前解決の状態確認

### cmd

```CloudShell
aws ec2 describe-vpc-attribute \
  --query EnableDnsSupport \
  --vpc-id ${VpcId}  \
  --attribute enableDnsSupport
```

### result

```CloudShell
{
    "Value": true
}
```

#### ■DNSホスト名をONにする

### cmd

```CloudShell
 aws ec2 modify-vpc-attribute \
  --vpc-id ${VpcId}  \
  --enable-dns-hostnames  '{"Value":true}' 
```

### result

```CloudShell
（なし）
```

#### ■DNSホスト名の状態確認

### cmd

```CloudShell
aws ec2 describe-vpc-attribute \
  --query EnableDnsHostnames \
  --vpc-id ${VpcId}  \
  --attribute enableDnsHostnames
```

### result

```CloudShell
{
    "Value": true
}
```

### ■Subnetの作成

作成したVPCの中にSubnetを4つ作成します。
Private Subnetが2つ、Public Subnetが2つです。

#### cmd1

```CloudShell
aws ec2 create-subnet \
    --vpc-id $VpcId \
    --availability-zone ap-northeast-1a \
    --cidr-block 10.0.0.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOnPublic}]"
```

#### result1

```CloudShell
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1a",
        "AvailabilityZoneId": "apne1-az4",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.0.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0f66f257f167a1d47",
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPublic"
            }
        ],
        "SubnetArn": "arn:aws:ec2:ap-northeast-1:152767562250:subnet/subnet-0f66f257f167a1d47",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
```

#### cmd2

```CloudShell
aws ec2 create-subnet \
    --vpc-id $VpcId \
    --availability-zone ap-northeast-1c \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOnPublic}]"
```

#### result2

```CloudShell
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1c",
        "AvailabilityZoneId": "apne1-az1",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.1.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0a1e2afffc8c140d8",
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPublic"
            }
        ],
        "SubnetArn": "arn:aws:ec2:ap-northeast-1:152767562250:subnet/subnet-0a1e2afffc8c140d8",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
```

#### cmd3

```CloudShell
aws ec2 create-subnet \
    --vpc-id $VpcId \
    --availability-zone ap-northeast-1a \
    --cidr-block 10.0.2.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOnPrivate}]"
```

#### result3

```CloudShell
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1a",
        "AvailabilityZoneId": "apne1-az4",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.2.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-049f0119237ff00a0",
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPrivate"
            }
        ],
        "SubnetArn": "arn:aws:ec2:ap-northeast-1:152767562250:subnet/subnet-049f0119237ff00a0",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
```

#### cmd4

```CloudShell
aws ec2 create-subnet \
    --vpc-id $VpcId \
    --availability-zone ap-northeast-1c \
    --cidr-block 10.0.3.0/24 \
    --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=ContainerHandsOnPrivate}]"
```

#### result4

```CloudShell
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-1c",
        "AvailabilityZoneId": "apne1-az1",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.3.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0ea89b6bc85e0ec61",
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPrivate"
            }
        ],
        "SubnetArn": "arn:aws:ec2:ap-northeast-1:152767562250:subnet/subnet-0ea89b6bc85e0ec61",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
```

#### 変数取得

```CloudShell
SubnetId1aPublic=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPublic" \
    "Name=availabilityZone,Values=ap-northeast-1a" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```CloudShell
SubnetId1cPublic=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPublic" \
    "Name=availabilityZone,Values=ap-northeast-1c" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```CloudShell
SubnetId1aPrivate=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPrivate" \
    "Name=availabilityZone,Values=ap-northeast-1a" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```CloudShell
SubnetId1cPrivate=`aws ec2 describe-subnets \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPrivate" \
    "Name=availabilityZone,Values=ap-northeast-1c" \
    --query "Subnets[*].SubnetId" \
    --output text`
```

```CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic: ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate: ${SubnetId1cPrivate}
EOF
```

#### 変数設定確認

```CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic: subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate: subnet-0ea89b6bc85e0ec61
```

### ■InternetGatewayの作成

#### cmd

```CloudShell
aws ec2 create-internet-gateway \
    --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=ContainerHandsOn}]"
```

#### result1

```CloudShell
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-0a511ba68ceb84ed8",
        "OwnerId": "152767562250",
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOn"
            }
        ]
    }
}
```

#### 変数設定

``` CloudShell
InternetGatewayId=`aws ec2 describe-internet-gateways \
    --query 'InternetGateways[*].InternetGatewayId' \
    --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn" \
    --output text`
```

``` CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic: ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate: ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
EOF
```

#### 変数設定確認

``` CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic: subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate: subnet-0ea89b6bc85e0ec61
InternetGatewayId : igw-0a511ba68ceb84ed8
```

### ■InternetGatewayをVPCにAttach

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

```CloudShell
available
```

### ■RouteTableの作成

#### cmd1

```CloudShell
aws ec2 create-route-table \
  --vpc-id ${VpcId} \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=ContainerHandsOnPublic}]"
```

#### result1

```CloudShell
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-00cf30796b25b9bc9",
        "Routes": [
            {
                "DestinationCidrBlock": "10.0.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPublic"
            }
        ],
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250"
    }
}
```

#### cmd2

```CloudShell
aws ec2 create-route-table \
  --vpc-id ${VpcId} \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=ContainerHandsOnPrivate}]"
```

#### result2

```CloudShell
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0afaac377925bca9a",
        "Routes": [
            {
                "DestinationCidrBlock": "10.0.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [
            {
                "Key": "Name",
                "Value": "ContainerHandsOnPrivate"
            }
        ],
        "VpcId": "vpc-0d3c1c88db46cfba7",
        "OwnerId": "152767562250"
    }
}
```

### ■RouteTableの確認

#### 変数設定

```CloudShell
RouteTableIdPublic=`aws ec2 describe-route-tables \
  --query "RouteTables[*].RouteTableId" \
  --filters "Name=vpc-id,Values=${VpcId}" \
  "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPublic" \
  --output text`
```

```CloudShell
RouteTableIdPrivate=`aws ec2 describe-route-tables \
  --query "RouteTables[*].RouteTableId" \
  --filters "Name=vpc-id,Values=${VpcId}" \
  "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOnPrivate" \
  --output text`
```

```CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic: ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate: ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
EOF
```

#### 変数設定確認

```CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic: subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate: subnet-0ea89b6bc85e0ec61
InternetGatewayId : igw-0a511ba68ceb84ed8
RouteTableIdPublic : rtb-00cf30796b25b9bc9
RouteTableIdPrivate : rtb-0afaac377925bca9a
```

### ■RouteTableにSubnetを紐付け

#### cmd1

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableIdPublic} \
  --subnet-id ${SubnetId1aPublic}
```

#### result1

```CloudShell
{
    "AssociationId": "rtbassoc-0e98cb5f6c54d5d83",
    "AssociationState": {
        "State": "associated"
    }
}
```

#### cmd2

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableIdPublic} \
  --subnet-id ${SubnetId1cPublic}
```

#### result2

```CloudShell
{
    "AssociationId": "rtbassoc-0f3ca785ae8675b6f",
    "AssociationState": {
        "State": "associated"
    }
}
```

#### cmd3

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableIdPrivate} \
  --subnet-id ${SubnetId1aPrivate}
```

#### result3

```CloudShell
{
    "AssociationId": "rtbassoc-07f8b8f8aa65d0df8",
    "AssociationState": {
        "State": "associated"
    }
}
```

#### cmd4

```CloudShell
aws ec2 associate-route-table \
  --route-table-id ${RouteTableIdPrivate} \
  --subnet-id ${SubnetId1cPrivate}
```

#### result4

```CloudShell
{
    "AssociationId": "rtbassoc-008c971373c02cf69",
    "AssociationState": {
        "State": "associated"
    }
}
```

### ■RouteTableにInternetGatewayを紐付け

#### cmd

```CloudShell
aws ec2 create-route \
  --route-table-id ${RouteTableIdPublic} \
  --destination-cidr-block "0.0.0.0/0" \
  --gateway-id ${InternetGatewayId}
```

#### result

```CloudShell
{
    "Return": true
}
```

### ■PublicSubnet用のSecurityGroup作成

#### cmd

```CloudShell
aws ec2 create-security-group \
  --group-name PublicSecurityGroup \
  --description "Public Security Group" \
  --vpc-id ${VpcId} \
  --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=ContainerHandsOn-PublicSecurityGroup}]"
```

#### result

```CloudShell
{
    "GroupId": "sg-01cc901415c240504",
    "Tags": [
        {
            "Key": "Name",
            "Value": "ContainerHandsOn-PublicSecurityGroup"
        }
    ]
}
```

#### 変数設定

```
PublicSecurityGroupsId=`aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].GroupId' \
  --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn-PublicSecurityGroup" \
    --output text`
```

### ■PrivateSubnet用のSecurityGroup作成

#### cmd

```CloudShell
aws ec2 create-security-group \
  --group-name PrivateSecurityGroup \
  --description "Private Security Group" \
  --vpc-id ${VpcId} \
  --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=ContainerHandsOn-PrivateSecurityGroup}]"
```

#### result

```CloudShell
{
    "GroupId": "sg-040aff209e1fe59cc",
    "Tags": [
        {
            "Key": "Name",
            "Value": "ContainerHandsOn-PrivateSecurityGroup"
        }
    ]
}
```

#### 変数設定

```CloudShell
PrivateSecurityGroupsId=`aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].GroupId' \
  --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn-PrivateSecurityGroup" \
    --output text`
```

```CloudShell
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic: ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate: ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
PublicSecurityGroupsId : ${PublicSecurityGroupsId}
PrivateSecurityGroupsId : ${PrivateSecurityGroupsId}
EOF
```

#### 変数設定確認
```CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic: subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate: subnet-0ea89b6bc85e0ec61
InternetGatewayId : igw-0a511ba68ceb84ed8
RouteTableIdPublic : rtb-00cf30796b25b9bc9
RouteTableIdPrivate : rtb-0afaac377925bca9a
PublicSecurityGroupsId : sg-01cc901415c240504
PrivateSecurityGroupsId : sg-040aff209e1fe59cc
```

### ■環境変数をメモ

Cloud9で使うため、取得した変数をエディターに残して下さい。

#### cmd

```CloudShell
clear; cat << EOF
export AccoutID="${AccoutID}"
export VpcId="${VpcId}"
export SubnetId1aPublic="${SubnetId1aPublic}"
export SubnetId1cPublic="${SubnetId1cPublic}"
export SubnetId1aPrivate="${SubnetId1aPrivate}"
export SubnetId1cPrivate="${SubnetId1cPrivate}"
export InternetGatewayId="${InternetGatewayId}"
export RouteTableIdPublic="${RouteTableIdPublic}"
export RouteTableIdPrivate="${RouteTableIdPrivate}"
export PublicSecurityGroupsId="${PublicSecurityGroupsId}"
export PrivateSecurityGroupsId="${PrivateSecurityGroupsId}"
EOF
```

#### result

```CloudShell
export AccoutID="152767562250"
export VpcId="vpc-0d3c1c88db46cfba7"
export SubnetId1aPublic="subnet-0f66f257f167a1d47"
export SubnetId1cPublic="subnet-0a1e2afffc8c140d8"
export SubnetId1aPrivate="subnet-049f0119237ff00a0"
export SubnetId1cPrivate="subnet-0ea89b6bc85e0ec61"
export InternetGatewayId="igw-0a511ba68ceb84ed8"
export RouteTableIdPublic="rtb-00cf30796b25b9bc9"
export RouteTableIdPrivate="rtb-0afaac377925bca9a"
export PublicSecurityGroupsId="sg-01cc901415c240504"
export PrivateSecurityGroupsId="sg-040aff209e1fe59cc"
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
  --subnet-id ${SubnetId1aPublic}  \
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
export SubnetId1aPublic="subnet-0ae475cbd47289960"
export SubnetId1cPublic="subnet-051a32873cc5c562b"
export SubnetId1aPrivate="subnet-01eb19ab0aeb0f6f1"
export SubnetId1cPrivate="subnet-0819c13fe959a0d1a"
export InternetGatewayId="igw-0db61da9fcd82b6eb"
export RouteTableIdPublic="rtb-0cfcfe4b74de83091"
export RouteTableIdPrivate="rtb-089389ec79a044951"
```

``` Cloud9
clear; cat << EOF
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
EOF
```

#### result

``` Cloud9
VpcId : vpc-08a77289b9b351429
SubnetId1aPublic : subnet-0ae475cbd47289960
SubnetId1cPublic : subnet-051a32873cc5c562b
SubnetId1aPrivate : subnet-01eb19ab0aeb0f6f1
SubnetId1cPrivate : subnet-0819c13fe959a0d1a
InternetGatewayId : igw-0db61da9fcd82b6eb
RouteTableIdPublic : rtb-0cfcfe4b74de83091
RouteTableIdPrivate : rtb-089389ec79a044951
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

## Docker Image作成

Duration: 0:05:00

### ■Cloud9上にdockerセットアップされていることを確認

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
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
AccoutID : ${AccoutID}
EOF
```

#### result

```Cloud9
VpcId : vpc-08a77289b9b351429
SubnetId1aPublic : subnet-0ae475cbd47289960
SubnetId1cPublic : subnet-051a32873cc5c562b
SubnetId1aPrivate : subnet-01eb19ab0aeb0f6f1
SubnetId1cPrivate : subnet-0819c13fe959a0d1a
InternetGatewayId : igw-0db61da9fcd82b6eb
RouteTableIdPublic : rtb-0cfcfe4b74de83091
RouteTableIdPrivate : rtb-089389ec79a044951
AccoutID : 378647896848
```

### ■DockerImageにTag付けを行う

#### cmd

```Cloud9
docker tag jaws-days-2022/container-hands-on:latest `echo ${AccoutID}`.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest
```

#### result

```Cloud9
（なし）
```

### ■DockerImageにTag付けの確認

#### cmd

```Cloud9
docker images --filter reference=`echo ${AccoutID}`.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest

```

#### result

```Cloud9
REPOSITORY                                                                            TAG       IMAGE ID       CREATED          SIZE
378647896848.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on   latest    fed9645afaae   21 minutes ago   202MB
```

### ■認証トークンを取得し、レジストリに対して Docker クライアントを認証します

#### cmd

```Cloud9
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin `echo ${AccoutID}`.dkr.ecr.ap-northeast-1.amazonaws.com
```

#### result

```Cloud9
WARNING! Your password will be stored unencrypted in /home/ec2-user/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### ■DockerImageをECRにPush

#### cmd

```Cloud9
docker push `echo ${AccoutID}`.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest
```

#### result

```Cloud9
61e740eb529d: Pushed 
58f84848d392: Pushed 
d5c24541b3aa: Pushed 
1a996540f50f: Pushed 
latest: digest: sha256:8d4d59636b92dffc4d4986c6a38a42fff9201d418aa21d96ad276a754d77d943 size: 1155
```

## VPCエンドポイント作成

Duration: 0:05:00

FargateをPrivateSubnetで稼働させるため、以下VPCエンドポイントを準備します。

- com.amazonaws.ap-northeast-1.s3
- com.amazonaws.ap-northeast-1.ecr.dkr
- com.amazonaws.ap-northeast-1.ecr.api
- com.amazonaws.ap-northeast-1.logs

### ■com.amazonaws.ap-northeast-1.s3

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Gateway \
    --service-name com.amazonaws.ap-northeast-1.s3 \
    --route-table-ids ${RouteTableIdPrivate} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
xxx
```

### ■インターフェース用のSecurityGroup作成

#### cmd

```Cloud9
aws ec2 create-security-group \
  --group-name InterfaceSecurityGroup \
  --description "Interface Security Group" \
  --vpc-id ${VpcId} \
  --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=ContainerHandsOn-InterfaceSecurityGroup}]"
```

#### result

```Cloud9
xxx
```

#### 変数設定

```Cloud9
InterfaceGroupId=`aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].GroupId' \
  --filters "Name=tag-key,Values=Name" \
    "Name=tag-value,Values=ContainerHandsOn-InterfaceSecurityGroup" \
    --output text`
```

```Cloud9
clear; cat << EOF
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
AccoutID : ${AccoutID}
InterfaceGroupId : ${InterfaceGroupId}
EOF
```

#### 変数設定確認

```
```

### ■com.amazonaws.ap-northeast-1.ecr.dkr

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.ecr.dkr \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${InterfaceGroupId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
```

### ■com.amazonaws.ap-northeast-1.ecr.api

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.ecr.api \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${InterfaceGroupId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
xxx
```

### ■com.amazonaws.ap-northeast-1.logs

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.logs \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${InterfaceGroupId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
```

## ALB作成

Duration: 0:05:00

### ■アプリケーションロードバランサーの作成

#### cmd

```Cloud9
aws elbv2 create-load-balancer \
    --name ContainerHandsOn \
    --subnets ${SubnetId1aPublic} ${SubnetId1cPublic} \
    --tags "Key=Name,Value=ContainerHandsOn"
```

#### result

```Cloud9
```

### ■ターゲットグループの作成

#### cmd

```Cloud9
aws elbv2 create-target-group \
    --name ContainerHandsOn \
    --protocol TCP \
    --port 80 \
    --target-type ip \
    --vpc-id ${VpcId}
```

## Fargate作成

Duration: 0:05:00

### ■クラスターの作成

#### cmd

```Cloud9
aws ecs create-cluster \
    --cluster-name ContainerHandsOn \
    --tags "key=Name,value=ContainerHandsOn"
```

#### result

```Cloud9
xxx
```

### ■タスク定義の作成

#### cmd

```Cloud9
cat << EOF > register-task-definition.json
{
    "family": "ContainerHandsOn", 
    "executionRoleArn": "arn:aws:iam::378647896848:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS", 
    "networkMode": "awsvpc", 
    "containerDefinitions": [
        {
            "name": "ContainerHandsOn", 
            "image": "${AccoutID}.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest", 
            "portMappings": [
                {
                    "containerPort": 80, 
                    "hostPort": 80, 
                    "protocol": "tcp"
                }
            ], 
            "essential": true
        }
    ], 
    "requiresCompatibilities": [
        "FARGATE"
    ], 
    "cpu": "256", 
    "memory": "512",
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    }
}
EOF
```

```Cloud9
aws ecs register-task-definition \
  --cli-input-json file://register-task-definition.json \
  --tags "key=Name,value=ContainerHandsOn"
```

#### result

```Cloud9
xxxx
```

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
