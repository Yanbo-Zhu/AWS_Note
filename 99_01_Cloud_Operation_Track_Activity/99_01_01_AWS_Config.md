
https://zhuanlan.zhihu.com/p/336676172#:~:text=%E4%B8%80%E3%80%81%E5%8A%9F%E8%83%BD%E7%AE%80%E4%BB%8B%201%20%E4%BD%BF%E7%94%A8%20AWS%20Config%20%E5%AE%9A%E4%B9%89%E9%A2%84%E7%BD%AE%E5%92%8C%E9%85%8D%E7%BD%AE%20AWS%20%E8%B5%84%E6%BA%90%E7%9A%84%E8%A7%84%E5%88%99%EF%BC%8C%E6%8C%81%E7%BB%AD%E5%AE%A1%E8%AE%A1%E5%92%8C%E8%AF%84%E4%BC%B0,Service%20%28SNS%29%20%E9%80%9A%E7%9F%A5%E5%92%8C%20Amazon%20CloudWatch%20%E4%BA%8B%E4%BB%B6%E3%80%82%203%20%E5%8F%AF%E4%BB%A5%E5%88%A9%E7%94%A8%E5%8F%AF%E8%A7%86%E5%8C%96%E6%8E%A7%E5%88%B6%E9%9D%A2%E6%9D%BF%E6%9D%A5%E6%9F%A5%E7%9C%8B%E6%95%B4%E4%BD%93%E5%90%88%E8%A7%84%E6%80%A7%E7%8A%B6%E6%80%81%E5%B9%B6%E5%BF%AB%E9%80%9F%E8%AF%86%E5%88%AB%E4%B8%8D%E5%90%88%E8%A7%84%E7%9A%84%E8%B5%84%E6%BA%90%E3%80%82

AWS Config is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources.

# 1 功能简介

![](image/Pasted%20image%2020240219234829.png)


AWS Config 服务提供了跟踪 AWS 资源清单以及更改，并评估 AWS 资源配置的能力。

- 持续监控
    - 持续监控和记录您的 AWS 资源的配置更改。
    - 一旦检测到原有状态发生了更改，系统将发送 Amazon Simple Notification Service (SNS) 通知，以便用户查看更改并采取相应措施。
- 运行问题排查
    - 借助 AWS Config，可以获取一份 AWS 资源配置更改的完整历史记录。
    - Config 与 AWS CloudTrail 集成，将配置更改与特定事件关联起来。

- 持续评估
    - 使用 AWS Config 定义预置和配置 AWS 资源的规则，持续审计和评估 AWS 资源配置。
    - 违反规则的资源配置或配置更改会自动触发 Amazon Simple Notification Service (SNS) 通知和 Amazon CloudWatch 事件。
    - 可以利用可视化控制面板来查看整体合规性状态并快速识别不合规的资源。

- 企业级合规监控
    - 借助多账户、多区域数据聚合，可以查看整个企业的合规状态。
- 变更管理
    - 可以追踪资源之间的关系并在做出更改前查看资源依赖关系。
    - 发生更改后，可以快速查看资源的配置历史记录并确定过去任一时间点上的资源配置详情。
- 支持第三方资源
    - 可以将第三方资源（例如 GitHub 存储库、Microsoft Active Directory 资源或任何本地服务器）的配置发布到 AWS 中。
    - 可以使用 AWS Config 控制台和 API 查看和监视资源库存和配置历史记录。
    - 可以创建 AWS Config 规则或一致性包，以根据最佳实践、内部策略和监管策略评估这些第三方资源。

# 2 基本概念

## 2.1 AWS资源

AWS 资源是指使用 AWS 管理控制台、AWS Command Line Interface (CLI)、AWS 开发工具包或 AWS 合作伙伴工具创建和管理的实体。AWS 资源的示例包括 Amazon EC2 实例、安全组、Amazon VPC 和 Amazon Elastic Block Store。

AWS Config 用唯一的标识符（例如，资源 ID 或 Amazon 资源名称 (ARN)）来标记每个资源。

## 2.2 配置历史

配置历史记录是指定资源在某个时间段的配置项集合。配置历史记录包含多种信息，例如资源首次创建的时间、过去一个月的资源配置情况以及昨天上午 9 点发生了哪些配置更改等。

AWS Config 可以将正在记录的各种资源类型的配置历史文件自动传输到指定的 Amazon S3 存储桶。

用户可以在 AWS Config 控制台中选择一项资源，并使用时间线浏览该资源以前的所有配置项。

## 2.3 配置项

配置项 代表AWS 资源在特定时间点具备的各种属性。配置项的组成部分包括元数据、属性、关系、当前配置以及相关事件。

只要检测到正在记录的资源类型发生变更，AWS Config 就会创建配置项。例如，如果 AWS Config 正在记录 Amazon S3 存储桶，则只要创建、更新或删除存储桶，AWS Config 就会创建配置项。

