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
Cloud-based computing usually involves groups of AWS resources, such as EC2 instances, EBS volumes, and so on, which must be created and managed collectively. For example, a web application typically requires application servers, database servers, load balancers, and so on. This group of instances is typically called a stack; a simple application server stack might look something like the following.

クラウドベースコンピューティングでは通常EC2インスタンス,EBSボリュームなどAWSリソースのグループを一緒に作成・管理する必要がある。例えばwebアプリケーションならアプリケーションサーバー、データベース・サーバー、ロードバランサーなどです。これらインスタンスグループを特にスタックと呼ぶことにします。

In addition to creating the instances and installing the necessary packages, you typically need a way to distribute applications to the application servers, monitor the stack's performance, manage security and permissions, and so on.

インスタンスを作り必要なパッケージをインストールすることに加えてディストリビューション固有のアプリケーションサーバー設定やパフォーマンスモニタ、セキュリティ管理などが必要になります。

AWS OpsWorks provides a simple and flexible way to create and manage stacks and applications. It supports a standard set of components—including application servers, database servers, load balancers, and more—that you can use to assemble your stack. These components all come with a standard configuration and are ready to run. However, you aren't limited to the standard components and configurations. AWS OpsWorks gives you the tools to customize the standard package configurations, install additional packages, and even create your own custom components. AWS OpsWorks also provides a way to manage related AWS resources, such as Elastic IP addresses and Amazon EBS volumes.

AWS OpsWorksは単純で柔軟なスタックとアプリケーションの作成・管理方法を提供します。標準的なアプリケーションサーバー、データーベースサーバー、ロードバランサー構成をスタックにまとめることができます。コンポーネントは全て標準設定済みで動作可能状態となっています。しかしながら標準的なコンポーネントと設定に制約されることはありません。AWS OpsWorksは追加パッケージのインストールやカスタムコンポーネント追加などを可能にするツールを提供します。AWS OpsWorksはまたElasitc IPアドレスやEBSボリュームなど関連するリソースを管理する方法も提供します。

Stacks

The stack is the core AWS OpsWorks component. It is basically a container for AWS resources—Amazon EC2 instances, Amazon EBS volumes, Elastic IP addresses, and so on—that have a common purpose and would be logically managed together. The stack helps you manage these resources as a group and also defines some default configuration settings, such as the instances' operating system and AWS region. If you want to isolate some stack components from direct user interaction, you can run the stack in a VPC.

スタック

スタックはAWS OpsWorksコンポーネントのコア部分です。EC2インスタンス、EBSボリューム、Elastic IPアドレスなど共通目的のAWS基本リソースで構成され全てを論理的に管理します。スタックはリソースをグループ単位で管理する手助けをしまた、インスタンのOSとAWSリージョンといって設定を定義します。もし、特定のスタックコンポーネントをユーザーからの直接アクセスから分離したいならVPC内でスタックを実行することができます。

Layers

You define the stack's components by adding one or more layers. A layer is basically a blueprint that specifies how to configure a set of Amazon EC2 instances for a particular purpose, such as serving applications or hosting a database server. You assign each instance to at least one layer, which determines what packages are to be installed on the instance, how they are configured, whether the instance has an Elastic IP address or Amazon EBS volume, and so on.

レイヤー

ひとつか複数のレイヤーを追加することでスタックのコンポーネントを定義できます。レイヤーはサーバーアプリケーションまたはデータベースホスティングといった特定のEC2インスタンス設定方法の基本設計となります。どのパッケージをインストールするかどの用に設定するかインスタンスにElastic IPアドレス、EBSボリュームをもたせるかを少なくともひとつのレイヤーに割り当てます。

If the built-in layers don't quite meet your requirements, you can customize or extend them by modifying packages' default configurations, adding custom Chef recipes to perform tasks such as installing additional packages, and more. You can also customize layers to work with AWS services that are not natively supported, such as using Amazon RDS as a database server. If that's still not enough, you can create a fully custom layer, which gives you complete control over which packages are installed, how they are configured, how applications are deployed, and more.

もしビルトインレイヤーが要求に合わないならカスタムChefレシピを追加することでカスタマイズや拡張をすることができます。もしそれでも不十分ならフルカスタムレイヤーを作ることができます。

Instances

