

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



# 3 HPA. VPA, CA 的比较 


https://blog.csdn.net/linux_s2018/article/details/132277329?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-132277329-blog-121381630.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-132277329-blog-121381630.235%5Ev43%5Econtrol

![[image/Pasted image 20250427134723.png]]


- cluster autoscaler，通过对集群进行 scale up/down 来满足应用需求，另外 Kubernetes 还有以下两种自动扩容、收缩的方法
- Vertical Pod Autoscaler：可以自动调整 Pod 使用的 CPU 和内存，使之符合应用需要
- Horizontal Pod Autoscaler：可以自动调整 deployment 中 Pod 的数量，或者 replication controller，replica set 来符合应用需要



# 4 Horizontal Pod Autoscaling

EKS 要支援 HPA (Horizontal Pod Autoscaler) 要先準備好什麼？

- 先準備好 [Metric Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html) ，依照文件步驟安裝即可。
    - https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html
- Scale pod deployments with Horizontal Pod Autoscaler
    - https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html
- Horizontal Pod Autoscaling
    - https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/


The Kubernetes Metrics Server is an aggregator of resource usage data in your cluster, and it isn’t deployed by default in Amazon EKS clusters. For more information, see [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) on GitHub. The Metrics Server is commonly used by other Kubernetes add ons, such as the [Scale pod deployments with Horizontal Pod Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html) or the [Kubernetes Dashboard](https://docs.aws.amazon.com/eks/latest/userguide/eks-managing.html). For more information, see [Resource metrics pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/) in the Kubernetes documentation. This topic explains how to deploy the Kubernetes Metrics Server on your Amazon EKS cluster.


The Kubernetes [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) automatically scales the number of Pods in a deployment, replication controller, or replica set based on that resource’s CPU utilization. This can help your applications scale out to meet increased demand or scale in when resources are not needed, thus freeing up your nodes for other applications. When you set a target CPU utilization percentage, the Horizontal Pod Autoscaler scales your application in or out to try to meet that target.





# 5 Cluster Autoscaler

- https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#auto-discovery-setup
- https://blog.csdn.net/weixin_41335923/article/details/121381630
- https://blog.csdn.net/qq_41846013/article/details/144350519?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7EPaidSort-2-144350519-blog-121381630.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7EPaidSort-2-144350519-blog-121381630.235%5Ev43%5Econtrol
- https://blog.csdn.net/linux_s2018/article/details/132277329?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-132277329-blog-121381630.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-132277329-blog-121381630.235%5Ev43%5Econtrol
- https://zhuanlan.zhihu.com/p/501209578
- https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#auto-discovery-setup  (官方的 )


当 Pod 失败或被重新安排到其他节点时，Kubernetes [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) 会自动调整集群中的节点数。Cluster Autoscaler 通常作为[部署](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/aws/examples)安装在集群中。它使用[领导选举](https://en.wikipedia.org/wiki/Leader_election)来确保高可用性，但一次只能由一个副本完成扩展。

CA（ cluster-autoscaler）是用来弹性伸缩kubernetes集群的。我们在使用kubernetes集群经常问到的一个问题是，我应该保持多大的节点规模来满足应用需求呢？ cluster-autoscaler的出现解决了这个问题，它可以自动的根据部署的应用所请求的资源量来动态的伸缩集群。

功能可以自动调整集群中 node 的数量以适应需求变化。
Cluster Autoscaler 一般以 Deployment 的方式部署在 K8s 中。AWS EKS Cluster Autoscaler 以 Amazon [EC2 Auto Scaling groups](https://zhida.zhihu.com/search?content_id=199511597&content_type=Article&match_order=1&q=EC2+Auto+Scaling+groups&zhida_source=entity)

服务为基础对 node 进行扩容。
本文以前文的 EKS 测试环境为基础，部署 Cluster Autoscaler，并测试自动扩容。


- **Scale up**: If the scheduler can’t place a pod on any available node due to resource limitations, the Cluster Autoscaler will add new nodes to the cluster.
- **Scale down**: If there are nodes with underutilized resources and no pending pods, the Cluster Autoscaler will remove those nodes to save costs.


---


cluster autoscaler 以 deployment 的形式运行在 EKS 中，通过 service account 赋予的权限来访问 AWS autoscaling group 资源，并控制 node（EC2）的增减。

默认情况下，当有新的 Pod 无法在现有 node 上 schedule 时会触发 scale up，当 node 空闲超过 10min 时，会触发 scale down。

cluster autoscaler 可以在 deployment 中指定参数来控制 node 增减的最大数量`（本文并未设置 --nodes=<min>:<max>:<asg-name>`），但此数量会受 nodegroup 本身设定 node 最大小数量的限制。

我们还可以观察 autoscaler 的日志来查看 sacle 的过程，这个日志比较多所以上面并没有展示

```text
kubectl logs pods/cluster-autoscaler-77cb5c59b7-mcfmj -n kube-system
```

设置 cluster autoscaler 的详细信息，还请参考 cluster autoscaler github 项目


## 5.1 Prerequisites

在部署集群 Cluster Autoscaler之前，您必须满足以下先决条件：
- 拥有现有 Kubernetes 集群 – 如果您没有集群，请参阅 创建 Amazon EKS 集群.  版本 Cluster Autoscaler需要Kubernetes v1.3.0或更高版本
- 权限 Cluster Autoscaler 需要能够检查和修改 EC2 Auto Scaling 组。建议使用服务帐户的IAM角色
- 身份管理 全集群自动缩放器功能策略或者最低IAM策略
- - AWS凭证 服务帐户的IAM角色
- 集群的现有 IAM OIDC 提供商。要确定是否具有 IAM OIDC 提供商，还是需要创建一个，请参阅 为集群创建 IAM OIDC 提供商。 OIDC OIDC 联合身份验证允许您的服务承担 IAM 角色并与 AWS 服务交互，而无需将凭证存储为环境变量
- 具有 Auto Scaling 组标签的节点组 – Cluster Autoscaler 要求 Auto Scaling 组带有以下标签，以便能够自动发现它们。
    - 如果您使用 eksctl 创建节点组，则节点组会自动应用这些标签。
    - 如果未使用 eksctl，则必须使用以下标签手动标记您的 Auto Scaling 组。有关更多信息，请参阅适用于 Linux 实例的 Amazon EC2 用户指南 中的标记 Amazon EC2 资源。

| 密钥                                         | 值       |
| :----------------------------------------- | :------ |
| `k8s.io/cluster-autoscaler/<cluster-name>` | `owned` |
| `k8s.io/cluster-autoscaler/enabled`        | `TRUE`  |

![[image/429fc7b7f1689226fe86578cd1f9b54d.png]]


### 5.1.1 **查看 EC2 Auto Scaling groups Tag**

Cluster Autoscaler 使用 EC2 Auto Scaling groups 服务对 node 进行扩容，我们需要确保 EKS 对应的 Auto Scaling groups 有适合的 Tag。

Cluster Autoscaler 通过 Tag 来识别哪些 Auto Scaling groups 属于其管辖范围。

如果我们使用 eksctl 命令或者 AWS 网页控制台来创建 node group，则所需 Tag 已经自动设置了，如果用其它方式创建的 node group 则需要确保其对应用的 Auto Scaling groups 中有以下两个 Tag

|Key|Value|
|---|---|

说明：“cluster-name”为 EKS 的名称

下面，我们在 EC2 界面查看一下我们之前创建的 EKS node group 对应的 Auto Scaling groups 信息。

在 AWS 中控台选择“EC2”，进入 EC2 界面，在左边列表中选择“Auto Scaling Groups”，在右边选中我们的 Auto Scaling groups

![](https://pica.zhimg.com/v2-bea097033eebccd034bff0488162561e_1440w.jpg)

说明：Auto Scaling groups 的名称在创建 node group 时自动创建，其中会包含 node group 的名称

选择“Details”后，拉到最下面，可以看到需要的两个 Tag 已经打好

![](https://pic2.zhimg.com/v2-494b1596f2ba362aee20f9bee671678f_1440w.jpg)


## 5.2 理论 

3.1 什么时候CA改变集群大小

扩容： 由于资源不足，pod调度失败，即有pod一直处于Pending状态。
缩容：node的资源利用率较低时，且此node上存在的pod都能被重新调度到其他node上运行。


3.2 什么样的节点不会被CA删除

    节点上有pod被PodDisruptionBudget控制器限制。
    节点上有命名空间是kube-system的pods。
    节点上的pod不是被控制器创建，例如不是被deployment, replica set, job, stateful set创建。
    节点上有pod使用了本地存储
    节点上pod驱逐后无处可去，即没有其他node能调度这个pod
    节点有注解：”cluster-autoscaler.kubernetes.io/scale-down-disabled”: “true”

3.3 如何防止节点被CA删除

可以通过给节点打上特定注解保证节点不给CA删除：
```
kubectl annotate node <nodename> cluster-autoscaler.kubernetes.io/scale-down-disabled=true
```


3.4 CA如何与HPA协同工作

HPA（Horizontal Pod Autoscaling）是k8s中pod的水平自动扩展，HPA的操作对象是RC、RS或Deployment对应的Pod，根据观察到的CPU等实际使用量与用户的期望值进行比对，做出是否需要增减实例数量的决策。
当CPU负载增加，HPA扩容pod，如果此pod因为资源不足无法被调度，则此时CA出马扩容节点。
当CPU负载减小，HPA减少pod，CA发现有节点资源利用率低甚至已经是空时，CA就会删除此节点。



### 5.2.1 CA monitor which metric 

**Cluster Autoscaler does not monitor CPU, Memory, or any CloudWatch metrics directly.**
Instead, **Cluster Autoscaler monitors the Kubernetes scheduler** and specifically checks:
- **Are there any Pods in Pending state because there aren’t enough resources (CPU/memory) available?**
- **Are there underutilized nodes that can be safely terminated without affecting running Pods?**

| x                                   | x                                                           |
| ----------------------------------- | ----------------------------------------------------------- |
| **Monitored by Cluster Autoscaler** | Pod scheduling status (Pending Pods), Node utilization      |
| **Not monitored**                   | CPUUtilization, MemoryUtilization, or any CloudWatch metric |

- **If there are Pending Pods** (Pods that can't be scheduled because of not enough CPU, memory, etc.), Cluster Autoscaler will **add more nodes**.
- **If there are underutilized nodes** (nodes with very few Pods), Cluster Autoscaler will **remove nodes** to save costs.

It looks at **Kubernetes scheduling**, **resource requests/limits**, and **Pod placement**, **not** raw performance metrics like CPU usage.


## 5.3 创建 IAM 策略和角色


创建授予 Cluster Autoscaler 使用 IAM 角色所需权限的 IAM 策略。将整个过程中的所有 `<example-values>`（包括 `<>`）替换为您自己的值。

### 5.3.1 创建一个 IAM 策略

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#auto-discovery-setup

Permissions required when using [ASG Autodiscovery](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#Auto-discovery-setup) and Dynamic EC2 List Generation (the default behaviour). In this example, only the second block of actions should be updated to restrict the resources/add conditionals:
like 
```
      "Resource": ["arn:aws:autoscaling:${YOUR_CLUSTER_AWS_REGION}:${YOUR_AWS_ACCOUNT_ID}:autoScalingGroup:*:autoScalingGroupName/${YOUR_ASG_NAME}"]
```


将以下内容保存到名为 `cluster-autoscaler-policy.json` 的文件中。如果现有节点组是使用 `eksctl` 创建的并且您使用了 `--asg-access` 选项，则此策略已存在，您可以跳至第 2 步。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "ec2:DescribeImages",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": ["*"]
    }
  ]
}
```

下面这个不用 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```


使用以下命令创建策略。您可以更改 `policy-name` 的值。

```sh
aws iam create-policy \
    --policy-name AmazonEKSClusterAutoscalerPolicy \
    --policy-document file://cluster-autoscaler-policy.json
```


记下输出中返回的 Amazon Resource Name (ARN)。您需要在后面的步骤中用到它。


![[image/v2-83fdfe7e4f62c1b8fcccaa24284524b0_1440w.jpg]]


#### 5.3.1.1 `"eks:DescribeNodegroup"`

The `"eks:DescribeNodegroup"` permission allows Cluster Autoscaler to pull labels and taints from the EKS DescribeNodegroup API for EKS managed nodegroups. (Note: When an EKS DescribeNodegroup API label and a tag on the underlying autoscaling group have the same key, the EKS DescribeNodegroup API label value will be saved by the Cluster Autoscaler over the autoscaling group tag value.) Currently the Cluster Autoscaler will only call the EKS DescribeNodegroup API when a managed nodegroup is created with 0 nodes and has never had any nodes added to it. Once nodes are added, even if the managed nodegroup is scaled back to 0 nodes, this functionality will not be called anymore. In the case of a Cluster Autoscaler restart, the Cluster Autoscaler will need to repopulate caches so it will call this functionality again if the managed nodegroup is at 0 nodes. Enabling this functionality any time there are 0 nodes in a managed nodegroup (even after a scale-up then scale-down) would require changes to the general shared Cluster Autoscaler code which could happen in the future.


**大意是：**

- **权限说明**：`eks:DescribeNodegroup` 这个 IAM 权限，允许 **Cluster Autoscaler** 通过调用 **EKS 的 DescribeNodegroup API**，拉取 **EKS 托管节点组（managed nodegroup）** 的 **标签（labels）和污点（taints）** 信息。
    
- **标签优先级**：如果某个 key 同时存在于节点组的 **EKS 标签（API返回的label）** 和 **底层 Auto Scaling Group 的标签（tag）**，那么 **Cluster Autoscaler** 会优先保存 **EKS API 的标签值**，而不是 Auto Scaling Group 的标签值。
    
- **调用时机**：
    
    - **只有在**托管节点组刚创建出来，且 **节点数为 0**，并且 **还没有任何节点加入过** 的情况下，Cluster Autoscaler 才会调用 `DescribeNodegroup` 来拉取这些信息。
        
    - **一旦**节点组中 **加入过节点**，哪怕之后又缩到 0 个节点，Cluster Autoscaler **也不会再调用** `DescribeNodegroup` API。
        
    - **如果** Cluster Autoscaler **重启了**（比如 pod 重启），它需要重新初始化内部缓存，这时如果节点组当前是 0 节点，它又会再次调用 `DescribeNodegroup`。
        
- **未来计划**：
    
    - 目前这种“只在最开始（且节点数量为 0）调用一次”的逻辑，是写在 **Cluster Autoscaler 公共代码**里的。
        
    - 如果以后想要在每次节点组回到 0 个节点时都重新拉取一次标签，需要改动 Cluster Autoscaler 的核心代码，**以后可能会改，但目前还没改。**
        



**总结一句话就是：**  
`eks:DescribeNodegroup` 这个权限目前只在节点组第一次创建且为 0 节点时由 Cluster Autoscaler 调用，用来同步标签和污点；后续不会频繁调用，除非 Autoscaler 重启。



### 5.3.2 更新一个 IAM Role 的信任策略 

https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html

**`aws iam update-assume-role-policy`** 是用来 **更新一个 IAM Role 的信任策略 (trust policy)** 的。
- 每个 IAM Role 都有一个**信任策略**，定义了**谁可以扮演（assume）这个角色**。
- 这个命令可以把一个新的信任策略直接覆盖到已有的 Role 上。

vim trues-policy.json

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<AccountID>:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/<OICD_ID>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-east-1.amazonaws.com/id/<OICD_ID>:sub": "system:serviceaccount:kube-system:cluster-autoscaler"
        }
      }
    }
  ]
}


