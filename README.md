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

# 参照

* [AWS OpsWorks](http://docs.aws.amazon.com/opsworks/latest/userguide/welcome.html)
