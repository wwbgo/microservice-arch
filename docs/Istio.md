## 核心特性

Istio 以统一的方式提供了许多跨服务网络的关键功能：

### 流量管理

Istio 简单的规则配置和流量路由允许您控制服务之间的流量和 API 调用过程。Istio 简化了服务级属性（如熔断器、超时和重试）的配置，并且让它轻而易举的执行重要的任务（如 A/B 测试、金丝雀发布和按流量百分比划分的分阶段发布）。

有了更好的对流量的可视性和开箱即用的故障恢复特性，您就可以在问题产生之前捕获它们，无论面对什么情况都可以使调用更可靠，网络更健壮。

### 安全

Istio 的安全特性解放了开发人员，使其只需要专注于应用程序级别的安全。Istio 提供了底层的安全通信通道，并为大规模的服务通信管理认证、授权和加密。有了 Istio，服务通信在默认情况下就是受保护的，可以让您在跨不同协议和运行时的情况下实施一致的策略——而所有这些都只需要很少甚至不需要修改应用程序。

Istio 是独立于平台的，可以与 Kubernetes（或基础设施）的网络策略一起使用。但它更强大，能够在网络和应用层面保护pod到 pod 或者服务到服务之间的通信。

### 可观察性

Istio 健壮的追踪、监控和日志特性让您能够深入的了解服务网格部署。通过 Istio 的监控能力，可以真正的了解到服务的性能是如何影响上游和下游的；而它的定制 Dashboard 提供了对所有服务性能的可视化能力，并让您看到它如何影响其他进程。

Istio 的 Mixer 组件负责策略控制和遥测数据收集。它提供了后端抽象和中介，将一部分 Istio 与后端的基础设施实现细节隔离开来，并为运维人员提供了对网格与后端基础实施之间交互的细粒度控制。

所有这些特性都使您能够更有效地设置、监控和加强服务的 SLO。当然，底线是您可以快速有效地检测到并修复出现的问题。

## 平台支持

Istio 独立于平台，被设计为可以在各种环境中运行，包括跨云、内部环境、Kubernetes、Mesos 等等。您可以在 Kubernetes 或是装有 Consul 的 Nomad 环境上部署 Istio。Istio 目前支持：

- Kubernetes 上的服务部署
    
- 基于 Consul 的服务注册
    
- 服务运行在独立的虚拟机上
    

## 整合和定制

Istio 的策略实施组件可以扩展和定制，与现有的 ACL、日志、监控、配额、审查等解决方案集成。

组件
-----------
### Envoy

Istio 使用 Envoy 代理的扩展版本。Envoy 是用 C++ 开发的高性能代理，用于协调服务网格中所有服务的入站和出站流量。Envoy 代理是唯一与数据平面流量交互的 Istio 组件。

Envoy 代理被部署为服务的 sidecar，在逻辑上为服务增加了 Envoy 的许多内置特性，例如:

- 动态服务发现
- 负载均衡
- TLS 终端
- HTTP/2 与 gRPC 代理
- 熔断器
- 健康检查
- 基于百分比流量分割的分阶段发布
- 故障注入
- 丰富的指标

这种 sidecar 部署允许 Istio 提取大量关于流量行为的信号作为属性。Istio 可以使用这些属性来实施策略决策，并将其发送到监视系统以提供有关整个网格行为的信息。

sidecar 代理模型还允许您向现有的部署添加 Istio 功能，而不需要重新设计架构或重写代码。您可以在设计目标中读到更多关于为什么我们选择这种方法的信息。

由 Envoy 代理启用的一些 Istio 的功能和任务包括:

- 流量控制功能：通过丰富的 HTTP、gRPC、WebSocket 和 TCP 流量路由规则来执行细粒度的流量控制。
- 网络弹性特性：重试设置、故障转移、熔断器和故障注入。
- 安全性和身份验证特性：执行安全性策略以及通过配置 API 定义的访问控制和速率限制。
- 基于 WebAssembly 的可插拔扩展模型，允许通过自定义策略实施和生成网格流量的遥测。

### Pilot

Pilot 为 Envoy sidecar 提供服务发现、用于智能路由的流量管理功能（例如，A/B 测试、金丝雀发布等）以及弹性功能（超时、重试、熔断器等）。

Pilot 将控制流量行为的高级路由规则转换为特定于环境的配置，并在运行时将它们传播到 sidecar。Pilot 将特定于平台的服务发现机制抽象出来，并将它们合成为任何符合 Envoy API 的 sidecar 都可以使用的标准格式。