```

```

###更新角色信任关系
aws iam update-assume-role-policy --role-name <role-name> --policy-document file://trust-policy.json

更新角色信任关系 
aws iam update-assume-role-policy --role-name AmazonEKSClusterAutoscalerRole --policy-document file://"trust-policy.json"
```

说明：IAM Role 中包括 IAM policy 和 trust relationship 两部分，我们先用 json 文件来定义 trust relationship 的内容。

trust-policy.json 在创建 Role 时，指定“Trust relationships”中的内容

    修改“252557384592”为自己的 AWS Account
    修改“us-east-1”为自己的 Region
    修改“OpenID Connect provider URL”为自己 EKS 的 OpenID Connect provider URL 中最后的字符串，如下所示
    在这里插入图片描述


![[image/v2-8d8f2873fe67f3136b2292d8e279f723_1440w.jpg]]



### 5.3.3 创建 IAM 角色
```
aws iam create-role \
  --role-name AmazonEKSClusterAutoscalerRole \
  --assume-role-policy-document file://"trust-policy.json"
 
###更新角色信任关系aws iam update-assume-role-policy --role-name <role-name> --policy-document file://trust-policy.json

更新角色信任关系 
aws iam update-assume-role-policy --role-name AmazonEKSClusterAutoscalerRole --policy-document file://"trust-policy.json"

