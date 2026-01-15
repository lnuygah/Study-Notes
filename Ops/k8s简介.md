# **Kubernetes（K8s）**

> **一句话**：Kubernetes（K8s）是一个**容器编排平台**，把“部署、扩缩容、自愈、发布、网络、存储、权限、可观测”等分布式系统的通用能力，变成**声明式配置 + 自动化控制**。

------



## **1. K8s 解决什么问题？（为什么需要它）**

当系统从“一个服务、几台机器”发展到“几十/上百服务、频繁发布、多环境、多团队协作”时，常见痛点：

- **部署复杂**：手工上机、写脚本、逐台更新，容易出错
- **服务不稳定**：实例挂了靠人发现，人肉重启
- **扩缩容困难**：流量突增时扩容慢，流量回落时浪费资源
- **发布风险高**：滚动升级、回滚、灰度发布都要大量自研
- **网络治理困难**：服务之间互相访问、负载均衡、域名路由配置复杂
- **配置与密钥混乱**：配置散落、改配置要重打镜像、密钥管理不统一
- **资源浪费/争抢**：CPU/内存无法约束，互相影响（噪声邻居）
- **可观测缺失**：出问题时不知道谁挂了、为什么挂、怎么复现

K8s 的价值：把这些“平台能力”沉淀为**统一的控制面（Control Plane）与标准 API**，你只需要描述“我要什么”，它负责让系统“变成那样”。

------



## **2. K8s 的核心思想（必须理解）**

### **2.1 声明式（Desired State）**

你不是写脚本告诉它“一步步怎么做”，而是描述期望：

- 我希望 web 服务跑 **3** 个副本
- 每个实例限制 **CPU/内存**
- 暴露 **80** 端口并提供稳定访问
- 发生故障要自动拉起
- 要能滚动升级、失败回滚

K8s 会持续对比：

- **期望状态（Desired）**
- **实际状态（Current）**

不一致就自动修正。

### **2.2 控制器模型（Controller + Reconcile Loop）**

K8s 的核心运行方式是“调谐循环”：

- 例如 Deployment 控制器发现：期望 3 个 Pod，但实际只有 2 个

  → 自动创建 1 个 Pod

- 发现某个 Pod 不健康

  → 删除并重建

- 发现版本变化（镜像 tag 变了）

  → 按策略滚动升级

你可以把控制器理解为“**一直在巡逻的机器人**”。

### **2.3 不可变基础设施（Immutable Infrastructure）**

最佳实践倾向：

- 镜像尽量不可变（版本化）
- 配置与密钥从镜像中剥离（ConfigMap/Secret）
- 需要数据就挂存储（PVC）
- 出问题尽量“替换实例”而非“修机器”

------



## **3. K8s 架构总览（组件与职责）**

### **3.1 集群角色：Control Plane vs Worker Node**

- **Control Plane（控制面）**：负责“决策与管理”
- **Worker Node（工作节点）**：负责“实际运行容器”

> 类比：控制面像“指挥中心”，工作节点像“工人和机器”。

------



## **4. Control Plane 关键组件（理解其作用）**

### **4.1 kube-apiserver（API 服务器）**

- 集群所有操作的入口（kubectl / controller / scheduler 都找它）
- 做认证、鉴权、准入控制（Admission）
- 将对象写入 etcd

### **4.2 etcd（集群数据库）**

- 保存整个集群的“真相”：对象状态（Deployment、Pod、Service…）
- 高可用场景必须保护好 etcd（备份、性能、稳定性）

### **4.3 kube-scheduler（调度器）**

- 决定“一个 Pod 应该被放到哪个 Node 上”
- 考虑因素：资源余量、亲和/反亲和、污点/容忍、拓扑分布等

### **4.4 kube-controller-manager（控制器管理器）**

- 运行各种控制器：Deployment/ReplicaSet/Node/Job 等
- 负责让“实际状态”收敛到“期望状态”

### **4.5 cloud-controller-manager（云控制器，可选）**

- 云厂商相关：LoadBalancer、路由、磁盘等资源对接（AWS/GCP/Azure）

------



## **5. Worker Node 关键组件（容器真正跑在这里）**

