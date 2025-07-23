

# 1 EBS EFS 比较


![](../image/Pasted%20image%2020240711182118.png)

|项目|**EBS（Elastic Block Store）**|**EFS（Elastic File System）**|
|---|---|---|
|📦 类型|块存储（Block Storage）|文件存储（File Storage）|
|🔗 挂载方式|只能挂载到 **一个 EC2 实例**（或一个 Pod）|可同时挂载到 **多个 EC2 实例 / Pod**|
|📍 使用范围|与单个 EC2 实例 / Pod 绑定|多实例/多Pod共享访问|
|🌐 网络协议|内部通过块设备连接（如 `/dev/xvda`）|NFS 协议（通常是 NFSv4）|
|🧠 使用场景|- 数据库存储（MySQL、PostgreSQL）  <br>- 高IO应用|- 多节点共享文件访问  <br>- 分布式应用  <br>- 用户目录存储|
|⚙️ 持久性|卷级别持久化|文件级别持久化|
|💲 计费方式|按 **卷大小（GiB）和 IOPS** 收费|按 **使用量（GiB）** 收费|
|☁️ 与 EKS 配合|通过 **EBS CSI Driver** 挂载为持久卷|通过 **EFS CSI Driver** 挂载为共享持久卷|
|🧩 动态扩展|需要手动调整卷大小|自动扩展，无需管理容量|


- 使用 **EBS**：
    - 如果你只需要一个 Pod/EC2 访问数据（例如数据库）
    - 需要高性能的磁盘读写（如 SSD）
- 使用 **EFS**：
    - 如果多个 Pod 需要共享访问相同数据（如 WordPress 共享上传目录）
    - 如果你需要简单的文件存储，容量自动增长


![](../image/Pasted%20image%2020240711182328.png)


# 2 使用EBS

- EBS provides block level storage volumes for use with EC2 & Container instances. 
- We can mount these volumes as devices on our EC2 & Container instances.
- EBS volumes that are attached to an instance are exposed as storage volumes that persist independently from the life of the EC2 or Container instance.
- We can dynamically change the configuration of a volume attached to an instance.
- AWS recommends EBS for data that must be quickly accessible and requires long-term persistence.
- EBS is well suited to both database-style applications that rely on random reads and writes, and to throughput-intensive applications that perform long, continuous reads and writes.

![](../image/Pasted%20image%2020240711182342.png)



![](../image/Pasted%20image%2020240711182449.png)


## 2.1 在 EKS 集群中使用 EBS CSI 驱动



### 2.1.1 使用 eksctl 

1 安装 EBS CSI Driver（通过 AWS 提供的 Helm chart 或 `eksctl`）

如果还没有 IAM 角色，你可以用 `eksctl` 创建：
```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster <your-cluster-name> \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --region <your-region>

eksctl create iamserviceaccount \
  --name my-serviceaccount \
  --namespace default \
  --cluster your-cluster-name \
  --attach-role-arn arn:aws:iam::<account-id>:role/existing-role \
  --approve
```

是的，eksctl create iamserviceaccount 会自动创建 IAM Role 和 Kubernetes ServiceAccount，并将它们绑定在一起，除非你指定使用已有的 IAM Role。
eksctl create iamserviceaccount 命令在 EKS 中创建的是一个 IAM 角色绑定到 Kubernetes ServiceAccount 的对象，用于让 Pod 通过该 ServiceAccount 安全地访问 AWS 资源。

- ✅ 创建一个新的 **IAM Role**，并附加你指定的 IAM Policy。
- ✅ 创建一个 **Kubernetes 的 ServiceAccount**，放在你指定的 namespace 里。
- ✅ 在这个 ServiceAccount 上打上注解：`eks.amazonaws.com/role-arn=...`。
- ✅ 把这个 IAM Role 配置为可以通过 **OIDC provider（IRSA）** 被该 ServiceAccount 使用。


- **Kubernetes ServiceAccount**
    - 在指定的命名空间中创建一个 `ServiceAccount`（或使用已有的）。
    - 名称由你通过 `--name` 参数指定。
- **IAM Role（带信任策略）**
    - 创建一个 AWS IAM 角色，允许被 EKS 节点上的 Pod 通过 IRSA（IAM Roles for Service Accounts）机制扮演。
    - 信任策略中会自动配置 `sts:AssumeRoleWithWebIdentity`，用于 EKS 的 OIDC 提供者。