```

说明：
role-name：自定义 Role 的名称  
assume-role-policy-document：指定本地 trust-policy 文件

---

**为角色附加策略**
```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::<AccountID>:policy/AmazonEKSClusterAutoscalerPolicy \
  --role-name AmazonEKSClusterAutoscalerRole

```


### 5.3.4 创建 service account


https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html#_step_2_create_and_associate_iam_role
https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html

Applications in a Pod’s containers can use an AWS SDK or the AWS CLI to make API requests to AWS services using AWS Identity and Access Management (IAM) permissions. Applications must sign their AWS API requests with AWS credentials. **IAM roles for service accounts (IRSA)** provide the ability to manage credentials for your applications, similar to the way that Amazon EC2 instance profiles provide credentials to Amazon EC2 instances. Instead of creating and distributing your AWS credentials to the containers or using the Amazon EC2 instance’s role, ==you associate an IAM role with a Kubernetes service account and configure your Pods to use the service account==. You can’t use IAM roles for service accounts with [local clusters for Amazon EKS on AWS Outposts](https://docs.aws.amazon.com/eks/latest/userguide/eks-outposts-local-cluster-overview.html).


Cluster Autoscaler requires the ability to examine and modify EC2 Auto Scaling Groups. We recommend using [IAM roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) to associate the Service Account that the Cluster Autoscaler Deployment runs as with an IAM role that is able to perform these functions. If you are unable to use IAM Roles for Service Accounts, you may associate an IAM service role with the EC2 instance on which the Cluster Autoscaler pod runs.

创建一个 service account 并使用 eksctl 或 AWS Management Console 向其附加 IAM 策略。为以下说明选择所需的选项卡。
使用 --asg-access 选项创建集群，
替换为eksctl为您创建的 IAM 策略的名称
策略名称类似于 `eksctl-<cluster-name>-nodegroup-ng-<xxxxxxxx>-PolicyAutoScaling`

---

- Replace my-service-account with the name of the Kubernetes service account that you want eksctl to create and associate with an IAM role. 
- Replace default with the namespace that you want eksctl to create the service account in. 
- Replace my-cluster with the name of your cluster. 
- Replace my-role with the name of the role that you want to associate the service account to. ==If it doesn’t already exist, eksctl creates it for you==. 
- replace 111122223333 with your account ID and my-policy with the name of an existing policy.


```
eksctl create iamserviceaccount --name my-service-account --namespace default --cluster my-cluster --role-name my-role \
    --attach-policy-arn arn:aws:iam::111122223333:policy/my-policy --approve
