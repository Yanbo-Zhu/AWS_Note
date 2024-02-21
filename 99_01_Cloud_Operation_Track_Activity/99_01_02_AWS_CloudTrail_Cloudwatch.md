
# 1 CloudTrail

==Auditing service for aws accounts==

随着基于AWS构建的应用程序的扩展和日常管理的增加，企业的合规审计以及在AWS上的安全防护成为一项越来越复杂的任务。对于企业而言，时刻面临着云上的基础设施以及应用程序被攻击的可能，因此，监控AWS账户行为的操作、识别可能的恶意行为和检测可能的错误配置是企业安全合规的重要内容。AWS CloudTrai记录的审计日志信息为企业的安全合规提供了基础的数据，根据CloudTrail的审计日志就可以构建出企业的云上审计方案。

![](image/Pasted%20image%2020240220093808.png)
## 1.1 什么是CloudTrail

CloudTrail是AWS提供的一项服务，用于记录用户、角色和服务所执行的操作，这些操作包括在AWS控制台、AWS命令行界面、AWS开放工具包以及API中执行的操作。CloudTrail将这些操作记录为事件，通过在CloudTrail中创建一个Trail，就可以持续记录AWS账户的活动和事件，从而帮助用户对AWS账户进行监管、合规性以及运营和风险审计。

> 关键词：事件，账户，审计日志，监控

AWS CloudTrail是亚马逊网络服务（AWS）的另一项服务，为你的AWS账户提供审计、治理、监控、合规性和风险监控。
与CloudWatch不同，CloudTrail是一个管理和治理工具，让你观察AWS账户相关活动的整个事件历史。
CloudTrail是一种记录服务，可以记录从任何外部工具产生的事件或行动，如AWS控制台、AWS CLI和SDK。你还可以使用CloudTrail轻松地检测你的账户中的任何异常活动

![](image/Pasted%20image%2020240221195506.png)


![](image/Pasted%20image%2020240221204651.png)



![](image/Pasted%20image%2020240221195554.png)


 **其好处是什么？**
通过它记录的事件历史，极大简化了安全分析、资源变化跟踪和故障排除的工作。

 **那么其工作原理是什么呢？**
AWS CloudTrail记录在给定的AWS设置中执行的活动，并检测任何不寻常的API使用，然后它还进行l事件跟踪和活动记录。产生的这些事件将被传递到AWS CloudTrail控制台、CloudWatch日志和S3桶。
通过使用 CloudWatch 事件和Alarms，CloudTrail 在发现任何异常事件时都会采取必要的行动。用户可以在 CloudTrail 控制台中查看所有最近的行动和事件，同时还可以使用历史记录下载 CloudTrail 活动记录。

## 1.2 use case example

![](image/Pasted%20image%2020240221195857.png)


## 1.3 事件类型 

在创建Trail时，用户可以选择记录管理事件、数据事件以及洞察事件，默认记录所有的管理事件。当AWS账户中发生事件时，CloudTrail会评估该事件是否与用户的Trail的配置匹配，只有匹配了的事件才会传送到Amazon S3存储桶。

管理事件/ Management Events
管理事件指AWS账户中对资源执行的管理操作，这包括读取、创建、删除和修改AWS的资产，例如EC2实例和S3存储桶。对于API活动，可以选择记录 **All**、**Read-only**或 **Write-only**管理事件，如果不想记录AWS KMS 和Amazon RDS 数据 API 事件，可以选择排除这两种事件。

数据事件/ DATA Events
数据事件指在创建的AWS资源内执行的数据请求操作，通常是一些涉及到大量数据活动的事件。创建Trail时默认不启动数据事件，因为它会产生额外的收费。在数据事件中，可以记录 Amazon S3 存储桶、AWS Lambda 函数、Amazon DynamoDB 表或这些表的组合的数据事件。

洞察事件/ Insight 
洞察事件指的是CloudTrail通过分析记录的管理事件，在识别出API调用相关的异常活动时记录的事件。洞察类型包括API调用率和API错误率，CloudTrail通过数学模型来确定账户的API事件的正常水平（称之为基线，在洞察事件开始前的7天内计算的），在调用率和错误率超出基线时就会生成洞察事件，并且投递到S3存储桶目标文件夹的的/CloudTrail-Insight目录下。

### 1.3.1 CloudTrail日志示例
a management event example 