### **5.1 kubelet**

- 每个 Node 上的代理
- 负责拉镜像、创建容器、上报 Pod 状态、执行探针

### **5.2 container runtime（容器运行时）**

- 例如 containerd、CRI-O
- 负责真正启动/停止容器

### **5.3 kube-proxy（网络代理）**

- 实现 Service 的负载均衡与转发规则（iptables/ipvs 等）
- 让 Service 能把流量“稳定地”分发到后端 Pod

------



## **6. 核心对象（从“最小单位”到“完整服务”）**

> 建议学习顺序：**Pod → Deployment → Service → Ingress → Config/Secret → Storage → AutoScaling**

### **6.1 Pod（最小调度单位）**

- Pod 是 K8s 调度和管理的最小单元

- 一个 Pod 里可以有多个容器（通常 1 个）

- 同 Pod 容器共享：

  

  - 网络命名空间（同一个 IP、localhost 可互通）
  - 存储卷（Volume）

  

- Pod 是“易变的”：随时可能被重建，**IP 会变**

**常见模式**

- Sidecar（旁车）：日志收集、代理等
- InitContainer：启动前先做初始化任务（迁移、生成配置、检查依赖）

------



### **6.2 Deployment / ReplicaSet（管理一组 Pod）**

- **Deployment**：描述应用部署方式与升级策略
- **ReplicaSet**：确保副本数满足（Deployment 背后使用 RS）

你通常只直接写 Deployment，不直接写 RS。

**Deployment 能力**

- 滚动升级（RollingUpdate）
- 回滚到历史版本（rollback）
- 扩缩容（调整 replicas）

------



### **6.3 Service（稳定入口 + 负载均衡）**

Pod 会重建、IP 会变，所以不能依赖 Pod IP。

Service 提供：

- 稳定的虚拟 IP（ClusterIP）
- 稳定的 DNS 名称（例如 api.default.svc.cluster.local）
- 将流量负载均衡到后端 Pod（通过 label selector 找 Pod）

**常见类型**

- ClusterIP：仅集群内访问（默认）
- NodePort：节点端口暴露（测试常用）
- LoadBalancer：云上创建负载均衡器（生产常用）

------



### **6.4 Ingress（HTTP/HTTPS 入口网关）**

Ingress 是“规则对象”，真正处理流量的是 **Ingress Controller**（例如 Nginx/Traefik）。

Ingress 能做：

- 域名路由：api.example.com → api-service
- 路径路由：/api → api-service，/web → web-service
- TLS 证书终止（HTTPS）

> Service 更偏“4 层”（TCP/UDP）稳定入口，Ingress 更偏“7 层”（HTTP/HTTPS）网关路由。

------



### **6.5 Namespace / Label / Selector（组织与治理的基础）**

- **Namespace**：逻辑隔离（dev/staging/prod）
- **Label**：给对象打标签（如 app=order tier=backend）
- **Selector**：按标签选择对象（Service 找 Pod、Deployment 管 Pod）

------



## **7. 配置与密钥（让镜像与环境解耦）**

### **7.1 ConfigMap（非敏感配置）**

- 环境变量注入：ENV=prod
- 文件挂载：生成配置文件供应用读取

### **7.2 Secret（敏感信息）**

- 数据库密码、API key 等

- 注意：默认仅 base64 编码，不等于加密

  生产应结合 KMS/加密 at-rest/外部密钥管理方案

------



## **8. 存储（让数据不随 Pod 消失）**

### **8.1 为什么需要 PV/PVC？**

- Pod 重建是常态
- 有状态服务（DB、队列、文件）需要持久化

### **8.2 PV / PVC / StorageClass**

- **PVC**：用户声明“我需要多大、什么性能的盘”
- **PV**：实际的存储资源
- **StorageClass**：动态供给策略（云盘、Ceph、NFS 等）

> 类比：PVC 像“申请单”，PV 像“实物硬盘”，StorageClass 像“采购/发放规则”。

------



## **9. 资源管理与调度（让集群“可控”）**

### **9.1 Requests / Limits**

- requests：调度器用来判断“要预留多少资源”
- limits：运行时硬限制，超了会被限制或 OOMKill

最佳实践：

