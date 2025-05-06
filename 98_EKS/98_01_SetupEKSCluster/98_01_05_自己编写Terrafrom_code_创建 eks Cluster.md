
https://medium.com/@tech_18484/step-by-step-guide-creating-an-eks-cluster-with-terraform-resources-iam-roles-for-service-df1c5e389811


# 1 VPC

https://community.aws/content/2j5ORbaCewzzUDNRlSAjzs3VgHu/building-an-aws-vpc-from-scratch-using-terraform

The entire network architecture of any cloud-based service is based on a rtual private cloud (VPC). AWS VPCs offer the required network segregation and enable security by efficiently managing aspects like subnets, routing, internet gateway, NAT gateway, DHCP, etc.

There are several considerations to be made while building a VPC for any project. Let’s start to build our VPC from the ground up using Terraform.

1. VPC in us-west-2 zone
    
2. Internet Gateway
    
3. 2 Public Subnets, one in each AZ
    
4. 2 Private Subnets, one in each AZ
    
5. Route Table configurations (main and 2nd)
    
6. NAT gateways
    

A VPC spans all the Availability Zones (AZ) in a region. It is always associated with a CIDR range (both IPv4 and IPv6) which defines the number of internal network addresses that may be used internally.

Within the VPC, we create subnets that are specific to AZs. It is possible to have multiple subnets in the same AZ. The purpose of subnets is to internally segregate resources contained in the VPC in every AZ. AWS Regions consist of multiple Availability Zones for DR purposes.

Our architecture contains two types of subnets – public and private. Public subnets enable internet access for the components hosted within them, while private subnets allow internet using NAT gateway.

An internet gateway is deployed and associated with the VPC to enable internet traffic within the VPC’s public subnets. Only one internet gateway can be associated with each VPC.



## 1.1 Create a Provider

```
provider "aws" {
  region = "us-west-2"
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

## 1.2 Create a VPC

To begin with, let us start by defining our VPC resource in Terraform. To specify a range of IP addresses in a VPC, a CIDR block needs to be provided. We have also provided a Name tag for identification.

```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "myvpc"
  }
}
```

## 1.3 Create Subnets


The VPC exists across all the Availability Zones in a region. While subnets are associated with a single AZ. The Oregon (us-west-2) region has two AZs, and we need one public and one private subnet in each AZ as per the diagram.

Firstly, we identify the CIDR ranges to be associated with the four new subnets we need to create. In our example, based on the CIDR range of the VPC I have identified the CIDR ranges and defined a couple of variables in our Terraform code (subnet.tf).

**Name :**

1. private-us-west-2a : 10.0.1.0/24
    
2. private-us-west-2b : 10.0.2.0/24
    
3. public-us-west-2a : 10.0.3.0/24
    
4. public-us-west-2b : 10.0.4.0/24


```
resource "aws_subnet" "private-us-west-2a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a"

  tags = {
    "Name"                            = "private-us-west-2a"
    "kubernetes.io/role/internal-elb" = "1"
    "kubernetes.io/cluster/demo"      = "owned"
  }
}

resource "aws_subnet" "private-us-west-2b" {
  vpc_id            = aws_vpc.main.id
          cidr_block        = "10.0.2.0/24"
  availability_zone = "us-west-2b"

  tags = {
    "Name"                            = "private-us-west-2b"
    "kubernetes.io/role/internal-elb" = "1"
    "kubernetes.io/cluster/demo"      = "owned"
  }
}

resource "aws_subnet" "public-us-west-2a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.3.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    "Name"                       = "public-us-west-2a"
    "kubernetes.io/role/elb"     = "1"
    "kubernetes.io/cluster/demo" = "owned"
  }
}

