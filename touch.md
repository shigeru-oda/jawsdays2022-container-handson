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

## 事前準備とVPC作成

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

### ■ECSの実行Roleの存在確認

#### cmd

```Cloud9
aws iam list-roles | grep "RoleName" | grep "ecsTaskExecutionRole"
```

#### result（存在する場合）

```Cloud9
            "RoleName": "ecsTaskExecutionRole",
```

#### result（存在しない場合）

```Cloud9
（なし）
```

### ■ECSの実行Role作成（上記で存在しない場合実行）

#### cmd

```Cloud9
cat << EOF > assume-role-policy-document.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

```Cloud9
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://assume-role-policy-document.json
```

#### result

```
{
    "Role": {
        "Path": "/",
        "RoleName": "ecsTaskExecutionRole",
        "RoleId": "AROASHENIAIFOV2DAKSWN",
        "Arn": "arn:aws:iam::152767562250:role/ecsTaskExecutionRole",
        "CreateDate": "2022-09-06T14:50:16+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "",
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ecs-tasks.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}
```

#### cmd

```Cloud9
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
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
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
EOF
```

#### 変数設定確認

```CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic : subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate : subnet-0ea89b6bc85e0ec61
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
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
EOF
```

#### 変数設定確認

``` CloudShell
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic : subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate : subnet-0ea89b6bc85e0ec61
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
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
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
SubnetId1cPublic : subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate : subnet-0ea89b6bc85e0ec61
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
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
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
SubnetId1cPublic : subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate : subnet-0ea89b6bc85e0ec61
InternetGatewayId : igw-0a511ba68ceb84ed8
RouteTableIdPublic : rtb-00cf30796b25b9bc9
RouteTableIdPrivate : rtb-0afaac377925bca9a
PublicSecurityGroupsId : sg-01cc901415c240504
PrivateSecurityGroupsId : sg-040aff209e1fe59cc
```

### ■PublicSubnetのInboundを追加

#### cmd

```CloudShell
aws ec2 authorize-security-group-ingress \
    --group-id ${PublicSecurityGroupsId} \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

#### result

```CloudShell
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-011c9bf434c9439a0",
            "GroupId": "sg-01cc901415c240504",
            "GroupOwnerId": "152767562250",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

### ■PrivateSubnetのInboundを追加

#### cmd

```CloudShell
aws ec2 authorize-security-group-ingress \
    --group-id ${PrivateSecurityGroupsId} \
    --protocol tcp \
    --port 80 \
    --source-group ${PublicSecurityGroupsId}
```

#### result

```CloudShell
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0eca9b6518889b4ca",
            "GroupId": "sg-040aff209e1fe59cc",
            "GroupOwnerId": "152767562250",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "ReferencedGroupInfo": {
                "GroupId": "sg-01cc901415c240504"
            }
        }
    ]
}
```

#### cmd

```CloudShell
aws ec2 authorize-security-group-ingress \
    --group-id ${PrivateSecurityGroupsId} \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0
```

#### result

```CloudShell
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0d637ce636f2f14d9",
            "GroupId": "sg-040aff209e1fe59cc",
            "GroupOwnerId": "152767562250",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 443,
            "ToPort": 443,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
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
  --automatic-stop-time-minutes 60 
