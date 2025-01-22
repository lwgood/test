1. [OSI七层模型 vs TCP/IP四层模型](./README.md/#osi七层模型-vs-tcpip四层模型)
2. [TCP 与 UDP 对比分析](./README.md/#tcp-与-udp-对比分析)
3. [MySQL 主从架构简述](./README.md/#mysql-主从架构简述)
4. [Redis 的四种模式分析对比](./README.md/#redis-的四种模式分析对比)
5. [K8S 核心组件](./README.md/#k8s-核心组件)
6. [K8S DaemonSet、Deployment、StatefulSet 和 ReplicaSet Pod 区别](./README.md/#k8s-daemonsetdeploymentstatefulset-和-replicaset-pod-区别)
7. [K8S 流量的治理](./README.md/#k8s-流量的治理)
8. [K8S 客户端访问完整过程](./README.md/#k8s-客户端访问完整过程)
9. [K8S Pod 调度规则/限制](./README.md/#k8s-pod-调度规则限制)
10. [502 Bad Gateway 错误分析](./README.md/#502-bad-gateway-错误分析)
11. [504 Gateway Timeout 错误分析](./README.md/#504-gateway-timeout-错误分析)

---
# OSI七层模型 vs TCP/IP四层模型

| 网络模型     | **OSI 七层模型**   | **TCP/IP 四层模型**   | **对应协议**                                                   |
|--------------|--------------------|-----------------------|---------------------------------------------------------------|
| **应用层**   | 应用层             | 应用层                | HTTP, FTP, SMTP, DNS, POP3, IMAP, TELNET |
| **表示层**   | 表示层             | -                     | - |
| **会话层**   | 会话层             | -                     | - |
| **传输层**   | 传输层             | 传输层                | TCP, UDP |
| **网络层**   | 网络层             | 网络层                | IP, ICMP, ARP, RIP |
| **数据链路层** | 数据链路层         | 数据链路层                | - |
| **物理层**   | 物理层             | -                | - |

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
  1. 开启二进制日志 (`binlog`)。设置唯一的 `server ID`。
  2. 创建具有 `REPLICATION SLAVE` 权限的复制账户。
- **从库配置**：
  1. 设置唯一的 `server ID`。
  2. 启用复制功能，记录主库的日志文件和位置来启动复制。

## 2. 复制过程
  1. 主库将所有更改操作记录到二进制日志 (`binlog`)。
  2. 从库通过 `IO_THREAD` 线程读取主库的 `binlog`，将其存储在自己的中继日志（`relay log`）中。
  3. 从库通过 `SQL_THREAD` 线程执行中继日志中的 SQL 语句，实现数据同步。

## 3. 同步方式
- **异步复制**: 主库无需等待从库确认即可提交事务，延迟小，但有数据丢失的风险。
- **半同步复制**: 主库至少等待一个从库确认事务已写入日志，较为可靠，延迟较小。
- **全同步复制**: 主库等待所有从库确认事务已写入日志，确保数据一致性，但增加延迟。

## 4. 高可用与读写分离
- **主库故障恢复**：
  1. 在主库发生故障时，可以手动或自动将某个从库提升为新的主库。
  2. 主主模式，使用 `Keepalived` 通过 `VIP` 的方式来统一对外提供服务，`Keepalived` 为非抢占 `nopreempt` 模式，避免故障节点恢复后抢占 `VIP`
- **读写分离**: 主库处理写操作，从库处理读操作，减轻主库压力，提升性能。

---
# Redis 的四种模式分析对比
| 部署模式     | 高可用性 | 扩展性 | 读写分离 | 数据存储方式                               | 主要优点               | 适用场景                 |
| ------------ | -------- | ------ | -------- | ------------------------------------------ | ---------------------- | ------------------------ |
| **单机模式** | 无       | 无     | 无       | 数据存储在单个节点内                       | 简单、性能高           | 小型项目、开发测试环境   |
| **主从模式** | 较低     | 低     | 支持     | 主节点存储数据，从节点复制主节点数据       | 读写分离，降低读取压力 | 读多写少的应用场景       |
| **哨兵模式** | 较高     | 低     | 支持     | 数据存储在主从节点，主节点故障时从节点接管 | 自动故障切换，高可用性 | 中小规模、高可用场景     |
| **集群模式** | 高       | 高     | 支持     | 数据分片存储在多个节点上                   | 水平扩展，性能优越     | 大规模分布式系统、高并发 |

---
# K8S 核心组件
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
| **特性**       | **Deployment**                     | **DaemonSet**                                  | **StatefulSet**                                 | **ReplicaSet**                     |
| -------------- | ---------------------------------- | ---------------------------------------------- | ----------------------------------------------- | ---------------------------------- |
| **无状态**     | 是                                 | 是                                             | 否                                              | 是                                 |
| **副本数**     | 自动水平扩展，副本数可调整         | 每个节点一个 Pod                               | 支持水平扩展，但每个 Pod 有独特身份             | 固定副本数，自动恢复               |
| **Pod 名称**   | 随机生成，无稳定名称               | 节点上的 Pod 名称由 DaemonSet 和哈希生成       | 稳定且有序，格式为 `<statefulset-name>-<index>` | 随机生成，无稳定名称               |
| **数据持久化** | 无                                 | 无                                             | 每个 Pod 可绑定持久化存储卷                     |                                    |
| **启动顺序**   | 无顺序要求，Pod 并发启动           | 每个节点上的 Pod 同时启动                      | 按顺序启动                                      | 无顺序要求，Pod 并发启动           |
| **扩展方式**   | 水平扩展，自动调整副本数           | 节点增加时自动创建 Pod，节点减少时自动清理     | 支持有序扩展和缩容，Pod 按顺序扩展和删除        | 水平扩展，自动调整副本数           |
| **故障响应**   | 自动恢复副本数，Pod 崩溃时自动重建 | 节点故障时自动清理旧 Pod，并在新节点上创建 Pod | Pod 崩溃时按顺序重新启动，保证顺序和稳定性      | 自动恢复副本数，Pod 崩溃时自动重建 |
| **常见应用**   | 无状态 Web 应用、微服务架构        | 节点级服务（如日志收集、监控代理等）           | 有状态服务（如数据库、消息队列）                | 副本控制，通常由 Deployment 管理   |

---
# K8S 流量的治理
- **Service**: 为 Pod 提供统一的访问入口，负责将流量分发到后端的 Pod。
- **Ingress**: 管理外部 HTTP/HTTPS 流量，根据 URL 路径或主机名将流量路由到不同的服务。
- **Affinity & Anti-affinity**: 通过亲和、反亲和控制 Pod 如何在集群中调度，确保某些 Pod 必须或避免运行在特定节点上，从而优化流量分布。
- **Service Mesh**: 服务网格如 `Istio` 提供流量路由、负载均衡、限流，灵活控制和监控微服务之间的流量。
- **HPA 自动扩缩容**: 根据 CPU、内存使用或自定义指标动态调整 Pod 数量，确保集群能根据流量水平自动伸缩。

---
# K8S 客户端访问完整过程
1. 客户端请求域名
2. DNS 解析 --> CNAME 解析为负载均衡器地址
3. 负载均衡器转发流量到 Ingress 控制器
4. Ingress 控制器根据路由规则处理请求
5. Ingress 路由流量到 Service
6. Service 路由流量到 Pod
7. Pod 处理请求并生成响应

---
# K8S Pod 调度规则/限制
- **Node Affinity (节点亲和性)**: 通过 `nodeAffinity` 设置调度规则，强制或软性限制 Pod 不被调度到特定节点。
- **Taints**: 在节点上设置污点（Taint），阻止 Pod 调度到该节点。
- **PodAntiAffinity (Pod 反亲和性)**: 设置 Pod 之间的反亲和性，避免 Pod 调度到与其他特定 Pod 相同的节点。

---
# 502 Bad Gateway 错误分析
`502 Bad Gateway` 错误通常表示服务器作为网关或代理时，**收到来自上游服务器的无效响应**，可能由后端服务不可用、配置错误或网络问题导致。

1. 服务不可用，后段服务崩溃或无响应。
2. 负载均衡器配置错误，无法将流量转发到 Ingress 控制器。
3. Ingress 路由规则配置错误，无法将流量转发到 service。
4. Service 配置错误，无法将流量转发到 Pod。
5. Pod 间无法通信。
6. 集群资源不足，导致服务无法响应。

---
# 504 Gateway Timeout 错误分析
`504 Gateway Timeout` 错误通常是由于 **服务器没有在规定时间内收到响应**，可能由服务性能、网络延迟、资源瓶颈等原因引起。通过检查后端服务、Ingress 配置、网络、资源使用等可以帮助定位并解决问题。

1. 后段服务器超时，处理请求变慢，无响应
2. Ingress 控制器等待后端响应超时
3. 网络堵塞，集群内部网络问题导致请求响应超时
4. 负载均衡器配置或健康检查失败
5. 集群资源不足/耗尽
6. Pod 资源不足/耗尽，健康检查失败
7. 外部依赖服务响应问题

---