resource "aws_subnet" "public-us-west-2b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  availability_zone       = "us-west-2b"
  map_public_ip_on_launch = true

  tags = {
    "Name"                       = "public-us-west-2b"
    "kubernetes.io/role/elb"     = "1"
    "kubernetes.io/cluster/demo" = "owned"
  }
}
```


---

解释 tags 

so for **private subnets**, the **Name** is just a simple tag that displays when a subnet is created, and following the **“kubernetes.io/role/internal-elb”** tag is used by Kubernetes to discover subnets where a private load balancer will be created. also, you need to tag your subnet with the cluster equal to the EKS cluster name **“kubernetes.io/cluster/demo”** In this case it's a demo and value can be owned if you only use it for Kubernetes.

> And also make sure that **map_public_ip_on_launch** is equal to **true** is only needed if you want to create public Kubernetes instance groups. so each Kubernetes worker will get public ip address which is usually not needed because most of the time you would use private subnet for instance groups and create public load balancer in public subnets. so for load balancer this attribute is not required.

You are only required to give a tag on the public subnet is **“kubernetes.io/role/elb”** equal to **one** this instructs Kubernetes to create a public load balancer in these subnets.


## 1.4 Create Internet Gateway

Since we have to build public subnets, we need to provide access to the internet in the given VPC. For this, the first thing that we need is an internet gateway.

```
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "igw"
  }
}
```


## 1.5 Create a NAT Gateway

You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

Now it's time to create a **NAT gateway,** it is used in a private subnet to allow services to connect to the internet and an important note is that we must make it inside the public subnets because it is required to send packets to the internet by the I**nternet gateway**.

For **NAT** we need to allocate the **elastic ip** address first. Then we can use it in the **aws_nat_gateway** resource.
```
resource "aws_eip" "nat" {
  vpc = true

  tags = {
    Name = "nat"
  }
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public-us-west-2a.id

  tags = {
    Name = "nat"
  }

  depends_on = [aws_internet_gateway.igw]
}
```


## 1.6 Create a Route tables

We already know that when a VPC is created, a main route table is created as well. The main route table is responsible for enabling the flow of traffic within the VPC.P

Here we have created two route tables one for Public and other Private. Public route table associated 2 public subnet and internet gateway. and Private route table associate 2 Private subnet and NAT Gateway.

**Route :**

1. **Public**
    
    1. **subnet associations**
        
        - public-us-west-2a : 10.0.3.0/24
            
        - public-us-west-2b : 10.0.4.0/24
            
    2. **Routes**
        
        - igw-XXXX : 0.0.0.0/0
            
2. **Private**
    
    1. **subnet associations**
        
        - private-us-west-2a : 10.0.1.0/24
            
        - private-us-west-2b : 10.0.2.0/24
            
    2. **Routes**
        
        - NAT-XXXXX : 0.0.0.0/0

```hcl
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      nat_gateway_id             = aws_nat_gateway.nat.id
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      gateway_id                 = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
    },
  ]

  tags = {
    Name = "private"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      gateway_id                 = aws_internet_gateway.igw.id
      nat_gateway_id             = ""
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
    },
  ]

  tags = {
    Name = "public"
  }
}

resource "aws_route_table_association" "private-us-west-2a" {
  subnet_id      = aws_subnet.private-us-west-2a.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private-us-west-2b" {
  subnet_id      = aws_subnet.private-us-west-2b.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "public-us-west-2a" {
  subnet_id      = aws_subnet.public-us-west-2a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public-us-west-2b" {
  subnet_id      = aws_subnet.public-us-west-2b.id
  route_table_id = aws_route_table.public.id
}
```



# 2 Launch and Manage AWS EKS Clusters

https://community.aws/content/2lm23m4MJ72qG0hi8ZrRywqF3me/effortlessly-launch-and-manage-aws-eks-clusters-with-terraform

## 2.1 IAM role with the AmazonEKSClusterPolicy


```
resource "aws_iam_role" "demo" {
  name = "eks-cluster-demo"
  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy_attachment" "demo-AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.demo.name
}

resource "aws_eks_cluster" "demo" {
  name     = "ashish"
  #cluster_version = "1.30"
  role_arn = aws_iam_role.demo.arn

  vpc_config {
    subnet_ids = [
      aws_subnet.private-us-west-2a.id,
      aws_subnet.private-us-west-2b.id,
      aws_subnet.public-us-west-2a.id,
      aws_subnet.public-us-west-2b.id
    ]
  }

  depends_on = [aws_iam_role_policy_attachment.demo-AmazonEKSClusterPolicy]
}
```

- Next, we are going to create a single instance group for Kubernetes. Similar to the EKS cluster, it requires an IAM role as well.

## 2.2 node group 

Next, we are going to create a single instance group for Kubernetes. Similar to the EKS cluster, it requires an IAM role as well.

```
resource "aws_iam_role" "nodes" {
  name = "eks-node-group-nodes"

  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
    Version = "2012-10-17"
  })
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.nodes.name
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.nodes.name
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.nodes.name
}

