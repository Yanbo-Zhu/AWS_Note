
ALB:  AWS Applicatiob Load balancer 


模式	控制器	AWS 负载均衡器	备注
模式 1	NGINX Ingress Controller	Classic ELB / NLB	简单、容易部署
模式 2	AWS Load Balancer Controller	ALB（推荐）	原生支持 Ingress，自带 HTTPS、路径、主机路由等功能



# 1 Classic NGINX Ingress + 外部 LoadBalancer（ELB）


- **部署 NGINX Ingress Controller**。
    
- **通过一个 Kubernetes LoadBalancer Service 暴露 Ingress Controller**。
    
- AWS 会自动为你创建一个 **Classic ELB 或 NLB**，用于接收外部流量，并将其转发到 Ingress Controller。
    
- Ingress Controller 根据规则将流量再转发到内部服务。



`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml`
此配置中的 `controller-service.yaml` 使用的是 `type: LoadBalancer`，AWS 会自动创建一个 ELB 实例。

你可以查看：`kubectl get svc -n ingress-nginx`

输出中会有一个 EXTERNAL-IP，即 AWS 创建的 ELB 地址。


# 2 使用 AWS ALB（Application Load Balancer）作为 Ingress Controller

这是在 **EKS 中的推荐做法**，更原生、灵活、支持路径和主机名路由。

步骤概览：
1. 安装 AWS Load Balancer Controller（ALB Controller）


```
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id> \
  --set image.repository=602401143452.dkr.ecr.<your-region>.amazonaws.com/amazon/aws-load-balancer-controller
```

（前提是你已经为 `aws-load-balancer-controller` 创建好 IAM Role 和 ServiceAccount）


创建 Ingress 资源
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-ingress
  namespace: default
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
    - host: my-app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

一旦这个 Ingress 被应用，ALB Controller 会自动在 AWS 上创建一个 **ALB 实例** 来服务此规则。




# 3 Certificate

Eine krasse Einschränkung ist dass das Certificate Management damit wieder nach außerhalb von Kubernetes verlagert wird, weil der ALB Ingress nur Certs benutzen kann die in AWS Certificate Manager vorhanden sind. 


是否能用 Private CA Management mit der aws api umgehen , 而不用  AWS Certificate Manager
答案是不行 
Man kann zwar einen AWS Dienst fürs Private CA Management an cert-manager anbinden. Aber das Verfahren hier ist ja, dass der ALB überhaupt nur Certs annimmt, die in ACM eingespielt wurden, und nicht solche aus Kubernetes Secrets. 

Man kriegt die Certs für den ALB Ingress über eine Annotation rein: 
`alb.ingress.kubernetes.io/certificate-arn`
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#certificate-arn

für das was unter "tls:" steht interessiert sich ALB gar nicht. 






