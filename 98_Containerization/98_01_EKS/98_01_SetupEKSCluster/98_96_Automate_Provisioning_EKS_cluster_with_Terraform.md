https://www.bilibili.com/video/BV1rV4y1o7bP/?spm_id_from=333.788.recommend_more_video.-1&vd_source=55e5cc2f534c16c73bbeb684e98c4195

# 1 Create VPC 


![](image/Pasted%20image%2020240712101805.png)


![](image/Pasted%20image%2020240712105250.png)


## 1.1 vpc.tf 

引用的了 某个 module 

![](image/Pasted%20image%2020240712103211.png)

![](image/Pasted%20image%2020240712104045.png)

使用的是 terraform registry 中的现成的 tf module 

enable_nat_gateway 
Description: Should be true if you want to provision NAT Gateways for each of your private networks

single_nat_gateway bool
Description: Should be true if you want to provision a single shared NAT Gateway across all of your private networks

enable_dns_hostnames
Description: Should be true to enable DNS hostnames in the VPC

---


![](image/Pasted%20image%2020240712104918.png)


tags 
For Cloud COntroller Manager 
指定 这个vpc 是被哪个 eks cluster 使用 ， 例子中 就是被 myapp-eks-cluster 使用 的 vpc
这其中的量的给出 (this assumption ) 是为了 kubenetes cloud contoller maanger 
![](image/Pasted%20image%2020240712104338.png)


public_subnet_tags
Description: Additional tags for the public subnets

![](image/Pasted%20image%2020240712104553.png)




private_subnet_tags
Description: Additional tags for the private subnets


还要创造 elb
![](image/Pasted%20image%2020240712104632.png)

public subnet 的 elb 的作用是 expost the port to outside, get the request from outside , can be accessible for outside, for externals adresse

private subnet 
for services and componets inside the private subnets 

## 1.2 terraform.tfvars
![](image/Pasted%20image%2020240712101958.png)



# 2 Create eks cluster 

1 
使用的是 terraform registry 中的现成的 tf module 
![](image/Pasted%20image%2020240712110449.png)


![](image/Pasted%20image%2020240712110000.png)

2 

![](image/Pasted%20image%2020240712110655.png)

选的是 Node Group 的方式 
worker_group参量 
![](image/Pasted%20image%2020240712110229.png)


3  Configure kubenetes provider

https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#authentication

![](image/Pasted%20image%2020240712111834.png)


get the info of eks-cluster ， 给到 kubenetes provider 

load_config_file  = "false"  # we don't want to load the default .kube/config, we create a new one to use it 
host 需要的的是 Endpoint of k8s cluster (API Cluster). (Optional) The hostname (in form of URI) of the Kubernetes API. Can be sourced from KUBE_HOST.



# 3 Check the cluster 和一起产生的 东西 

1 Cluster 
![](image/Pasted%20image%2020240712113247.png)

![](image/Pasted%20image%2020240712113255.png)


2 Configuration 
![](image/Pasted%20image%2020240712113307.png)


3 一起创造出来的 IAM role

![](image/Pasted%20image%2020240712113354.png)

one for eks 
one for ec2 


4 EC2 instance  
![](image/Pasted%20image%2020240712113431.png)

![](image/Pasted%20image%2020240712113458.png)


5 node 是上一节中 通过 worker_group 创造的 

![](image/Pasted%20image%2020240712113152.png)




# 4 用 vpc module 创造出来的VPC 

![](image/Pasted%20image%2020240712113522.png)



## 4.1 route tables of vpc 


Route table 
![](image/Pasted%20image%2020240712113604.png)




这个route table 是route request to internet gateway, in order to connect to internet 
应该是 assocaited with the public subnets 的 

![](image/Pasted%20image%2020240712114555.png)

这个route table 是route request to nat gateway, in order to let worker node connect to master node 
应该是 assocaited with the private  subnets 的 

![](image/Pasted%20image%2020240712114802.png)


![](image/Pasted%20image%2020240712114156.png)




### 4.1.1 Route table of VPC 的官方解释 

A short answer:
Destination: the packet's final destination
Target: where the packet should go next, to get it one step closer to the intended destination.


You are planning a business trip from the US to Paris.
Destination: Paris
Target: US airport, fly to Paris.

Each route has a destination and target field.
- _Destination_ field specifies the pattern that a request must match with its destination address ([IP or CIDR range](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)).
- _Target_ field specifies where such a request should be routed. It could be `local` (i.e. to targets in this VPC) or `your-internet-gateway-ID` in case those requests should be routed to the gateway for external/somewhere-else access. [A list of possible target values is here](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Route_Tables.html).
\
Here's an example assuming a VPC with addresses 10.0.0.0/16
```
| Destination | Target      |
|:------------|------------:|
| 10.0.0.0/16 | local       |
|  0.0.0.0/0  | your-igw-id |
```

This will route any request with into-the-VPC destination to local targets in the VPC. If the pattern 10.0.0.0/16 is not met, the any-ipv4-address aka 0.0.0.0/0 will be considered and the request will be routed to the internet gateway you specified with ID your-igw-id.

The order of the routes is irrelevant since the most specific route will be chosen.


## 4.2 随着VPC一起创造的Subnets 

![](image/Pasted%20image%2020240712114954.png)

![](image/Pasted%20image%2020240712115018.png)



## 4.3 随着VPC一起创造出来的Security group


![](image/Pasted%20image%2020240712115102.png)



这个 SG allows all traffic within this cluster 
![](image/Pasted%20image%2020240712115434.png)


这个 是 worker node 的 SG
This is for the comunication between the Worker node 
![](image/Pasted%20image%2020240712115638.png)





这个 SG 是为了  communication between master node and worker node 
![](image/Pasted%20image%2020240712115736.png)

![](image/Pasted%20image%2020240712115609.png)




# 5 connect one worker node with kubectl

![](image/Pasted%20image%2020240712115951.png)
![](image/Pasted%20image%2020240712120109.png)

update-kubeconfig 

 ![](image/Pasted%20image%2020240712120244.png)


kubectl get node
![](image/Pasted%20image%2020240712120311.png)


kubectl get pod 
![](image/Pasted%20image%2020240712120347.png)
















# 6 deploy a simple application on one worker node 

![](image/Pasted%20image%2020240712120413.png)



A loadbalancer was created  becuse we create a service wity type Loadbalancer 
![](image/Pasted%20image%2020240712120437.png)

![](image/Pasted%20image%2020240712120510.png)

关联的都是 public subnet of each AZ 

![](image/Pasted%20image%2020240712120600.png)

![](image/Pasted%20image%2020240712120641.png)



