resource "aws_eks_node_group" "private-nodes" {
  cluster_name    = aws_eks_cluster.demo.name
  node_group_name = "private-nodes"
  node_role_arn   = aws_iam_role.nodes.arn

  subnet_ids = [
    aws_subnet.private-us-west-2a.id,
    aws_subnet.private-us-west-2b.id
  ]

  capacity_type  = "ON_DEMAND"
  instance_types = ["t3.small"]

  scaling_config {
    desired_size = 1
    max_size     = 5
    min_size     = 0
  }

  update_config {
    max_unavailable = 1
  }

  labels = {
    role = "general"
  }

  # taint {
  #   key    = "team"
  #   value  = "devops"
  #   effect = "NO_SCHEDULE"
  # }

  # launch_template {
  #   name    = aws_launch_template.eks-with-disks.name
  #   version = aws_launch_template.eks-with-disks.latest_version
  # }

  depends_on = [
    aws_iam_role_policy_attachment.nodes-AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.nodes-AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.nodes-AmazonEC2ContainerRegistryReadOnly,
  ]
}

# resource "aws_launch_template" "eks-with-disks" {
#   name = "eks-with-disks"

#   key_name = "local-provisioner"

#   block_device_mappings {
#     device_name = "/dev/xvdb"

#     ebs {
#       volume_size = 50
#       volume_type = "gp2"
#     }
#   }
# }
```


Above first policy are **AmazonEKSWorkerNodePolicy** which is required for ec2 and EKS cluster access 
the second is **AmazonEKS_CNI_Policy** for Kubernetes networking configuration and 
the last one is **AmazonEC2ContainerRegistryReadOnly** which allows to download and run docker images from the ECR repository.

And in the node group resource, you have many options to configure Kubernetes worker and where we specify cluster name, node group, and role name along with the two private subnets. if you need nodes with public IP, replace private with public subnet id.

for capacity, you can use on-demand and spot instances( that are much cheaper but can be taken away by was at any time

> For scaling its important to note scaling configuration and by default eks cluster will not autoscale your nodes. you need to deploy additional component on k8s called cluster autoscaler.
> 
> But to define minimum and maximum number of nodes you can use the **min** and **max** attributes and eks will use the setting to create the autoscaling group on your behalf and then autoscaller will simply adjust the desired sized based on the load.

You can also create some labels and taints for your nodes. you can use the label to instruct the Kubernetes scheduler to use a particular node group by using node affinity or node selector.


## 2.3 manage permissions for the applications deployed in your Kubernetes cluster, 

To manage permissions for the applications deployed in your Kubernetes cluster, you have two options.

1. **Attach Policies to Kubernetes Nodes**: You can attach IAM policies directly to the Kubernetes nodes. In this case, all the pods running on those nodes will inherit the same IAM permissions, meaning every pod will have the same level of access to AWS resources.
    
2. **Use OpenID Connect (OIDC) Provider**: Alternatively, you can create an OpenID Connect (OIDC) provider for your EKS cluster. ==This method allows you to grant IAM permissions based on the service account associated with each pod==. Using service accounts gives you more granular control over the permissions assigned to individual pods, rather than granting permissions at the node level.
    1. Amazon EKS supports using OpenID Connect (OIDC) identity providers as a method to authenticate users to your cluster. OIDC identity providers can be used with, or as an alternative to AWS Identity and Access Management (IAM). https://docs.aws.amazon.com/eks/latest/userguide/authenticate-oidc-identity-provider.html



The configuration file for this setup will be named `terraform/iam-oidc.tf`.


```
data "tls_certificate" "eks" {
  url = aws_eks_cluster.demo.identity[0].oidc[0].issuer
}

