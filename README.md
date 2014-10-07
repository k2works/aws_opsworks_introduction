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
        "FromPort": "22",
        "ToPort": "22",
        "CidrIp": "0.0.0.0/0"
      },
      {
        "IpProtocol": "tcp",
        "FromPort": "80",
        "ToPort": "80",
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
    "InstanceType": "m1.small"
  }
},
```

# 参照

* [AWS OpsWorks](http://docs.aws.amazon.com/opsworks/latest/userguide/welcome.html)

* [Using Amazon RDS with Amazon Virtual Private Cloud (VPC)](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)