- **IAM Policy 附加**
    - 这个 policy 必须已经创建了 ， 才可以 attach 
    - 它会将你通过 `--attach-policy-arn` 指定的权限策略附加到上述角色上，比如：`arn:aws:iam::aws:policy/AmazonEBSCSIDriverPolicy`
- attach 一个已经有的 iam role 
    - 你可以改用 `--attach-role-arn` 参数，这样 `eksctl` 就不会创建 IAM Role，只会创建 Kubernetes ServiceAccount 并绑定已有角色：
- **IAM Role 和 Kubernetes ServiceAccount 的关联**
    - `eksctl` 会通过 `eks.amazonaws.com/role-arn` 注解将 IAM 角色与 `ServiceAccount` 关联起来，使得该账号下的 Pod 能以该 IAM 身份与 AWS 服务交互。


使用 eksctl 安装（推荐）
```
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster <your-cluster-name> \
  --region <your-region> \
  --service-account-role-arn <iam-role-arn>
```


2 创建 StorageClass

```
# ebs-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
```


3 创建 PVC 和 Pod 来测试挂载
```
# ebs-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 4Gi
```

```
# ebs-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-using-ebs
spec:
  containers:
    - name: app
      image: busybox
      command: [ "sleep", "3600" ]
      volumeMounts:
        - mountPath: "/data"
          name: ebs-volume
  volumes:
    - name: ebs-volume
      persistentVolumeClaim:
        claimName: ebs-claim
```



# 3 EFS 

## 3.1 在 EKS 集群中使用 EBS CSI 驱动
### 3.1.1 使用 Terraform 


1 IAM Role for Service Account (IRSA)
需要先给 driver 创建 IAM 权限。
```
resource "aws_iam_policy" "efs_csi_driver" {
  name        = "AmazonEKS_EFS_CSI_Driver_Policy"
  description = "EFS CSI driver policy for EKS"
  policy      = file("efs-csi-driver-policy.json") # 可以从 AWS 官方下载 JSON
}

resource "aws_iam_role" "efs_csi_driver" {
  name = "eks-efs-csi-driver-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "efs_csi_driver_attach" {
  role       = aws_iam_role.efs_csi_driver.name
  policy_arn = aws_iam_policy.efs_csi_driver.arn
}

```


2 （2）EFS CSI Driver Helm Chart 安装

```
resource "helm_release" "efs_csi_driver" {
  name       = "aws-efs-csi-driver"
  repository = "https://kubernetes-sigs.github.io/aws-efs-csi-driver/"
  chart      = "aws-efs-csi-driver"
  namespace  = "kube-system"

  set {
    name  = "controller.serviceAccount.create"
    value = "false"
  }

  set {
    name  = "controller.serviceAccount.name"
    value = "efs-csi-controller-sa"  # 这里要和你上面创建的 ServiceAccount 对应
  }

  set {
    name  = "region"
    value = var.region
  }
}
```


或者用 直接用  resource "aws_eks_addon" 

```
resource "aws_eks_addon" "aws_efs_csi_driver" {
  cluster_name                = aws_eks_cluster.cluster.name
  addon_name                  = "aws-efs-csi-driver"
  addon_version               = var.addon_versions["aws-efs-csi-driver"]
  resolve_conflicts_on_create = "OVERWRITE"
  resolve_conflicts_on_update = "PRESERVE"
  service_account_role_arn    = aws_iam_role.service_accounts["aws-efs-csi-driver"].arn

  depends_on = [
    aws_eks_node_group.all,
    aws_efs_mount_target.efs_csi
  ]
}
```



3 创建 StorageClass
```
resource "kubernetes_storage_class" "efs" {
  metadata {
    name = "efs-sc"
  }

  provisioner = "efs.csi.aws.com"

  reclaim_policy = "Retain"
  volume_binding_mode = "Immediate"
}

```



4 然后，你就可以在 Kubernetes 中像下面这样申请 PVC 了：
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```


### 3.1.2 使用 terraform 例子 in IVU


#### 3.1.2.1 EKS Infrastructure: 创立 一个 efs  File systems

在 terraform-aws-eks\modules\cluster\main.tf 中

```hcl

#### __________________________________________________________________ EFS configuration _____ ####

resource "aws_efs_file_system" "efs_csi" {
  encrypted        = true
  performance_mode = "generalPurpose"
  throughput_mode  = "elastic"

  tags = merge(local.common_tags, { Name = "${local.resource_prefix}" })
}