resource "aws_iam_openid_connect_provider" "eks" {
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = [data.tls_certificate.eks.certificates[0].sha1_fingerprint]
  url             = aws_eks_cluster.demo.identity[0].oidc[0].issuer
}
```


### 2.3.1 IAM, Kubernetes, and OpenID Connect (OIDC) background information

https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html
https://docs.aws.amazon.com/eks/latest/userguide/authenticate-oidc-identity-provider.html

In 2014, AWS Identity and Access Management added support for federated identities using OpenID Connect (OIDC). This feature allows you to authenticate AWS API calls with supported identity providers and receive a valid OIDC JSON web token (JWT). You can pass this token to the AWS STS `AssumeRoleWithWebIdentity` API operation and receive IAM temporary role credentials. You can use these credentials to interact with any AWS service, including Amazon S3 and DynamoDB.

Each JWT token is signed by a signing key pair. The keys are served on the OIDC provider managed by Amazon EKS and the private key rotates every 7 days. Amazon EKS keeps the public keys until they expire. If you connect external OIDC clients, be aware that you need to refresh the signing keys before the public key expires. Learn how to [Fetch signing keys to validate OIDC tokens](https://docs.aws.amazon.com/eks/latest/userguide/irsa-fetch-keys.html).

Kubernetes has long used service accounts as its own internal identity system. Pods can authenticate with the Kubernetes API server using an auto-mounted token (which was a non-OIDC JWT) that only the Kubernetes API server could validate. These legacy service account tokens don’t expire, and rotating the signing key is a difficult process. In Kubernetes version `1.12`, support was added for a new `ProjectedServiceAccountToken` feature. This feature is an OIDC JSON web token that also contains the service account identity and supports a configurable audience.

Amazon EKS hosts a public OIDC discovery endpoint for each cluster that contains the signing keys for the `ProjectedServiceAccountToken` JSON web tokens so external systems, such as IAM, can validate and accept the OIDC tokens that are issued by Kubernetes.


在 2014 年，AWS Identity and Access Management（IAM）增加了对使用 OpenID Connect（OIDC）进行联合身份验证的支持。  
这个功能允许你通过受支持的身份提供商（IdP）进行身份验证，获取一个有效的 OIDC JSON Web Token（JWT）。  
你可以将这个 token 传递给 AWS 的 STS（Security Token Service）中的 `AssumeRoleWithWebIdentity` API 操作，从而获得 IAM 的临时角色凭证。  
使用这些凭证，你就可以访问包括 Amazon S3 和 DynamoDB 在内的所有 AWS 服务。

每个 JWT token 都是由一对签名密钥对签名的。  
这些密钥由 Amazon EKS 管理的 OIDC 提供方（OIDC provider）提供，其中私钥每 7 天轮换一次。  
Amazon EKS 会保留公钥直到它们过期。  
如果你连接了外部 OIDC 客户端，需要注意在公钥过期之前刷新签名密钥。  
你可以学习如何获取签名密钥来验证 OIDC token。

Kubernetes 早就使用 ServiceAccount（服务账户）作为其内部身份验证系统。  
Pod 可以通过一个自动挂载的 token（最初是非 OIDC 格式的 JWT）向 Kubernetes API Server 进行身份验证，这种旧版的 ServiceAccount token 只有 Kubernetes API Server 本身能验证。  
这些旧版 token 不会过期，而且签名密钥的轮换过程非常困难。  
从 Kubernetes 1.12 版本开始，新增了 `ProjectedServiceAccountToken` 特性。  
这种新 token 是 OIDC 格式的 JWT，不仅包含了 ServiceAccount 的身份信息，还支持配置 Audience（受众）。

Amazon EKS 会为每个集群托管一个公共的 OIDC 发现端点，其中包含了 `ProjectedServiceAccountToken` JWT 的签名公钥。  
这样，外部系统（例如 IAM）就可以验证并接受 Kubernetes 签发的 OIDC token。




## 2.4 结果 


**EKS :**

![](https://community.aws/_next/image?url=https%3A%2F%2Fassets.community.aws%2Fa%2F2ox1DyHizxWmckpnoS03GRizUOW%2F12-p.webp%3FimgSize%3D1599x314&w=3840&q=75)

**IAM Role:**

1. **eks-cluster-demo**
    

![](https://community.aws/_next/image?url=https%3A%2F%2Fassets.community.aws%2Fa%2F2ox1ptAiFuXIosenHNEbz2EeTZm%2F3-pn.webp%3FimgSize%3D1504x775&w=3840&q=75)

1. **eks-node-group-nodes**
    

![](https://community.aws/_next/image?url=https%3A%2F%2Fassets.community.aws%2Fa%2F2ox1MwatlZXwuKc7og9vOp41MaU%2F1-pn.webp%3FimgSize%3D1466x716&w=3840&q=75)

- **Identity providers**
    

![](https://community.aws/_next/image?url=https%3A%2F%2Fassets.community.aws%2Fa%2F2ox1bCTMSURP8IfYs6wvjCuyk8u%2F2-pn.webp%3FimgSize%3D1901x913&w=3840&q=75 "Identity providers")

Identity providers


# 3 Test configuration

https://community.aws/content/2owpkcMiX4bBoidvHcXhVXiAAIJ/effortless-aws-eks-cluster-autoscaling-using-terraform

## 3.1 testing the provider first

```
data "aws_iam_policy_document" "test_oidc_assume_role_policy" {
  statement {
    actions = ["sts:AssumeRoleWithWebIdentity"]
    effect  = "Allow"

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.eks.url, "https://", "")}:sub"
      values   = ["system:serviceaccount:default:aws-test"]
    }

    principals {
      identifiers = [aws_iam_openid_connect_provider.eks.arn]
      type        = "Federated"
    }
  }
}

