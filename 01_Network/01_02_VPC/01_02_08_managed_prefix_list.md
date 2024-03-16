
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






# 2 例子： 


## 2.1 在 security group 中 使用 managed prefix list 

Customer-managed Prefix List is great option to have to centrally manage and track a list of CIDR blocks allowed to ingress an ENI by referencing Prefix Lists in Security Groups, a single Prefix List instance can be referenced by one or many Security Groups within the same account or cross-account.



https://sjramblings.io/aws_managed_prefixes

To allow access to only S3 from my instance, I create a rule as follows instead of just allowing the default outbound any.

![Prefix Lists](https://cdn.hashnode.com/res/hashnode/image/upload/v1678725901243/X7xs3xKmc.png?auto=compress&auto=compress,format&format=webp)

![Outbound Rule](https://cdn.hashnode.com/res/hashnode/image/upload/v1678725931511/-4eHv1MjS.png?auto=compress&auto=compress,format&format=webp)

Behind the scenes, the Prefix list ID contains a list of CIDR blocks that cover all the IP address ranges for the S3 service in the target region.


---

This is especially useful in scenarios where you have fleet of EC2 instances where you like to allow the same network traffic sources to ingress on Port 22 to perform administration tasks, these fleet EC2 instances could scatter across multiple VPCs, and may even be scattered across multiple AWS accounts.

![](image/Pasted%20image%2020240305223306.png)

Often, we add a new Source CIDR to all Security Groups as we allow a new machine to perform administration tasks to the same fleet of EC2 instances, or even remove (or not when we forget) a CIDR Source when a machine is retired. In the past we would have modified each and every one of these Security Groups.

Here is how we can leverage Customer-managed Prefix Lists with Security Groups:

![](image/Pasted%20image%2020240305223325.png)

Here, under the same Security Group rules outcome we externalise the CIDR values into a Prefix List and reference the list in all 3 Security Groups; in the case of Security Groups spanning across multiple AWS accounts the Prefix Lists can be shared with other AWS accounts using Resource Access Manager (RAM). 
Now, we can allow a new machine to perform administration tasks across the entire fleet of EC2 instances by only adding a new CIDR Source to a single location, conversely, we can remove a machine by deleting a CIDR Source. There is also an added benefit of reduced effort in the need to identify which Security Groups have a rule for an IP address if we were to remove access across the entire fleet using this pattern – because it is maintained in a single location.



## 2.2 Prefix List – Subnet Route Table Reference

Another way to use Prefix Lists is to use them to centrally manage and track a list of CIDR block destinations to route traffic out of a Subnet’s Route Table to the same Target, a Prefix List can be referenced by one or many Subnet Route Tables within the same account or cross-account using RAM.



Below, we have a scenario with 3 different Route Tables across the two VPCs, with each Route Table with the same Transit Gateway Target for the same set of Destinations; and also the same Destinations routed to their respective Egress Only Internet Gateway (EIGW) for their VPC.
![](image/Pasted%20image%2020240305223150.png)


Here is how we can leverage Customer-managed Prefix Lists with Subnet Route Tables:
![](image/Pasted%20image%2020240305223204.png)


We have externalised the Destination CIDR values of the 3 Route Tables into 2 separate Prefix Lists: 1st Prefix List contains the CIDR block values of Destinations routed for the EIGW in their respective VPC; the 2nd Prefix List contains CIDR block values of Destinations routed for the same Transit Gateway instance all VPCs is an attachment of.



## 2.3 Transit Gateway Route Table 

Lastly, in a Transit Gateway Route Table you have the option to either to define static routes or have routes dynamically propagated from a Transit Gateway attachment. You also have the option to use a Prefix List for routing.

Here is how we can leverage Customer-managed Prefix Lists with Transit Gateway Route Tables:

![](image/Pasted%20image%2020240305223510.png)

To reference a Prefix List in a Transit Gateway Route Table, you have to reference it under the "Prefix list references" section:

![](image/Pasted%20image%2020240305223915.png)

# 3 Considerations

- The aggregated total Max Entries of all Prefix Lists referenced by a resource (e.g. a Security Group) is counted towards the resource's quota - not the aggregated total of actual entries of all Prefix Lists. Be conscious of the Prefix List you reference in a resource, does the resource referencing the Prefix List require all the CIDR values offered in the list? if not, you are not using Prefix Lists economically.
- If the same Prefix List instance is referenced by multiple AWS resources then consistency is enforced - operational effort is reduced due to fewer changes by not having to change a values in multiple locations.
- Before you add or remove a CIDR value from a Prefix List, consider the flow on impact it may have to the downstream resources that reference this list, as you may inadvertently terminate some traffic flow, or worse, open up traffic to sources you don't intend to.

# 4 Conclusion

One of the things I have noticed during my short time in consulting so far is that organising Cloud resources (in particular Networking), structuring them correctly and consistently across multiple environments will set up a solid foundation for organisations in the long term, however, it is often an area that is overlooked and is only paid attention to when the rate of innovation is slowed down due to complexities and inconsistencies. Prefix Lists is a great option to have to improve consistency and operational efficiencies.

Here I have only detailed the basic use of Customer-managed Prefix Lists, but in my other blog I have a more advanced use case leveraging Prefix Lists: [Work-around for cross-account Transit Gateway Security Group Reference](https://chiwaichan.co.nz/2022/05/28/work-around-for-cross-account-transit-gateway-security-group-reference)
