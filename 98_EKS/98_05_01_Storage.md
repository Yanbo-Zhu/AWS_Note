
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


# 1 使用EBS

- EBS provides block level storage volumes for use with EC2 & Container instances. 
- We can mount these volumes as devices on our EC2 & Container instances.
- EBS volumes that are attached to an instance are exposed as storage volumes that persist independently from the life of the EC2 or Container instance.
- We can dynamically change the configuration of a volume attached to an instance.
- AWS recommends EBS for data that must be quickly accessible and requires long-term persistence.
- EBS is well suited to both database-style applications that rely on random reads and writes, and to throughput-intensive applications that perform long, continuous reads and writes.

![](../image/Pasted%20image%2020240711182342.png)



![](../image/Pasted%20image%2020240711182449.png)


## 1.1 如何在 EKS 集群中使用 EBS CSI 驱动



### 1.1.1 使用 eksctl 

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
```

eksctl create iamserviceaccount 命令在 EKS 中创建的是一个 IAM 角色绑定到 Kubernetes ServiceAccount 的对象，用于让 Pod 通过该 ServiceAccount 安全地访问 AWS 资源。
- **Kubernetes ServiceAccount**
    - 在指定的命名空间中创建一个 `ServiceAccount`（或使用已有的）。
    - 名称由你通过 `--name` 参数指定。
- **IAM Role（带信任策略）**
    - 创建一个 AWS IAM 角色，允许被 EKS 节点上的 Pod 通过 IRSA（IAM Roles for Service Accounts）机制扮演。
    - 信任策略中会自动配置 `sts:AssumeRoleWithWebIdentity`，用于 EKS 的 OIDC 提供者。
- **IAM Policy 附加**
    - 它会将你通过 `--attach-policy-arn` 指定的权限策略附加到上述角色上，比如：
        `arn:aws:iam::aws:policy/AmazonEBSCSIDriverPolicy`
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



### 1.1.2 使用 Terraform 


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

# 2 使用 RDS


![](../image/Pasted%20image%2020240711201315.png)

高可用: 链接两个不同的RDS

![](../image/Pasted%20image%2020240711201345.png)










