



# 1 create 

You can create an IAM role using eksctl with the following command:
```
eksctl create iamserviceaccount \
  --cluster=eks-demo \
  --namespace=kube-system \
  --name=cluster-autoscaler \
  --attach-policy-arn=<ARN from the previous step> \
  --override-existing-serviceaccounts \
  --approve
```


In Amazon EKS (Elastic Kubernetes Service), an **IAM service account** refers to a Kubernetes service account that is **linked to an AWS IAM role**. This allows **pods in your EKS cluster** to securely and automatically assume AWS IAM roles and access AWS services (like S3, DynamoDB, SQS, etc.) — without needing to store AWS credentials in the pod.


Normally, pods don’t have AWS credentials. But with IAM roles for service accounts (IRSA), you can associate a Kubernetes service account with an IAM role, and the pod using that service account gets temporary credentials through the AWS SDK.

Let’s say you have a pod that needs to read from an S3 bucket. Instead of embedding AWS credentials in the container:
1. You create an **IAM role** with the necessary S3 permissions.
2. You create a **Kubernetes service account** in your EKS cluster.
3. You **associate the IAM role with the Kubernetes service account** using annotations.    
4. Your pod uses this service account → gets IAM role → has access to S3 via temporary credentials.

Benefits 
- ✅ Fine-grained access control (per pod)
- ✅ No hardcoded credentials
- ✅ Secure and scalable
- ✅ Works with AWS STS (temporary credentials)


