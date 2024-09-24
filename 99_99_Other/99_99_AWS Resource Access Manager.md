https://shazi.info/aws-resource-access-manager-%E8%A7%A3%E6%B1%BA%E8%B7%A8%E5%B8%B3%E8%99%9F%E7%AE%A1%E7%90%86%E7%9A%84%E5%95%8F%E9%A1%8C%E4%BB%A5%E5%8F%8A%E5%BB%B6%E4%BC%B8%E8%AD%B0%E9%A1%8C/

https://www.infoq.cn/article/96mvi4j0wznejaqhljzz

# 1 总览 

AWS Resource Access Manager (AWS RAM) helps you securely share your resources across AWS accounts, within your organization or organizational units (OUs), and with AWS Identity and Access Management (IAM) roles and users for supported resource types.

Resource Access Manager 簡稱 RAM，是 2018 re:Invent 發表的新服務，主要是針對 Multi-Account 的客群提供服務，並且支援多項當天在 re:Invent 新發表的服務，如 Transit Gateways, Route 53 DNS Resolver。
资源共享不收取任何费用。


分享服務資源給其他帳號，相同的設定由一個帳號設定就可以套用到所有帳號，例如：Subnet, DNS Resolver Rule … 等，該服務免費。

新推出的 AWS Resource Access Manager (RAM) 可以方便 AWS 账户之间的资源共享。通过它可以在您的 AWS 组织内部共享资源，可以从控制台、CLI 或通过一组 API 来使用。我们现已推出对 Route 53 解析器规则的支持（昨天已在 Shaun 的优秀博文中宣布）并且将很快支持更多资源类型。


如要共享资源，您只需创建一个资源共享并为其命名，向其中添加一个或多个资源，然后向其他 AWS 账户授予访问权限即可。每个资源共享就好比一个购物车，可以容纳不同类型的资源。您可以共享您拥有的任何资源，但您无法再次共享他人向您共享的资源。您可以与组织、组织单元 (OU) 或 AWS 账户共享资源。您还可以控制是否可以将组织之外的账户添加到特定的资源共享中。

AWS Resource Access Manager 中 shared resorce ist regional service。  切换不同的 region, 原本的AWS Resource Access Manager 中 shared resorce ， 会不见 
# 2 支援服務

- Capacity Reservations
- Traffic mirror targets
- Transit Gateways
- VPC Subnet
- License Manager (configurations)
- Route 53 Resolver (forwarding rules)
- DB Cluster

RAM 支援的 Transit Gateways, Route 53 Resolver, License Manager 都是可以統一控管設定，避免重工的情況。

Subnet 蠻特殊的，被 Shard 的帳號可以不用建 VPC、Subnet，直接在 EC2 上就選的到 subnet-xxxx (shard) 字樣的 Subnet，共享 Subnet 的概念讓 Multi-Account 每個帳號要做的事情變少了。


# 3 开通 

组织的主账户必须在 RAM 控制台的设置页面启用共享：
这个 只在 master accout of a aws organization 中 才有 的选项。 
在其他的 aws account 中不会出现 这个 checkbox 
  

![](https://static001.infoq.cn/resource/image/c2/51/c2d5462b1ded47d0ba9a4a3df005e251.jpg)

然后与组织内的其他账户共享资源，无需进一步的操作即可实现资源共享（RAM 利用了将账户添加到组织时完成的握手便利）。与组织之外的账户共享资源时将会发出一份邀请，必须接受该邀请才能向该账户共享资源。
与某个账户（姑且称之为消费账户）共享资源时，将会在相关控制台页面显示共享的资源以及消费账户拥有的资源。同样，Describe/List 调用将会返回共享的资源和消费账户拥有的资源。
资源共享可以添加标签，您可以引用 IAM 策略中的标签来创建基于标签的权限系统。您可以随时添加和删除资源共享的账户和资源。

---


在其他的 aws account 中不会出现 这个 checkbox 
![](image/Pasted%20image%2020240302211525.png)


# 4 AWS Resource Access Manager 的使用



我会打开 RAM 控制台，然后单击 Create a resource share（创建资源共享）以开始使用：

![](https://static001.infoq.cn/resource/image/35/90/3506b5ea46386253134f52078ac82490.png)

  

我会输入我的共享名称 (CompanyResolvers) 然后选择我希望添加的资源：

  

![](https://static001.infoq.cn/resource/image/8f/b3/8f6313f17e258939b29bd5c21de822b3.png)
如我早先所提到，我们将很快增加更多的资源类型！

----


我会输入我希望与之共享资源的主体（组织、组织单位或 AWS 账户），然后单击 Create resource share（创建资源共享）：
![](https://static001.infoq.cn/resource/image/da/ca/daf68a42c3a12985409e14b72136bdca.png)
我组织之外的其他账户将会收到邀请。邀请将在控制台中显示，并且可以在控制台中接受。接受邀请后，如果拥有恰当的 IAM 权限，这些账户将可以访问资源。

用如下方法可以查询 aws Organzaition Unit 的 ID
![](image/Pasted%20image%2020240302211731.png)


---
RAM 还允许我集中访问我共享的所有资源，以及向我共享的所有资源：

![](https://static001.infoq.cn/resource/image/dd/9c/dd5e70c6d9cdb7aae524c40539fa639c.png)

您还可以使用 CreateResourceShare、UpdateResourceShare、GetResourceShareInvitations 和 AcceptResourceShareInvitation 等函数实现共享过程的自动化。当然，您可以使用 IAM 策略来管理这些函数在交易两侧的使用。