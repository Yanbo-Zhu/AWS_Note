https://www.bilibili.com/video/BV1Fs4y1d7qP/?spm_id_from=333.337.search-card.all.click&vd_source=55e5cc2f534c16c73bbeb684e98c4195


![](../image/Pasted%20image%2020240711204230.png)




# 1 Create EKS IAM Role

![](../image/Pasted%20image%2020240711204416.png)


![](../image/Pasted%20image%2020240711205236.png)


# 2 create VPC for EKS worker node 

why do need another VPC: EKS cluster  needs specific networking configuration 

1 
Worker Nodes need specific Firewall configurations, sothat the Master Node can communicate the Worker Node 
![](../image/Pasted%20image%2020240711205658.png)

Bes practice : configure Public Subnet and Private Subnet 
![](../image/Pasted%20image%2020240711205838.png)



Through IAM Role you give Kubernetes permission to change VPCconfigurations 
![](../image/Pasted%20image%2020240711210003.png)



2 
结论是 wo create this VPC (for Worker Node) with EKS specific configuration  
aws 上有 cloudformation template 我们可以直接用 去create the VPC


![](../image/Pasted%20image%2020240711210311.png)


Template 下载
![](../image/Pasted%20image%2020240711210415.png)

![](../image/Pasted%20image%2020240711210424.png)



![](../image/Pasted%20image%2020240711210855.png)


# 3 create a eks cluster (master node)

![](../image/Pasted%20image%2020240711211103.png)


![](../image/Pasted%20image%2020240711211115.png)


Giving the networking info of worker nodes 
![](../image/Pasted%20image%2020240711211413.png)


----
public and private mode of entrypoint 

![](../image/Pasted%20image%2020240711211647.png)

![](../image/Pasted%20image%2020240711211658.png)


---
Logging through Cloudwatch logs 

![](../image/Pasted%20image%2020240711211730.png)


最后的结果

![](../image/Pasted%20image%2020240711211903.png)



# 4 Connect to EKS Cluster locally with kubectl 


create kubeconfig file 

![](../image/Pasted%20image%2020240711212823.png)


![](../image/Pasted%20image%2020240711212928.png)

![](../image/Pasted%20image%2020240711213007.png)



# 5 Create Node Groups for generating the Pods


With Node Group all necessary processes are automatically installed, 比如说 Containerd, kubelet , kube-proxy in Worker Node , 我们自己就不用费劲去手动安装了. 




![](../image/Pasted%20image%2020240711213354.png)




## 5.1 EC2 Role for Node Group (worker Nodes)

A EC2 Role that policies are attacthed to it so that kubelet has the permission to exceute the tasks 

![](../image/Pasted%20image%2020240711213748.png)


![](../image/Pasted%20image%2020240711213820.png)


![](../image/Pasted%20image%2020240711213851.png)

![](../image/Pasted%20image%2020240711213919.png)


![](../image/Pasted%20image%2020240711214137.png)



## 5.2 Add Node Group to EKS Cluster 


![](../image/Pasted%20image%2020240711214315.png)

![](../image/Pasted%20image%2020240711214325.png)


![](../image/Pasted%20image%2020240711214351.png)

![](../image/Pasted%20image%2020240711215134.png)



然后发现 两个 ec2 instance 已经被制造了 

![](../image/Pasted%20image%2020240711215157.png)

![](../image/Pasted%20image%2020240711215321.png)





 