resource "aws_security_group" "efs" {
  name                   = "${local.resource_prefix}-efs"
  description            = "Security group for EFS to allow access from EKS cluster ${var.cluster_name}."
  revoke_rules_on_delete = true
  tags                   = merge(local.common_tags, { Name = "${local.resource_prefix}-efs" })
  vpc_id                 = local.vpc_id
}

resource "aws_vpc_security_group_ingress_rule" "efs" {
  for_each = local.node_subnet_cidr_blocks

  security_group_id = aws_security_group.efs.id
  description       = "Allow NFS access to EFS filesystem from EKS cluster ${var.cluster_name} subnet ${each.key}."
  cidr_ipv4         = each.value
  ip_protocol       = "tcp"
  from_port         = 2049
  to_port           = 2049
  tags              = merge(local.common_tags, { Name = "${local.resource_prefix}-nfs-from-${each.key}" })
}


## In deinem Terraform-Code mit aws_efs_mount_target wird nur dafür gesorgt, dass das EFS-Dateisystem in den angegebenen Subnetzen netzwerkseitig verfügbar ist. Wo genau das Dateisystem im Dateisystembaum (z. B. unter /mnt/efs) eingebunden wird, muss man  später manuell festlegen – auf dem EC2-Server

resource "aws_efs_mount_target" "efs_csi" {
  for_each = toset(local.node_private_subnets)

  file_system_id  = aws_efs_file_system.efs_csi.id
  subnet_id       = data.aws_subnet.node_private[each.value].id
  security_groups = [aws_security_group.efs.id]
}

