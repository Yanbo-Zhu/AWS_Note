
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



https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html


Transit Gateway is a networking service that connects multiple VPCs, on-premises networks, or remote networks using a centralized hub-and-spoke architecture. Transit Gateway acts as a transit point for traffic between VPCs, providing a scalable and flexible way to manage your network connectivity. With Transit Gateway, you can easily route traffic between VPCs, enforce security policies, and integrate with other AWS services, such as AWS Transit Gateway Network Manager or AWS Global Accelerator.

![](image/Pasted%20image%2020240224151452.png)



The following diagram shows a transit gateway with three VPC attachments. The route table for each of these VPCs includes the local route and routes that send traffic destined for the other two VPCs to the transit gateway.

![](image/Pasted%20image%2020240307111045.png)

The following is an example of a default transit gateway route table for the attachments shown in the previous diagram. The CIDR blocks for each VPC propagate to the route table. Therefore, each attachment can route packets to the other two attachments.

|Destination|Target|Route type|
|---|---|---|
|`VPC A CIDR`|`Attachment for VPC A`|propagated|
|`VPC B CIDR`|`Attachment for VPC B`|propagated|
|`VPC C CIDR`|`Attachment for VPC C`|propagated|



## 2.1 Advantage and Disadvantage

Benefits of Transit Gateway
- **Scalable and flexible:** Transit Gateway can connect up to 5,000 VPCs and route traffic between VPCs in different AWS accounts or regions, as well as on-premises networks or remote networks.
- **Centralized management:** Transit Gateway allows you to manage your network resources in a single place, using features such as route tables, security groups, or VPN attachments.
- **Advanced features:** Transit Gateway supports route propagation, VPN failover, or domain name system (DNS) resolution, which can enhance your network performance and security.

Limitations of Transit Gateway
- **Higher cost:** Transit Gateway can be more expensive than VPC peering, especially for small-scale networks or simple architectures. You may incur additional charges for data transfer, VPN connections, or NAT gateways.
- **Complexity:** Transit Gateway has a more complex setup process than VPC peering, requiring you to create and configure multiple components such as route tables, attachments, or prefixes.
- **Limited control:** Transit Gateway may not offer as much granular control over network traffic as VPC peering, especially if you need to implement custom routing or security policies.


## 2.2 Transit Gateway Attachment 

将 transit Gateway 和某一个资源绑定。 就会产生 产生一个 attachement , 这个 attachment 有独立的 id 

A transit gateway attachment is both a source and a destination of packets. You can attach the following resources to your transit gateway:
- One or more VPCs. 
    - AWS Transit Gateway deploys an elastic network interface within VPC subnets, which is then used by the transit gateway to route traffic to and from the chosen subnets. You must have at least one subnet for each Availability Zone, which then enables traffic to reach resources in every subnet of that zone. During attachment creation, resources within a particular Availability Zone can reach a transit gateway only if a subnet is enabled within the same zone. If a subnet route table includes a route to the transit gateway, traffic is only forwarded to the transit gateway if the transit gateway has an attachment in the subnet of the same Availability Zone.
- One or more VPN connections
- One or more AWS Direct Connect gateways
- One or more Transit Gateway Connect attachments
- One or more transit gateway peering connections
- A transit gateway attachment can be both a source and a destination of packets


## 2.3 Routing

Your transit gateway routes IPv4 and IPv6 packets between attachments using transit gateway route tables. You can configure these route tables to propagate routes from the route tables for the attached VPCs, VPN connections, and Direct Connect gateways. 
You can also add static routes to the transit gateway route tables. When a packet comes from one attachment, it is routed to another attachment using the route that matches the destination IP address.

For transit gateway peering attachments, only static routes are supported.

![](image/Pasted%20image%2020240305173703.png)

### 2.3.1 Route Association and Route Propagation


https://nileshjoshi.medium.com/overview-of-transit-vpc-aws-transit-gateway-deep-dive-37631fb2a6f6

Nice! We are done with VPC attachments and now it’s time to associate these attachments under TGW routes. So let’s jump to Transit Gateway Route Tables choose each route and associate the VPC attachments.

    Association: FROM which Attachment (VPC/VPN) the traffic can be initiated. You can associate a transit gateway attachment with a single route table.

    Propagation: TO which Attachment(VPC/VPN) traffic can be routed to. You can create a propagation of transit gateway attachment with multiple route tables.


### 2.3.2 route table

https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html#tgw-route-tables-overview

Route table evaluation differs between whether you're using a VPC route table or a transit gateway route table.

The following example shows a VPC route table. The VPC local route has the highest priority, followed by the routes that are the most specific. When a static route and a propagated route have the same destination, the static route has a higher priority.

|Destination|Target|Priority|
|---|---|---|
|10.0.0.0/16|local|1|
|192.168.0.0/16|pcx-12345|2|
|172.31.0.0/16|vgw-12345 (static) or<br><br>tgw-12345 (static)|2|
|172.31.0.0/16|vgw-12345 (propagated)|3|
|0.0.0.0/0|igw-12345|4|

The following example shows a transit gateway route table. 
If you prefer the AWS Direct Connect gateway attachment to the VPN attachment, use a BGP VPN connection and propagate the routes in the transit gateway route table.

| Destination    | Attachment (Target)                    | Resource type              | Route type           | Priority |
| -------------- | -------------------------------------- | -------------------------- | -------------------- | -------- |
| 10.0.0.0/16    | tgw-attach-123 \| vpc-1234             | VPC                        | Static or propagated | 1        |
| 192.168.0.0/16 | tgw-attach-789 \| vpn-5678             | VPN                        | Static               | 2        |
| 172.31.0.0/16  | tgw-attach-456 \| dxgw_id              | AWS Direct Connect gateway | Propagated           | 3        |
| 172.31.0.0/16  | tgw-attach-789 \| tgw-connect-peer-123 | Connect                    | Propagated           | 4        |
| 172.31.0.0/16  | tgw-attach-789 \| vpn-5678             | VPN                        | Propagated           | 5        |





# 3 Transit Gateway 和 vpc peering 的区别 

一个 vpc peering 只能连接两个 vpc 
Transit Gateway 可以连接 多个 vpc 

While VPC peering is a simple and low-cost solution for connecting two VPCs within the same region, Transit Gateway provides a more advanced and centralized approach for connecting multiple VPCs across regions, accounts, and on-premises networks.

![](image/Pasted%20image%2020240224152255.png)