Each layer has at least one instance. An instance represents an Amazon EC2 instance and defines its basic configuration, such as operating system and size. Other configuration settings, such as Elastic IP addresses or Amazon EBS volumes, are defined by the instance's layer. In addition, each layer has an associated set of Chef recipes that AWS OpsWorks runs on the layer's instances at key points in an instance's life cycle. A layer's Setup recipes perform tasks such as installing and configuring the layer's packages and starting daemons; the Deploy recipes handle downloading applications; and so on. You can also run recipes manually, at any time.

インスタンス

それぞれのレイヤーには少なくともひとつのインスタンスが存在します。インスタンスはEC2インスタンを表し、OSやサイズなど基本的な設定を定義します。Elastic IPアドレスやEBSボリュームなどはインスタンスのレイヤーで定義されます。加えてそれぞれのレイヤーにはインスタンスライフサイクルのキーポイントで実行されるChefレシピのセットが構成されています。レイヤーセットアップレシピはレイヤーのパッケージインストール・設定・デーモン起動・アプリケーションデプロイなどのタスクで構成されています。また、レシピはいつでも手動実行できます。

When you start an instance, AWS OpsWorks launches an Amazon EC2 instance using the configuration settings specified by the instance and its layer. After the Amazon EC2 has booted, AWS OpsWorks installs an agent that handles communication between the instance and the service. AWS OpsWorks then runs the layer's Setup recipes to install, configure, and start the layer's software, followed by the Deploy recipes, which install any applications.

インスタンス起動時、AWS OpsWorksはレイヤー固有の設定を使ってEC2インスタンスを起動します。EC2起動後、AWS OpsWorksはインスタンスとサービス間のやりとりをするエージェントをインストールします。それからAWS OpsWorksはレイヤーのソフトウェアをインストール、設定そして起動させるセットアップレシピを実行しアプリケーションをインストールするためデプロイレシピを完了させます。

For example, after you start an instance that belongs to the example's PHP App Server layer, AWS OpsWorks launches an Amazon EC2 instance and then runs a set of recipes that install, configure, and start a PHP application server and install PHP applications. An instance can even belong to multiple layers. In that case, AWS OpsWorks runs the recipes for each layer so you can, for example, have an instance that supports a PHP application server and a MySQL database server.

You can start instances in several ways.

24/7 instances are started manually and run until you stop them.

Load-based instances are automatically started and stopped by AWS OpsWorks, based on specified load metrics, such as CPU utilization. They allow your stack to automatically adjust the number of instances to accommodate variations in incoming traffic.

Time-based instances are run by AWS OpsWorks on a specified daily and weekly schedule. They allow your stack to automatically adjust the number of instances to accommodate predictable usage patterns.

AWS OpsWorks also supports autoheal instances. If an agent stops communicating with the service, AWS OpsWorks automatically stops and restarts the instance.

Apps

You store applications and related files in a repository such as an Amazon S3 bucket. Each application is represented by an app, which specifies the application type and contains the information that AWS OpsWorks needs to deploy the application from the repository to your instances. You can deploy apps in two ways:

Automatically—When you start an app server instance, AWS OpsWorks automatically deploys all apps of the appropriate type; Java apps are deployed to the Java App Server layer's instances, and so on.

Manually—If you have a new app or want to update an existing one, you can manually deploy it to your online instances.

When you deploy an app, AWS OpsWorks runs the Deploy recipes on the stack's instances. The app server layer's Deploy recipes download the app from the repository to the instance and perform related tasks such as configuring the server and restarting the daemon. You typically have AWS OpsWorks run the Deploy recipes on the entire stack, which allows the other layers' instances to modify their configuration appropriately. However, you can limit deployment to a subset of instances if, for example, you want to test a new app before deploying it to every app server instance.


アプリ

Amazon S3バケットなどレポジトリにアプリケーションと関連ファイルを保存できる。それぞれのアプリケーションはアプリケーションタイプとAWS OpsWorksがレポジトリからインスタンスにアプリケーションをデプロイするのに必要とする情報で構成されている。２つの方法でアプリケーションをデプロイできる。

自動-サーバーインスタンを起動した時、AWS OpsWorksは自動的に全ての適正なアプリをデプロイします。JavaアプリはJavaアプリサーバーレイヤーのインスタンスにデプロイされるといった具合です。

手動-既に存在するアプリを更新したい時、オンラインスタンスに手動デプロイができます。