```

If the role or service account already exist, the previous command might fail. `eksctl` has different options that you can provide in those situations. For more information run `eksctl create iamserviceaccount --help`.



```sh
eksctl create iamserviceaccount \
  --cluster=<my-cluster> \
  --namespace=kube-system \
  --name=cluster-autoscaler \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/<AmazonEKSClusterAutoscalerPolicy> \
  --override-existing-serviceaccounts \
  --approve
```


### 5.3.5 Confirm that the IAM role’s trust policy is configured correctly.
https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html#irsa-confirm-role-configuration



- ```
    aws iam get-role --role-name my-role --query Role.AssumeRolePolicyDocument
    ```
    
    An example output is as follows.
    
    `{     "Version": "2012-10-17",     "Statement": [         {             "Effect": "Allow",             "Principal": {                 "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"             },             "Action": "sts:AssumeRoleWithWebIdentity",             "Condition": {                 "StringEquals": {                     "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:default:my-service-account",                     "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com"                 }             }         }     ] }`
    
- Confirm that the policy that you attached to your role in a previous step is attached to the role.
    
- ```
    aws iam list-attached-role-policies --role-name my-role --query "AttachedPolicies[].PolicyArn" --output text
    ```
    
    An example output is as follows.
    
                   `arn:aws:iam::111122223333:policy/my-policy`
    
- Set a variable to store the Amazon Resource Name (ARN) of the policy that you want to use. Replace `my-policy` with the name of the policy that you want to confirm permissions for.
    
- ```
    export policy_arn=arn:aws:iam::111122223333:policy/my-policy
    ```
    
- View the default version of the policy.
    
- ```
    aws iam get-policy --policy-arn $policy_arn
    ```
    
    An example output is as follows.
    
    `{     "Policy": {         "PolicyName": "my-policy",         "PolicyId": "EXAMPLEBIOWGLDEXAMPLE",         "Arn": "arn:aws:iam::111122223333:policy/my-policy",         "Path": "/",         "DefaultVersionId": "v1",         [...]     } }`
    
- View the policy contents to make sure that the policy includes all the permissions that your Pod needs. If necessary, replace `1` in the following command with the version that’s returned in the previous output.
    
- ```
    aws iam get-policy-version --policy-arn $policy_arn --version-id v1
    ```
    
    An example output is as follows.
    
    `{     "Version": "2012-10-17",     "Statement": [         {             "Effect": "Allow",             "Action": "s3:GetObject",             "Resource": "arn:aws:s3:::my-pod-secrets-bucket"         }     ] }`
    
    If you created the example policy in a previous step, then your output is the same. If you created a different policy, then the `example` content is different.
    
- Confirm that the Kubernetes service account is annotated with the role.
    

```
kubectl describe serviceaccount my-service-account -n default
```

An example output is as follows.

`Name:                my-service-account Namespace:           default Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::111122223333:role/my-role Image pull secrets:  <none> Mountable secrets:   my-service-account-token-qqjfl Tokens:              my-service-account-token-qqjfl [...]`

## 5.4 部署 Cluster Autoscaler

### 5.4.1 部署 Cluster Autoscaler 使用yaml 


0 运行以下命令下载 Cluster Autoscaler 的 yaml 文件

```text
#设置代理
export http_proxy=YOUR_PROXY_SERVER
export https_proxy=YOUR_PROXY_SERVER
#下载yaml文件
wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

