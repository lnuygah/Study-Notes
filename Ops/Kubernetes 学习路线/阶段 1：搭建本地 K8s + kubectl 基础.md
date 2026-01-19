# 阶段 1：本地搭建 K8s + kubectl 基础（Mac M1 跟练）

### 你会学到的目标知识点（阶段 1）

1. **[kubectl 是什么](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/?utm_source=chatgpt.com)**：K8s 的命令行客户端，通过 API Server 管理集群（你后面所有练习都靠它）。
2. **本地集群怎么来**：用 [`kind`](https://kind.sigs.k8s.io/docs/user/quick-start/?utm_source=chatgpt.com)（Kubernetes in Docker）或 [`minikube`](https://minikube.sigs.k8s.io/docs/start/?utm_source=chatgpt.com) 在本机启动一个 K8s。
3. **kubeconfig / context**：kubectl 通过 kubeconfig 记录“连哪个集群、用什么身份”。（多集群切换的关键）
4. **最基础资源可视化**：`nodes / namespaces / pods` 的查看、创建、删除
5. **最小“动手闭环”**：启动一个 Pod → 查看 → 进 Pod/看日志（简单）→ 删除

阶段 1 的核心：你要能 **在本机启动一个 K8s 集群**，并用 **kubectl** 完成最小闭环：
 **连上集群 → 看 nodes/ns/pods → 创建 Pod → 查看/describe → port-forward 访问 → 删除清理**。

------

## 1.0 目标知识点 1：kubectl 是什么？（你后面所有操作都靠它）

### 概念解释

- **kubectl** 是 Kubernetes 的命令行客户端。
- 它通过 **Kubernetes API Server** 对集群进行操作（创建/查看/更新/删除资源）。
- 你写 YAML、做 apply、排查 events/logs，都是用 kubectl 触发。

### 跟练：安装 + 验证（Mac M1）

```bash
brew install kubectl
kubectl version --client
```

### 验证点（你应看到）

- 输出中能看到 `Client Version: v...`

![8](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/8.png)

## 1.1 目标知识点 2：minikube 是什么？它如何在本机“造”出一个 K8s

### 概念解释

- [**minikube**](https://minikube.sigs.k8s.io/docs/start/?utm_source=chatgpt.com&arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download) 用来在本地快速启动一个 Kubernetes，用于学习与开发，核心目标就是“一条命令起集群”。 
- [minikube 支持多种运行方式（driver）：可以跑在 **VM / 容器 / 裸机** 等不同环境里。](https://minikube.sigs.k8s.io/docs/drivers/?utm_source=chatgpt.com) 
- 在 **Mac M1** 上，最省事的是用 **Docker driver**（你的 Docker 环境已在阶段 0 搭好）。

------

## 1.2 Step 1：安装 minikube（Mac M1）

### 跟练：安装与版本检查

```bash
brew install minikube
minikube version
```

### 验证点

- 输出里能看到 minikube 的版本信息

![9](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/9.png)

------

## 1.3 Step 2：用 Docker driver 启动集群（Mac M1 推荐）

> minikube 官方“Start”页面强调：有 Docker（或 VM 环境）即可 `minikube start`。 

### 跟练：启动（明确指定 driver）

```bash
minikube start --driver=docker

# 如遇到拉取镜像慢可以手动进行docker pull 
docker pull kicbase/stable:v0.0.48
# 然后在启动指定已存在镜像
minikube start --driver=docker --base-image=kicbase/stable:v0.0.48 --delete-on-failure
```

### 跟练：确认集群状态

```bash
minikube status
kubectl cluster-info
kubectl get nodes -o wide
```

### 验证点（你应看到）

- `minikube status` 显示 Running/Configured
- `kubectl get nodes` 至少 1 个节点，且 `STATUS=Ready`

![10](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/10.png)

![11](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/11.png)

---

## 1.4 目标知识点 3：kubeconfig / context（kubectl 为什么知道连哪个集群）

### 概念解释

- kubectl 会读取 `~/.kube/config`（kubeconfig）
- 其中 **context** 把 “cluster + user + 默认 namespace” 组合成一个“当前工作环境”
- 以后你可能同时有 minikube / 云集群 / 测试集群，切换就靠 context

### 跟练：查看当前 context 与列表

```bash
kubectl config get-contexts
kubectl config current-context
```

### 验证点

- 能看到 `minikube` 作为一个 context（通常当前就是它）

![12](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/12.png)

---

## 1.5 目标知识点 4：Namespace（资源的逻辑隔离）

### 概念解释

- **Namespace** 用于隔离资源（dev/staging/prod 常用）
- 很多对象（Pod/Deployment/Service）都是 namespace 级资源
- Node 是集群级资源（不属于 namespace）

### 跟练：查看系统默认 namespace

```bash
kubectl get ns
```

### 验证点

- 能看到 `default`、`kube-system` 等

![13](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/13.png)

------

## 1.6 Step 3：创建练习 namespace，并设为默认（强烈推荐）

### 跟练：创建 `learn` namespace

```bash
kubectl create ns learn
kubectl get ns
```

### 跟练：把当前 context 默认 namespace 设为 `learn`

```bash
kubectl config set-context --current --namespace=learn
kubectl config view --minify | grep namespace
```

### 验证点

- `grep namespace` 显示 `namespace: learn`
   （之后练习就不用总加 `-n learn`）

![14](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/14.png)

------

## 1.7 目标知识点 5：Pod（阶段 1 只学最小运行单元的基本操作）

### 概念解释

- **Pod** 是 K8s 里承载容器的最小调度单位
- 阶段 1 只要求：创建、查看、describe、删除（排障先从 Events 开始）

------

## 1.8 Step 4：创建一个 Pod（nginx）→ 查看 → describe 看 Events

### 跟练：创建 Pod（命令式）

```bash
kubectl run web --image=nginx:1.27 --port=80
```

### 跟练：查看 Pod 状态

```bash
kubectl get pods -o wide
```

### 跟练：describe（必须练）

```bash
kubectl describe pod web
```

### 验证点

- Pod 最终为 `Running`
- `describe` 里能看到：
  - Pod IP / Node
  - 镜像信息
  - **Events**（后续排障最关键入口）

![15](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/15.png)

------

## 1.9 目标知识点 6：Label（K8s 的“分组标签”，后面 Service/Deployment 都靠它）

### 概念解释

- Label 是 `key=value`（如 `app=demo`）
- 你可以用 `-l` 按标签过滤资源
- 后面 Service 用 selector 选后端 Pod，本质也是 label 匹配

### 跟练：创建带 label 的 Pod + selector 过滤

```bash
kubectl run api --image=nginx:1.27 --labels=app=demo,tier=backend
kubectl run front --image=nginx:1.27 --labels=app=demo,tier=frontend

kubectl get pods -l app=demo
kubectl get pods -l tier=backend
```

### 验证点

- `-l app=demo` 能一次看到 `api` 和 `front`
- `-l tier=backend` 只看到 `api`

![17](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/17.png)

------

## 1.10 目标知识点 7：声明式 YAML（从 run 过渡到 apply，这是 K8s 正统用法）

### 概念解释

- `kubectl run` 是命令式：快，但不适合长期维护
- YAML + `kubectl apply` 是声明式：可版本管理、可复用、可审计（生产更常用）

### 跟练：写最小 Pod YAML 并 apply

#### 1）创建 `pod.yaml`

```yaml
cat > pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  labels:
    app: demo-yaml
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
EOF
```

#### 2）apply 并查看

```
kubectl apply -f pod.yaml
kubectl get pod demo-pod -o wide
kubectl describe pod demo-pod
```

### 验证点

- `demo-pod` 为 `Running`
- `describe` Events 无明显异常

![18](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/18.png)

------

## 1.11（强烈推荐）目标知识点 8：port-forward（还没学 Service/Ingress 前的最快访问方式）

### 概念解释

- `port-forward` 可以把 Pod 端口临时映射到本机端口，便于调试
- 阶段 1 用它完成“我真的访问到了 Pod”这个关键体验

### 跟练：port-forward 并 curl 访问

#### 1）开启端口转发（保持该终端不退出）

```
kubectl port-forward pod/demo-pod 8080:80
```

#### 2）新开一个终端验证

```
curl -I http://localhost:8080
```

### 验证点

- `curl -I` 返回 `HTTP/1.1 200 OK`
- port-forward 终端会输出转发日志
- `Ctrl+C` 停止 port-forward

![19](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/19.png)

------

## 1.12 Step 5：清理练习资源（保持环境干净）

### 跟练：删除命令式创建的 Pod

```
kubectl delete pod web api front
```

### 跟练：删除 YAML 创建的 Pod

```
kubectl delete -f pod.yaml
```

### 验证点

```
kubectl get pods
```

- 显示 `No resources found`

![19](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/19.png)

------

## 1.13 常见坑（Mac M1 + Docker driver）

### 坑 1：`Cannot connect to the Docker daemon`

**原因**：Docker 没启动（Docker Desktop 或 Colima 未运行）
 **处理**：先启动 Docker，再执行：

```
minikube start --driver=docker
```

### 坑 2：`kubectl` 连的不是 minikube

**处理**：

```
kubectl config get-contexts
kubectl config use-context minikube
kubectl cluster-info
```

### 坑 3：Pod 一直 Pending

**处理**：先看 Events（最关键）

```
kubectl describe pod <pod名>
```

------

## 1.14 阶段 1 最终验收清单（你做到这些就算过关）

-  `minikube start --driver=docker` 成功，节点 Ready 
-  会看 context：`get-contexts` / `current-context`
-  会 namespace：创建 `learn` 并设为默认
-  会 Pod 基本操作：run / get / describe / delete
-  会 label：创建带 label 的 Pod + `-l` 过滤
-  会 YAML：apply / get / delete
-  会 port-forward：本机 curl 访问 Pod

------

## 1.15（可选）彻底清理 minikube 集群

### 停止（下次可继续用）

```
minikube stop
```

### 删除（完全重来）

```
minikube delete
```

------

如果你阶段 1 跑通了，我可以继续按同样格式展开 **阶段 2**，并且只用 minikube：
 我会给你 4 个“故意制造的故障案例”（ImagePullBackOff / CrashLoopBackOff / 端口不通 / readiness 不通过），让你把 `describe → Events → logs` 的排障套路练到肌肉记忆。





## 1.8步骤补充：

如遇到节点内docker拉取镜像失败需要更换镜像源处理方式如下：让 minikube 节点里的 Docker daemon 使用 DaoCloud 镜像加速（根治版）

> 目标：在 minikube 节点内把 `/etc/docker/daemon.json` 增加 `registry-mirrors: ["https://docker.m.daocloud.io"]`，并重启 docker 服务。

------

### 0 先清理掉当前失败的 Pod（避免 BackOff 干扰）

```bash
kubectl delete pod web
```

------

### 1 备份节点上的 daemon.json（强烈建议）

```bash
minikube ssh -- 'sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.bak && echo "backup ok"'
```

验证备份存在：

```bash
minikube ssh -- 'ls -l /etc/docker/daemon.json*'
```

------

### 2 写入“带 mirror 的 daemon.json”（保留你原来的配置）

当前节点的 daemon.json 是：

```json
{
	"exec-opts": ["native.cgroupdriver=cgroupfs"],
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "100m"
	},
	"storage-driver": "overlay2"
}
```

我们在这个基础上加上 `registry-mirrors`：

```bash
minikube ssh -- "sudo sh -c 'cat > /etc/docker/daemon.json << EOF
{
  \"exec-opts\": [\"native.cgroupdriver=cgroupfs\"],
  \"log-driver\": \"json-file\",
  \"log-opts\": {\"max-size\": \"100m\"},
  \"storage-driver\": \"overlay2\",
  \"registry-mirrors\": [\"https://docker.m.daocloud.io\"]
}
EOF'"
```

校验文件内容：

```bash
minikube ssh -- 'cat /etc/docker/daemon.json'
```

------

### 3 重启 minikube 节点里的 Docker 服务

优先试 systemd（大多数情况 ok）：

```bash
minikube ssh -- 'sudo systemctl restart docker'
```

如果提示没有 systemctl，则用备用：

```bash
minikube ssh -- 'sudo service docker restart || sudo rc-service docker restart || true'
```

然后确认 docker 进程正常：

```bash
minikube ssh -- 'docker info | sed -n "/Registry Mirrors:/,/Live Restore Enabled/p"'
```

✅ 应看到 `Registry Mirrors:` 下出现 `https://docker.m.daocloud.io`

------

### 4 在节点内直接验证拉取（最关键一步）

```bash
minikube ssh -- 'docker pull nginx:1.27'
```

如果这一步成功，说明节点 docker daemon 已经稳定可拉。

------

### 5 回到 K8s：重新创建 Pod

```bash
kubectl run web --image=nginx:1.27 --port=80
kubectl get pods -o wide
kubectl describe pod web | tail -n 30
```

✅ 预期：Pod 很快变 `Running`，Events 不再出现 EOF。

### 回滚（如果你想恢复原状）

```bash
minikube ssh -- 'sudo mv /etc/docker/daemon.json.bak /etc/docker/daemon.json && sudo systemctl restart docker'
```

------

## ![16](/Users/chensibin/workplace/company/dotnetcode/GitHub/Study-Notes/Ops/Kubernetes 学习路线/images/16.png)