CloudTrail存储到S3的日志文件以JSON的格式编写，每个文件中包含一条或多条记录，如下所示为CloudTrail记录的一个示例事件，关于事件中每个字段的详细含义，可以参考[CloudTrail record contents](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html)。

```text
{
    "eventVersion": "1.08",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::123456789012:user/Alice",
        "accountId": "123456789012",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "userName": "Alice"
    },
    "eventTime": "2022-03-24T21:11:59Z",
    "eventSource": "iam.amazonaws.com",
    "eventName": "CreateUser",
    "awsRegion": "us-east-2",
    "sourceIPAddress": "127.0.0.1",
    "userAgent": "aws-cli/1.3.2 Python/2.7.5 Windows/7",
    "requestParameters": {"userName": "Bob"},
    "responseElements": {"user": {
        "createDate": "Mar 24, 2022 9:11:59 PM",
        "userName": "Bob",
        "arn": "arn:aws:iam::123456789012:user/Bob",
        "path": "/",
        "userId": "EXAMPLEUSERID"
    }}
}
```

这条日志显示了IAM用户Alice使用AWS CLI调用了CreateUser操作创建了名为Bob的新用户。对于CloudTrail记录的事件，包含了一些重要的字段，例如执行操作的AWS用户名称以及访问密钥ID（userIdentity字段），执行操作的详细信息（eventTime、eventSource和eventName字段）。对于管理事件和数据事件，日志还包含了requestParameters和responseElements来提供了执行操作的请求参数以及响应元素，通过响应元素可以帮助用户确认操作的执行情况，如果执行失败的话，日志中会包含errorCode和errorMessage两个字段来说明失败的原因。

## 1.4 SLS的CloudTrail审计方案

对于海量的审计数据，往往需要较大的存储成本。相对于一些商业化SaaS，例如DataDog、Sumo Logic等，SLS只需要极低的成本就可以开启CloudTrail的审计方案，例如每天写入1百万条日志，并且用户开启了智能冷热分层存储（热存设置为30天），那么用户一年的存储成本大约为253元（约39美元）。

除了成本极低以外，使用SLS作为AWS CloudTrail的监控分析平台具有以下优势：

- 集成了AWS CloudTrail审计应用，简化日志接入流程，提供一站式审计服务。
- 免费提供查询、报表分析以及告警监控的功能，极低的存储成本。对于告警监控功能，只收取语音通知和短信通知费用。
- 灵活、高性能的查询分析能力。支持PB级数据实时查询与分析，提供10多种查询运算符、10多种机器学习函数、100多个SQL函数，支持千亿级数据高性能分析，能够快速支撑审计场景下的查询监控。
- 丰富的报表和告警内置规则，强大的智能告警系统。应用内置了开箱即用的仪表盘，包括总览、登录审计、S3数据事件、IAM审计、网络和安全审计报表，还内置了涵盖账号安全、多种AWS云产品操作合规以及API调用等关键方向的告警规则。另外，日志服务告警是一站式告警监控、降噪、事务管理、通知分派的智能运维平台，具有高可用和高可靠性。
- SLS具有强大完善的开放平台能力。支持通过SDK/API查询消费，也提供开箱即用的组件与其他平台工具对接，包括数仓如MaxCompute和OSS等、流计算平台如Flink和Spark Streaming等、可视化框架如Grafana和DataV等

### 1.4.1 将CloudTrail日志接入到SLS

SLS集成的AWS CloudTrail审计应用的工作原理如下图所示，在AWS CloudTrail中创建Trail后，还需要在Amazon SQS中创建队列，才能拉取到AWS CloudTrail数据。在AWS上需要进行的准备工作如下：

