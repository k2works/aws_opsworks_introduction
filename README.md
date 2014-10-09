AWS OpsWorks入門
===
# 目的
AWS OpesWorksを使ってインフラ環境を構築できるようになる。

# 前提
| ソフトウェア     | バージョン    | 備考         |
|:---------------|:-------------|:------------|
| AWS OpsWorks   |API Version 2013-02-18  |             |
| AWS CloudFormation  |API Version 2010-05-15        |             |
|           　　　|        |             |

# 構成
+ [AWS OpsWorksとは](#1)
+ [はじめに](#2)
+ [RDSと連携する](#3)

# 詳細
## <a name="1">AWS OpsWorksとは</a>

クラウドベースコンピューティングでは通常EC2インスタンス,EBSボリュームなどAWSリソースのグループを一緒に作成・管理する必要がある。例えばwebアプリケーションならアプリケーションサーバー、データベース・サーバー、ロードバランサーなどです。これらインスタンスグループを特にスタックと呼ぶことにします。

インスタンスを作り必要なパッケージをインストールすることに加えてディストリビューション固有のアプリケーションサーバー設定やパフォーマンスモニタ、セキュリティ管理などが必要になります。


AWS OpsWorksは単純で柔軟なスタックとアプリケーションの作成・管理方法を提供します。標準的なアプリケーションサーバー、データーベースサーバー、ロードバランサー構成をスタックにまとめることができます。コンポーネントは全て標準設定済みで動作可能状態となっています。しかしながら標準的なコンポーネントと設定に制約されることはありません。AWS OpsWorksは追加パッケージのインストールやカスタムコンポーネント追加などを可能にするツールを提供します。AWS OpsWorksはまたElasitc IPアドレスやEBSボリュームなど関連するリソースを管理する方法も提供します。


スタック

スタックはAWS OpsWorksコンポーネントのコア部分です。EC2インスタンス、EBSボリューム、Elastic IPアドレスなど共通目的のAWS基本リソースで構成され全てを論理的に管理します。スタックはリソースをグループ単位で管理する手助けをしまた、インスタンのOSとAWSリージョンといって設定を定義します。もし、特定のスタックコンポーネントをユーザーからの直接アクセスから分離したいならVPC内でスタックを実行することができます。

レイヤー

ひとつか複数のレイヤーを追加することでスタックのコンポーネントを定義できます。レイヤーはサーバーアプリケーションまたはデータベースホスティングといった特定のEC2インスタンス設定方法の基本設計となります。どのパッケージをインストールするかどの用に設定するかインスタンスにElastic IPアドレス、EBSボリュームをもたせるかを少なくともひとつのレイヤーに割り当てます。


もしビルトインレイヤーが要求に合わないならカスタムChefレシピを追加することでカスタマイズや拡張をすることができます。もしそれでも不十分ならフルカスタムレイヤーを作ることができます。

インスタンス

それぞれのレイヤーには少なくともひとつのインスタンスが存在します。インスタンスはEC2インスタンを表し、OSやサイズなど基本的な設定を定義します。Elastic IPアドレスやEBSボリュームなどはインスタンスのレイヤーで定義されます。加えてそれぞれのレイヤーにはインスタンスライフサイクルのキーポイントで実行されるChefレシピのセットが構成されています。レイヤーセットアップレシピはレイヤーのパッケージインストール・設定・デーモン起動・アプリケーションデプロイなどのタスクで構成されています。また、レシピはいつでも手動実行できます。

インスタンス起動時、AWS OpsWorksはレイヤー固有の設定を使ってEC2インスタンスを起動します。EC2起動後、AWS OpsWorksはインスタンスとサービス間のやりとりをするエージェントをインストールします。それからAWS OpsWorksはレイヤーのソフトウェアをインストール、設定そして起動させるセットアップレシピを実行しアプリケーションをインストールするためデプロイレシピを完了させます。

アプリ

Amazon S3バケットなどレポジトリにアプリケーションと関連ファイルを保存できる。それぞれのアプリケーションはアプリケーションタイプとAWS OpsWorksがレポジトリからインスタンスにアプリケーションをデプロイするのに必要とする情報で構成されている。２つの方法でアプリケーションをデプロイできる。

自動-サーバーインスタンを起動した時、AWS OpsWorksは自動的に全ての適正なアプリをデプロイします。JavaアプリはJavaアプリサーバーレイヤーのインスタンスにデプロイされるといった具合です。

手動-既に存在するアプリを更新したい時、オンラインスタンスに手動デプロイができます。

アプリをデプロイした時、AWS OpsWorksはスタックのインスタンス上でデプロイレシピを実行します。アプリサーバーレイヤーのデプロイレシピはレポジトリからアプリをインスタンスにダウンロードしてサーバー設定やデーモン再起動など関連するタスクを実行します。特別に他のレイヤーのインスタンの設定を適正にするためにAWS OpsWorksを実行することもできます。しかしながら、全てのサーバーにデプロイする前に新しいアプリをテストしたいなどの場合にはデプロイを制限することもできます。

## <a name="2">はじめに</a>
### シンプルなアプリケーションサーバースタックを作る
#### スタックを作る

_template/OpsWorks-step2-01.json_

```json
"myStack": {
  "Type": "AWS::OpsWorks::Stack",
  "Properties": {
    "Name": {
      "Ref": "AWS::StackName"
    },
    "ServiceRoleArn": {
      "Fn::GetAtt": [ "OpsWorksServiceRole", "Arn"]
    },
    "DefaultInstanceProfileArn": {
      "Fn::GetAtt": [ "OpsWorksInstanceProfile", "Arn"]
    }
  }
},
```

#### PHPアプリケーションサーバーレイヤーを追加する

_template/OpsWorks-step2-02.json_

```json
"myLayer": {
  "Type": "AWS::OpsWorks::Layer",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "MyPHPApp",
    "Type": "php-app",
    "Shortname": "php-app",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true"
  }
},
```

#### PHPアプリケーションサーバーレイヤーにインスタンスを追加する

_template/OpsWorks-step2-03.json_

```json
"myInstance1": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "LayerIds": [
      {
        "Ref": "myLayer"
      }
    ],
    "InstanceType": "m1.small"
  }
},
```

#### アプリケーションを作成してデプロイする

_template/OpsWorks-step2-03.json_

```json
"myLayer": {
  "Type": "AWS::OpsWorks::Layer",
  "DependsOn": "myApp",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "MyPHPApp",
    "Type": "php-app",
    "Shortname": "php-app",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true"
  }
},

"myApp": {
  "Type": "AWS::OpsWorks::App",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "SimplePHPApp",
    "Type": "php",
    "AppSource": {
      "Type": "git",
      "Url": "git://github.com/amazonwebservices/opsworks-demo-php-simple-app.git",
      "Revision": "version1"
    },
    "Attributes": {
      "DocumentRoot": " "
    }
  }
},
```

![001](https://farm4.staticflickr.com/3928/15434098266_5f0a062b31.jpg)

![002](https://farm4.staticflickr.com/3936/15270568438_7c15abbfae.jpg)

### バックエンドにデータストアを追加する
#### バックエンドデータベースを追加

_template/OpsWorks-step3-01.json_

```json
"myLayer2": {
  "Type": "AWS::OpsWorks::Layer",
  "DependsOn": "myApp",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "MySQL",
    "Type": "db-master",
    "Shortname": "db-master",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true",
    "Attributes": {
          "MysqlRootPassword" : "mysqlpasswd"
      },
    "VolumeConfigurations": [
    {
        "MountPoint": "/vol/mysql",
        "NumberOfDisks": "1",
        "Size": "10"
    }
    ]
  }
},
・・・
"myInstance2": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "LayerIds": [
      {
        "Ref": "myLayer2"
      }
    ],
    "InstanceType": "m1.small"
  }
},
```

![003](https://farm6.staticflickr.com/5600/15456839712_71f1cdee0a.jpg)

#### SimplePHPAppを更新する

_template/OpsWorks-step3-02.json_

```json
"myApp": {
  "Type": "AWS::OpsWorks::App",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "SimplePHPApp",
    "Type": "php",
    "AppSource": {
      "Type": "git",
      "Url": "git://github.com/amazonwebservices/opsworks-demo-php-simple-app.git",
      "Revision": "version2"
    },
    "Attributes": {
      "DocumentRoot": "web"
    }
  }
},
```

#### カスタムクックブックをスタックに追加する

_template/OpsWorks-step3-03.json_

```json
"myStack": {
  "Type": "AWS::OpsWorks::Stack",
  "Properties": {
    "Name": {
      "Ref": "AWS::StackName"
    },
    "ServiceRoleArn": {
      "Fn::GetAtt": [ "OpsWorksServiceRole", "Arn"]
    },
    "DefaultInstanceProfileArn": {
      "Fn::GetAtt": [ "OpsWorksInstanceProfile", "Arn"]
    },
    "CustomCookbooksSource" : {
        "Type" : "git",
        "Url" : "git://github.com/amazonwebservices/opsworks-example-cookbooks.git"
    },
    "UseCustomCookbooks" : "true"
  }
},
```

#### レシピを実行する

_template/OpsWorks-step3-04.json_

```json
"myLayer2": {
  "Type": "AWS::OpsWorks::Layer",
  "DependsOn": "myApp",
  "Properties": {
    "StackId": {
      "Ref": "myStack"
    },
    "Name": "MySQL",
    "Type": "db-master",
    "Shortname": "db-master",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true",
    "Attributes": {
          "MysqlRootPassword" : "mysqlpasswd"
      },
    "VolumeConfigurations": [
    {
        "MountPoint": "/vol/mysql",
        "NumberOfDisks": "1",
        "Size": "10"
    }
    ],
    "CustomRecipes" : {
      "Configure" : [ ],
      "Deploy" : ["phpapp::dbsetup"],
      "Setup" : [ ],
      "Shutdown" : [ ],
      "Undeploy" : [ ]
      }
  }
},
```

#### SimplePHPApp, Version 2をデプロイする

![004](https://farm6.staticflickr.com/5599/15270419359_639c317b7d.jpg)

#### SimplePHPAppを実行する

![005](https://farm6.staticflickr.com/5598/15434098316_353c2cb0f2.jpg)

[PHPのサンプルアプリVersion2が動かない](https://github.com/k2works/aws_opsworks_introduction/issues/1)

### スタックをスケールアウトする

#### VPC内でスタックを実行する

_template/OpsWorks-step4-01.json_

#### ロードバランサーを追加する

_template/OpsWorks-step4-02.json_

```json
"ELB": {
  "Type": "AWS::ElasticLoadBalancing::LoadBalancer",
  "Properties": {
    "SecurityGroups": [
      {
        "Ref": "ELBSecurityGroup"
      }
    ],
    "Subnets": [
      {
        "Ref": "PublicSubnet"
      }
    ],
    "Listeners": [
      {
        "LoadBalancerPort": "80",
        "InstancePort": "80",
        "Protocol": "HTTP"
      }
    ],
    "HealthCheck": {
      "Target": "HTTP:80/",
      "HealthyThreshold": "3",
      "UnhealthyThreshold": "5",
      "Interval": "90",
      "Timeout": "60"
    }
  }
},
"ELBAttachment": {
  "Type": "AWS::OpsWorks::ElasticLoadBalancerAttachment",
  "Properties": {
    "ElasticLoadBalancerName": {
      "Ref": "ELB"
    },
    "LayerId": {
      "Ref": "OpsWorksLayer"
    }
  }
},
"ELBSecurityGroup": {
  "Type": "AWS::EC2::SecurityGroup",
  "Properties": {
    "GroupDescription" : "Allow inbound access to the ELB",
    "VpcId": {
      "Ref": "VPC"
    },
    "SecurityGroupIngress": [
      {
        "IpProtocol": "tcp",
        "FromPort": "80",
        "ToPort": "80",
        "CidrIp": "0.0.0.0/0"
      }
    ],
    "SecurityGroupEgress": [
      {
        "IpProtocol": "tcp",
        "FromPort": "80",
        "ToPort": "80",
        "CidrIp": "0.0.0.0/0"
      }
    ]
  }
},
```

```json
"OpsWorksSecurityGroup": {
  "Type": "AWS::EC2::SecurityGroup",
  "Properties": {
    "GroupDescription" : "Allow inbound requests from the ELB to the OpsWorks instances",
    "VpcId": {
      "Ref": "VPC"
    },
    "SecurityGroupIngress": [
      {
        "IpProtocol": "tcp",
        "FromPort": "80",
        "ToPort": "80",
        "SourceSecurityGroupId": {
          "Ref": "ELBSecurityGroup"
        }
      }
    ]
  }
},
```

```json
"OpsWorksLayer": {
  "Type": "AWS::OpsWorks::Layer",
  "Metadata" : {
    "Comment" : "OpsWorks instances require outbound Internet access. Using DependsOn to make sure outbound Internet Access is estlablished before creating instances in this layer."
  },
  "DependsOn": [ "NATIPAddress", "PublicRoute", "PublicSubnetRouteTableAssociation", "PrivateRoute", "PrivateSubnetRouteTableAssociation"],
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "Name": "MyPHPApp",
    "Type": "php-app",
    "Shortname": "php-app",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true",
    "CustomSecurityGroupIds": [
      {
        "Ref": "OpsWorksSecurityGroup"
      }
    ]
  }
},
```

#### PHPアプリケーションサーバーインスタンス追加

_template/OpsWorks-step4-03.json_

```json
"OpsWorksInstance1": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "LayerIds": [
      {
        "Ref": "OpsWorksLayer"
      }
    ],
    "InstanceType": "m1.small"
  }
},
"OpsWorksInstance2": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "LayerIds": [
      {
        "Ref": "OpsWorksLayer"
      }
    ],
    "InstanceType": "m1.small"
  }
},
"OpsWorksApp": {
  "Type": "AWS::OpsWorks::App",
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "Name": "MyPHPApp",
    "Type": "php",
    "AppSource": {
      "Type": "git",
      "Url": "git://github.com/amazonwebservices/opsworks-demo-php-simple-app.git",
      "Revision": "version1"
    },
    "Attributes": {
      "DocumentRoot": " "
    }
  }
},
```

```json
"OpsWorksLayer": {
  "Type": "AWS::OpsWorks::Layer",
  "Metadata" : {
    "Comment" : "OpsWorks instances require outbound Internet access. Using DependsOn to make sure outbound Internet Access is estlablished before creating instances in this layer."
  },
  "DependsOn": [ "NATIPAddress", "PublicRoute", "PublicSubnetRouteTableAssociation", "PrivateRoute", "PrivateSubnetRouteTableAssociation", "OpsWorksApp"],
・・・
```

#### PHPアプリケーションの確認

![006](https://farm6.staticflickr.com/5599/15465836952_baf5fb3602.jpg)

![002](https://farm4.staticflickr.com/3936/15270568438_7c15abbfae.jpg)

## <a name="3">RDSと連携する</a>

### VPCに対応したスタックを作る

_template/OpsWorks-step5-01.json_

### RDSを作る

_template/OpsWorks-step5-02.json_

```json
"Parameters" : {
  "DBName": {
    "Default": "MyDatabase",
    "Description" : "The database name",
    "Type": "String",
    "MinLength": "1",
    "MaxLength": "64",
    "AllowedPattern" : "[a-zA-Z][a-zA-Z0-9]*",
    "ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
  },
  "DBUsername": {
    "Default": "admin",
    "NoEcho": "true",
    "Description" : "The database admin account username",
    "Type": "String",
    "MinLength": "1",
    "MaxLength": "16",
    "AllowedPattern" : "[a-zA-Z][a-zA-Z0-9]*",
    "ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
  },
  "DBPassword": {
    "Default": "password",
    "NoEcho": "true",
    "Description" : "The database admin account password",
    "Type": "String",
    "MinLength": "8",
    "MaxLength": "41",
    "AllowedPattern" : "[a-zA-Z0-9]*",
    "ConstraintDescription" : "must contain only alphanumeric characters."
  },
  "DBClass" : {
    "Default" : "db.m1.small",
    "Description" : "Database instance class",
    "Type" : "String",
    "AllowedValues" : [ "db.m1.small", "db.m1.large", "db.m1.xlarge", "db.m2.xlarge", "db.m2.2xlarge", "db.m2.4xlarge" ],
    "ConstraintDescription" : "must select a valid database instance type."
  },
  "DBAllocatedStorage" : {
    "Default": "5",
    "Description" : "The size of the database (Gb)",
    "Type": "Number",
    "MinValue": "5",
    "MaxValue": "1024",
    "ConstraintDescription" : "must be between 5 and 1024Gb."
  }
},
・・・
"OpsWorksServiceRole": {
  "Type": "AWS::IAM::Role",
  "Properties": {
    "AssumeRolePolicyDocument": {
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": [
              "opsworks.amazonaws.com"
            ]
          },
          "Action": [
            "sts:AssumeRole"
          ]
        }
      ]
    },
    "Path": "/",
    "Policies": [
      {
        "PolicyName": "opsworks-service",
        "PolicyDocument": {
          "Statement": [
            {
              "Effect": "Allow",
              "Action": [
                "ec2:*",
                "iam:PassRole",
                "cloudwatch:GetMetricStatistics",
                "elasticloadbalancing:*",
                "rds:*"
              ],
              "Resource": "*"
            }
          ]
        }
      }
    ]
  }
},
・・・
"MyDBSubnetGroup" : {
  "Type" : "AWS::RDS::DBSubnetGroup",
  "Properties" : {
    "DBSubnetGroupDescription" : "Subnets available for the RDS DB Instance",
    "SubnetIds" : [ { "Ref" : "PublicSubnet" },
                    { "Ref" : "PrivateSubnet" } ]
  }
},
"myVPCSecurityGroup" : {
    "Type" : "AWS::EC2::SecurityGroup",
    "Properties" :
    {
       "GroupDescription" : "Security group for RDS DB Instance.",
       "VpcId" : {
         "Ref" : "VPC"
       },
      "SecurityGroupIngress": [
      {
        "IpProtocol": "tcp",
        "FromPort": "3306",
        "ToPort": "3306",
        "CidrIp": "0.0.0.0/0"
      },
      {
        "IpProtocol": "icmp",
        "FromPort": "-1",
        "ToPort": "-1",
        "CidrIp": "0.0.0.0/0"
      }
    ],
    "SecurityGroupEgress": [
      {
        "IpProtocol": "-1",
        "CidrIp": "0.0.0.0/0"
      }
    ]
    }
},
"MyDB" : {
  "Type" : "AWS::RDS::DBInstance",
  "Properties" : {
    "DBName" : { "Ref" : "DBName" },
    "AllocatedStorage" : { "Ref" : "DBAllocatedStorage" },
    "DBInstanceClass" : { "Ref" : "DBClass" },
    "Engine" : "MySQL",
    "EngineVersion" : "5.5",
    "MasterUsername" : { "Ref" : "DBUsername" } ,
    "MasterUserPassword" : { "Ref" : "DBPassword" },
    "DBSubnetGroupName" : { "Ref" : "MyDBSubnetGroup" },
    "VPCSecurityGroups" : [ { "Ref" : "myVPCSecurityGroup" }  ]
  }
}
},
```

### レイヤーにRDSを追加してアプリケーションに対応づける

_template/OpsWorks-step5-03.json_

```json
"OpsWorksLayer": {
  "Type": "AWS::OpsWorks::Layer",
  "Metadata" : {
    "Comment" : "OpsWorks instances require outbound Internet access. Using DependsOn to make sure outbound Internet Access is estlablished before creating instances in this layer."
  },
  "DependsOn": [ "PublicRoute", "PublicSubnetRouteTableAssociation", "PrivateSubnetRouteTableAssociation"],
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "Name": "MyPHPApp",
    "Type": "php-app",
    "Shortname": "php-app",
    "EnableAutoHealing": "true",
    "AutoAssignElasticIps": "false",
    "AutoAssignPublicIps": "true",
    "CustomSecurityGroupIds": [
      {
        "Ref": "OpsWorksSecurityGroup"
      }
    ]
  }
},
"myApp": {
  "Type": "AWS::OpsWorks::App",
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "Name": "SimplePHPApp",
    "Type": "php",
    "AppSource": {
      "Type": "git",
      "Url": "git://github.com/amazonwebservices/opsworks-demo-php-simple-app.git",
      "Revision": "version2"
    },
    "Attributes": {
      "DocumentRoot": "web"
    }
  }
},
"myInstance1": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "StackId": {
      "Ref": "OpsWorksStack"
    },
    "LayerIds": [
      {
        "Ref": "OpsWorksLayer"
      }
    ],
    "InstanceType": "m1.small",
    "SubnetId" : {
      "Ref": "PublicSubnet"
    }
  }
},
```

### VPC内でRDSに接続できるか確認

![007](https://farm3.staticflickr.com/2945/15298615418_50c9d4d4dd.jpg)

![008](https://farm6.staticflickr.com/5616/15485268145_b6d4b89b49.jpg)

![009](https://farm4.staticflickr.com/3929/15482123691_b36f97d710.jpg)

```bash
$ ssh -i my-key.pem ec2-user@54.64.44.45
$ mysql -h am1n7ngscv4mg6x.ccebcwxdebti.ap-northeast-1.rds.amazonaws.com -uadmin -ppassword
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 27
Server version: 5.5.37-log Source distribution

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### OpsWorksからRDSにアクセスできるようにする

_template/OpsWorks-step5-03.json_

```json
"OpsWorksServiceRole": {
  "Type": "AWS::IAM::Role",
  "Properties": {
    "AssumeRolePolicyDocument": {
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": [
              "opsworks.amazonaws.com"
            ]
          },
          "Action": [
            "sts:AssumeRole"
          ]
        }
      ]
    },
    "Path": "/",
    "Policies": [
      {
        "PolicyName": "opsworks-service",
        "PolicyDocument": {
          "Statement": [
            {
              "Effect": "Allow",
              "Action": [
                "ec2:*",
                "iam:PassRole",
                "cloudwatch:GetMetricStatistics",
                "elasticloadbalancing:*",
                "rds:*"
              ],
              "Resource": "*"
            }
          ]
        }
      }
    ]
  }
```

![010](https://farm3.staticflickr.com/2945/15298361829_86a9aabc24.jpg)

![011](https://farm6.staticflickr.com/5600/15298615338_a7734fe0a9.jpg)

![012](https://farm4.staticflickr.com/3939/15298693497_13ba958dee.jpg)

![013](https://farm3.staticflickr.com/2945/15298693487_e193931a8c.jpg)

# 参照

* [AWS OpsWorks](http://docs.aws.amazon.com/opsworks/latest/userguide/welcome.html)

* [Using Amazon RDS with Amazon Virtual Private Cloud (VPC)](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)
