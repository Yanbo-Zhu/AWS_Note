

https://youtu.be/ydxEeVAqVdo?si=oeaQOZuOEZuKaHvX

# 1 理论 

private Subnet can not download packages form internet 

从内部经过 NAT Gateway 访问互联网是可能的 

![](image/Pasted%20image%2020240221162358.png)


从外部经过 NAT Gateway 是不可能的
![](image/Pasted%20image%2020240221162424.png)


# 2 Setup process

1 create VPC
2 create private subnet and public subnet within VPC 
4 create Internet gateway
3 create route table, attach internet gateway to route table , so that the public Subnet can connect to internet Gateway 

![](image/Pasted%20image%2020240221163757.png)

![](image/Pasted%20image%2020240221163819.png)

4 create NAT Gateway
==NAT Gateway will be associated with our public subnet with the internet gateway, because the internet gateway is only available inside our public subnet 
NAT Gateway will provide a route to the private subnet to get an internet access ==

NAT Gateway 需要被安置在 public subnet 中 

![](image/Pasted%20image%2020240221164602.png)


绑定 一个 public subnet 
![](image/Pasted%20image%2020240221164846.png)

connectivity type 
应该选 public 
Private:  用这个话 ec2 instance in private subnet 只能连接到 其他的 VPC, 不能连接到 internet 

create Elastic ip addresse 
![](image/Pasted%20image%2020240221165145.png)

add tags 
![](image/Pasted%20image%2020240221165204.png)




5 update the route table in private subnet 
so the private subnet can connect to NAT Gateway 

![](image/Pasted%20image%2020240221165910.png)




6 create Ec2 instance in public subnet and private subnet 
在创造 EC2 instance 的时候， 就顺便把 security group 给创造了 

 看到 private EC2 Instance 没有 public ip addresse , 只有 private ipv4 addresse. 所以 无法送互联网直接访问 这个 instance 
 (auto-assign publoic IP 选为 Disable)
 ![](image/Pasted%20image%2020240221192631.png)
 
 ![](image/Pasted%20image%2020240221180605.png)

## 2.1 test the NAT Gateway 

login ec2 instance in public subnet , then 在 这个 ec2 instance 中 去 login the ec2 instance in private subnet 。 因为 the ec2 instance in private subnet 无法直接从 internet 中访问 


在 private ec2 instance 的属性页面， 可以看到 在 public ec2 instance 中 使用 `ssh -i "nat-gateway-demo-key.pem" ubuntu@12.0.2.98 `  就可以连接到 private ec2 instance 了 

![](image/Pasted%20image%2020240221180810.png)


![](image/Pasted%20image%2020240221181003.png)



