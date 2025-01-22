1. [TCP 与 UDP 对比分析](./README.md/#tcp-与-udp-对比分析)
2. [MySQL 主从架构简述](./README.md/#mysql-主从架构简述)
3. [Redis 的四种模式分析对比](./README.md/#redis-的四种模式分析对比)
4. [K8S 组件组成](./README.md/#k8s-组件组成)
5. [K8S DaemonSet、Deployment、StatefulSet 和 ReplicaSet Pod 区别](./README.md/#k8s-daemonsetdeploymentstatefulset-和-replicaset-pod-区别)
6. [K8S 流量的治理](./README.md/#k8s-流量的治理)
7. [K8S 客户端访问完整过程](./README.md/#k8s-客户端访问完整过程)
8. [K8S Pod 调度规则/限制](./README.md/#k8s-pod-调度规则限制)
9. [502 Bad Gateway 错误分析](./README.md/#502-bad-gateway-错误分析)
10. [504 Gateway Timeout 错误分析](./README.md/#504-gateway-timeout-错误分析)

---

# TCP 与 UDP 对比分析

| 特性         | **TCP**                                                                    | **UDP**                                                               |
| ------------ | -------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **可靠性**   | 高，数据完整性、顺序传输                                                   | 低，保证数据完整性、无序传输                                          |
| **速度**     | 较慢，需三次握手和确认                                                     | 较快，无连接，数据传输没有握手与确认过程                              |
| **丢包处理** | 丢包重传，确保数据可靠到达                                                 | 不保证丢包重传，适合容忍丢包的场景                                    |
| **使用场景** | 需要保证数据完整性和一致性的场景，如：HTTP/HTTPS、FTP 文件传输、数据库连接 | 视频直播（实时性要求高、在线游戏（实时互动、DNS 查询（简短请求/响应） |
| **延迟**     | 较高，三次握手与确认增加延迟                                               | 较低，数据直接发送，无需等待确认                                      |
| **流量控制** | 支持流量控制（通过窗口大小调整）                                           | 不支持流量控制，适用于对流量控制要求不高的场景                        |

---

# MySQL 主从架构简述

## 1. 启用方式

- **主库配置**：
  - 开启二进制日志 (`binlog`)。设置唯一的 `server ID`。
  - 创建具有 `REPLICATION SLAVE` 权限的复制账户。
- **从库配置**：
  - 设置唯一的 `server ID`。
  - 启用复制功能，记录主库的日志文件和位置来启动复制。

## 2. 复制过程

- 主库将所有更改操作记录到二进制日志 (`binlog`)。
- 从库通过 `IO_THREAD` 线程读取主库的 `binlog`，将其存储在自己的中继日志（`relay log`）中。
- 从库通过 `SQL_THREAD` 线程执行中继日志中的 SQL 语句，实现数据同步。

## 3. 同步方式

- **异步复制**：
  - 主库无需等待从库确认即可提交事务，延迟小，但有数据丢失的风险。
- **半同步复制**：
  - 主库至少等待一个从库确认事务已写入日志，较为可靠，延迟较小。
- **全同步复制**：
  - 主库等待所有从库确认事务已写入日志，确保数据一致性，但增加延迟。

## 4. 高可用与读写分离

- **主库故障恢复**：
  - 在主库发生故障时，可以手动或自动将某个从库提升为新的主库。
  - 主主模式，使用 `Keepalived` 通过 `VIP` 的方式来统一对外提供服务，`Keepalived` 为非抢占 `nopreempt` 模式，避免故障节点恢复后抢占 `VIP`
- **读写分离**：
  - 主库处理写操作，从库处理读操作，减轻主库压力，提升性能。

---

# Redis 的四种模式分析对比

| 部署模式     | 高可用性 | 扩展性 | 读写分离 | 数据存储方式                               | 主要优点               | 适用场景                 |
| ------------ | -------- | ------ | -------- | ------------------------------------------ | ---------------------- | ------------------------ |
| **单机模式** | 无       | 无     | 无       | 数据存储在单个节点内                       | 简单、性能高           | 小型项目、开发测试环境   |
| **主从模式** | 较低     | 低     | 支持     | 主节点存储数据，从节点复制主节点数据       | 读写分离，降低读取压力 | 读多写少的应用场景       |
| **哨兵模式** | 较高     | 低     | 支持     | 数据存储在主从节点，主节点故障时从节点接管 | 自动故障切换，高可用性 | 中小规模、高可用场景     |
| **集群模式** | 高       | 高     | 支持     | 数据分片存储在多个节点上                   | 水平扩展，性能优越     | 大规模分布式系统、高并发 |

---

# K8S 组件组成

| **组件**                    | **描述**                                                                          |
| --------------------------- | --------------------------------------------------------------------------------- |
| **Master 节点**             | 负责管理集群，包含调度、API 接口等关键组件。                                      |
| **kube-apiserver**          | Kubernetes 的 API 接口，所有与集群相关的操作都通过它进行交互。                    |
| **kube-scheduler**          | 负责将 pod 分配到合适的工作节点（Node），根据资源和策略调度。                     |
| **kube-controller-manager** | 负责集群控制器的管理，确保集群状态与期望状态保持一致。                            |
| **etcd**                    | 一个分布式的键值存储系统，用于存储 Kubernetes 的所有配置信息和集群状态。          |
| **Node 节点**               | 集群中的工作节点，运行容器和应用程序。                                            |
| **kubelet**                 | 安装在每个 Node 节点上的代理，确保容器按照预期运行，并向 API 服务器报告节点状态。 |
| **kube-proxy**              | 负责集群中的网络代理，确保各服务之间的流量正确路由。                              |
| **Container Runtime**       | 容器运行时，负责启动和管理容器，如 Docker、containerd、CRI-O 等。                 |

---

# K8S DaemonSet、Deployment、StatefulSet 和 ReplicaSet Pod 区别

| 特性             | **DaemonSet**                                 | **Deployment**                            | **StatefulSet**                              | **ReplicaSet**                         |
| ---------------- | --------------------------------------------- | ----------------------------------------- | -------------------------------------------- | -------------------------------------- |
| **用途**         | 每个节点运行一个 Pod               | 无状态应用 | 管理有状态应用，提供稳定标识和存储           | 确保指定副本数的 Pod 副本存在          |
| **副本数**       | 每个节点一个 Pod 副本                         | 动态调整副本数                            | 固定副本数，每个 Pod 有稳定标识              | 动态调整副本数                         |
| **Pod 标识**     | 每个 Pod 都有唯一名称               | Pod 无固定标识，副本可以相同              | 每个 Pod 有唯一的名称和标识                  | Pod 无固定标识，副本可以相同           |
| **存储**         | 无持久化存储                                  | 无持久化存储                              | 提供持久化存储（每个 Pod 都有持久化存储）    | 无持久化存储                           |
| **网络标识**     | 每个 Pod 都有稳定的网络标识                   | Pod 的网络标识随调度改变                  | 每个 Pod 的网络标识稳定且唯一                | Pod 的网络标识随调度改变               |
| **用途场景**     | 运行集群级别的服务，如日志收集、监控          | 无状态服务，如 Web 应用，API 服务器       | 有状态服务，如数据库，分布式缓存             | 无状态服务，如 Web 应用，API 服务器    |
| **Pod 启动顺序** | 无特定顺序                                    | 并行启动                                  | 按顺序启动，确保每个 Pod 的依赖关系          | 并行启动                               |
| **Pod 健康检查** | 无序列要求                                    | 滚动更新，提供健康检查与回滚              | 按顺序更新，支持每个 Pod 的健康检查          | 滚动更新，提供健康检查与回滚           |
| **扩展方式**     | 无扩展，自动在每个节点上部署 Pod              | 支持水平扩展，自动增加或减少副本数        | 支持水平扩展，按顺序部署并维护有状态数据     | 无法直接扩展，通过 Deployment 调用     |
| **故障恢复**     | 每个节点保证一个 Pod，自动调度到新节点        | 滚动更新时重建 Pod，确保可用性            | Pod 重建时按顺序恢复，每个 Pod 保持数据一致  | 只负责副本数，Pod 出现故障时会重新调度 |
| **常见应用**     | 日志收集（如 Fluentd）、监控（如 Prometheus） | Web 服务、API 服务、无状态应用            | 数据库、消息队列、有状态应用（如 Cassandra） | Web 服务、API 服务、无状态应用         |

---

# K8S 流量的治理

## 1. Service

- 为 Pod 提供统一的访问入口，负责将流量分发到后端的 Pod，实现负载均衡。

## 2. Ingress

- 管理外部 HTTP/HTTPS 流量，根据 URL 路径或主机名将流量路由到不同的服务。

## 3. 亲和、反亲和（Affinity & Anti-affinity）

- 通过亲和、反亲和控制 Pod 如何在集群中调度，确保某些 Pod 必须或避免运行在特定节点上，从而优化流量分布。

## 4. 服务网格 (Service Mesh)

- 服务网格如 `Istio` 提供流量路由、负载均衡、限流，灵活控制和监控微服务之间的流量。

## 5. HPA 自动扩缩容 (Horizontal Pod Autoscaler)

- 根据 CPU、内存使用或自定义指标动态调整 Pod 数量，确保集群能根据流量水平自动伸缩。

### Kubernetes 实现流量治理主要依靠以上几种方式：

- Service 和 Ingress）确保流量均匀分配到集群中的各个 Pod。
- 亲和性确保某些请求始终路由到同一 Pod。
- 服务网格控制 Pod 间的流量，更高级的流量路由、策略控制和监控能力。
- HPA 根据流量水平动态扩缩容，保证资源的弹性伸缩。

---

# K8S 客户端访问完整过程

客户端请求域名 --> 2. DNS 解析 --> 3. CNAME 解析为负载均衡器地址 --> 4. 负载均衡器转发流量到 Ingress 控制器
--> 5. Ingress 控制器根据路由规则处理请求 --> 6. Ingress 路由流量到 Service --> 7. Service 路由流量到 Pod
--> 8. Pod 处理请求并生成响应 --> 9. 响应通过 Service 返回 --> 10. 响应通过 Ingress 或负载均衡器返回客户端

---

# K8S Pod 调度规则/限制

1. **Node Affinity (节点亲和性)**

   - 通过 `nodeAffinity` 设置调度规则，强制或软性限制 Pod 不被调度到特定节点。

2. **Taints and Tolerations (污点与容忍)**

   - 在节点上设置污点（Taint），默认阻止 Pod 调度到该节点；仅允许具有相应容忍度（Toleration）的 Pod 调度上去。

3. **PodAntiAffinity (Pod 反亲和性)**
   - 设置 Pod 之间的反亲和性，避免 Pod 被调度到与某些特定 Pod 相同的节点。

---

# 502 Bad Gateway 错误分析

- `502 Bad Gateway` 错误通常表示服务器作为网关或代理时，**收到来自上游服务器的无效响应**，可能由后端服务不可用、配置错误或网络问题导致。

| **原因**             | **描述**                       | **排查方法**              |
| -------------------- | ------------------------------ | ------------------------- |
| **服务不可用**       | 后端服务崩溃或未响应           | 检查 Pod 状态和日志       |
| **Ingress 配置错误** | 路由配置不正确                 | 检查 Ingress 配置和日志   |
| **Service 配置问题** | 无法正确路由流量               | 检查 Service 配置是否正确 |
| **负载均衡器问题**   | 代理服务器或负载均衡器配置错误 | 查看负载均衡器配置和日志  |
| **网络问题**         | Pod 间无法通信                 | 检查网络插件和 Pod 状态   |
| **资源不足**         | 集群资源不足，导致服务无法响应 | 检查集群资源使用情况      |

---

# 504 Gateway Timeout 错误分析

- `504 Gateway Timeout` 错误通常是由于 **服务器没有在规定时间内收到响应**，可能由服务性能、网络延迟、资源瓶颈等原因引起。通过检查后端服务、Ingress 配置、网络、资源使用等可以帮助定位并解决问题。

| **原因**                       | **描述**                                        | **排查方法**                                                |
| ------------------------------ | ----------------------------------------------- | ----------------------------------------------------------- |
| **后端服务超时**               | 后端服务处理请求变慢，未能及时响应              | 查看后端服务日志，分析是否存在性能瓶颈                      |
| **Ingress 控制器超时**         | Ingress 控制器等待后端响应超时                  | 检查 Ingress 配置的 `timeout` 设置，查看控制器日志          |
| **网络拥塞或延迟**             | 集群内部网络问题导致请求未及时传递              | 检查网络插件（如 Calico、Flannel），检查网络延迟或丢包      |
| **负载均衡问题**               | 外部负载均衡器配置或健康检查问题                | 查看负载均衡器配置和日志，确保健康检查正常                  |
| **资源耗尽**                   | 后端服务或集群资源（CPU、内存）耗尽             | 查看集群资源使用情况（`kubectl top`），确认是否存在资源瓶颈 |
| **Pod 资源限制或健康检查失败** | Pod 资源限制过低或健康检查失败，导致 Pod 未响应 | 检查 Pod 的资源请求与限制，查看健康检查配置                 |
| **数据库或外部依赖慢**         | 后端服务依赖的数据库或外部服务响应慢            | 检查数据库性能或外部依赖的响应状态                          |

---