```

#### result

```CloudShell
{
    "environmentId": "aa2999a731eb4178aed99069f1b683aa"
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

``` Cloud9
clear; cat << EOF
AccoutID : ${AccoutID}
VpcId : ${VpcId}
SubnetId1aPublic : ${SubnetId1aPublic}
SubnetId1cPublic : ${SubnetId1cPublic}
SubnetId1aPrivate : ${SubnetId1aPrivate}
SubnetId1cPrivate : ${SubnetId1cPrivate}
InternetGatewayId : ${InternetGatewayId}
RouteTableIdPublic : ${RouteTableIdPublic}
RouteTableIdPrivate : ${RouteTableIdPrivate}
PublicSecurityGroupsId : ${PublicSecurityGroupsId}
PrivateSecurityGroupsId : ${PrivateSecurityGroupsId}
EOF
```

#### result

``` Cloud9
AccoutID : 152767562250
VpcId : vpc-0d3c1c88db46cfba7
SubnetId1aPublic : subnet-0f66f257f167a1d47
SubnetId1cPublic : subnet-0a1e2afffc8c140d8
SubnetId1aPrivate : subnet-049f0119237ff00a0
SubnetId1cPrivate : subnet-0ea89b6bc85e0ec61
InternetGatewayId : igw-0a511ba68ceb84ed8
RouteTableIdPublic : rtb-00cf30796b25b9bc9
RouteTableIdPrivate : rtb-0afaac377925bca9a
PublicSecurityGroupsId : sg-01cc901415c240504
PrivateSecurityGroupsId : sg-040aff209e1fe59cc
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
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:152767562250:repository/jaws-days-2022/container-hands-on",
        "registryId": "152767562250",
        "repositoryName": "jaws-days-2022/container-hands-on",
        "repositoryUri": "152767562250.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on",
        "createdAt": "2022-09-06T13:21:42+00:00",
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

#### cmd

```Cloud9
cat << EOF > Dockerfile
FROM php:7.4.0-apache
COPY src/ /var/www/html/
EOF
```

```Cloud9
mkdir src
```

```Cloud9
cat << EOF > ./src/index.php
<!DOCTYPE html>
<html lang="ja">
  <head>
    <title>Hello! Jaws Days 2022!!</title>
  </head>
  <body>
    <p>Hello! Jaws Days 2022!!</p>
    <?php echo gethostname(); ?>
  </body>
</html>
EOF
```

```Cloud9
ls -l ./Dockerfile  ./src/index.php
```

#### result

```Cloud9
-rw-rw-r-- 1 ec2-user ec2-user  47 Sep  6 13:26 ./Dockerfile
-rw-rw-r-- 1 ec2-user ec2-user 190 Sep  6 13:26 ./src/index.php
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
REPOSITORY                          TAG       IMAGE ID       CREATED          SIZE
jaws-days-2022/container-hands-on   latest    98c14fa37bab   10 seconds ago   414MB
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

- Cloud9のヘッダ部分の`Preview`-> `Preview Runnnig Application`のボタン押下  
- `Hello! Jaws Days 2022!!`と記載された画面が表示されること  

![img](./image/img4-1.png)

Positive
: 何か間違ってコンテナを止めたい場合には以下を実行ください

```
docker stop $(docker ps -q)
docker rm $(docker ps -q -a)
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
152767562250.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on   latest    98c14fa37bab   10 minutes ago   414MB
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
The push refers to repository [152767562250.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on]
9f8ad0ecf420: Pushed 
536365481ed8: Pushed 
4b3693c51878: Pushed 
677c3ce9f0b4: Pushed 
c08f4d9c281b: Pushed 
ed13170590f7: Pushed 
37cbdda31557: Pushed 
9691e5d7a4c7: Pushed 
6a4d393f0795: Pushed 
e38834ac7561: Pushed 
ec64f555d498: Pushed 
840f3f414cf6: Pushed 
17fce12edef0: Pushed 
831c5620387f: Pushed 
latest: digest: sha256:1fb5a3ac5ea1e7d60c24f371564a8c2c7bbc8072be0e80a3c53c30e1cae3ffd3 size: 3242
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
{
    "VpcEndpoint": {
        "PolicyDocument": "{\"Version\":\"2008-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"*\",\"Resource\":\"*\"}]}", 
        "VpcId": "vpc-0d3c1c88db46cfba7", 
        "Tags": [
            {
                "Value": "ContainerHands", 
                "Key": "Name"
            }
        ], 
        "NetworkInterfaceIds": [], 
        "SubnetIds": [], 
        "RequesterManaged": false, 
        "PrivateDnsEnabled": false, 
        "State": "available", 
        "ServiceName": "com.amazonaws.ap-northeast-1.s3", 
        "RouteTableIds": [
            "rtb-0afaac377925bca9a"
        ], 
        "Groups": [], 
        "OwnerId": "152767562250", 
        "VpcEndpointId": "vpce-035934e1a19b46f78", 
        "VpcEndpointType": "Gateway", 
        "CreationTimestamp": "2022-09-06T14:14:54.000Z", 
        "DnsEntries": []
    }
}
```

### ■com.amazonaws.ap-northeast-1.ecr.dkr

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.ecr.dkr \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${PrivateSecurityGroupsId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
{
    "VpcEndpoint": {
        "VpcId": "vpc-0d3c1c88db46cfba7", 
        "Tags": [
            {
                "Value": "ContainerHands", 
                "Key": "Name"
            }
        ], 
        "NetworkInterfaceIds": [
            "eni-09e0a49241653c549", 
            "eni-05e27340a7008c6c5"
        ], 
        "SubnetIds": [
            "subnet-0ea89b6bc85e0ec61", 
            "subnet-049f0119237ff00a0"
        ], 
        "RequesterManaged": false, 
        "PrivateDnsEnabled": true, 
        "State": "pending", 
        "ServiceName": "com.amazonaws.ap-northeast-1.ecr.dkr", 
        "RouteTableIds": [], 
        "Groups": [
            {
                "GroupName": "PrivateSecurityGroup", 
                "GroupId": "sg-040aff209e1fe59cc"
            }
        ], 
        "OwnerId": "152767562250", 
        "VpcEndpointId": "vpce-0dceb08e8bffc9a0c", 
        "VpcEndpointType": "Interface", 
        "CreationTimestamp": "2022-09-06T14:15:53.158Z", 
        "DnsEntries": [
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0dceb08e8bffc9a0c-nuehayud.dkr.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0dceb08e8bffc9a0c-nuehayud-ap-northeast-1a.dkr.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0dceb08e8bffc9a0c-nuehayud-ap-northeast-1c.dkr.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "ZONEIDPENDING", 
                "DnsName": "dkr.ecr.ap-northeast-1.amazonaws.com"
            }, 
            {
                "HostedZoneId": "ZONEIDPENDING", 
                "DnsName": "*.dkr.ecr.ap-northeast-1.amazonaws.com"
            }
        ]
    }
}
```

### ■com.amazonaws.ap-northeast-1.ecr.api

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.ecr.api \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${PrivateSecurityGroupsId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
{
    "VpcEndpoint": {
        "VpcId": "vpc-0d3c1c88db46cfba7", 
        "Tags": [
            {
                "Value": "ContainerHands", 
                "Key": "Name"
            }
        ], 
        "NetworkInterfaceIds": [
            "eni-084e01fac365b44eb", 
            "eni-0f8846781b4fb82ee"
        ], 
        "SubnetIds": [
            "subnet-0ea89b6bc85e0ec61", 
            "subnet-049f0119237ff00a0"
        ], 
        "RequesterManaged": false, 
        "PrivateDnsEnabled": true, 
        "State": "pending", 
        "ServiceName": "com.amazonaws.ap-northeast-1.ecr.api", 
        "RouteTableIds": [], 
        "Groups": [
            {
                "GroupName": "PrivateSecurityGroup", 
                "GroupId": "sg-040aff209e1fe59cc"
            }
        ], 
        "OwnerId": "152767562250", 
        "VpcEndpointId": "vpce-0e45e7f04848a0600", 
        "VpcEndpointType": "Interface", 
        "CreationTimestamp": "2022-09-06T14:16:25.698Z", 
        "DnsEntries": [
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0e45e7f04848a0600-kef4k347.api.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0e45e7f04848a0600-kef4k347-ap-northeast-1c.api.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-0e45e7f04848a0600-kef4k347-ap-northeast-1a.api.ecr.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "ZONEIDPENDING", 
                "DnsName": "api.ecr.ap-northeast-1.amazonaws.com"
            }
        ]
    }
}
```

### ■com.amazonaws.ap-northeast-1.logs

#### cmd

```Cloud9
aws ec2 create-vpc-endpoint \
    --vpc-id ${VpcId} \
    --vpc-endpoint-type Interface \
    --service-name com.amazonaws.ap-northeast-1.logs \
    --subnet-ids ${SubnetId1aPrivate} ${SubnetId1cPrivate} \
    --security-group-id ${PrivateSecurityGroupsId} \
    --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=ContainerHands}]"
```

#### result

```Cloud9
{
    "VpcEndpoint": {
        "VpcId": "vpc-0d3c1c88db46cfba7", 
        "Tags": [
            {
                "Value": "ContainerHands", 
                "Key": "Name"
            }
        ], 
        "NetworkInterfaceIds": [
            "eni-04b15926212e8ea31", 
            "eni-0514bbba136ec6108"
        ], 
        "SubnetIds": [
            "subnet-0ea89b6bc85e0ec61", 
            "subnet-049f0119237ff00a0"
        ], 
        "RequesterManaged": false, 
        "PrivateDnsEnabled": true, 
        "State": "pending", 
        "ServiceName": "com.amazonaws.ap-northeast-1.logs", 
        "RouteTableIds": [], 
        "Groups": [
            {
                "GroupName": "PrivateSecurityGroup", 
                "GroupId": "sg-040aff209e1fe59cc"
            }
        ], 
        "OwnerId": "152767562250", 
        "VpcEndpointId": "vpce-05ae88af55de2bf71", 
        "VpcEndpointType": "Interface", 
        "CreationTimestamp": "2022-09-06T14:16:50.374Z", 
        "DnsEntries": [
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-05ae88af55de2bf71-5ltns582.logs.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-05ae88af55de2bf71-5ltns582-ap-northeast-1c.logs.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "Z2E726K9Y6RL4W", 
                "DnsName": "vpce-05ae88af55de2bf71-5ltns582-ap-northeast-1a.logs.ap-northeast-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "ZONEIDPENDING", 
                "DnsName": "logs.ap-northeast-1.amazonaws.com"
            }
        ]
    }
}
```

## ALB作成

Duration: 0:05:00

### ■アプリケーションロードバランサーの作成

#### cmd

```Cloud9
aws elbv2 create-load-balancer \
    --name ContainerHandsOn \
    --subnets ${SubnetId1aPublic} ${SubnetId1cPublic} \
    --security-groups ${PublicSecurityGroupsId} \
    --tags "Key=Name,Value=ContainerHandsOn"
```

#### result

```Cloud9
{
    "LoadBalancers": [
        {
            "IpAddressType": "ipv4", 
            "VpcId": "vpc-0d3c1c88db46cfba7", 
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:loadbalancer/app/ContainerHandsOn/75145ed42d9a3867", 
            "State": {
                "Code": "provisioning"
            }, 
            "DNSName": "ContainerHandsOn-610375823.ap-northeast-1.elb.amazonaws.com", 
            "SecurityGroups": [
                "sg-08822a4834ed05f46"
            ], 
            "LoadBalancerName": "ContainerHandsOn", 
            "CreatedTime": "2022-09-06T14:21:35.730Z", 
            "Scheme": "internet-facing", 
            "Type": "application", 
            "CanonicalHostedZoneId": "Z14GRHDCWA56QT", 
            "AvailabilityZones": [
                {
                    "SubnetId": "subnet-0a1e2afffc8c140d8", 
                    "LoadBalancerAddresses": [], 
                    "ZoneName": "ap-northeast-1c"
                }, 
                {
                    "SubnetId": "subnet-0f66f257f167a1d47", 
                    "LoadBalancerAddresses": [], 
                    "ZoneName": "ap-northeast-1a"
                }
            ]
        }
    ]
}
```

### ■ターゲットグループの作成

#### cmd

```Cloud9
aws elbv2 create-target-group \
    --name ContainerHandsOn \
    --protocol HTTP \
    --port 80 \
    --target-type ip \
    --health-check-protocol HTTP \
    --health-check-port traffic-port \
    --health-check-path /index.php \
    --vpc-id ${VpcId}
```

#### result

```Cloud9
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7624dfd8f556a2d9",
            "TargetGroupName": "ContainerHandsOn",
            "Protocol": "HTTP",
            "Port": 80,
            "VpcId": "vpc-0d3c1c88db46cfba7",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "traffic-port",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 30,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 5,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/index.php",
            "Matcher": {
                "HttpCode": "200"
            },
            "TargetType": "ip",
            "ProtocolVersion": "HTTP1",
            "IpAddressType": "ipv4"
        }
    ]
}
```

### ■アプリケーションロードバランサーのDNS取得

#### cmd

```Cloud9
LoadBalancersDnsName=`aws elbv2 describe-load-balancers \
  --names ContainerHandsOn \
  --query "LoadBalancers[*].DNSName" \
  --output text`
```

```Cloud9
echo ${LoadBalancersDnsName}
```

#### result

```Cloud9
ContainerHandsOn-610375823.ap-northeast-1.elb.amazonaws.com
```

### ■アプリケーションロードバランサーのARN取得

#### cmd

```Cloud9
LoadBalancerArn=`aws elbv2 describe-load-balancers \
  --names ContainerHandsOn \
  --query "LoadBalancers[*].LoadBalancerArn" \
  --output text`
```

```Cloud9
echo ${LoadBalancerArn}
```

#### result

```Cloud9
arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:loadbalancer/app/ContainerHandsOn/75145ed42d9a3867
```

### ■ターゲットグループのARN取得

#### cmd

```Cloud9
TargetGroupArn=`aws elbv2 describe-target-groups \
  --names ContainerHandsOn \
  --query "TargetGroups[*].TargetGroupArn" \
  --output text`
```

```Cloud9
echo ${TargetGroupArn}
```

#### result

```Cloud9
arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7f11fa9e9d635ce9
```

### ■リスナーの追加

#### cmd

```Cloud9
aws elbv2 create-listener \
    --load-balancer-arn ${LoadBalancerArn} \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=${TargetGroupArn}
```

#### result

```Cloud9
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:listener/app/ContainerHandsOn/75145ed42d9a3867/1245c4be3b3230b7",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:loadbalancer/app/ContainerHandsOn/75145ed42d9a3867",
            "Port": 80,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7f11fa9e9d635ce9",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7f11fa9e9d635ce9",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
            ]
        }
    ]
}
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
{
    "cluster": {
        "status": "ACTIVE", 
        "defaultCapacityProviderStrategy": [], 
        "statistics": [], 
        "capacityProviders": [], 
        "tags": [
            {
                "value": "ContainerHandsOn", 
                "key": "Name"
            }
        ], 
        "clusterName": "ContainerHandsOn", 
        "settings": [
            {
                "name": "containerInsights", 
                "value": "disabled"
            }
        ], 
        "registeredContainerInstancesCount": 0, 
        "pendingTasksCount": 0, 
        "runningTasksCount": 0, 
        "activeServicesCount": 0, 
        "clusterArn": "arn:aws:ecs:ap-northeast-1:152767562250:cluster/ContainerHandsOn"
    }
}
```


### ■タスク定義の作成

#### cmd

```Cloud9
cat << EOF > register-task-definition.json
{
    "family": "ContainerHandsOn", 
    "executionRoleArn": "arn:aws:iam::${AccoutID}:role/ecsTaskExecutionRole", 
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
{
    "taskDefinition": {
        "taskDefinitionArn": "arn:aws:ecs:ap-northeast-1:152767562250:task-definition/ContainerHandsOn:1",
        "containerDefinitions": [
            {
                "name": "ContainerHandsOn",
                "image": "152767562250.dkr.ecr.ap-northeast-1.amazonaws.com/jaws-days-2022/container-hands-on:latest",
                "cpu": 0,
                "portMappings": [
                    {
                        "containerPort": 80,
                        "hostPort": 80,
                        "protocol": "tcp"
                    }
                ],
                "essential": true,
                "environment": [],
                "mountPoints": [],
                "volumesFrom": []
            }
        ],
        "family": "ContainerHandsOn",
        "executionRoleArn": "arn:aws:iam::152767562250:role/ecsTaskExecutionRole",
        "networkMode": "awsvpc",
        "revision": 1,
        "volumes": [],
        "status": "ACTIVE",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            },
            {
                "name": "ecs.capability.execution-role-ecr-pull"
            },
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
            },
            {
                "name": "ecs.capability.task-eni"
            }
        ],
        "placementConstraints": [],
        "compatibilities": [
            "EC2",
            "FARGATE"
        ],
        "runtimePlatform": {
            "cpuArchitecture": "X86_64",
            "operatingSystemFamily": "LINUX"
        },
        "requiresCompatibilities": [
            "FARGATE"
        ],
        "cpu": "256",
        "memory": "512",
        "registeredAt": "2022-09-06T14:59:53.594000+00:00",
        "registeredBy": "arn:aws:sts::152767562250:assumed-role/AWSReservedSSO_AWSAdministratorAccess_8fe206b83490a022/ShigeruOda"
    },
    "tags": [
        {
            "key": "Name",
            "value": "ContainerHandsOn"
        }
    ]
}
```

### ■サービスの作成

#### cmd

```Cloud9
aws ecs create-service \
    --cluster ContainerHandsOn \
    --service-name ContainerHandsOn \
    --task-definition ContainerHandsOn:1 \
    --desired-count 2 \
    --launch-type FARGATE \
    --platform-version LATEST \
    --network-configuration "awsvpcConfiguration={subnets=[${SubnetId1aPrivate},${SubnetId1cPrivate}],securityGroups=[${PrivateSecurityGroupsId}],assignPublicIp=DISABLED}" \
   --load-balancers targetGroupArn=${TargetGroupArn},containerName=ContainerHandsOn,containerPort=80 
```

#### result

```Cloud9
{
    "service": {
        "serviceArn": "arn:aws:ecs:ap-northeast-1:152767562250:service/ContainerHandsOn/ContainerHandsOn",
        "serviceName": "ContainerHandsOn",
        "clusterArn": "arn:aws:ecs:ap-northeast-1:152767562250:cluster/ContainerHandsOn",
        "loadBalancers": [
            {
                "targetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7624dfd8f556a2d9",
                "containerName": "ContainerHandsOn",
                "containerPort": 80
            }
        ],
        "serviceRegistries": [],
        "status": "ACTIVE",
        "desiredCount": 2,
        "runningCount": 0,
        "pendingCount": 0,
        "launchType": "FARGATE",
        "platformVersion": "LATEST",
        "platformFamily": "Linux",
        "taskDefinition": "arn:aws:ecs:ap-northeast-1:152767562250:task-definition/ContainerHandsOn:1",
        "deploymentConfiguration": {
            "deploymentCircuitBreaker": {
                "enable": false,
                "rollback": false
            },
            "maximumPercent": 200,
            "minimumHealthyPercent": 100
        },
        "deployments": [
            {
                "id": "ecs-svc/9015413324295259653",
                "status": "PRIMARY",
                "taskDefinition": "arn:aws:ecs:ap-northeast-1:152767562250:task-definition/ContainerHandsOn:1",
                "desiredCount": 2,
                "pendingCount": 0,
                "runningCount": 0,
                "failedTasks": 0,
                "createdAt": "2022-09-07T02:14:18.688000+00:00",
                "updatedAt": "2022-09-07T02:14:18.688000+00:00",
                "launchType": "FARGATE",
                "platformVersion": "1.4.0",
                "platformFamily": "Linux",
                "networkConfiguration": {
                    "awsvpcConfiguration": {
                        "subnets": [
                            "subnet-0ea89b6bc85e0ec61",
                            "subnet-049f0119237ff00a0"
                        ],
                        "securityGroups": [
                            "sg-040aff209e1fe59cc"
                        ],
                        "assignPublicIp": "DISABLED"
                    }
                },
                "rolloutState": "IN_PROGRESS",
                "rolloutStateReason": "ECS deployment ecs-svc/9015413324295259653 in progress."
            }
        ],
        "roleArn": "arn:aws:iam::152767562250:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS",
        "events": [],
        "createdAt": "2022-09-07T02:14:18.688000+00:00",
        "placementConstraints": [],
        "placementStrategy": [],
        "networkConfiguration": {
            "awsvpcConfiguration": {
                "subnets": [
                    "subnet-0ea89b6bc85e0ec61",
                    "subnet-049f0119237ff00a0"
                ],
                "securityGroups": [
                    "sg-040aff209e1fe59cc"
                ],
                "assignPublicIp": "DISABLED"
            }
        },
        "healthCheckGracePeriodSeconds": 0,
        "schedulingStrategy": "REPLICA",
        "createdBy": "arn:aws:iam::152767562250:role/aws-reserved/sso.amazonaws.com/ap-northeast-1/AWSReservedSSO_AWSAdministratorAccess_8fe206b83490a022",
        "enableECSManagedTags": false,
        "propagateTags": "NONE",
        "enableExecuteCommand": false
    }
}
```

## 動作確認１

Duration: 0:05:00

### ■アドレス確認

#### cmd

```Cloud9
echo "http://"${LoadBalancersDnsName}
```

#### result

```Cloud9
http://ContainerHandsOn-610375823.ap-northeast-1.elb.amazonaws.com
```

### ■画面確認

- 上記で取得されたアドレスをChromeなどのブラウザに貼り付け、以下のような表示になること
- 更新を行うと2行目のhostnameが変更されていること（ALBで負荷分散されている確認）

![img](./image/img9-1.png)

![img](./image/img9-2.png)

## 変数整理

### ここまで取得された変数を整理

#### cmd

```Cloud9
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
export LoadBalancerArn="${LoadBalancerArn}"
export TargetGroupArn="${TargetGroupArn}"
export LoadBalancersDnsName="${LoadBalancersDnsName}"
EOF
```

#### result

```Cloud9
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
export LoadBalancerArn="arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:loadbalancer/app/ContainerHandsOn/75145ed42d9a3867"
export TargetGroupArn="arn:aws:elasticloadbalancing:ap-northeast-1:152767562250:targetgroup/ContainerHandsOn/7f11fa9e9d635ce9"
export LoadBalancersDnsName="ContainerHandsOn-610375823.ap-northeast-1.elb.amazonaws.com"
```

## CodeBuild作成

Duration: 0:05:00

## CodeDeploy作成

Duration: 0:05:00

## CodePipeline作成

Duration: 0:05:00

## Dockerコンテナ再ビルド(Codeシリーズを利用)

Duration: 0:05:00

## 動作確認２

Duration: 0:05:00

## 片付け

Duration: 0:05:00

### 途中の見出し

Positive
: 何かお知らせを書きたい時のボックス

Negative
: 何か注意点などを書きたい時のボックス
