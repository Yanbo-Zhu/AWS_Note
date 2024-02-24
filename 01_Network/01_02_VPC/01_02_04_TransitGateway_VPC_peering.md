
https://www.cloudthat.com/resources/blog/connecting-amazon-vpcs-a-comprehensive-comparison-between-vpc-peering-and-transit-gateway

# 1 VPC Peering


VPC peering is a networking solution that connects two VPCs in the same or different AWS regions using private IP addresses. VPC peering creates a direct, one-to-one connection between two VPCs, bypassing the internet and providing low-latency, high-bandwidth connectivity.

With VPC peering, you can share resources such as EC2 instances, Elastic Block Store (EBS) volumes, security groups, and route traffic between VPCs.

![](image/Pasted%20image%2020240224150459.png)


## 1.1 Advantage and Disadvantage

Benefits of VPC Peering
- **Simple and easy to set up:** VPC peering can be created with just a few clicks in the AWS Management Console and does not require additional hardware or software.
- **Low latency and high throughput:** VPC peering allows you to communicate with instances in another VPC as if they were in the same network without incurring data transfer charges or exposing your traffic to the public internet.
- **Flexible and scalable:** VPC peering can connect VPCs across different accounts or regions and be deleted or modified anytime.

Limitations of VPC Peering
- **Regional constraints:** VPC peering can only connect VPCs within the same AWS region, which may limit its usefulness for global deployments.
- **Overlapping IP addresses:** VPC peering requires that the IP address ranges of the peered VPCs do not overlap, which can be challenging when migrating workloads or merging multiple VPCs.
- **Limited scalability:** VPC peering supports a one-to-one connection model, which may not be suitable for large-scale networks or complex architectures.


# 2 Transit Gateway

Transit Gateway is a networking service that connects multiple VPCs, on-premises networks, or remote networks using a centralized hub-and-spoke architecture. Transit Gateway acts as a transit point for traffic between VPCs, providing a scalable and flexible way to manage your network connectivity. With Transit Gateway, you can easily route traffic between VPCs, enforce security policies, and integrate with other AWS services, such as AWS Transit Gateway Network Manager or AWS Global Accelerator.

![](image/Pasted%20image%2020240224151452.png)

## 2.1 Advantage and Disadvantage

Benefits of Transit Gateway
- **Scalable and flexible:** Transit Gateway can connect up to 5,000 VPCs and route traffic between VPCs in different AWS accounts or regions, as well as on-premises networks or remote networks.
- **Centralized management:** Transit Gateway allows you to manage your network resources in a single place, using features such as route tables, security groups, or VPN attachments.
- **Advanced features:** Transit Gateway supports route propagation, VPN failover, or domain name system (DNS) resolution, which can enhance your network performance and security.

Limitations of Transit Gateway
- **Higher cost:** Transit Gateway can be more expensive than VPC peering, especially for small-scale networks or simple architectures. You may incur additional charges for data transfer, VPN connections, or NAT gateways.
- **Complexity:** Transit Gateway has a more complex setup process than VPC peering, requiring you to create and configure multiple components such as route tables, attachments, or prefixes.
- **Limited control:** Transit Gateway may not offer as much granular control over network traffic as VPC peering, especially if you need to implement custom routing or security policies.



# 3 Transit Gateway 和 vpc peering 的区别 

一个 vpc peering 只能连接两个 vpc 
Transit Gateway 可以连接 多个 vpc 

While VPC peering is a simple and low-cost solution for connecting two VPCs within the same region, Transit Gateway provides a more advanced and centralized approach for connecting multiple VPCs across regions, accounts, and on-premises networks.

![](image/Pasted%20image%2020240224152255.png)


