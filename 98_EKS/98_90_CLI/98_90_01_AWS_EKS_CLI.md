
`aws eks` 本身并不会直接部署 Pods，它更像是负责“平台本身”的管理，比如：
- 创建/删除集群
- 管理节点组
- 管理集群 IAM 权限
- 获取 kubeconfig 配置等
而具体部署服务、Ingress、ConfigMap、Secrets 等，还是需要用 `kubectl`。

# 1 list-clusters

```
aws eks list-clusters --profile ivu-cloud-e2x

⚡ 🦄  aws eks list-clusters --profile ivu-cloud-e2x
{
    "clusters": [
        "dev"
    ]
}
```


# 2 describe-cluster


```
aws eks describe-cluster --name dev --region eu-central-1

aws eks describe-cluster --profile ivu-cloud-e2x

 ⚡ 🦄  aws eks describe-cluster --name dev --profile ivu-cloud-e2x
{
    "cluster": {
        "name": "dev",
        "arn": "arn:aws:eks:eu-central-1:681290371536:cluster/dev",
        "createdAt": "2024-06-17T11:58:19.666000+02:00",
        "version": "1.30",
        "endpoint": "https://08A0408DF33822C2DE2D141CFECDB47F.gr7.eu-central-1.eks.amazonaws.com",
        "roleArn": "arn:aws:iam::681290371536:role/eks-dev-cluster",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-06398a4a5670a57ed",
                "subnet-0fe5d8ec644987d02",
                "subnet-0f93d13299632685e",
                "subnet-083d08782d95ec2d5"
            ],
            "securityGroupIds": [
                "sg-085a92c78ecd21b72"
            ],
            "clusterSecurityGroupId": "sg-03d6cd5a89d764909",
            "vpcId": "vpc-025e448680853a8a8",
            "endpointPublicAccess": false,
            "endpointPrivateAccess": true,
            "publicAccessCidrs": []
        },
        "kubernetesNetworkConfig": {
            "serviceIpv4Cidr": "10.232.0.0/18",
            "ipFamily": "ipv4"
        },
        "logging": {
            "clusterLogging": [
                {
                    "types": [
                        "api",
                        "audit",
                        "authenticator",
                        "controllerManager",
                        "scheduler"
                    ],
                    "enabled": true
                }
            ]
        },
        "identity": {
            "oidc": {
                "issuer": "https://oidc.eks.eu-central-1.amazonaws.com/id/08A0408DF33822C2DE2D141CFECDB47F"
            }
        },
        "status": "ACTIVE",
        "certificateAuthority": {
            "data": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJUlAvTXRXaE5BTzB3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBMk1UY3dPVFU0TVRGYUZ3MHpOREEyTVRVeE1EQXpNVEZhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUM2OERTUWJTQTR4WTF5T3RkeFpwQ1djZkJ0SE10dzVRc2pNL2JNbm9CTUdpUjVSdW1nVUN1Q0dpbHYKR3NDUzRhRWMyYndVRHhHbUtLeUk2ZHJZT0JJVnVZMGhVZlh5V1FCMWNDM1g2d2J3eGR1QWtMZ1VvSzJMS29OZgpOVk5sSElNWkZmbFRuRUxUVFFJczFUK1h0N215K0lDckNPRlVyMGtsdWUwMVFnRzA1K25lT0ZjbktWdURmTUd3CnNGU0szT04xa2JacGJDMkNGemx5TzVjZEJpbzVEVlVIVUx4RjFLdnJpUmRFblp5MlB1cEI4NkJFWitjdVpLVzcKTVNVc0xDajBrT01RRkMyTUxXS0NxcGFGK0lOdStyUmFTVGd2dWlna1ZCNXRjeUdYWkx0QWd3aWtqQ2NxaGZjWgp1TFhaK3EvVUZ5RTRTK3NKb0s5L0Zpa1pBVExWQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUeEFMVEhvK3M2a3Zrb1o1YXU0aXpRRmdoUWlqQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQklBNHZ0NDg1MApiYXBPWnlNenR3STJ0Zm8xRVFuT0dkTTlUcG1BMlIwRmZGdHVaVFo4STcvbEwxZU91ME8yZ0ljeVV6eVk0ckZsCmxOYTAyRkZKTGViRDVMSTMxQURlaGR6UzRpNlVBNitJbUFPbXkyV2RlTG1hNUQxbWpid2RKcVpOZmZLc25oSC8KdVlvMDFFQkNxTDF1Sm1KbXJhYmt4YVA0b3RzamsxUmVteGZWdWhUNEkxalRieG96VGE2M0JJcXB6YzR5RWYvLwoySVBybVJjRGRNZUFQSHRvQWEvQ2tkczhGMDhhRm1BNEFaVmF6ZWxWWEhORlVlSG9lYnVwYkJsd2o2OUlXZGdkCloweHcyVWFNZmtqT0Q4VUtHb1JhWmlFRURZSXlsemhwbTJCQWFFTDBhVE5KeCtxdFBOWDFyS0loYUtRdVpzRDEKNjZOTTZEOUdvT0hPCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
        },
        "platformVersion": "eks.29",
        "tags": {
            "Cluster": "dev",
            "Name": "dev"
        },
        "encryptionConfig": [
            {
                "resources": [
                    "secrets"
                ],
                "provider": {
                    "keyArn": "arn:aws:kms:eu-central-1:681290371536:key/171770b8-cf70-450e-a31c-f5b47f7aed60"
                }
            }
        ],
        "health": {
            "issues": []
        },
        "accessConfig": {
            "authenticationMode": "API_AND_CONFIG_MAP"
        },
        "upgradePolicy": {
            "supportType": "EXTENDED"
        }
    }
}


```


# 3 create-cluster

创建一个新的 EKS 集群（**前提：IAM 角色和 VPC 已准备好**）


```
aws eks create-cluster \
  --name my-eks-cluster \
  --region us-west-2 \
  --role-arn arn:aws:iam::123456789012:role/eksClusterRole \
  --resources-vpc-config subnetIds=subnet-abc123,subnet-def456,securityGroupIds=sg-0123456789abcdef0

```


# 4 wait cluster-active

等待集群创建完成（否则后续命令会报错）

aws eks wait cluster-active --name my-eks-cluster --region us-west-2


# 5 update-kubeconfig: 下载集群的 kubeconfig 到本地以便 `kubectl` 使用

下载集群的 kubeconfig 到本地以便 `kubectl` 使用

```
aws eks update-kubeconfig --name my-eks-cluster --region us-west-2

```


# 6 delete-cluster

删除集群
```
aws eks delete-cluster --name my-eks-cluster --region us-west-2

```


# 7 create-nodegroup

添加节点组（使用 aws eks create-nodegroup）


```
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name my-node-group \
  --scaling-config minSize=1,maxSize=3,desiredSize=2 \
  --disk-size 20 \
  --subnets subnet-xxxx subnet-yyyy \
  --instance-types t3.medium \
  --ami-type AL2_x86_64 \
  --node-role arn:aws:iam::123456789012:role/EKSNodeRole \
  --region eu-central-1
```





