

EKS does not manage worker nodes, it is up to you to setup the worker nodes
1. Self-managed nodes
2. Managed node group
3. Fargate

更多见 [[../98_01_01_总览#4 Worker Nodes|98_01_01_总览]]

# 1 Role and Policy 

- IAM Roles:
    - `K8S-Worker-Node-Role`: Worker Node 的身份，提供以下的 Policies:
        - `AmazonEKSWorkerNodePolicy`
        - `AmazonEC2ContainerRegistryReadOnly`
        - `AmazonEKS_CNI_Policy`

attach this roles to the new node group or a ec2 instance which is not in a node group

[[../98_01_SetupEKSCluster/98_01_01_Create_EKScluster_with_AWS_Management_Console#5.1 EC2 Role for Node Group (worker Nodes)|98_01_01_Create_EKScluster_with_AWS_Management_Console]]

# 2 Managed node group

## 2.1 随着 EKS Cluster 的setup 去建立 Node Group

[[../98_01_SetupEKSCluster/98_01_01_Create_EKScluster_with_AWS_Management_Console#5.2 Add Node Group to EKS Cluster|98_01_01_Create_EKScluster_with_AWS_Management_Console]]

[[../98_01_SetupEKSCluster/98_01_03_使用eksctl去创建一个 Cluster#2 使用 eksctl 去创建一个 Cluster eksctl create cluster|98_01_03_使用eksctl去创建一个 Cluster]]


## 2.2 提前自己建立好一个 AutoSaclingGroup(ASG), 将他加入到 eks cluster 中 

[[../98_01_SetupEKSCluster/98_01_02_使用aws_eks_去创建一个Cluster#2.1 建置 Master Nodes|98_01_02_使用aws_eks_去创建一个Cluster]]

[[../98_01_SetupEKSCluster/98_01_03_使用eksctl去创建一个 Cluster#3 将节点组添加到已存在的集群|98_01_03_使用eksctl去创建一个 Cluster]]



## 2.3 AutoScaling  in node Group 
在 Amazon EKS 中，**EKS Managed Node Groups** 本身已经**内置了 Auto Scaling 功能**，你可以在创建节点组时或之后启用它，并设定：
- 最小节点数 (`minSize`)
- 最大节点数 (`maxSize`)
- 所需节点数 (`desiredSize`)
这些参数由 **Auto Scaling Group (ASG)** 支持（==每个 Node Group 背后都有一个 ASG==），并且可以配合 Cluster Autoscaler 实现**按需自动扩缩容**。


# 3 Auto Scaler

EKS 要支援 HPA (Horizontal Pod Autoscaler) 要先準備好什麼？
先準備好 [Metric Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html) ，依照文件步驟安裝即可。




