
https://kubedemy.io/aws-eks-part-18-kubernetes-authentication-with-iam-users

# 1 AWS EKS Cluster Owner:

If you run `kubectl auth whoami` command with the cluster owner, you will get the following result. As you can see, this user is bound to `system:masters` Kubernetes group, which is a group with the highest privileges in the Kubernetes.

```
 ⚡ 🦄  kubectl auth whoami
ATTRIBUTE             VALUE
Username              kubernetes-admin
UID                   aws-iam-authenticator:681290371536:AROAZ5IASCHIAULTZ2QIF
Groups                [system:masters system:authenticated]
Extra: accessKeyId    [ASIAZ5IASCHIACZEZHGJ]
Extra: arn            [arn:aws:sts::681290371536:assumed-role/Admin_ADFS/YZH@ivu.de]
Extra: canonicalArn   [arn:aws:iam::681290371536:role/Admin_ADFS]
Extra: principalId    [AROAZ5IASCHIAULTZ2QIF]
Extra: sessionName    [YZH@ivu.de]
```



# 2 kubenetes中的RBAC (Rola-based Access Control) 结合AWS中的IAM的user 

![](image/Pasted%20image%2020240711154750.png)

![](image/Pasted%20image%2020240711154812.png)

传统k8s 

![](image/Pasted%20image%2020240711155004.png)


# 3 ConfigMap for EKS Authentification


## 3.1 What is AWS IAM Authenticator?

AWS IAM Authenticator is a tool that allows IAM credentials to authenticate and access Kubernetes clusters. In AWS EKS, it runs as a part of the cluster control plane and is managed by AWS as part of their EKS service. You can also use it for non-EKS clusters and create a way to access non-EKS clusters using the AWS IAM service.


## 3.2 IAM Authentication in AWS EKS Clusters:

In EKS clusters, all IAM principals are defined in `aws-auth` ConfigMap within `kube-system` namespace. This configuration is managed by both the EKS service and the cluster administrator. EKS edits this config to add/remove IAM Roles for worker nodes to allow them to join the cluster. The cluster admin can also edit this configuration to add/remove additional users/roles to the cluster. So, to add a new principal, we need to edit this resource. You can see a sample configuration here:

kubectl describe -n kube-system configmaps aws-auth

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::231144931069:role/Kubedemy_EKS_Managed_Nodegroup_Role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - rolearn: arn:aws:iam::231144931069:role/KubernetesAdmin
      username: admin:{{SessionName}}
      groups:
        - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::231144931069:user/kubedemy
      username: kubedemy
      groups:
        - system:masters
  mapAccounts: |
    - "231144931069"
    - "112233445566"
```

mapRoles is a list of IAM roles authorized to access the cluster. For example, EC2 instances which want to join the cluster. You can also add a specific role for cluster administration and allow the role to be assumed by IAM users.

mapUsers is a list of IAM users authorized to access the cluster. Users are individuals like kubedemy user. They can login into the cluster and access the resources based on Kubernetes Roles attached to their username and groups.

mapAccounts is a list of AWS accounts authorized to access the cluster. As you can see, we don’t define usernames and groups for mapped accounts. IAM Authenticator automatically maps IAM ARN from these accounts to the username.

---

## 3.3 ivu eks 中的参数 
```
 ⚡ 🦄  kubectl describe -n kube-system configmaps aws-auth
Name:         aws-auth
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
mapRoles:
----
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::681290371536:role/eks-dev-nodes
  username: system:node:{{EC2PrivateDNSName}}

- groups:
  - system:masters
  rolearn: arn:aws:iam::681290371536:role/Admin_ADFS
  username: admin_adfs
- groups:
  - system:masters
  rolearn: arn:aws:iam::681290371536:role/aws-reserved/sso.amazonaws.com/eu-central-1/AWSReservedSSO_AWS_Administrator_1f3f89ef4f1adcde
  username: admin_sso


BinaryData
====

Events:  <none>
```


## 3.4 应用案例 


### 3.4.1 join a worker node with master node 

[[../98_01_SetupEKSCluster/98_01_02_使用aws_eks_去创建一个Cluster#2.2.1 join a worker node with master node|98_01_02_使用aws_eks_去创建一个Cluster]]

### 3.4.2 EKS Authentication with IAM User setup procedure:

https://kubedemy.io/aws-eks-part-18-kubernetes-authentication-with-iam-users

见 [[../98_01_SetupEKSCluster/98_01_02_使用aws_eks_去创建一个Cluster#2.2.2 修改并添加 IAM 用户或角色|98_01_02_使用aws_eks_去创建一个Cluster]]




