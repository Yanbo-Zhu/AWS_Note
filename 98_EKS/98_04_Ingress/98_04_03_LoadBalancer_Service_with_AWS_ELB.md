
Kubernetes 中的 `LoadBalancer` 类型的 Service 是用来 **将集群内的服务暴露到集群外部（公网）** 的一种方式。它通常配合云服务商的负载均衡器（如 AWS ELB、GCP LB、Azure LB）使用。
当你创建一个 `LoadBalancer` 类型的 Service，Kubernetes 会==自动==向云服务商申请一个公网的 Load Balancer，并将其流量转发到你集群内部的 Pods。


LoadBalancer_Service
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80         # 外部访问的端口
      targetPort: 8080 # Pod 内部容器监听的端口
```


云平台自动分配 Load Balancer

- **AWS**：创建 ELB（Elastic Load Balancer）
    
- **GCP**：创建 GCP Load Balancer
    
- **Azure**：创建 Azure Load Balancer



你可以用 `kubectl get svc` 查看分配的 **EXTERNAL-IP**：

```
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
my-app-service    LoadBalancer   10.0.30.107    3.92.185.222       80:30698/TCP   5m
```