resource "aws_iam_role" "test_oidc" {
  assume_role_policy = data.aws_iam_policy_document.test_oidc_assume_role_policy.json
  name               = "test-oidc"
}

resource "aws_iam_policy" "test-policy" {
  name = "test-policy"

  policy = jsonencode({
    Statement = [{
      Action = [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation"
      ]
      Effect   = "Allow"
      Resource = "arn:aws:s3:::*"
    }]
    Version = "2012-10-17"
  })
}

resource "aws_iam_role_policy_attachment" "test_attach" {
  role       = aws_iam_role.test_oidc.name
  policy_arn = aws_iam_policy.test-policy.arn
}

output "test_policy_arn" {
  value = aws_iam_role.test_oidc.arn
}
```


## 3.2 create a pod to test IAM roles for service accounts


Check kubectl pods using command line:Next is to create a pod to test IAM roles for service accounts. First, we are going to omit annotations to bind the service account with the role. The way it works, you create a service account and use it in your pod spec. It can be anything, deployment, statefulset, or some jobs. Give it a name

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-test
  namespace: default
---
apiVersion: v1
kind: Pod
metadata:
  name: aws-cli
  namespace: default
spec:
  serviceAccountName: aws-test
  containers:
  - name: aws-cli
    image: amazon/aws-cli
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "while true; do sleep 30; done;" ]
  tolerations:
  - operator: Exists
    effect: NoSchedule
```


Then you need to apply it using `kubectl apply -f <folder/file>` command.

```
kubectl apply -f k8s/aws-test.yaml
```


**Verify via command line :**

```
[ec2-user@ip-172-31-22-236 myeks]$ kubectl get po -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   aws-node-tgs9f             2/2     Running   0          50m
kube-system   coredns-7bb495d866-prlzp   1/1     Running   0          52m
kube-system   coredns-7bb495d866-x5g5s   1/1     Running   0          52m
kube-system   kube-proxy-ktcn9           1/1     Running   0          50m

[ec2-user@ip-172-31-22-236 myeks]$ kubectl get po -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
default       aws-cli                    1/1     Running   0          2m13s
kube-system   aws-node-tgs9f             2/2     Running   0          52m
kube-system   coredns-7bb495d866-prlzp   1/1     Running   0          54m
kube-system   coredns-7bb495d866-x5g5s   1/1     Running   0          54m
kube-system   kube-proxy-ktcn9           1/1     Running   0          52m

```

Let's add missing annotation to the service account and redeploy the pod. Don't forget to replace `XXXXXXXXX` with your AWS account number.

```
---
...
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::xxxxxxxxx:role/test-oidc
...
```


Then you need to apply it using `kubectl apply -f <folder/file>` command.

```
kubectl apply -f k8s/aws-test.yaml
```

# 4 Create load balancer 

## 4.1 Create public load balancer on EKS