```


#### 3.1.2.2 EKS Infrastructure: 创建 iam policy, iam role, service_account


在 terraform-aws-eks\modules\cluster\main.tf 中

aws-efs-csi-driver.json
```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Action": [
              "iam:CreateServiceLinkedRole"
          ],
          "Resource": "*",
          "Condition": {
              "StringEquals": {
                  "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:DescribeAccountAttributes",
              "ec2:DescribeAddresses",
              "ec2:DescribeAvailabilityZones",
              "ec2:DescribeInternetGateways",
              "ec2:DescribeVpcs",
              "ec2:DescribeVpcPeeringConnections",
              "ec2:DescribeSubnets",
              "ec2:DescribeSecurityGroups",
              "ec2:DescribeInstances",
              "ec2:DescribeNetworkInterfaces",
              "ec2:DescribeTags",
              "ec2:GetCoipPoolUsage",
              "ec2:DescribeCoipPools",
              "elasticloadbalancing:DescribeLoadBalancers",
              "elasticloadbalancing:DescribeLoadBalancerAttributes",
              "elasticloadbalancing:DescribeListeners",
              "elasticloadbalancing:DescribeListenerCertificates",
              "elasticloadbalancing:DescribeSSLPolicies",
              "elasticloadbalancing:DescribeRules",
              "elasticloadbalancing:DescribeTargetGroups",
              "elasticloadbalancing:DescribeTargetGroupAttributes",
              "elasticloadbalancing:DescribeTargetHealth",
              "elasticloadbalancing:DescribeTags",
              "elasticloadbalancing:DescribeTrustStores",
              "elasticloadbalancing:DescribeListenerAttributes"
          ],
          "Resource": "*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "cognito-idp:DescribeUserPoolClient",
              "acm:ListCertificates",
              "acm:DescribeCertificate",
              "iam:ListServerCertificates",
              "iam:GetServerCertificate",
              "waf-regional:GetWebACL",
              "waf-regional:GetWebACLForResource",
              "waf-regional:AssociateWebACL",
              "waf-regional:DisassociateWebACL",
              "wafv2:GetWebACL",
              "wafv2:GetWebACLForResource",
              "wafv2:AssociateWebACL",
              "wafv2:DisassociateWebACL",
              "shield:GetSubscriptionState",
              "shield:DescribeProtection",
              "shield:CreateProtection",
              "shield:DeleteProtection"
          ],
          "Resource": "*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:AuthorizeSecurityGroupIngress",
              "ec2:RevokeSecurityGroupIngress"
          ],
          "Resource": "*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:CreateSecurityGroup"
          ],
          "Resource": "*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:CreateTags"
          ],
          "Resource": "arn:aws:ec2:*:*:security-group/*",
          "Condition": {
              "StringEquals": {
                  "ec2:CreateAction": "CreateSecurityGroup"
              },
              "Null": {
                  "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:CreateTags",
              "ec2:DeleteTags"
          ],
          "Resource": "arn:aws:ec2:*:*:security-group/*",
          "Condition": {
              "Null": {
                  "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                  "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "ec2:AuthorizeSecurityGroupIngress",
              "ec2:RevokeSecurityGroupIngress",
              "ec2:DeleteSecurityGroup"
          ],
          "Resource": "*",
          "Condition": {
              "Null": {
                  "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:CreateLoadBalancer",
              "elasticloadbalancing:CreateTargetGroup"
          ],
          "Resource": "*",
          "Condition": {
              "Null": {
                  "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:CreateListener",
              "elasticloadbalancing:DeleteListener",
              "elasticloadbalancing:CreateRule",
              "elasticloadbalancing:DeleteRule"
          ],
          "Resource": "*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:AddTags",
              "elasticloadbalancing:RemoveTags"
          ],
          "Resource": [
              "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
              "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
              "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
          ],
          "Condition": {
              "Null": {
                  "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                  "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:AddTags",
              "elasticloadbalancing:RemoveTags"
          ],
          "Resource": [
              "arn:aws:elasticloadbalancing:*:*:listener/net/*/*/*",
              "arn:aws:elasticloadbalancing:*:*:listener/app/*/*/*",
              "arn:aws:elasticloadbalancing:*:*:listener-rule/net/*/*/*",
              "arn:aws:elasticloadbalancing:*:*:listener-rule/app/*/*/*"
          ]
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:ModifyLoadBalancerAttributes",
              "elasticloadbalancing:SetIpAddressType",
              "elasticloadbalancing:SetSecurityGroups",
              "elasticloadbalancing:SetSubnets",
              "elasticloadbalancing:DeleteLoadBalancer",
              "elasticloadbalancing:ModifyTargetGroup",
              "elasticloadbalancing:ModifyTargetGroupAttributes",
              "elasticloadbalancing:DeleteTargetGroup",
              "elasticloadbalancing:ModifyListenerAttributes"
          ],
          "Resource": "*",
          "Condition": {
              "Null": {
                  "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:AddTags"
          ],
          "Resource": [
              "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
              "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
              "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
          ],
          "Condition": {
              "StringEquals": {
                  "elasticloadbalancing:CreateAction": [
                      "CreateTargetGroup",
                      "CreateLoadBalancer"
                  ]
              },
              "Null": {
                  "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
              }
          }
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:RegisterTargets",
              "elasticloadbalancing:DeregisterTargets"
          ],
          "Resource": "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*"
      },
      {
          "Effect": "Allow",
          "Action": [
              "elasticloadbalancing:SetWebAcl",
              "elasticloadbalancing:ModifyListener",
              "elasticloadbalancing:AddListenerCertificates",
              "elasticloadbalancing:RemoveListenerCertificates",
              "elasticloadbalancing:ModifyRule"
          ],
          "Resource": "*"
      }
  ]
}

```

iam policy
```
resource "aws_iam_policy" "aws_efs_csi_driver" {
  name        = "${local.resource_prefix}-aws-efs-csi-driver"
  description = "Policy for AWS EFS CSI drivers of EKS cluster ${var.cluster_name}."
  tags        = merge(local.common_tags, { Name = "${var.cluster_name}-aws-efs-csi-driver" })
  policy      = file("${path.module}/files/iam_policies/aws-efs-csi-driver.json")
}