## 2.4 配置记录器

配置记录器以配置项目的形式将受支持资源的配置存储在账户中。用户必须先创建并启动配置记录器，然后才能开始记录。
默认情况下，配置记录器会记录 AWS Config 运行的区域内所有受支持的资源。您可以创建一个自定义配置记录器，仅记录您指定的资源类型。

## 2.5 配置快照

配置快照是受支持资源的配置项的集合。配置快照可以完整展示被记录的资源及其配置的相关信息。
可以将配置快照传输到您指定的 Amazon Simple Storage Service (Amazon S3) 存储桶。

## 2.6 配置流

配置流是一个自动更新的列表，列出了 AWS Config 正在记录的资源的所有配置项。每当资源被创建、修改或删除时，AWS Config 会创建一条配置项并将其添加到配置流。配置流在运行时会使用 Amazon Simple Notification Service (Amazon SNS) 主题。

## 2.7 资源关系

AWS Config 会查找您账户中的 AWS 资源，然后创建 AWS 资源关系图。 

# 3 工作原理

- 初始配置：打开 AWS Config 之后，它会先查找所有受支持的 AWS 资源，并为每个资源生成一个配置项。
- 变更：AWS Config 会在某个资源的配置更改时生成配置项，并在启动配置记录器后，保留配置项的历史记录。
- 定时：AWS Config 还会跟踪不是由 API 发起的配置更改。AWS Config 会定期检查资源配置，并针对已更改的配置生成配置项。

默认情况下，AWS Config 会为区域内每个支持的资源创建配置项。如果不希望 AWS Config 为所有支持的资源都创建配置项，可以指定希望其跟踪的资源类型。

如果使用的是 AWS Config 规则，则 AWS Config 会持续评估 AWS 资源是否具备所需设置。根据具体规则，AWS Config 会在配置更改时或定期进行资源评估。每个规则都与一个 AWS Lambda 函数关联，其中包含规则的评估逻辑，该函数会返回被评估资源的合规性状态。
如果某个资源不符合某项规则的条件，那么 AWS Config 会将该资源和规则标记为不合规。当某个资源的合规性状态发生更改时，AWS Config 会向 Amazon SNS 主题发送通知。

