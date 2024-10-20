
ALB:  AWS Applicatiob Load balancer 
# 1 Certificate

Eine krasse Einschränkung ist dass das Certificate Management damit wieder nach außerhalb von Kubernetes verlagert wird, weil der ALB Ingress nur Certs benutzen kann die in AWS Certificate Manager vorhanden sind. 


是否能用 Private CA Management mit der aws api umgehen , 而不用  AWS Certificate Manager
答案是不行 
Man kann zwar einen AWS Dienst fürs Private CA Management an cert-manager anbinden. Aber das Verfahren hier ist ja, dass der ALB überhaupt nur Certs annimmt, die in ACM eingespielt wurden, und nicht solche aus Kubernetes Secrets. 

Man kriegt die Certs für den ALB Ingress über eine Annotation rein: 
`alb.ingress.kubernetes.io/certificate-arn`
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#certificate-arn

für das was unter "tls:" steht interessiert sich ALB gar nicht. 