![](https://pic3.zhimg.com/v2-1e83807c8f4cb81ff3cb70eb40b010fa_1440w.jpg)


1 

用文本编辑打开 cluster-autoscaler-autodiscover.yaml 文件，进行以下操作

`查找并替换“<YOUR CLUSTER NAME>”为我们 EKS 的名称`

![](https://pic4.zhimg.com/v2-3dd4fac8365df5f691026f6c6cb7cb09_1440w.jpg)

  
在 EKS 的名称“tsEKS”下面，并添加以下两行  
```text
- --balance-similar-node-groups
- --skip-nodes-with-system-pods=false
```


把 cluster-autoscaler 的镜像版本换成上面查到的版本 1.21.1

更改之后结果如下，保存退出

![](https://pic3.zhimg.com/v2-771323c65e68b87301b0b408bca423c8_1440w.jpg)

  

2 
直接在集群中部署即可，简化的yaml如下所示，启动参数按需添加，其中{{MIN}}是最小节点数，{{MAX}}是最大节点数

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cluster-autoscaler
  labels:
    k8s-app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: cluster-autoscaler
  template:
    metadata:
      labels:
        k8s-app: cluster-autoscaler
    spec:
      containers:
        - image: cluster-autoscaler:latest
          name: cluster-autoscaler
          command:
            - ./cluster-autoscaler
            - --nodes={{MIN}}:{{MAX}}:k8s-worker-asg-1
```

### 5.4.2 部署 Cluster Autoscaler  使用 kubectl 命令 

要部署 Cluster Autoscaler，请完成以下步骤。建议您查看 [部署注意事项](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/cluster-autoscaler.html#ca-deployment-considerations) 并优化 Cluster Autoscaler 部署，然后再将其部署到生产集群。

0 运行以下命令下载 Cluster Autoscaler 的 yaml 文件

```text
#设置代理
export http_proxy=YOUR_PROXY_SERVER
export https_proxy=YOUR_PROXY_SERVER
#下载yaml文件
wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

![](https://pic3.zhimg.com/v2-1e83807c8f4cb81ff3cb70eb40b010fa_1440w.jpg)


---


1 部署 Cluster Autoscaler。
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

![[image/v2-5ceaa76c244b4ef0d18a4f48f408b3da_1440w.jpg]]


查看 Cluster Autoscaler pod

```text
kubectl get pods -n kube-system
```

![](https://pic3.zhimg.com/v2-71aa59a90e8818d9738914abfbda98d4_1440w.jpg)


---


2 使用您以前创建的 IAM 角色的 ARN 对 `cluster-autoscaler` 服务账户添加注释。将`<示例值>`替换为您自己的值。

```sh
kubectl annotate serviceaccount cluster-autoscaler \
  -n kube-system \
  eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT_ID>:role/<AmazonEKSClusterAutoscalerRole>
```

说明：我们也可以在 cluster-autoscaler-autodiscover.yaml 文件中直接修改 service account “cluster-autoscaler”增加 annotations

```
#文件
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::xxxxx:role/<Role-name>  # Add the IAM role created in the above C section.
  name: cluster-autoscaler
  namespace: kube-system

```



---



3 使用以下命令修补部署以向 Cluster Autoscaler Pod 添加 `cluster-autoscaler.kubernetes.io/safe-to-evict` 注释。
```sh
kubectl patch deployment cluster-autoscaler \
  -n kube-system \
  -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'
```

说明：同样的，我们也可以在 cluster-autoscaler-autodiscover.yaml 文件中直接修改 deployment 增加 annotations

如果有 autoscaler 有新的镜像，我们可以通过以下命令设置直接在 deployment 中更改镜像版本

```text
kubectl set image deployment cluster-autoscaler \
  -n kube-system \
  cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v<1.21.n>
```


---


4 使用以下命令编辑 Cluster Autoscaler 部署

`kubectl -n kube-system edit deployment.apps/cluster-autoscaler`

编辑 cluster-autoscaler 容器命令，将 `<YOUR CLUSTER NAME>（包括 <>）`替换为您的集群名称，然后添加以下选项。
    --balance-similar-node-groups
    --skip-nodes-with-system-pods=false

```yaml
    spec:
      containers:
      - command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```

保存并关闭该文件以应用更改。



---


5
在 Web 浏览器中从 GitHub 打开 Cluster Autoscaler 版本页面，找到与您集群的 Kubernetes 主版本和次要版本相匹配的 Cluster Autoscaler 最新版本。例如，如果您集群的 Kubernetes 版本是 1.21，则查找以 1.21 开头的最新 Cluster Autoscaler 版本。记录该版本的语义版本号 (1.21.*n*) 以在下一步中使用。


---


6
使用以下命令，将 Cluster Autoscaler 映像标签设置为您在上一步中记录的版本。将 1.21.*n* 替换为您自己的值。

```sh
kubectl set image deployment cluster-autoscaler \
  -n kube-system \
  cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v<1.21.n>
```


## 5.5 Auto-Discovery Setup

Auto-Discovery Setup is the preferred method to configure Cluster Autoscaler.

To enable this, provide the `--node-group-auto-discovery` flag as an argument whose value is a list of tag keys that should be looked for. ==For example, `--node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<cluster-name>` will find the ASGs that have at least all the given tags. Without the tags, the Cluster Autoscaler will be unable to add new instances to the ASG as it has not been discovered. ==

In the example, a value is not given for the tags and in this case any value will be ignored and will be arbitrary - only the tag name matters. Optionally, the tag value can be set to be usable and custom tags can also be added. For example, `--node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled=foo,k8s.io/cluster-autoscaler/<cluster-name>=bar,my-custom-tag=custom-value`. Now the ASG tags must have the correct values as well as the custom tag to be successfully discovered by the Cluster Autoscaler.


Example deployment:

```
kubectl apply -f examples/cluster-autoscaler-autodiscover.yaml
```

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.26.2
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
            # - --nodes=1:10:k8s-worker-asg-1    # 不要和  --node-group-auto-discovery 一起使用 
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```


Cluster Autoscaler will respect the minimum and maximum values of each Auto Scaling Group. It will only adjust the desired value.


----
### 5.5.1 Cluster Autoscaler  如何去确定 每个 node 的 capability 

Each Auto Scaling Group should be composed of instance types that provide approximately equal capacity. For example, ASG "xlarge" could be composed of m5a.xlarge, m4.xlarge, m5.xlarge, and m5d.xlarge instance types, because each of those provide 4 vCPUs and 16GiB RAM. Separately, ASG "2xlarge" could be composed of m5a.2xlarge, m4.2xlarge, m5.2xlarge, and m5d.2xlarge instance types, because each of those provide 8 vCPUs and 32GiB RAM.


==Cluster Autoscaler will attempt to determine the CPU, memory, and GPU resources provided by an Auto Scaling Group based on the instance type specified in its Launch Configuration or Launch Template==. It will also examine any overrides provided in an ASG's Mixed Instances Policy. If any such overrides are found, only the first instance type found will be used. See [Using Mixed Instances Policies and Spot Instances](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#Using-Mixed-Instances-Policies-and-Spot-Instances) for details.


### 5.5.2 Cluster Autoscaler  根据 ASG 的 tag 去做一些行为 

下面的标签都是 打在 ASG 上面的

When scaling up from 0 nodes, the Cluster Autoscaler reads ASG tags to derive information about the specifications of the nodes i.e labels and taints in that ASG. Note that it does not actually apply these labels or taints - this is done by an AWS generated user data script. It gives the Cluster Autoscaler information about whether pending pods will be able to be scheduled should a new node be spun up for a particular ASG with the asumption the ASG tags accurately reflect the labels/taint actually applied.


 这些 Label、Taint 必须在节点启动时通过 kubelet 配置应用上，Cluster Autoscaler 不会主动为你设置节点上的 taints。

**NOTE:** It is your responsibility to ensure such labels and/or taints are applied via the node's kubelet configuration at startup. Cluster Autoscaler will not set the node taints for you.


---

Recommendations:

- It is recommended to use a second tag like `k8s.io/cluster-autoscaler/<cluster-name>` when `k8s.io/cluster-autoscaler/enabled` is used across many clusters to prevent ASGs from different clusters having conflicts. An ASG must contain at least all the tags specified and as such secondary tags can differentiate between different clusters ASGs.
- To prevent conflicts, do not provide a `--nodes` argument if `--node-group-auto-discovery` is specified.
- Be sure to add `autoscaling:DescribeLaunchConfigurations` or `ec2:DescribeLaunchTemplateVersions` to the `Action` list of the IAM Policy used by Cluster Autoscaler, depending on whether your ASG utilizes Launch Configurations or Launch Templates.
- If Cluster Autoscaler adds a node to the cluster, and the node has taints applied when it joins the cluster that Cluster Autoscaler was unaware of (because the tag wasn't supplied in ASG), this can lead to significant confusion and misbehaviour.


- 如果你在多个 EKS 集群共用 `k8s.io/cluster-autoscaler/enabled` 标签，建议加上一个专门的 `k8s.io/cluster-autoscaler/<集群名字>` 标签来区分不同集群，避免冲突。
    
- 如果使用了 `--node-group-auto-discovery`，就不要再使用 `--nodes` 参数，避免配置冲突。
    
- IAM Policy 中应该加上 `autoscaling:DescribeLaunchConfigurations` 或 `ec2:DescribeLaunchTemplateVersions` 权限（具体取决于 ASG 是用 Launch Configuration 还是 Launch Template）。
    
- 如果扩容的新节点带有 Cluster Autoscaler 未知的 taints（因为 ASG 没有正确打 tag），可能导致调度混乱或异常行为。


---

当你的应用使用了 `NodeSelector`，为了让 Cluster Autoscaler 能正确识别哪个 Auto Scaling Group（ASG）可以扩容，需要在 ASG 上打上特定的标签。

The following is only required if scaling up from 0 nodes. The Cluster Autoscaler will require the label tag on the ASG should a deployment have a NodeSelector, else no scaling will occur as the Cluster Autoscaler does not realise the ASG has that particular label. The tag is of the format `k8s.io/cluster-autoscaler/node-template/label/<label-name>`: `<label-value>` is the name of the label and the value of each tag specifies the label value.

Example tags:
- `k8s.io/cluster-autoscaler/node-template/label/foo`: `bar`


---

**污点（Taint）要求：**  
同样地，为了防止扩容出带有污点但无法运行待调度 Pod 的节点，需要在 ASG 上打上 taint 的标签。

The following is only required if scaling up from 0 nodes. The Cluster Autoscaler will require the taint tag on the ASG, else tainted nodes may get spun up that cannot actually have the pending pods run on it. The tag is of the format `k8s.io/cluster-autoscaler/node-template/taint/<taint-name>`:`<taint-value:taint-effect>` is the name of the taint and the value of each tag specifies the taint value and effect with the format `<taint-value>:<taint-effect>`.

Example tags:

- `k8s.io/cluster-autoscaler/node-template/taint/dedicated`: `true:NoSchedule`

---

资源（Resource）描述（从 1.14 版本起支持）：
Cluster Autoscaler 可以通过标签识别每个 ASG 提供的资源量


From version 1.14, Cluster Autoscaler can also determine the resources provided by each Auto Scaling Group via tags. The tag is of the format `k8s.io/cluster-autoscaler/node-template/resources/<resource-name>`. `<resource-name>` is the name of the resource, such as `ephemeral-storage`. The value of each tag specifies the amount of resource provided. The units are identical to the units used in the `resources` field of a Pod specification.

Example tags:

- `k8s.io/cluster-autoscaler/node-template/resources/ephemeral-storage`: `100G`



---

自动缩放选项（Auto Scaling Options）：
你可以在 ASG 上通过标签单独覆盖 Cluster Autoscaler 的一些全局配置

ASG labels can specify autoscaling options, overriding the global cluster-autoscaler settings for the labeled ASGs. Those labels takes the same values format as the cluster-autoscaler command line flags they override (a float or a duration, encoded as string). Currently supported autoscaling options (and example values) are:

- `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownutilizationthreshold`: `0.5` (overrides `--scale-down-utilization-threshold` value for that specific ASG)
- `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledowngpuutilizationthreshold`: `0.5` (overrides `--scale-down-gpu-utilization-threshold` value for that specific ASG)
- `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownunneededtime`: `10m0s` (overrides `--scale-down-unneeded-time` value for that specific ASG)
- `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownunreadytime`: `20m0s` (overrides `--scale-down-unready-time` value for that specific ASG)
- `k8s.io/cluster-autoscaler/node-template/autoscaling-options/ignoredaemonsetsutilization`: `true` (overrides `--ignore-daemonsets-utilization` value for that specific ASG)


## 5.6 Manual configuration

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#manual-configuration

Cluster Autoscaler can also be configured manually if you wish by passing the `--nodes` argument at startup. The format of the argument is `--nodes=<min>:<max>:<asg-name>`, where `<min>` is the minimum number of nodes, `<max>` is the maximum number of nodes, and `<asg-name>` is the Auto Scaling Group name.

You can pass multiple `--nodes` arguments if you have multiple Auto Scaling Groups you want Cluster Autoscaler to use.

**NOTES**:

- Both `<min>` and `<max>` must be within the range of the minimum and maximum instance counts specified by the Auto Scaling group.
- When manual configuration is used, all Auto Scaling groups must use EC2 instance types that provide equal CPU and memory capacity.

Examples:

 One ASG Setup (min: 1, max: 10, ASG Name: k8s-worker-asg-1)
```
kubectl apply -f examples/cluster-autoscaler-one-asg.yaml
```

Multiple ASG Setup
```
kubectl apply -f examples/cluster-autoscaler-multi-asg.yaml
```


## 5.7 查看 Cluster Autoscaler 日志

部署 Cluster Autoscaler 之后，您可以查看日志并验证它在监控您的集群负载。

使用以下命令查看您的 Cluster Autoscaler 日志。


您可以在一个 (扩展) 代码行中执行所有这些操作：
```sh
I0926 23:15:55.165842       1 static_autoscaler.go:138] Starting main loop
I0926 23:15:55.166279       1 utils.go:595] No pod using affinity / antiaffinity found in cluster, disabling affinity predicate for this loop
I0926 23:15:55.166293       1 static_autoscaler.go:294] Filtering out schedulables
I0926 23:15:55.166330       1 static_autoscaler.go:311] No schedulable pods
I0926 23:15:55.166338       1 static_autoscaler.go:319] No unschedulable pods
I0926 23:15:55.166345       1 static_autoscaler.go:366] Calculating unneeded nodes
I0926 23:15:55.166357       1 utils.go:552] Skipping ip-192-168-3-111.<region-code>.compute.internal - node group min size reached
I0926 23:15:55.166365       1 utils.go:552] Skipping ip-192-168-71-83.<region-code>.compute.internal - node group min size reached
I0926 23:15:55.166373       1 utils.go:552] Skipping ip-192-168-60-191.<region-code>.compute.internal - node group min size reached
I0926 23:15:55.166435       1 static_autoscaler.go:393] Scale down status: unneededOnly=false lastScaleUpTime=2019-09-26 21:42:40.908059094 ...
I0926 23:15:55.166458       1 static_autoscaler.go:403] Starting scale down
I0926 23:15:55.166488       1 scale_down.go:706] No candidates for scale down

