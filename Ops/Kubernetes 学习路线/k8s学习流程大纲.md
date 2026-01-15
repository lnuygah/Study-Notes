# Kubernetes 学习路线（阶段验收 + YAML 模板 + Case）

> 建议用本地 **kind** 或 **minikube** 练习。全程只需要 `kubectl` + 一个集群环境。
>  每个阶段：**目标 → 步骤 → 验收 → Case → YAML 模板**（能直接套用）。

------

## 阶段 0：前置基础（Docker + 基础网络）【1–3 天】

### 目标

- 会做镜像：Dockerfile / build / run
- 理解端口、DNS、反向代理的基本概念

### 步骤

- 写一个简单 Web（任意语言）+ Dockerfile
- `docker build`，`docker run -p 8080:8080`
- 修改代码 → 重新打包 → 跑起新版本

### 阶段验收

- 能独立写 Dockerfile 并跑一个可访问的服务
- 能解释：镜像 vs 容器、端口映射、环境变量注入

### Case

- “Hello API”：返回 `{version:"v1"}`，改成 v2 后重新打镜像验证升级

------

## 阶段 1：搭建本地 K8s + kubectl 基础【0.5–1 天】

### 目标

- 有一个可用集群
- 会基本查看资源：node/pod/ns

### 步骤

- 安装 kind/minikube
- 安装 kubectl
- `kubectl get nodes`
- `kubectl get ns`

### 阶段验收

- `kubectl get nodes` 输出 Ready
- 能切换 namespace：`-n` 参数

### Case

- 在 `default` 里创建一个 `nginx` Pod 并删除它

### YAML 模板：Pod（最小可运行）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  labels:
    app: demo
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
```

> 常用命令

```bash
kubectl apply -f pod.yaml
kubectl get pod -o wide
kubectl describe pod demo-pod
kubectl delete pod demo-pod
```

------

## 阶段 2：YAML 基础 + kubectl 排障套路【2–4 天】

### 目标

- 会写/读 YAML
- 掌握排障三板斧：get / describe / logs

### 步骤

- 故意制造错误（镜像不存在、端口不对）
- 学会从 Events 找原因

### 阶段验收

- 看到 `ImagePullBackOff` 能定位为镜像/仓库问题
- 看到 `CrashLoopBackOff` 能用 logs 定位启动失败原因

### Case

- 写一个 Pod 用不存在镜像 `nginx:badtag`，观察 Events 并修正

### YAML 模板：错误示例（用于练习）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-image-pod
spec:
  containers:
    - name: web
      image: nginx:badtag
```

------

## 阶段 3：Deployment（发布与副本管理）【3–5 天】

### 目标

- 理解 Pod 易变、Deployment 管副本
- 会滚动升级、回滚、扩缩容

### 步骤

- 创建 Deployment（replicas=3）
- 修改镜像 tag 触发滚动更新
- 回滚到旧版本
- 改 replicas 做扩缩容

### 阶段验收

- `kubectl rollout status deploy/...` 能看到升级过程
- 能解释 ReplicaSet 的作用（Deployment 背后）
- 能回滚：`kubectl rollout undo`

### Case

- 部署 `nginx:1.26` → 升级到 `nginx:1.27` → 回滚到 1.26

### YAML 模板：Deployment（可直接套用）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deploy
  labels:
    app: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "300m"
              memory: "256Mi"
```

> 常用命令

```bash
kubectl apply -f deploy.yaml
kubectl get deploy,rs,pod -l app=demo
kubectl set image deploy/demo-deploy web=nginx:1.26
kubectl rollout status deploy/demo-deploy
kubectl rollout undo deploy/demo-deploy
kubectl scale deploy/demo-deploy --replicas=5
```

------

## 阶段 4：Service（稳定访问入口）【2–3 天】

### 目标

- 理解：Pod IP 会变，Service 提供稳定访问 + 负载均衡
- 会用 ClusterIP / NodePort（本地练习）

### 步骤

- 给上阶段的 Deployment 建 Service
- 在集群内访问 Service（DNS）
- 用 NodePort 暴露到本机访问（本地环境常用）

### 阶段验收

- 删除某些 Pod 后，Service 仍可访问（自动切换到其他 Pod）
- 能解释 selector 与 endpoints 的关系

### Case

- NodePort 暴露 nginx，在浏览器访问 `http://localhost:<nodePort>`