1. 在CloudTrail中创建一个Trail，具体参考[创建一个Trail](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail-by-using-the-console.html)
2. 在SQS中创建一个Queue，具体参考[创建一个队列](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-create-queue.html)
3. 在第一步创建的Trail对应的S3 Bucket中，配置事件通知，目的地选择第2步创建的Queue，具体参考[S3配置事件通知](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)

  
![](https://pic3.zhimg.com/80/v2-0d71831371f2e8cdab9e4cc84c27a87e_720w.webp)

  

完成上面的步骤后，就可以在[AWS CloudTrai审计应用](https://link.zhihu.com/?target=https%3A//sls.console.aliyun.com/lognext/app/awslens/aws_cloudtrail%3Fresource%3D/aws-data-access)中创建对应的导入任务，导入任务的参数配置可以参考[创建配置](https://link.zhihu.com/?target=https%3A//help.aliyun.com/document_detail/423444.html)。

## 1.5 开启您的安全审计方案

将CloudTrail日志导入到SLS后，就可以轻松开启您的安全审计方案，具体包括了以下能力。

### 1.5.1 探索CloudTrail日志-查询分析

在导入CloudTrail日志的时候，SLS会自动根据CloudTrail日志的字段创建索引，因此用户可以直接在查询分析框内输入对应的查询分析语句来探索CloudTrail日志。下图所示为查询15分钟内所有事件名称为**GetObject**的日志，返回的内容包括查询到的符合条件的日志总条数以及原始日志。

![](https://pic3.zhimg.com/80/v2-edabf6daddb6e9837db134511db4401e_720w.webp)

除了根据索引字段过滤和查询原始的CloudTrail日志，还可以利用SQL进行统计分析。下图所示为统计1天内每隔15分钟所产生的日志数量，并且以线图的形式将统计分析的结果展示出来。

![](https://pic4.zhimg.com/80/v2-778dbd2abe44cc61192b9c88810e8ce7_720w.webp)

  
### 1.5.2 数据可视化-报表

AWS CloudTrail审计应用提供开箱即用的仪表盘，包括总览、登录审计、S3数据事件、IAM审计、网络和安全审计，用于分析和审计AWS账户的各类事件。下图所示为总览和S3数据事件的内置报表，更多信息可以参考[查看数据报表](https://link.zhihu.com/?target=https%3A//help.aliyun.com/document_detail/425083.html)。除了内置的报表以外，用户还可以根据自己的需求深度自定义报表，在查询分析页面将统计分析结果保存到仪表盘就可以完成报表的定制。

![](https://pic1.zhimg.com/80/v2-c77ffc5f434f9f8f279cf1238e717110_720w.webp)

  
![](https://pic2.zhimg.com/80/v2-9b82acfed2b16ba8fd83765321feffa1_720w.webp)


### 1.5.3 实时检测安全威胁-监控告警

AWS CloudTrail应用内置众多开箱即用的告警规则，用户开启后，SLS就会实时监控导入的CloudTrail日志，一旦检测到与告警规则相匹配的事件时，日志服务就会产生一个告警，并且根据用户配置的告警策略和行动策略进行告警通知。应用内置的告警规则涵盖了账号安全、多种AWS云产品操作合规以及API调用等关键方向的审计日志，除此以外，用户也可以根据自己的需求自定义告警规则，监测符合自身安全合规的审计日志。


## 1.6 Pricing 

![](image/Pasted%20image%2020240221200408.png)

## 1.7 示例

the historic access to the dynamodb table of all control panel or management operations 

![](image/Pasted%20image%2020240221200647.png)


# 2 Cloudwatch 


==Monitoring service for applications ==

![](image/Pasted%20image%2020240221204017.png)

Cloudwatch Rules (formly called Cloudwatch event): 建立 serverless cron jobs, in order to invoke a certain function or perform a certain action at a regular intervals or fixed intervals 
Cloudwatch EventBridge: are application events that you can integrate into an eventbus and respond to programmatically 

This 2 things are not really related to cloudwatch. They just kind of evolved over time and still exist within the cloudwatch section console 

# 3 CloudWatch 和 CloudTrail 有什么区别


包括但不限于下面的一些区别。

|比较项|CloudWatch|CloudTrail|
|---|---|---|
|监控方向|一个针对AWS资源和应用的监控服务，报告应用程序日志，是一个接近实时的系统事件流，描述了你的AWS资源的变化的服务|一项网络服务，记录你的AWS账户中的API活动，提供关于AWS账户中发生的具体信息，是一个更注重在你的AWS账户中进行的AWS API调用的服务|
|基本操作|默认为您的资源提供免费的基本监控，如EC2实例、EBS卷和RDS DB实例|当创建AWS账户时，CloudTrail也被默认启用|
|能力|可收集和跟踪指标，收集和监控日志文件，并设置警报|记录了谁提出了请求、使用的服务、执行的操作、操作的参数以及AWS服务返回的响应元素等信息，然后存储到指定位置|
|监控频率|在基本监控中以 5 分钟为周期交付指标数据，在详细监控中以 1 分钟为周期交付指标数据；其日志代理默认每五秒发送一次日志数据|在 API 调用后 15 分钟内提供事件|

  
![](image/Pasted%20image%2020240221204728.png)