下图展示了平台适配器和 Envoy 代理如何交互。

![https://istio.io/latest/zh/docs/ops/deployment/architecture/discovery.svg](https://istio.io/latest/zh/docs/ops/deployment/architecture/discovery.svg)

1. 平台启动一个服务的新实例，该实例通知其平台适配器。
2. 平台适配器使用 Pilot 抽象模型注册实例。
3. **Pilot** 将流量规则和配置派发给 Envoy 代理，来传达此次更改。

这种松耦合允许 Istio 在 Kubernetes、Consul 或 Nomad 等多种环境中运行，同时维护相同的 operator 接口来进行流量管理。

您可以使用 Istio 的流量管理 API 来指示 Pilot 优化 Envoy 配置，以便对服务网格中的流量进行更细粒度地控制。

### Citadel

Citadel 通过内置的身份和证书管理，可以支持强大的服务到服务以及最终用户的身份验证。您可以使用 Citadel 来升级服务网格中的未加密流量。使用 Citadel，operator 可以执行基于服务身份的策略，而不是相对不稳定的 3 层或 4 层网络标识。可以使用 Istio 的授权特性来控制谁可以访问您的服务。

### Galley

Galley 是 Istio 的配置验证、提取、处理和分发组件。它负责将其余的 Istio 组件与从底层平台（例如 Kubernetes）获取用户配置的细节隔离开来。


实例演示
-----------
# 流量管理

``` 配置路由
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml -n test
```
``` 查看路由配置
kubectl get destinationrules -o yaml -n test
```

### 请求路由

``` 配置路由到 Bookinfo 微服务的 v1 版本(不带星级评分)， reviews 服务v1
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml -n test
```
``` 根据用户路由, 用户jason路由到v2(带星级评分), 其他情况还是v1
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml -n test
```

### 故障注入

注意 reviews:v2 服务对 ratings 服务的调用具有 10 秒的硬编码连接超时
``` 人为注入延迟
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml -n test
```
reviews 和 ratings 服务间的超时被硬编码为 10 秒

在 productpage 和 reviews 服务之间也有一个 3 秒的硬编码的超时，再加 1 次重试，一共 6 秒

这种类型的错误可能发生在典型的由不同的团队独立开发不同的微服务的企业应用程序中。 
Istio 的故障注入规则可以帮助您识别此类异常，而不会影响最终用户。

``` 注入 HTTP abort 故障
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml -n test
```

### 流量转移

``` 将所有流量打到v1
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml -n test
```
``` 把 50% 的流量从 reviews:v1 转移到 reviews:v3
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml -n test
```
金丝雀部署

### 请求超时

默认情况下，超时是禁用的

路由规则 timeout 字段来指定

### 熔断

熔断，是创建弹性微服务应用程序的重要模式。熔断能够使您的应用程序具备应对来自故障、潜在峰值和其他 未知网络因素影响的能力。

### 镜像

流量镜像，也称为影子流量，是一个以尽可能低的风险为生产带来变化的强大的功能。镜像会将实时流量的副本发送到镜像服务。镜像流量发生在主服务的关键请求路径之外

### Ingress

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "*"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /headers
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```

### Egress


# 安全

### 认证

``` JWT
apiVersion: "authentication.istio.io/v1alpha1"
kind: "Policy"
metadata:
  name: "jwt-example"
spec:
  targets:
  - name: httpbin
  origins:
  - jwt:
      issuer: "https://ids.dev.fbox.flexem.net/core"
      jwksUri: "https://ids.dev.fbox.flexem.net/core/.well-known/openid-configuration/jwks"
  principalBinding: USE_ORIGIN
```

### 授权

``` spec: 字段为空值 {}，意思是不允许任何流量
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  {}
```
```
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: "details-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: details
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
```

# 策略

### 策略检查功能

在默认的 Istio 安装配置中，策略检查功能是关闭的

安装选项中加入--set values.global.disablePolicyChecks=false 和 --set values.pilot.policy.enabled=true

### 速率限制

```
kubectl apply -f samples/bookinfo/policy/mixer-rule-productpage-ratelimit.yaml
```

### 请求头和路由控制

### 拒绝请求和黑白名单


# 可观察性

### 指标度量

### 日志

### 分布式追踪

- `x-request-id`
- `x-b3-traceid`
- `x-b3-spanid`
- `x-b3-parentspanid`
- `x-b3-sampled`
- `x-b3-flags`
- `x-ot-span-context`



### 网络可视化

Kiali

### 远程访问遥测