アプリをデプロイした時、AWS OpsWorksはスタックのインスタンス上でデプロイレシピを実行します。アプリサーバーレイヤーのデプロイレシピはレポジトリからアプリをインスタンスにダウンロードしてサーバー設定やデーモン再起動など関連するタスクを実行します。特別に他のレイヤーのインスタンの設定を適正にするためにAWS OpsWorksを実行することもできます。しかしながら、全てのサーバーにデプロイする前に新しいアプリをテストしたいなどの場合にはデプロイを制限することもできます。

Customizing your Stack

The built-in layers provide standard functionality that is sufficient for many purposes. However, you might customize them a bit. AWS OpsWorks provides a variety of ways to customize layers to meet your specific requirements:

You can modify how AWS OpsWorks configures packages by overriding attributes that represent the various configuration settings, or by even overriding the templates used to create configuration files.

You can extend an existing layer by providing your own custom recipes to perform tasks such as running scripts or installing and configuring nonstandard packages.

You can create a fully custom layer by providing a set of recipes that handle all the tasks of installing packages, deploying apps, and so on.

You package your custom recipes and related files in one or more cookbooks and store the cookbooks in a repository such Amazon S3. You can run your custom recipes manually, but AWS OpsWorks also lets you automate the process by supporting a set of five lifecycle events:

Setup occurs on a new instance after it successfully boots.

Configure occurs on all of the stack's instances when an instance enters or leaves the online state.

Deploy occurs when you deploy an app.

Undeploy occurs when you delete an app.

Shutdown occurs when you stop an instance.

Each layer comes with a set of built-in recipes assigned to each of these events. When a lifecycle event occurs on a layer's instance, AWS OpsWorks runs the associated recipes. For example, when a Deploy event occurs on an app server instance, AWS OpsWorks runs the layer's built-in Deploy recipes to download the app.

You can run custom recipes automatically by assigning them to a layer's lifecycle events. AWS OpsWorks then automatically runs them for you after the built-in recipes have finished. For example, if you want to install an additional package on a layer's instances, write a recipe to install the package and assign it to the layer's Setup event. AWS OpsWorks will run it automatically on each of the layer's new instances, after the built-in Setup recipes are finished.

Resource Management

With AWS OpsWorks, you can use any of your account's Elastic IP address and Amazon EBS volume resources in a stack. You can use the AWS OpsWorks console or API to register resources with a stack, attach registered resources to or detach them from instances, and move resources from one instance to another.

Security and Permissions

AWS OpsWorks integrates with AWS Identity and Access Management (IAM) to provide robust ways of controlling how users access AWS OpsWorks, including the following:

How individual users can interact with each stack, such as whether they can create stack resources such as layers and instances, or whether they can use SSH to connect to a stack's EC2 instances.

How AWS OpsWorks can act on your behalf to interact with AWS resources such as Amazon EC2 instances.

How apps that run on AWS OpsWorks instances can access AWS resources such as Amazon S3 buckets.

How to manage public SSH keys and connect to an instance with SSH.

Monitoring and Logging

AWS OpsWorks provides several features to help you monitor your stack and troubleshoot issues with your stack and any custom recipes:

CloudWatch monitoring, which is summarized for your convenience on the OpsWorks Monitoring page.

CloudTrail support. which logs API calls made by or on behalf of AWS OpsWorks in your AWS account.

A Ganglia master layer that can be used to collect and display detailed monitoring data for the instances in your stack.

An event log that lists all events in your stack.

Chef logs that describe the details of what transpired for each lifecycle event on each instance, such as which recipes were run and what errors occurred.

CLI, SDK, and AWS CloudFormation Templates

In addition to the graphical console, AWS OpsWorks also supports a command-line interface (CLI) and SDKs for multiple languages that can be used to perform any operation. Consider these features:

The AWS OpsWorks CLI is part of the AWS CLI, and can be used to perform any operation from the command-line.

The AWS CLI supports multiple AWS services and can be installed on Windows, Linux, or OS X systems.

AWS OpsWorks is included in AWS Tools for Windows PowerShell and can be used to perform any operation from a PowerShell command line.

The AWS OpsWorks SDKs are included in the AWS SDK and can be used to perform any operation from applications implemented in: Java, JavaScript (browser-based and Node.js), .NET, PHP, Python (boto), or Ruby.

You can also use AWS CloudFormation templates to provision stacks. For some examples, see AWS OpsWorks Snippets.


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