### YAML 模板：Service（ClusterIP）

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-svc
spec:
  selector:
    app: demo
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
```

### YAML 模板：Service（NodePort，用于本地访问）

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-svc-nodeport
spec:
  selector:
    app: demo
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

------

## 阶段 5：Ingress（HTTP/HTTPS 网关路由）【2–4 天】

### 目标

- 会用域名/路径把流量路由到不同 Service
- 理解：Ingress 需要 Ingress Controller（Nginx/Traefik）

### 步骤

- 安装 Ingress Controller（kind/minikube 都有官方方式）
- 创建两个服务：`web` 与 `api`（都可用 nginx 模拟）
- Ingress 配置 `/web` 与 `/api` 路由

### 阶段验收

- 同一个域名：`/web`、`/api` 命中不同后端
- 能解释：Ingress 是规则对象，Controller 才是真正处理流量的组件

### Case

- `http://demo.local/web` → web-svc
- `http://demo.local/api` → api-svc

### YAML 模板：Ingress（路径路由）

> 注意：不同 Controller 的 `ingressClassName` 可能不同（如 `nginx`），本地按实际改。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ing
spec:
  ingressClassName: nginx
  rules:
    - host: demo.local
      http:
        paths:
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
```

------

## 阶段 6：ConfigMap / Secret（配置与密钥）【2–3 天】

### 目标

- 配置从镜像剥离
- 会用环境变量 / 挂载文件方式注入配置
- Secret 管理敏感信息

### 步骤

- 创建 ConfigMap：设置 `APP_ENV=dev`
- Deployment 引用 ConfigMap 注入 env
- 创建 Secret：设置 `DB_PASSWORD=...`
- 用 `envFrom` 或单项 `valueFrom` 注入

### 阶段验收

- 修改 ConfigMap 后能触发应用更新（常见做法是滚动重启）
- 能解释：Secret 默认只是 base64，不等于安全加密（生产要加强）

### Case

- 应用读取 `APP_ENV` 打印到日志；改 ConfigMap 后滚动重启验证生效

### YAML 模板：ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  APP_ENV: "dev"
  LOG_LEVEL: "info"
```

### YAML 模板：Secret（stringData 更好写）

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
type: Opaque
stringData:
  DB_PASSWORD: "p@ssw0rd"
```

### YAML 模板：Deployment 引用 ConfigMap/Secret

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-config-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-config
  template:
    metadata:
      labels:
        app: demo-config
    spec:
      containers:
        - name: app
          image: nginx:1.27
          env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: demo-config
                  key: APP_ENV
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: demo-secret
                  key: DB_PASSWORD
```

------

## 阶段 7：Probe（健康检查）【2–3 天】

### 目标

- 理解 readiness vs liveness
- 能用探针避免“未就绪却接流量”和“假死不重启”

### 步骤

- readinessProbe：启动延迟 10s 才就绪
- livenessProbe：接口挂了触发重启（练习即可）

### 阶段验收

- readiness 未通过时 Service 不把流量打进去（Endpoints 变化）
- liveness 失败会导致容器重启（restartCount 增加）

### Case

- 用 nginx 的 `/` 做健康检查；故意写错路径看 readiness 失败

### YAML 模板：readiness + liveness

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-probe
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-probe
  template:
    metadata:
      labels:
        app: demo-probe
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
            failureThreshold: 3
