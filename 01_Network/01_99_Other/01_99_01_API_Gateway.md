

Amazon API Gateway 是一项AWS服务，用于创建、发布、维护、监控和保护任意规模的 REST、HTTP 和 WebSocket API。API 开发人员可以创建能够访问 AWS 或其他 Web 服务以及存储在 [AWS 云](https://aws.amazon.com/what-is-cloud-computing/)中的数据的 API。作为 API Gateway API 开发人员，您可以创建 API 以在您自己的客户端应用程序中使用。或者，您可以将您的 API 提供给第三方应用程序开发人员。有关更多信息，请参阅 [谁使用 API Gateway？](https://docs.aws.amazon.com/zh_cn/apigateway/latest/developerguide/api-gateway-overview-developer-experience.html#apigateway-who-uses-api-gateway)。

API Gateway 创建符合下列条件的 RESTful API：
- 基于 HTTP 的。
- 启用无状态客户端-服务器通信。
- 实施标准 HTTP 方法例，如 GET、POST、PUT、PATCH 和 DELETE。


![](image/Pasted%20image%2020240228213602.png)

# 1 AWS 无服务器基础设施的一部分

API Gateway 与 AWS Lambda 共同构成 AWS 无服务器基础设施中面向应用程序的部分。对于调用公开 AWS 服务的应用程序，您可以使用 Lambda 与所需的服务交互，并通过 API Gateway 中的 API 方法来使用 Lambda 函数。AWS Lambda 在高可用性计算基础设施上运行代码。它会进行必要的计算资源执行和管理工作。为了支持无服务器应用程序，API Gateway 可以支持与 AWS Lambda 的[简化代理集成](http://docs.aws.amazon.com/zh_cn/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html)和 HTTP 终端节点。

## 1.1 调用 API Gateway API

应用程序开发人员使用名为 `execute-api` 的 API Gateway 服务组件来调用在 API Gateway 中创建或部署的 API。底层编程实体由创建的 API 公开。此类 API 有若干种调用方式。您可以使用 API Gateway 控制台来测试调用 API。您可以使用 CURL 或 Postman 等 REST API 客户端来调用 API，也可以使用 API Gateway 为 API 生成的软件开发工具包来调用 API。

请注意 `apigateway` 和 `execute-api` API Gateway 服务组件之间的差异。在设置 IAM 权限策略以便构建或调用 API 时，请在选择服务组件时引用相应的服务组件名称。

## 1.2 API Gateway 的优势

API Gateway 可以帮助您提供可靠、安全并可以扩展的移动和 Web 应用程序后端。API Gateway 让您能够将移动和 Web 应用程序安全地连接到托管在 AWS Lambda 上的业务逻辑、托管在 Amazon EC2 上的 API 或者托管在 AWS 内部或外部的其他可公开寻址的 Web 服务。借助 API Gateway，您可以针对后端服务创建和运行 API。例如，您不需要开发和维护基础设施来处理授权和访问控制、流量管理、监控和分析、版本管理和软件开发工具包 (SDK) 的生成工作。

API Gateway 专为 Web 和移动开发人员而设计，可以为内部构建或第三方生态系统合作伙伴构建的移动应用程序、Web 应用程序和服务器应用程序提供安全可靠的后端 API 访问。API 背后的业务逻辑可以由 API Gateway 代理调用的可公开访问的终端节点提供，也可以完全作为一个 Lambda 函数来运行。