```


## 5.8 验证


默认扩大节点组节点数量
扩大规模如何运作？
纵向扩展在 API 服务器上创建一个监视来查找所有 Pod。它每 10 秒检查一次任何不可调度的 pod（可通过--scan-interval标志配置）。当 Kubernetes 调度程序无法找到可以容纳 pod 的节点时，该 pod 就无法调度。例如，Pod 可以请求任何集群节点上可用的更多 CPU。不可调度的 pod 通过其 PodCondition 进行识别。每当 Kubernetes 调度程序无法找到运行 pod 的位置时，它就会将“schedulable”PodCondition 设置为 false，并将 Reason 设置为“unschedulable”。如果不可调度的 Pod 列表中有任何项目，Cluster Autoscaler 会尝试找到新的位置来运行它们。

默认缩减节点组节点数量
每 10 秒（可通过--scan-interval标志配置），如果不需要扩展，Cluster Autoscaler 会检查哪些节点是不需要的。当满足以下所有条件时，将考虑删除节点：
1，该节点上运行的所有 Pod 的 CPU 和内存请求总和（默认情况下包括DaemonSet Pod和Mirror Pod--ignore-daemonsets-utilization ，但可以使用和--ignore-mirror-pods-utilization标志进行配置）小于该节点可分配的 50%。（在 1.1.0 之前，使用节点容量而不是可分配的容量。）可以使用 --scale-down-utilization-threshold标志配置利用率阈值。
2，节点上运行的所有 pod（默认情况下在所有节点上运行的 pod 除外，例如清单运行 pod 或 daemonset 创建的 pod）都可以移动到其他节点。请参阅 哪些类型的 Pod 可以阻止 CA 删除节点？部分了解有关哪些 pod 不满足此条件的更多详细信息，即使其他地方有空间容纳它们。在检查此情况时，所有可移动吊舱的新位置都会被记住。这样，Cluster Autoscaler 就知道每个 Pod 可以移动到哪里，以及哪些节点在 Pod 迁移方面依赖于哪些其他节点。当然，最终调度程序可能会将 Pod 放置在其他位置。
3，它没有缩小禁用注释（请参阅如[何防止 Cluster Autoscaler 缩小特定节点？](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#how-can-i-prevent-cluster-autoscaler-from-scaling-down-a-particular-node)）



### 5.8.1 
```sh
#测试集群自动绽放程序是否启动及角色是否已附加
$ kubectl get pods -n kube-system
$ kubectl exec -n kube-system cluster-autoscaler-xxxxxx-xxxxx  env | grep AWS