```

------

## 阶段 8：存储 PV/PVC（让数据不丢）【3–5 天】

### 目标

- 理解 PV/PVC/StorageClass
- 会给应用挂持久化卷

### 步骤（本地 kind 可用 hostPath 做练习）

- 创建 PVC
- Pod 挂载到某目录
- 写入文件 → 删除 Pod → 重建 → 文件仍在（同卷）

### 阶段验收

- Pod 重建后数据仍存在（PVC 绑定的 PV 未变）
- 能解释：PVC 是申请、PV 是资源、StorageClass 是供给规则

### Case

- 运行一个 busybox Pod，往 `/data/hello.txt` 写内容，重建后仍读到

### YAML 模板：PVC（通用写法，StorageClass 按环境改）

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### YAML 模板：Pod 挂载 PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pvc-pod
spec:
  containers:
    - name: bb
      image: busybox:1.36
      command: ["sh", "-c", "echo hello > /data/hello.txt; sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: demo-pvc
```

------

## 阶段 9：资源管理 + HPA 自动扩缩容【3–5 天】

### 目标

- 会设置 requests/limits
- 会用 HPA 基于 CPU 自动扩容（需要 metrics）

### 步骤

- 安装 metrics-server（本地）
- Deployment 设置 requests/limits
- 创建 HPA（min=2 max=10）
- 压测触发扩容（简单 curl loop 即可）

### 阶段验收

- CPU 上升时副本数自动增加
- CPU 下降后能缩回
- 能解释：HPA 基于 metrics，没 metrics 不会工作

### Case

- 部署一个会消耗 CPU 的 demo（或用现成镜像），压测后观察 `kubectl get hpa`

### YAML 模板：HPA（autoscaling/v2）

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: demo-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demo-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

------

## 阶段 10：RBAC（权限）入门【2–4 天】

### 目标

- 理解 ServiceAccount、Role、RoleBinding
- 让某个应用只能读某些资源（最小权限）

### 步骤

- 创建 ServiceAccount
- 创建 Role（只允许 get/list pods）
- 绑定 RoleBinding
- 用该账号访问 API（概念练习即可）

### 阶段验收

- 能解释 namespace 内 Role 与全局 ClusterRole 区别
- 能看懂报错：Forbidden（权限不足）

### Case

- 在 `default` 创建一个只读 Pod 列表权限的 ServiceAccount

### YAML 模板：RBAC（namespace 级）

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: demo-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: demo-sa-read-pods
  namespace: default
subjects:
  - kind: ServiceAccount
    name: demo-sa
    namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

------

## 阶段 11：综合实战（把“一个小系统”完整跑起来）【1–2 周】

### 目标

把前面所有关键对象串起来，形成“可上线”的最小闭环：

- Deployment + Service + Ingress
- ConfigMap/Secret
- requests/limits + HPA
- readiness/liveness
- PVC（如有状态组件）
- 基本排障流程

### 阶段验收

- 能完成一次无损滚动发布
- 能在故障时 10 分钟内定位（events + logs）
- 能解释每个对象存在的理由（不是照抄 YAML）

### Case（推荐做这套最小系统）

- `web`：前端（nginx）
- `api`：后端（先用 nginx 模拟也行）
- 路由：Ingress `/web` `/api`
- 配置：ConfigMap 注入环境
- 伸缩：HPA
- 健康：probe

------

# 附：推荐目录结构（便于管理与复用）

```tex
k8s-demo/
  00-namespace/
  01-deploy/
  02-service/
  03-ingress/
  04-config/
  05-secret/
  06-storage/
  07-hpa/
  08-rbac/
```

------

# 附：通用排障命令清单（建议背）

```bash
# 看状态
kubectl get pod -A
kubectl get deploy,svc,ing -A

# 看事件（最重要）
kubectl describe pod <pod>

# 看日志
kubectl logs <pod> -c <container> --tail=200

# 进入容器排查
kubectl exec -it <pod> -c <container> -- sh

# 看 Service 是否有后端
kubectl get endpoints <svc>

# 发布状态
kubectl rollout status deploy/<name>
kubectl rollout history deploy/<name>
kubectl rollout undo deploy/<name>
```