![](https://pic4.zhimg.com/80/v2-a3a4710c6ffde6a217de298458ca7db7_720w.webp)

# 4 AWS Config

## 4.1 首页查看

  

![](https://pic4.zhimg.com/80/v2-7f2b0fc92110bcf13115f6d65bb113f3_720w.webp)

  

## 4.2 配置项查看

- 资源清单页支持按照资源类别和标签进行过滤查看。
![](https://pic2.zhimg.com/80/v2-05bb4decc704c7e17d89a9af7f4fb1b9_720w.webp)


- 点击资源标识符可以查看资源详情。右上角可以查看配置时间线、合规时间线。
![](https://pic2.zhimg.com/80/v2-b39a4d958465a22743dbfd58825c1ba5_720w.webp)

- 配置时间线

- Changes表示变更次数，单击可以跳转到CloudTrail变更项。
![](https://pic2.zhimg.com/80/v2-d76c951faa5a62254c37bf9634ddd2a9_720w.webp)

  

![](image/Pasted%20image%2020240219235422.png)

## 4.3 配置项管理

默认情况下，配置记录器会记录 AWS Config 运行的区域内所有受支持的资源。用户也可以创建一个自定义配置记录器，仅记录指定的资源类型。

如果不希望 AWS Config 记录所有支持资源的更改，则可以对其进行自定义，以使其仅记录特定类型的资源更改。

要记录的资源类型:
- **All resources (所有资源)**– AWS Config 会使用下列选项记录所有受支持的资源：
    - **Record all resources supported in this region (记录此区域中支持的所有资源)** – AWS Config 将记录每种受支持类型的区域性资源的配置更改。AWS Config 添加对新区域资源类型的支持后，它将自动开始记录该类型的资源。  
    - **Include global resources (包括全球性资源)** – AWS Config 将受支持类型的全局性资源包含在它记录的资源（例如 IAM 资源）中。AWS Config 添加对新全球性资源类型的支持后，它将自动开始记录该类型的资源。  
- **Specific types (特定类型)** – AWS Config 仅记录您指定的 AWS 资源类型的配置更改。  

![](https://pic4.zhimg.com/80/v2-add55d93d3dc7e2642295ec4b7ae8d1f_720w.webp)

  

## 4.4 高级查询

高级查询功能提供了单一查询终端节点和强大的查询语言以获得当前资源状态元数据，而无需执行特定于服务的描述 API 调用。可以使用配置聚合器跨多个账户和 AWS 区域从中央账户运行相同的查询。


![](https://pic1.zhimg.com/80/v2-e15146c8038f626e0db323bd5c2d59f8_720w.webp)

  

  
![](https://pic3.zhimg.com/80/v2-28445a77fc4803d210363d6548089c5a_720w.webp)

  

# 5 AWS Config 托管和自定义规则

AWS Config 规则表示特定 AWS 资源或整个 AWS 账户所需的配置设置。如果某个资源违反了规则，AWS Config 会将该资源和规则标记为不合规，并且 AWS Config 会通过 Amazon SNS 通知用户。

激活一项规则后，AWS Config 会将资源与规则中的条件进行比较。完成这一初始评估后，AWS Config 会在每次触发评估时继续执行评估。规则中会定义评估触发器，可以包括以下类型：
- 配置更改 – 当与规则范围匹配的任何资源的配置更改时，AWS Config 将触发评估。
- 通过定义规则的范围来选择哪些资源触发评估。
- 定期 – AWS Config 按照您选择的频率运行评估（例如，每 24 小时）。

![](image/Pasted%20image%2020240219235254.png)

## 5.1 合规查看

![](https://pic2.zhimg.com/80/v2-da751eb6104c5a94c1b21eff25230b35_720w.webp)

  

## 5.2 内置规则

[https://docs.aws.amazon.com/zh_cn/config/latest/developerguide/managed-rules-by-aws-config.html](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/zh_cn/config/latest/developerguide/managed-rules-by-aws-config.html)

aws 已经 提供了一些 rules, 你可以直接将这些 rules 归入到 某个 aws config 中 

评估页面：

![](https://pic2.zhimg.com/80/v2-62e0f210f62d5172e31da0ddc01f06e9_720w.webp)

## 5.3 自定义规则

可以通过与一个 AWS Lambda 函数关联实现自定义规则，评估 AWS 资源是否符合规则的逻辑。

## 5.4 修正不合规资源

- 手动修正
- 自动修正

# 6 多账户多区域数据聚合

聚合器是一种 AWS Config 资源类型，用于从以下内容收集 AWS Config 配置和合规性数据：
- 多个账户和多个区域。
- 单个账户和多个区域。
- AWS Organizations中的所有账户。

  
![](https://pic2.zhimg.com/80/v2-c407e1b1e4676200749d731b66103f11_720w.webp)

## 6.1 聚合器视图

![](https://pic4.zhimg.com/80/v2-be6d350b1cd6a0c239e2274188da4ebb_720w.webp)

## 6.2 添加聚合器

当源账户是单个账户时，需要授权。如果要聚合的源账户是 AWS Organizations 的一部分，则不需要授权。

![](https://pic2.zhimg.com/80/v2-4b89e193da8fdf2d4c0999793d88f51d_720w.webp)

## 6.3 账号授权

![](https://pic2.zhimg.com/80/v2-65216f2452b8c9c873fae3aa3a278275_720w.webp)

  

# 7 安全性

## 7.1 Service-Linked Role

AWS使用名为 **AWSServiceRoleForConfig** 的service-linked role代替用户访问其他的云服务。
- AWSServiceRoleForConfig 信任 [http://config.amazonaws.com](https://link.zhihu.com/?target=http%3A//config.amazonaws.com) 服务来代入该角色。
- AWSServiceRoleForConfig 角色的权限策略包含对 AWS Config 资源的只读和只写权限以及对 AWS Config 支持的其他服务中的资源的只读权限。

[http://config.amazonaws.com](https://link.zhihu.com/?target=http%3A//config.amazonaws.com)通过扮演AWSServiceRoleForConfig角色，来获取资源信息，并生成配置项。

## 7.2 AWS Config 管理权限

要允许用户管理 AWS Config，必须向 IAM 用户授予显式权限以使其能够执行与 AWS Config 任务关联的操作。

[完全访问权限](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/zh_cn/config/latest/developerguide/recommended-iam-permissions-using-aws-config-console-cli.html%23full-config-permission)：设置和管理 AWS Config 的用户必须具有完全访问权限。如果具有完全访问权限，用户可以提供 AWS Config 向其传递数据的 Amazon S3 和 Amazon SNS 终端节点、为 AWS Config 创建角色，以及打开和关闭记录功能。

只读权限：即，内置的AWSConfigUserAccess权限。使用 AWS Config 但无需设置 AWS Config 的用户应获得只读权限。如果具有只读权限，用户可以查找资源的配置或者按标签搜索资源。

## 7.3 针对 Amazon S3 存储桶的权限

默认情况下，所有 Amazon S3 存储桶和对象都是私有的。只有作为创建存储桶的 AWS 账户的资源所有者可以访问该存储桶。但是，资源所有者可以选择将访问权限授予其他资源和用户。

如果 AWS Config 自动创建 Amazon S3 存储桶，则这些权限会自动添加到 Amazon S3 存储桶中。但是，如果指定现有 Amazon S3 存储桶，则必须确保该 S3 存储桶具有相应权限。

  

![](https://pic2.zhimg.com/80/v2-985f1c5d48c8614c9f421e8fbd919941_720w.webp)