#测试命令
kubectl scale deployment autoscaler-demo --replicas=50

#扩展日志
I1025 13:48:42.975037       1 scale_up.go:529] Final scale-up plan: [{eksctl-xxx-xxx-xxx-nodegroup-ng-xxxxx-NodeGroup-xxxxxxxxxx 2->3 (max: 8)}]

#获取节点组信息
eksctl get nodegroup --cluster <cluster-name>
#调整节点组现有节点数
eksctl scale nodegroup --cluster <cluster-name> --name <nodegroup-name> --nodes <number>
#调整节点组最小节点数
eksctl scale nodegroup --cluster <cluster-name> --name <nodegroup-name> --nodes-min <number>
#调整节点组最大节点数
eksctl scale nodegroup --cluster <cluster-name> --name <nodegroup-name> --nodes-max <number>

```


### 5.8.2 

新建文件 testpod.yaml，并粘贴以下内容

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

说明：testpod.yaml 创建一个简单的 deployment，其中运行一个 nignx Pod

运行以下命令创建 deployment

```text
kubectl apply -f testpod.yaml
kubectl get deployments
```

可以看到这个 deployment 中有一个 pod 在运行

![](https://pica.zhimg.com/v2-dbb9132e713a26d4f52bd83ca194a872_1440w.jpg)

---


 **scale up**

下面我们把 deployment 中运行 pod 的数量改成 5 个，运行以下命令修改 deployment

```text
kubectl edit deployment/nginx-deployment
```

kubectl edit 会打开默认的文本编辑器，我们把 replicas 改成 5


![](https://pic4.zhimg.com/v2-85cb0ee7f3de12528e6f3c7b6b94f05f_1440w.jpg)

  

然后保存退出

![](https://pic3.zhimg.com/v2-a56bb4a9c85c3ddfec22a9368acf344c_1440w.jpg)

  

这时再查看 deployment 和 pod 的信息

```text
kubectl get deployments
kubectl get pods
```

可以看到目前只有一个 pod 是 running，剩下 4 个是 pending 状态

![](https://pic4.zhimg.com/v2-7d1ba736655f60ac91373570cd211667_1440w.jpg)

![](https://pic1.zhimg.com/v2-b6c78451a657d7d13fb1b1a4c96f27ea_1440w.jpg)

注意：如果在你的环境中 5 个 pod 马上就运行成功了，说明资源足够，那我们可以把 replicas 改成 10 或者更大，以触发扩容。

我们用 describe 命令查看其中一个 Pod

```text
kubectl describe pods/nginx-deployment-66b6c48dd5-642b9
```

可以看到自动扩展已经触发，但达到最大 group size 所以失败了

```text
Events:
  Type     Reason             Age                  From                Message
  ----     ------             ----                 ----                -------
  Normal   NotTriggerScaleUp  49s (x121 over 20m)  cluster-autoscaler  pod didn't trigger scale-up: 1 max node group size reached
  Warning  FailedScheduling   26s (x21 over 21m)   default-scheduler   0/2 nodes are available: 2 Too many pods.
