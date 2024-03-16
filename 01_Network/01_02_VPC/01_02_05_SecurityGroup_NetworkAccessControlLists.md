


# 1 SecurityGroup 和 NetworkACL 比较 

https://medium.com/@cloud_tips/difference-between-security-groups-and-network-access-control-list-3332c5612efc
https://docs.aws.amazon.com/whitepapers/latest/aws-best-practices-ddos-resiliency/security-groups-and-network-acls-bp5.html

AWS security group 和 network ACL都定义了网络访问规则，用于控制哪些inbond和outbond traffic被允许或者被禁止。不同之处在于：

There is no additional charge for using security groups or network ACLs.

security group：
- 应用于ec2 instance的流量访问控制.  security groups allow you to control inbound and outbound traffic at the instance level,
    - 一个 security group 可以包含 多个和 instances. 这些 instances 必须在一个 vpc 中， 但是可以在多个 subnets 中 ， 多个 az 中 
- 所有 traffic to a security group 在默认状态下 都是被否定的.  All internet traffic to a security group is implicitly denied unless you create an _allow_ rule to permit the traffic.
- 0.0.0.0/0 表示 除了已经被 其他的 rule 定义的ip range 之外的 任何 ip addresss 
- Stateful (有状态)防火墙： 你可以在一个SG里边只指定Inbond规则，无需指定outbond规则，当inbond规则允许流量进入后，所有响应(outbond)流量会自动放行
    - This means that if you allow traffic in one direction, traffic is allowed in the other direction

Network ACL：
- 作用于单个subnet里边的所有主机。network ACLs offer similar capabilities at the VPC subnet level
- 每一个Subnet都有对应的Network ACL。 
- Stateless 防火墙: 不管你的流量是主动发起的请求流量，还是服务端的响应流量，都需要检查相应的访问控制规则. 
    - you must explicitly allow traffic in both directions.
    - subnets network ACLs are stateless , both inbound and outbound rules need to be updated to enable communication between the web and database tier 

关于修改inbound and outbound rule：You can modify the rules for a security group at any time; you can’t modify the rules for a network ACL until you disassociate it from the subnet.
You can’t block specific IP addresses using a security group; you can block specific IP addresses using a network ACL.

---

AWS security group：
- 应用于ec2 instance的流量访问控制。同一个security group可以跨不同的subnet甚至availability zone ，但是不能跨VPC。(一个VPC可以跨多个Availability zone, 一个Availability zone就是一个数据中心，一个region下边可以有多个Availability zone)。因为 一个 security group 可以包含多个 ec2 instance, 这些 ec2 instance 是可以在不同 subnet, 不同的 availability zone 的 
- 在定义SG时，可以指定Inbound/outbond policy, 比如在定义inbound policy时，可以指定哪些源主机可以访问本机(目的主机)的哪些端口。比如我们为一个web server 定义一个SG，而这个web server我们想任何主机都可以访问它，包括从internet访问，这时目的端口可以指定80， 源主机就可以指定0.0.0.0，表示任何主机都可以访问。
- SG一个有趣的功能是，在设定规则时，源主机可以不指定IP地址，而是指定一个SG，表示所有属于该SG的主机。也就是说SG不经可以设定访问规则，还可以代表一组主机哦。 
- 另外, SG是stateful (有状态)防火墙，你可以在一个SG里边只指定Inbond规则，无需指定outbond规则，当inbond规则允许流量进入后，所有响应(outbond)流量会自动放行。下文有详细介绍什么是有状态防火墙。

Network ACL：
- 作用于单个subnet里边的所有主机。每一个Subnet都有对应的Network ACL。

比如下边图示中，左下角的那个private subnet的security group A里的主机与 security group B里边的主机互相通信，那traffic要受security group A和security group B控制，但是不受Network ACL控制的，因为流量没有出subnet。而下图左上角的主机要与左下角的主机通讯，那流量即受security group控制， 也受Network ACL控制，因为流量出了subnet。

![](image/Pasted%20image%2020240224155248.png)


# 2 防火墙有状态无状态

下图介绍了什么是有状态防火墙，什么是无状态防火墙。

![](image/Pasted%20image%2020240224155409.png)
比如一个客户端浏览器访问一个web server，web server IP是10.2.1.10，监听端口是80，客户端IP是10.1.1.1，端口是65188。客户端服务器之间有一个防火墙，做控制流量。这个时候，防火墙设置规则时，有两种方案：

有状态防火墙方案：(stateful firewall)
这种方式，只针对主动发起方发起的流量设置访问规则，比如我们的例子，主动发起方是客户端浏览器，那只需要设置从浏览器（IP 10.1.1.1, port 65188）到web server (IP 10.2.1.10, port 80)的流量设置规则，不需要针对web server到浏览器的响应流量再单独设置一条规则，==默认如果放行从浏览器到web server的流量，那从web server到浏览器的响应流量也是允许的。==
就像下边图片说的，一个有状态防火墙，是自动允许返回的流量的。（请思考：假设我们在防火墙上设置了一条规则，允许浏览器访问web server。那现在浏览器没有访问web server， 但是出于某种原因，虽然不太可能，web server主动发送流量给客户端呢的65188端口，这种情况防火墙会允许么？）

无状态防火墙方案：(stateless firewall) 
像下边图中说的，无状态防火墙不管你的流量是主动发起的请求流量，还是服务端的响应流量，都需要检查相应的访问控制规则，看有没有对应的规则允许放行。

==AWS的Security Group，使用的是有状态防火墙，而Network ACL使用的是无状态防火墙，需要为请求、响应流量都这只段都的规则，这点设置规则时要注意。==

请思考：既然Security Group和Network ACL都对应防火墙的访问控制规则，那他们分别在哪里设置？是在每个主机的防火墙里，比如Linux的IP table，还是在撞门的网络设备，比如防火墙，路由器等等？


# 3 security group

另外，默认的security group允许同一个security group里边的主机通讯，即使是跨不同subnet的主机，Security group允许同一个Security group的instances之间互相访问，是由如下这条规则决定的，可以看到允许source是这个security group本身的主机访问，也就是允许该security group内所有资源互相访问。

![](image/Pasted%20image%2020240224155728.png)



<u>NAT gateway 应该被 安放在 public subnet</u>
Provision a NAT gateway in a public subnet, the private subnet's route tables can be modified to add a default route the point to the NAT gateway. This configuration ensures that outbound internet traffic form the ec2 instance in the private subnets is routed through the NAT gateway, allowing them to comuincation with xx server im internet.

Provisoin a NAT gateway in a private subnet would not allow the ec2 instances in the private subnets to commmunicate with internt. 

NAT gateway are designed to be deployed in oublic subnets to provide internet connectitivity to private subnets.


