

# 1 Destination and Target 

A short answer:
Destination: the packet's final destination
Target: where the packet should go next, to get it one step closer to the intended destination.

e.g.
You are planning a business trip from the US to Paris.
Destination: Paris
Target: US airport, fly to Paris.


```
| Destination | Target      |
|:------------|------------:|
| 10.0.0.0/16 | local       |
|  0.0.0.0/0  | your-igw-id |
```
This will route any request with into-the-VPC destination to local targets in the VPC.  如果你的目的地的地址 match the pattern 10.0.0.0/16, 你就会被分到 local subnet内

If the pattern 10.0.0.0/16 is not met, the any-ipv4-address aka 0.0.0.0/0 will be considered and the request will be routed to the internet gateway you specified with ID your-igw-id.  0.0.0.0/0 代表 任何其他的 ip 地址, 除了 (10.0.0.0/16, etc) 就是在route table 中 没有被定义的 ip 地址 .   Here all traffic will be passed on to the Internet Gateway


## 1.1 Destination 

Destination => IP address/CIDR range .
Destination field specifies the pattern that a request must match with its destination address (IP or CIDR range).
This is an IP Address or a CIDR Range. To plan this out, consider outgoing traffic from any instance inside a VPC. This outgoing traffic will have some destination IP Address, and the route will just handle the request to the target, which is the second part of the entry.
## 1.2 Target 

Target => Where you want to send the traffic for the specified destination
Target field specifies where such a request should be routed. It could be local (i.e. to targets in this VPC) or your-internet-gateway-ID in case those requests should be routed to the gateway for external/somewhere-else access. A list of possible target values is here.
Target is just the destination the request will be redirected to for getting processed.

例子

| x                | x                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------- |
| local            | if the destination is my local subnet, mention target as "local"                   |
| Internet gateway | The Internet gateway is one of the targets (e.g. routing traffic to the internet). |
|                  |                                                                                    |
|                  |                                                                                    |
可以填写为
NAT Gateway
Virtual Private Gateway
VPC endpoint
VPC peering connection etc. depending on your architecture