```

![](https://pic1.zhimg.com/v2-24a398b0befb0685e246d48cd29d85ca_1440w.jpg)

原因在于，我们之前创建的 node group，最大 node 数量是 2，我们目前的 node 数量已经达到最大

node group 信息如下

![](https://pic3.zhimg.com/v2-a2c5747ce7453026b7d6123269a3ce90_1440w.jpg)

  

autoscaling groups 里的信息

![](https://picx.zhimg.com/v2-103ac077ff6aaf91e3876dfbdbfda921_1440w.jpg)

  

下面，我们调整 node group 的 max size

在 EKS 界面，选择我们的“tsEKS”，选择“Configuration”，选择“Compute”，勾选 node group 名称“tsEKSnodeGrp”，点击“Edit”

![](https://pic4.zhimg.com/v2-b4a72982fa5471cbb429f041bec802ad_1440w.jpg)

  

把“Maximum size”改成 3，然后拖到最下面，点击“Save changes”

![](https://pica.zhimg.com/v2-8eb297fecbc99c65e127174d955ae136_1440w.jpg)

  

![](https://pic3.zhimg.com/v2-1eb7837e30746af69c63b7af80a06a04_1440w.jpg)

更改完成

![](https://pic2.zhimg.com/v2-7deb266c76962234891e21225669c901_1440w.jpg)

  

过一会儿，我们刷新界面，可以看到新的 node

![](https://pica.zhimg.com/v2-27d4106ddd857d49cc5b0744a8b3b902_1440w.jpg)

  

通过命令也可以看到新的 node

![](https://pic3.zhimg.com/v2-0034a7caea17da5c54bd48b5de396c02_1440w.jpg)

  

这时再观察 deployment 和 pod，可以看到所有 pod 都正常运行了，说明扩容成功了

![](https://picx.zhimg.com/v2-9588150b8afbcd6fee7c4b5cacc2ed93_1440w.jpg)

  ---
  
**scale down**

下面我们用上面同样的方法，把 deployment 中 pod 数量改回 1 个

![](https://pica.zhimg.com/v2-af536a426805a39c3475a4797b4f1ff0_1440w.jpg)

  

过大概 10 几分钟，可以看到新增的 node 已经停止

![](https://pic2.zhimg.com/v2-5d1d8865104bb848f8d0386237d727e1_1440w.jpg)

## 5.9 常见问题

报错一：
```
caused by: InvalidIdentityToken: No OpenIDConnect provider found in your account for https://oidc.eks.us-east-1.amazonaws.com/id/274A18041DB4CF680FA22A5EF99FDFE3
```

解决：
使用 eksctl 为集群创建 IAM OIDC 身份提供商
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

确定集群是否拥有现有 IAM OIDC 提供商。
检索集群的 OIDC 提供商 ID 并将其存储在变量中。
oidc_id=$(aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
确定您的账户中是否已存在具有您的集群 ID 的 IAM OIDC 提供商。

---

报错二：

如果部署后发现，Pod 反复重启，最后处于“CrashLoopBackOff”状态

![](https://pic1.zhimg.com/v2-d0b875a95cc96bf8511958da1e805eb6_1440w.jpg)


查看 Pod 日志，运行以下命令把 pod 日志重定向到本地文件 autoscaler.log

```
kubectl logs pods/cluster-autoscaler-77cb5c59b7-mcfmj -n kube-system >autoscaler.log
```

```
0617 07:29:49.853336       1 aws_manager.go:262] Failed to regenerate ASG cache: WebIdentityErr: failed to retrieve credentials
caused by: AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
	status code: 403, request id: ff336b02-e997-47f2-8551-8e00efa05049
F0617 07:29:49.853387       1 aws_cloud_provider.go:430] Failed to create AWS Manager: WebIdentityErr: failed to retrieve credentials
caused by: AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity
	status code: 403, request id: ff336b02-e997-47f2-8551-8e00efa05049
```

解决：

在 AWS IAM 中控台选择上面创建的 role “ AmazonEKSClusterAutoscalerRole”，点击“Edit trust relationship”

![[image/v2-8d2de9682c94051dad3e68f1b591a5ff_1440w.jpg]]


记下以下两个 ID

![](https://pica.zhimg.com/v2-8ff568e9947ff24eaf9d3d320182e966_1440w.jpg)

  
确保这两个 ID 与 EKS OpenID Connect provider URL 中的 ID 一致，修改一致后，autoscaler 的 Pod 就会自动重建成功

![[image/v2-d46317e2e4290bd1c075db85d5e8a2b2_1440w.jpg]]


---


修改OICD供应商ID，更新角色信任策略
aws iam update-assume-role-policy --role-name AmazonEKSClusterAutoscalerRole --policy-document file://"trust-policy.json"

aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
如果返回了输出，则表示您的集群已经有 IAM OIDC 提供商，您可以跳过下一步。如果没有返回输出，则您必须为集群创建 IAM OIDC 提供商。


使用以下命令为您的集群创建 IAM OIDC 身份提供商。将 my-cluster 替换为您自己的值。
eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve






