 
![](../image/Pasted%20image%2020240711220202.png)


# 1 准备工作 

install eksctl
configure AWs admin User credentials 



# 2 使用 eksctl 去创建一个 Cluster: eksctl create cluster 

https://www.youtube.com/watch?v=CukYk43agA4

相关命令 
![](../image/Pasted%20image%2020240711114205.png)

![](../image/Pasted%20image%2020240711114221.png)

![](../image/Pasted%20image%2020240711114253.png)


----


创造一个 cluster 

eksctl create cluster -n clusterl --nodegroup-name ng1 --region us-east-1 --node-type t2.mocrp --nodes 2 

![](../image/Pasted%20image%2020240711114442.png)

----

例子2 

![](../image/Pasted%20image%2020240711221207.png)


![](../image/Pasted%20image%2020240711221143.png)


---

例子3 使用 cluster.yaml 
`eksctl create cluster -f cluster.yaml`

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-west-2

nodeGroups:
  - name: ng-1
    instanceType: t3.medium
    desiredCapacity: 2
    minSize: 1
    maxSize: 3
    ssh:
      allow: true
```

![](../image/Pasted%20image%2020240711221157.png)


可以看到 node 被创造了 , 但是 pod 还没有 

![](../image/Pasted%20image%2020240711221517.png)



Role 也被自动创造了 
![](../image/Pasted%20image%2020240711221620.png)


VPC 也被创造  (with name eksctl )
![](../image/Pasted%20image%2020240711222319.png)


subnets  也被创造了 
![](../image/Pasted%20image%2020240711221909.png)



Cluster, two worker node (maps to 2 instances )
![](../image/Pasted%20image%2020240711222530.png)

per node we have  1 cordns, 1 aws-node, 1 kube-porxy 
![](../image/Pasted%20image%2020240711222636.png)



# 3 将节点组添加到已存在的集群 eksctl create nodegroup


```
eksctl create nodegroup \
  --cluster my-cluster \
  --region us-west-2 \
  --name extra-nodes \
  --node-type t3.large \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```



# 4 Connect to the new created cluster 

在执行 上面的 eksctl create cluster 命令的时候， log 中 显示了 通过 kubectl get nodes 能够得到 新创建的 node 的信息 

![](../image/Pasted%20image%2020240711130646.png)

在自己的计算机上执行 kubectl config view 得到新创建的 cluster 的信息 

![](../image/Pasted%20image%2020240711130916.png)

执行 kubectl get nodes 

![](../image/Pasted%20image%2020240711131021.png)


# 5 通过 http 的方式去访问 eks cluster 中的 kubernetes dashboard 

![](../image/Pasted%20image%2020240711160358.png)

![](../image/Pasted%20image%2020240711160415.png)

![](../image/Pasted%20image%2020240711160427.png)


# 6 eksctl delete cluster 

![](../image/Pasted%20image%2020240711131054.png)


# 7 获取集群 kubeconfig（如果配置丢失）


`eksctl utils write-kubeconfig --region us-west-2 --cluster my-cluster`



# 8 查询信息 eksctl get cluster/nodegroup 

```
 ⚡ 🦄  eksctl get cluster --profile ivu-cloud-e2x
NAME    REGION          EKSCTL CREATED
dev     eu-central-1    False


 ⚡ 🦄  eksctl get nodegroup --profile ivu-cloud-e2x --cluster dev
CLUSTER NODEGROUP       STATUS  CREATED                 MIN SIZE        MAX SIZE        DESIRED CAPACITY        INSTANCE TYPE   IMAGE ID                ASG NAME                                                TYPE
dev     dev-prefix      ACTIVE  2025-04-16T11:02:09Z    1               7               3                       r5.2xlarge      AL2023_x86_64_STANDARD  eks-dev-prefix-decb1f21-b454-cd0c-ac3b-4cc75ddc776c     managed


```