```

----

创建一些信息 之后 用于 创建 iam role 和 role_policy_binding 

```sh
locals {
  service_account_roles = merge(
    {
      aws-load-balancer-controller = {
        service_account = "kube-system:aws-load-balancer-controller"
        policy_arns     = [aws_iam_policy.aws_load_balancer_controller.arn]
      }
      aws-ebs-csi-driver = {
        service_account = "kube-system:ebs-csi-controller-sa"
        policy_arns     = ["arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy"]
      }
      aws-efs-csi-driver = {
        service_account = "kube-system:efs-csi-controller-sa"
        policy_arns     = [aws_iam_policy.aws_efs_csi_driver.arn]
      }
      vpc-cni = {
        service_account = "kube-system:aws-node"
        policy_arns     = ["arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"]
      }
    },
    var.configure_loki_buckets ? {
      loki-s3 = {
        service_account = "loki:loki-sa"
        policy_arns     = [aws_iam_policy.loki_s3.arn]
      }
    } : {},
    length(local.external_dns_policies) > 0 ? {
      external-dns = {
        service_account = "external-dns:external-dns-sa"
        policy_arns     = local.external_dns_policies
      }
    } : {},
    local.configure_mountpoint_s3_csi_driver ? {
      aws-mountpoint-s3-csi-driver = {
        service_account = "kube-system:s3-csi-driver-sa"
        policy_arns     = local.mountpoint_s3_policies
      }
    } : {}
  )

  service_account_role_policy_attachments_list = flatten([
    for service_account_iam_role_name, service_account_role in local.service_account_roles :
    [
      for index in range(length(service_account_role.policy_arns)) :
      {
        role       = "${local.resource_prefix}-${service_account_iam_role_name}"
        policy_arn = service_account_role.policy_arns[index]
        index      = index
      }
    ]
  ])

  service_account_role_policy_attachments = {
    for attachment in local.service_account_role_policy_attachments_list :
    "${attachment.role}-${attachment.index}" => {
      role       = attachment.role
      policy_arn = attachment.policy_arn
    }
  }
}
```



----

创建 iam role 
这些 role 之后 绑定 kubernetes 中的 service account 
kubernetes cluster 的 pod 如果有这个 service account , 他就可以 形式 iam role 拥有的权力 

"If a Pod in a Kubernetes cluster is associated with this ServiceAccount, it can assume the IAM role and inherit its permissions."

```
resource "aws_iam_role" "service_accounts" {
  for_each = local.service_account_roles

  name               = "${local.resource_prefix}-${each.key}"
  description        = "IAM role for service account ${each.key} of EKS cluster ${var.cluster_name}"
  tags               = merge(local.common_tags, { Name = "${local.resource_prefix}-${each.key}" })
  assume_role_policy = <<-EOF
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Principal": {
                  "Federated": "${aws_iam_openid_connect_provider.cluster.arn}"
              },
              "Action": "sts:AssumeRoleWithWebIdentity",
              "Condition": {
                  "StringEquals": {
                      "${aws_iam_openid_connect_provider.cluster.url}:aud": "sts.amazonaws.com",
                      "${aws_iam_openid_connect_provider.cluster.url}:sub": "system:serviceaccount:${each.value.service_account}"
                  }
              }
          }
      ]
  }
  EOF
}

```



---

绑定 iam_role 和 iam policy

```
resource "aws_iam_role_policy_attachment" "service_accounts" {
  for_each = local.service_account_role_policy_attachments

  role       = each.value.role
  policy_arn = each.value.policy_arn

  depends_on = [
    aws_iam_role.service_accounts,
    aws_iam_policy.aws_load_balancer_controller,
    aws_iam_policy.aws_efs_csi_driver,
    aws_iam_policy.mountpoint_s3
  ]
}
```



#### 3.1.2.3 EKS Infrastructure:  使用 aws eks addon: efs csi driver  和 service account 的创造 

在 terraform-aws-eks\modules\cluster\main.tf 中

==`aws_eks_addon` 资源 **会自动创建 Kubernetes ServiceAccount**。 名字为 kube-system:ebs-csi-controller-sa==   . 就不用特意去 创造一个 service account 了 


```
resource "aws_eks_addon" "aws_efs_csi_driver" {
  cluster_name                = aws_eks_cluster.cluster.name
  addon_name                  = "aws-efs-csi-driver"
  addon_version               = var.addon_versions["aws-efs-csi-driver"]
  resolve_conflicts_on_create = "OVERWRITE"
  resolve_conflicts_on_update = "PRESERVE"
  service_account_role_arn    = aws_iam_role.service_accounts["aws-efs-csi-driver"].arn

  depends_on = [
    aws_eks_node_group.all,
    aws_efs_mount_target.efs_csi
  ]
}
```


service_account_role_arn 的作用是将一个 AWS IAM Role 与 Kubernetes 中的某个 ServiceAccount 关联起来，从而让运行在 EKS 上的 Pod 能够 以这个 IAM Role 的权限运行。这是通过 IRSA（IAM Roles for Service Accounts） 实现的。


- service_account_role_arn:
    - `aws_iam_role.service_accounts["aws-efs-csi-driver"]` 的 ARN 赋给该 EKS 插件（Addon）使用的 ServiceAccount。
    - 它只是在安装 EKS 插件（比如 `aws-efs-csi-driver`）时，告诉插件使用你指定的 IAM Role（通过 `service_account_role_arn`）绑定的 ServiceAccount。
- 也就是说，这个 EFS CSI Driver 插件在集群中部署时，会通过 IRSA 使用这个 IAM Role。
- 该 IAM Role 应该已经配置了一个 **trust policy**，信任 Kubernetes 中这个 ServiceAccount，可以“assume role”。


Wenn du das Addon deployst, kannst du im Cluster prüfen:
```sh
efs-csi-controller-sa

 ⚡ 🦄  kubectl get sa efs-csi-controller-sa -n kube-system -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::681290371536:role/eks-dev-aws-efs-csi-driver
  creationTimestamp: "2024-06-17T10:12:34Z"
  labels:
    app.kubernetes.io/name: aws-efs-csi-driver
  name: efs-csi-controller-sa
  namespace: kube-system
  resourceVersion: "1801"
  uid: 4e390781-66db-4132-8f09-a68e87a416b8
