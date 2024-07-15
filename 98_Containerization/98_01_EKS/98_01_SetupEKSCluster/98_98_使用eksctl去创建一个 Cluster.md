 
![](image/Pasted%20image%2020240711220202.png)


# 1 准备工作 

install eksctl
configure AWs admin User credentials 



# 2 使用 eksctl 去创建一个 Cluster 

https://www.youtube.com/watch?v=CukYk43agA4

相关命令 
![](image/Pasted%20image%2020240711114205.png)

![](image/Pasted%20image%2020240711114221.png)

![](image/Pasted%20image%2020240711114253.png)


--

创造一个 cluster 

eksctl create cluster -n clusterl --nodegroup-name ng1 --region us-east-1 --node-type t2.mocrp --nodes 2 

![](image/Pasted%20image%2020240711114442.png)

----

例子2 

![](image/Pasted%20image%2020240711221207.png)


![](image/Pasted%20image%2020240711221143.png)


![](image/Pasted%20image%2020240711221157.png)


可以看到 node 被创造了 , 但是 pod 还没有 

![](image/Pasted%20image%2020240711221517.png)



Role 也被自动创造了 
![](image/Pasted%20image%2020240711221620.png)


VPC 也被创造  (with name eksctl )
![](image/Pasted%20image%2020240711222319.png)


subnets  也被创造了 
![](image/Pasted%20image%2020240711221909.png)



Cluster, two worker node (maps to 2 instances )
![](image/Pasted%20image%2020240711222530.png)

per node we have  1 coreds, 1 aws-node, 1 kube-porxy 
![](image/Pasted%20image%2020240711222636.png)



# 3 Connect to the new created cluster 

在执行 上面的 eksctl create cluster 命令的时候， log 中 显示了 通过 kubectl get nodes 能够得到 新创建的 node 的信息 

![](image/Pasted%20image%2020240711130646.png)

在自己的计算机上执行 kubectl config view 得到新创建的 cluster 的信息 

![](image/Pasted%20image%2020240711130916.png)

执行 kubectl get nodes 

![](image/Pasted%20image%2020240711131021.png)


# 4 通过 http 的方式去访问 eks cluster 中的 dashboard 

![](image/Pasted%20image%2020240711160358.png)

![](image/Pasted%20image%2020240711160415.png)

![](image/Pasted%20image%2020240711160427.png)


# 5 delete the cluster 

![](image/Pasted%20image%2020240711131054.png)

