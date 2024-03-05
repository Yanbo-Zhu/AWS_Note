
https://www.chiwaichan.co.nz/2022/05/28/leveraging-aws-prefix-lists/
https://docs.aws.amazon.com/vpc/latest/userguide/managed-prefix-lists.html

# 1 two types of Prefix Lists:

    AWS-managed Prefix Lists: as the name indicates these lists are managed by AWS, and they are used to maintain a set of IP address ranges for AWS services, e.g. S3, DynamoDB and CloudFront.
    Customer-managed Prefix Lists: these are created and maintained by anyone who has access to the AWS Console, AWS APIs or AWS SDKs. This is what we will be focusing on.



## 1.1 What is an AWS-managed Prefix List

https://docs.aws.amazon.com/vpc/latest/userguide/working-with-aws-managed-prefix-lists.html

AWS-managed prefix lists are created and maintained by AWS and are available to anyone with an AWS account. 
这里面的内容 是 aws 自己创造的 ， 不是我们认为 输入的 
==A prefix list is a collection of one or more IP CIDR blocks used to simplify the configuration and management of security groups and routing tables.==

There are customer-managed prefix lists and AWS-managed prefix lists. **This blog post focuses on AWS-managed prefix lists for Amazon CloudFront.**
You can simply **find them in AWS Management Console, under VPC, Managed prefix lists.**

**AWS-managed lists include updated IP addresses of AWS services which you can add to security groups or route tables** to better manage what service or who can reach your VPC.

The following AWS-managed prefix lists are available:
- **Amazon S3: com.amazonaws.region.s3**
- **Amazon DynamoDB: com.amazonaws.region.dynamodb**
- **Amazon CloudFront: com.amazonaws.global.cloudfront.origin-facing**

|AWS service|Prefix list name|Weight|
|---|---|---|
|[Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html#managed-prefix-list)|com.amazonaws.global.cloudfront.origin-facing|55|
|Amazon DynamoDB|com.amazonaws.`region`.dynamodb|1|
|AWS Ground Station|com.amazonaws.global.groundstation|5|
|[Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-router-firewall-rules.html)|com.amazonaws.`region`.ipv6.route53-healthchecks|25|
|com.amazonaws.`region`.route53-healthchecks|25|
|Amazon S3|com.amazonaws.`region`.s3|1|
|Amazon S3 Express One Zone|com.amazonaws.`region`.s3express|6|
|[Amazon VPC Lattice](https://docs.aws.amazon.com/vpc-lattice/latest/ug/security-groups.html#managed-prefix-list)|com.amazonaws.`region`.vpc-lattice|10|
|com.amazonaws.`region`.ipv6.vpc-lattice|10|

## 1.2 Customer-managed Prefix 

AWS VPC Customer-managed Prefix List is a great tool to have available as it provides the ability to track and maintain a list of CIDR block values, which can then be referenced by other AWS Networking components in their rules or route tables. Each Prefix List supports either IPv4 or IPv6 based addresses, and a number of expected Max Entries for the list must be defined; the number of entries in the list cannot exceed the Max Entries.

You can use Prefix List to maintain a list of CIDR blocks of Subnets or VPCs; or, track a list of similiar IP addresses based on a grouping of your choice, e.g. EC2 instances with a certain function - you can even track CIDR values of Subnets, VPCs and EC2 within the same list.

### 1.2.1 Let's create a Prefix List in the AWS Console

![](image/Pasted%20image%2020240305181203.png)



# 2 原理解释 


This is especially useful in scenarios where you have fleet of EC2 instances where you like to allow the same network traffic sources to ingress on Port 22 to perform administration tasks, these fleet EC2 instances could scatter across multiple VPCs, and may even be scattered across multiple AWS accounts.

![](image/Pasted%20image%2020240305181351.png)

Often, we add a new Source CIDR to all Security Groups as we allow a new machine to perform administration tasks to the same fleet of EC2 instances, or even remove (or not when we forget) a CIDR Source when a machine is retired. In the past we would have modified each and every one of these Security Groups.

Here is how we can leverage Customer-managed Prefix Lists with Security Groups:

![](image/Pasted%20image%2020240305181429.png)

Here, under the same Security Group rules outcome we externalise the CIDR values into a Prefix List and reference the list in all 3 Security Groups; 

Share the prefix list to other aws acoount 
in the case of Security Groups spanning across multiple AWS accounts the Prefix Lists can be shared with other AWS accounts using Resource Access Manager (RAM). 
Now, we can allow a new machine to perform administration tasks across the entire fleet of EC2 instances by only adding a new CIDR Source to a single location, conversely, we can remove a machine by deleting a CIDR Source. 
There is also an added benefit of reduced effort in the need to identify which Security Groups have a rule for an IP address if we were to remove access across the entire fleet using this pattern – because it is maintained in a single location.


# 3 例子： 在 security group 中 使用 managed prefix list 

https://sjramblings.io/aws_managed_prefixes

To allow access to only S3 from my instance, I create a rule as follows instead of just allowing the default outbound any.

![Prefix Lists](https://cdn.hashnode.com/res/hashnode/image/upload/v1678725901243/X7xs3xKmc.png?auto=compress&auto=compress,format&format=webp)

![Outbound Rule](https://cdn.hashnode.com/res/hashnode/image/upload/v1678725931511/-4eHv1MjS.png?auto=compress&auto=compress,format&format=webp)

Behind the scenes, the Prefix list ID contains a list of CIDR blocks that cover all the IP address ranges for the S3 service in the target region.