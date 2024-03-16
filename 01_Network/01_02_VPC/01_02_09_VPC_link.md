
VPC links enable you to create private integrations that connect your HTTP API routes to private resources in a VPC, such as Application Load Balancers or Amazon ECS container-based applications. To learn more about creating private integrations, see [Working with private integrations for HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-private.html).

A private integration uses a VPC link to encapsulate connections between API Gateway and targeted VPC resources. You can reuse VPC links across different routes and APIs.

When you create a VPC link, API Gateway creates and manages [elastic network interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html) for the VPC link in your account. This process can take a few minutes. When a VPC link is ready to use, its state transitions from `PENDING` to `AVAILABLE`.

# 1 Connecting an API Gateway to a VPC using VPC Link

VPC Link 通常和 API Gateway 一起使用 ， 为了  connect an API Gateway to a VPC ==without exposing your VPC resources (e.g. Load Balancers, EC2) to the internet.==

https://manurana.medium.com/tutorial-connecting-an-api-gateway-to-a-vpc-using-vpc-link-682a21281263

A very typical deployment architecture for smaller startups is to have an API Gateway at the front, which passes on requests to an ELB, which in turn distributes them to a bunch of EC2 instances. ELBs and EC2s are typically inside a VPC. 
For an API Gateway to use an ELB as the HTTP endpoint for integration, ==the ELB needs to be exposed to the internet==
This implies that an API request can theoretically be made directly to the ELB, bypassing all the rules configured on the API Gateway. This is a concern.

You can allow only requests originating from the API Gateway on your app by configuring client certificates, but that does not prevent flooding style DDOS attacks on the ELB (and the EC2s). ==An ideal solution is to create a link between the API Gateway and the ELB which is NOT exposed to the internet.==

Such a solution does exist, and it is called a **VPC Link**. This works only with a Network Load Balancer (NLB). An NLB works at the TCP layer, and cannot terminate SSL. But that’s just fine because the connection between the API Gateway and the NLB will be through the VPC Link, which seems to be a VPN tunnel of sorts. The SSL termination will be handled at the API Gateway itself.

So let us build an API that interfaces with a non-internet facing NLB through a VPC Link. We will configure the API as an **HTTP Proxy Integration**, which passes all requests directly to the NLB. This should work for most simple cases.