```

---

如何创造一个 service account (其实不用做)

```
resource "kubernetes_service_account" "efs_csi_controller" {
  metadata {
    name      = "efs-csi-controller-sa"
    namespace = "kube-system"

    annotations = {
      "eks.amazonaws.com/role-arn" = aws_iam_role.service_accounts["aws-efs-csi-driver"].arn
    }
  }
}
```


#### 3.1.2.4 Kubenetes Cluster: 创造 StorageClass

```
  efs_storage_class_manifest = <<-EOF
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: efs-sc
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
    provisioner: efs.csi.aws.com
    reclaimPolicy: Delete
    parameters:
      provisioningMode: efs-ap
      fileSystemId: ${data.aws_efs_file_system.eks.id}
      directoryPerms: "700" # without specifying it, there is an error "failed to provision volume with StorageClass ... minimum field size of 3 ..."
      basePath: "/" # to avoid too long EFS path names
      subPathPattern: "$${.PVC.namespace}/$${.PVC.name}"
      ensureUniqueDirectory: "false" # to avoid too long EFS path names
      # gidRangeStart: "1000" # optional
      # gidRangeEnd: "2000" # optional
      # reuseAccessPoint: "false" # optional
  EOF
```


部署这个 stroage class into cluster 
```
resource "kubernetes_manifest" "efs_storage_class" {
  manifest = yamldecode(local.efs_storage_class_manifest)
}
```


## 3.2 #### Kubenetes Cluster: 给  efs-csi-controller 这个Deployment 加补丁  

  Patch the Deployment efs-csi-controller which serves for efs-csi-driver in eks cluster. 
  The original Deploymnet resource was already created along with provisioning the resource "aws_eks_addon" with name "aws_efs_csi_driver" in modules\cluster\main.tf. 
  This manifest is used to patch the Deployment resource in order to add the --delete-access-point-root-dir=true argument to the efs-csi-controller Deployment. This is needed to delete the contents of the EFS Access Point when deleting a Persistent Volume (PV) with reclaimPolicy: Delete.
  
  See also:
  https://github.com/kubernetes-sigs/aws-efs-csi-driver/issues/411#issuecomment-819205846
  https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/deploy/kubernetes/base/controller-deployment.yaml#L40-L41

```
  efs_csi_controller_deployment_manifest = <<-EOF
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: efs-csi-controller
      namespace: kube-system
      annotations:
        kubectl.kubernetes.io/restartedAt: "2025-04-26T12:34:56Z"  #  is used to force a redeployment when the timestamp is updated. Terraform dynamically replaces the timestamp during the terraform apply.
    spec:
      template:
        spec:
          containers:
            - name: efs-plugin
              args:
                - --delete-access-point-root-dir=true  # reclaimPolicy: Delete works as expected. Provisioned volume and the underlying access points are deleted. However, EFS does not delete Access Point contents when deleting an Access Point. We provided --delete-access-point-root-dir to the driver and if enabled, driver will delete access point contents while deleting PV .
                - --logtostderr
  EOF
```


```# resource kubernetes_manifest can also work with patching functionality. 
resource "kubernetes_manifest" "efs_csi_controller_deployment" {
  manifest = yamldecode(local.efs_csi_controller_deployment_manifest)
}
```



# 4 使用 RDS


![](../image/Pasted%20image%2020240711201315.png)

高可用: 链接两个不同的RDS

![](../image/Pasted%20image%2020240711201345.png)










