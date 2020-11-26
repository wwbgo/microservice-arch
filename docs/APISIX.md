# 架构
![https://github.com/apache/apisix/raw/master/doc/images/apisix.png](https://github.com/apache/apisix/raw/master/doc/images/apisix.png)

## 特性

你可以把 Apache APISIX 当做流量入口，来处理所有的业务数据，包括动态路由、动态上游、动态证书、 A/B 测试、金丝雀发布(灰度发布)、蓝绿部署、限流限速、抵御恶意攻击、监控报警、服务可观测性、服务治理等。

- **全平台**
    
    - 云原生: 平台无关，没有供应商锁定，无论裸机还是 Kubernetes，APISIX 都可以运行。
    - 运行环境: OpenResty 和 Tengine 都支持。
    - 支持 ARM64: 不用担心底层技术的锁定。
- **多协议**
    
    - TCP/UDP 代理: 动态 TCP/UDP 代理。
    - 动态 MQTT 代理: 支持用 `client_id` 对 MQTT 进行负载均衡，同时支持 MQTT 3.1 和 5.0 两个协议标准。
    - gRPC 代理：通过 APISIX 代理 gRPC 连接，并使用 APISIX 的大部分特性管理你的 gRPC 服务。
    - gRPC 协议转换：支持协议的转换，这样客户端可以通过 HTTP/JSON 来访问你的 gRPC API。
    - Websocket 代理
    - Proxy Protocol
    - Dubbo 代理：基于 Tengine，可以实现 Dubbo 请求的代理。
    - HTTP(S) 反向代理
    - SSL：动态加载 SSL 证书。
- **全动态能力**
    
    - 热更新和热插件: 无需重启服务，就可以持续更新配置和插件。
    - 代理请求重写: 支持重写请求上游的`host`、`uri`、`schema`、`enable_websocket`、`headers`信息。
    - 输出内容重写: 支持自定义修改返回内容的 `status code`、`body`、`headers`。
    - Serverless: 在 APISIX 的每一个阶段，你都可以添加并调用自己编写的函数。
    - 动态负载均衡：动态支持有权重的 round-robin 负载平衡。
    - 支持一致性 hash 的负载均衡：动态支持一致性 hash 的负载均衡。
    - 健康检查：启用上游节点的健康检查，将在负载均衡期间自动过滤不健康的节点，以确保系统稳定性。
    - 熔断器: 智能跟踪不健康上游服务。
    - 代理镜像: 提供镜像客户端请求的能力。
- **精细化路由**
    
    - 支持全路径匹配和前缀匹配
    - 支持使用 Nginx 所有内置变量做为路由的条件，所以你可以使用 `cookie`, `args` 等做为路由的条件，来实现灰度发布、A/B 测试等功能
    - 支持各类操作符做为路由的判断条件，比如 `{"arg_age", ">", 24}`
    - 支持自定义路由匹配函数
    - IPv6：支持使用 IPv6 格式匹配路由
    - 支持路由的自动过期(TTL)
    - 支持路由的优先级
    - 支持批量 Http 请求
- **安全防护**
    
    - 多种身份认证方式: key-auth, JWT, basic-auth, wolf-rbac
    - IP 黑白名单
    - Referer 白名单
    - IdP 支持: 支持外部的身份认证服务，比如 Auth0，Okta，Authing 等，用户可以借此来对接 Oauth2.0 等认证方式
    - 限制速率
    - 限制请求数
    - 限制并发
    - 防御 ReDoS(正则表达式拒绝服务)：内置策略，无需配置即可抵御 ReDoS
    - CORS：为你的 API 启用 CORS
    - URI 拦截器：根据 URI 拦截用户请求
    - 请求验证器
- **运维友好**
    
    - OpenTracing 可观测性: 支持 Apache Skywalking 和 Zipkin
    - 对接外部服务发现：除了内置的 etcd 外，还支持 `Consul` 和 `Nacos` 的 DNS 发现模式，以及 Eureka
    - 监控和指标: Prometheus
    - 集群：APISIX 节点是无状态的
    - 高可用：支持配置同一个集群内的多个 etcd 地址。
    - 控制台: 内置控制台来操作 APISIX 集群。
    - 版本控制：支持操作的多次回滚。
    - CLI: 使用命令行来启动、关闭和重启 APISIX。
    - 单机模式: 支持从本地配置文件中加载路由规则，在 kubernetes(k8s) 等环境下更友好。
    - 全局规则：允许对所有请求执行插件，比如黑白名单、限流限速等。
    - 高性能：在单核上 QPS 可以达到 18k，同时延迟只有 0.2 毫秒。
    - 故障注入
    - REST Admin API: 使用 REST Admin API 来控制 Apache APISIX，默认只允许 127.0.0.1 访问，你可以修改 `conf/config.yaml` 中的 `allow_admin` 字段，指定允许调用 Admin API 的 IP 列表。同时需要注意的是，Admin API 使用 key auth 来校验调用者身份，**在部署前需要修改 `conf/config.yaml` 中的 `admin_key` 字段，来保证安全。**
    - 外部日志记录器：将访问日志导出到外部日志管理工具
- **高度可扩展**
    
    - 自定义插件: 允许挂载常见阶段，例如`init`, `rewrite`，`access`，`balancer`,`header filer`，`body filter` 和 `log` 阶段。
    - 自定义负载均衡算法：可以在 `balancer` 阶段使用自定义负载均衡算法。
    - 自定义路由: 支持用户自己实现路由算法。