- Next, let's deploy the sample application and expose it using public and private load balancers. The first is a deployment object with a base nginx image. File name is `k8s/deployment.yaml`.
见  https://community.aws/content/2owpkcMiX4bBoidvHcXhVXiAAIJ/effortless-aws-eks-cluster-autoscaling-using-terraform#create-public-load-balancer-on-eks

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
        - name: web
          containerPort: 80
        resources:
          requests:
            memory: 256Mi
            cpu: 250m
          limits:
            memory: 256Mi
            cpu: 250m
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - general
      # tolerations:
      # - key: team
      #   operator: Equal
      #   value: devops
      #   effect: NoSchedules.
```


To expose the application to the internet, you can create a Kubernetes service of a type load balancer and use annotations to configure load balancer properties. By default, Kubernetes will create a load balancer in public subnets, so you don't need to provide any additional configurations. Also, if you want a new network load balancer instead of the old classic load balancer, you can add aws-load-balancer-type equal to nlb. Call it `k8s/public-lb.yaml`


```
---
apiVersion: v1
kind: Service
metadata:
  name: public-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: web
```


Create both deployment and the service objects.
```
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/public-lb.yaml
```
- Find load balancer in AWS console by name. Verify that LB was created in public subnets




## 4.2 Create private load balancer on EKS

- Sometimes if you have a large infrastructure with many different services, you have a requirement to expose the application only within your VPC. For that, you can create a private load balancer.
    
- To make it private, you need additional annotation: aws-load-balancer-internal and then provide the CIDR range. Usually, you use 0.0.0.0/0 to allow any services within your VPC to access it. Give it a name `private-lb.yaml`.

见 https://community.aws/content/2owpkcMiX4bBoidvHcXhVXiAAIJ/effortless-aws-eks-cluster-autoscaling-using-terraform

```
---
apiVersion: v1
kind: Service
metadata:
  name: private-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
    service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: web
```


# 5 depolyen EKS autoscaller 

## 5.1 depolyen EKS autoscaller 方式1 


Finally, we got to the EKS autoscaller. We will be using OpenID connect provider to create an IAM role and bind it with the autoscaller. Let's create an IAM policy and role first. It's similar to the previous one, but autoscaller will be deployed in the kube-system namespace.

```
data "aws_iam_policy_document" "eks_cluster_autoscaler_assume_role_policy" {
  statement {
    actions = ["sts:AssumeRoleWithWebIdentity"]
    effect  = "Allow"

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.eks.url, "https://", "")}:sub"
      values   = ["system:serviceaccount:kube-system:cluster-autoscaler"]
    }

    principals {
      identifiers = [aws_iam_openid_connect_provider.eks.arn]
      type        = "Federated"
    }
  }
}

resource "aws_iam_role" "eks_cluster_autoscaler" {
  assume_role_policy = data.aws_iam_policy_document.eks_cluster_autoscaler_assume_role_policy.json
  name               = "eks-cluster-autoscaler"
}

resource "aws_iam_policy" "eks_cluster_autoscaler" {
  name = "eks-cluster-autoscaler"

  policy = jsonencode({
    Statement = [{
      Action = [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ]
      Effect   = "Allow"
      Resource = "*"
    }]
    Version = "2012-10-17"
  })
}

resource "aws_iam_role_policy_attachment" "eks_cluster_autoscaler_attach" {
  role       = aws_iam_role.eks_cluster_autoscaler.name
  policy_arn = aws_iam_policy.eks_cluster_autoscaler.arn
}

output "eks_cluster_autoscaler_arn" {
  value = aws_iam_role.eks_cluster_autoscaler.arn
}
```


> And note on **values = [“system:serviceaccount:kube-system:cluster-autoscaler”]** field in the I am policy document that specify the service account name within the kube-system namespace in your kubernetes cluster.

---

Next, we deploy Cluster Autoscaler. To do so, you must use the Amazon Resource Names (ARN) number of the IAM role created in our earlier step.The content intended to save into a file (make sure you copy all of the content presented over the next page):Modify below two lines

- **Line 8 :** change IAM Role name
- **Line 159 :** **--node-group-auto-discovery =** This is used by CA to discover the Auto Scaling group ba
- sed on its tag.


```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::424432388155:role/eks-cluster-autoscaler
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
    verbs: ["create","list","watch"]
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
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.31.1
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
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/ashish
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt #/etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
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



## 5.2 depolyen EKS autoscaller 方式1 

1  **Create IAM Role for Cluster Autoscaler**

