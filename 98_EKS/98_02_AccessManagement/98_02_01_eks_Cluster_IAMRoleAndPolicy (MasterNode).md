

準備以下身份給 K8s Cluster 使用：

- IAM User `au-eks-admin`: 用來建立 EKS Cluster 的 IAM 身份
- IAM Roles:
    - `K8s-Master-Node-Role`: Master Node 的身份，提供以下的 Policies:
        - `AmazonEKSClusterPolicy`
        - `AmazonEKSServicePolicy`



![](../image/Pasted%20image%2020240711204416.png)


![](../image/Pasted%20image%2020240711205236.png)