- 生产一定要设置 requests/limits，防止互相抢资源

### **9.2 调度进阶（了解即可）**

- nodeSelector：简单指定节点标签
- affinity/anti-affinity：让服务分散/聚合
- taints/tolerations：让某些节点只跑特定工作负载

------



## **10. 自动扩缩容（让系统随流量变化）**

### **10.1 HPA（Horizontal Pod Autoscaler）**

- 根据 CPU/内存/自定义指标自动调节副本数
- 依赖 metrics（如 metrics-server / Prometheus adapter）

### **10.2 其他扩缩**

- VPA：自动调节 requests/limits（更进阶）
- Cluster Autoscaler：自动增减节点（云上常用）

------



## **11. 健康检查（决定“是否接流量、是否重启”）**

### **11.1 readinessProbe（就绪探针）**

- 未就绪：Service 不会把流量打给它
- 常用于：启动慢、依赖未准备好（DB 未连上）

### **11.2 livenessProbe（存活探针）**

- 探测失败：容器会被重启
- 常用于：死锁、假死（进程还在但不工作）

> 一句话：readiness 决定“接不接流量”，liveness 决定“重不重启”。

------



## **12. 发布策略（上线的关键）**

### **12.1 Rolling Update（滚动发布）**

- 逐步替换旧 Pod 为新 Pod
- 配合 readinessProbe 能保证无损发布

### **12.2 回滚（Rollback）**

- 新版本出问题，快速回退到历史 revision

### **12.3 灰度/金丝雀（Canary）**

- 一部分流量到新版本
- 常通过 Ingress/Service Mesh 实现更细粒度控制

------



## **13. 安全体系（生产必须重视）**

### **13.1 RBAC（权限控制）**

- Role/ClusterRole：权限集合
- RoleBinding/ClusterRoleBinding：把权限赋给用户/服务账号
- ServiceAccount：Pod 运行时身份

### **13.2 NetworkPolicy（网络隔离）**

- 默认集群可能“全互通”
- NetworkPolicy 可以限制命名空间/服务之间的访问（需要 CNI 支持）

### **13.3 Pod Security（安全基线）**

- 限制特权容器、root 运行、宿主机挂载等
- 生产建议启用更严格的策略

------



## **14. 可观测性（会用才能稳定）**

### **14.1 日志**

- kubectl logs 查看容器日志
- 多容器 Pod 要指定 -c

### **14.2 事件（Events）**

- kubectl describe 看 Events 是排障第一入口

  ImagePullBackOff、CrashLoopBackOff、FailedScheduling 都在这里解释得很清楚

### **14.3 指标**

- metrics-server：HPA、基础资源指标
- Prometheus/Grafana：更完整的监控体系

------



## **15. 最常见的“运行链路”（从 YAML 到跑起来）**

1. 你 kubectl apply 创建 Deployment
2. API Server 写入 etcd
3. Deployment Controller 创建 ReplicaSet
4. ReplicaSet 创建 Pod
5. Scheduler 给 Pod 选 Node
6. Node 上 kubelet 拉镜像并启动容器
7. readinessProbe 通过后，Service 才开始转发流量
8. 如果 Pod 挂了，控制器发现偏离期望状态 → 重建

------



## **16. 常见故障与排查思路（强烈建议记住）**

**固定排查顺序：**

1. kubectl get pods 看状态（Pending / CrashLoopBackOff / ImagePullBackOff）
2. kubectl describe pod xxx 看 **Events**
3. kubectl logs xxx 看日志
4. 若网络问题：检查 Service selector、Endpoints、端口一致性、Ingress 规则
5. 若调度问题：看 requests/limits、节点资源、taints/tolerations
6. 若启动慢：readiness/liveness、初始延迟、超时参数

------



## **17. 你学到什么程度算“会 K8s”？**

达到以下能力基本就能应对大部分业务场景：

- 能写 Deployment/Service/Ingress 把服务跑起来
- 能用 ConfigMap/Secret 管配置
- 能设置 requests/limits + HPA
- 能用日志 + events 快速定位问题
- 理解 Pod 易变、Service 稳定入口、Ingress 网关路由、PVC 持久化这些核心模型