Create a new IAM policy and IAM role that will allow Cluster Autoscaler to interact with the Auto Scaling groups and EC2 instances in your AWS account.

Terraform: Create IAM Policy
```
resource "aws_iam_policy" "cluster_autoscaler_policy" {
  name        = "ClusterAutoscalerPolicy"
  description = "IAM policy for Cluster Autoscaler to scale nodes in EKS"

  policy = <<POLICY
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:UpdateAutoScalingGroup",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:AttachInstances",
                "autoscaling:DetachInstances",
                "ec2:DescribeInstances"
            ],
            "Resource": "*"
        }
    ]
}
POLICY
}
```


2 Create IAM Role and Attach the Policy

This IAM role will allow Cluster Autoscaler to assume the policy.
```
resource "aws_iam_role" "cluster_autoscaler_role" {
  name = "cluster-autoscaler-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action    = "sts:AssumeRole"
        Principal = {
          Service = "eks.amazonaws.com"
        }
        Effect    = "Allow"
        Sid       = ""
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "cluster_autoscaler_policy_attachment" {
  role       = aws_iam_role.cluster_autoscaler_role.name
  policy_arn = aws_iam_policy.cluster_autoscaler_policy.arn
}

```

3 Create the Service Account in Kubernetes

You need to create a Kubernetes ServiceAccount for the Cluster Autoscaler and associate it with the IAM Role you just created. This can be done either via eksctl or manually using kubectl.

Service Account Definition (YAML):
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-autoscaler
  namespace: kube-system
```


Apply it using kubectl:
```
kubectl apply -f cluster-autoscaler-service-account.yaml
```

Alternatively, you can use eksctl to automatically create the IAM Role and the ServiceAccount for you.
```
eksctl create iamserviceaccount \
  --region <region> \
  --name cluster-autoscaler \
  --namespace kube-system \
  --cluster <cluster-name> \
  --attach-policy-arn arn:aws:iam::<aws-account-id>:policy/ClusterAutoscalerPolicy \
  --approve \
  --override-existing-serviceaccounts
```



4  **Deploy Cluster Autoscaler**

You can deploy Cluster Autoscaler either by using Helm or by applying a Kubernetes manifest directly.

**Option 1: Helm Deployment** (Recommended for easier configuration)

```
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=<EKS_CLUSTER_NAME> \
  --set awsRegion=<AWS_REGION> \
  --set rbac.create=false \
  --set serviceAccount.name=cluster-autoscaler
```

**Option 2: Direct YAML Deployment**

Here is an example of a Kubernetes `Deployment` YAML for Cluster Autoscaler.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
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
      serviceAccountName: cluster-autoscaler
      containers:
        - name: cluster-autoscaler
          image: "k8s.gcr.io/cluster-autoscaler:v1.22.0"
          command:
            - ./cluster-autoscaler
            - --v=4
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --nodes=1:10:<YOUR-ASG-NAME>
            - --balance-similar-node-groups=true
            - --expander=least-waste
            - --scale-down-delay-after-add=10m
            - --scale-down-unneeded-time=10m
            - --scale-down-utilization-threshold=0.5
          env:
            - name: AWS_REGION
              value: <AWS_REGION>
      nodeSelector:
        kubernetes.io/os: linux
      tolerations:
        - key: "node.kubernetes.io/not-ready"
          operator: "Exists"
          effect: "NoSchedule"
        - key: "node.kubernetes.io/unreachable"
          operator: "Exists"
          effect: "NoSchedule"

```

---


Configure Cluster Autoscaler to Scale Based on Auto Scaling Group

In the Cluster Autoscaler deployment, ensure you specify your Auto Scaling Group (ASG) and the desired node scaling limits (e.g., min and max nodes).

You can do this by adjusting the --nodes argument in the deployment:

```
--nodes=1:10:<YOUR-ASG-NAME>
```

**Verify Installation and Logs**

To verify if the Cluster Autoscaler is working correctly, check its logs:
```
kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

You should see logs indicating that the autoscaler is monitoring node utilization and adjusting the cluster size as needed.


## 5.3 Check 

and check all pods are up and running by following the command:

```
kubectl get pod -n kube-system
```

![[image/1_aiqwRFfp3O8RyVFOUdjg7A.webp]]


