# 引导 Kubernetes 控制平面

在本实验中，你将引导 Kubernetes 控制平面。将在控制机器上安装以下组件：Kubernetes API 服务器、调度器和控制器管理器。

## 先决条件

将 Kubernetes 二进制文件和 systemd 单元文件复制到 `server` 实例：

```bash
scp \
  downloads/kube-apiserver \
  downloads/kube-controller-manager \
  downloads/kube-scheduler \
  downloads/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

本实验中的命令必须在控制实例 `server` 上运行。使用 `ssh` 命令登录到控制实例。例如：

```bash
ssh root@server
```

## 配置 Kubernetes 控制平面

创建 Kubernetes 配置目录：

```bash
mkdir -p /etc/kubernetes/config
```

### 安装 Kubernetes 控制器二进制文件

安装 Kubernetes 二进制文件：

```bash
{
  chmod +x kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl
    
  mv kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl \
    /usr/local/bin/
}
```

### 配置 Kubernetes API 服务器

```bash
{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
    kube-api-server.key kube-api-server.crt \
    service-accounts.key service-accounts.crt \
    encryption-config.yaml \
    /var/lib/kubernetes/
}
```

创建 `kube-apiserver.service` systemd 单元文件：

```bash
mv kube-apiserver.service \
  /etc/systemd/system/kube-apiserver.service
```

### 配置 Kubernetes 控制器管理器

将 `kube-controller-manager` kubeconfig 移动到适当位置：

```bash
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

创建 `kube-controller-manager.service` systemd 单元文件：

```bash
mv kube-controller-manager.service /etc/systemd/system/
```

### 配置 Kubernetes 调度器

将 `kube-scheduler` kubeconfig 移动到适当位置：

```bash
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

创建 `kube-scheduler.yaml` 配置文件：

```bash
mv kube-scheduler.yaml /etc/kubernetes/config/
```

创建 `kube-scheduler.service` systemd 单元文件：

```bash
mv kube-scheduler.service /etc/systemd/system/
```

### 启动控制器服务

```bash
{
  systemctl daemon-reload
  
  systemctl enable kube-apiserver \
    kube-controller-manager kube-scheduler
    
  systemctl start kube-apiserver \
    kube-controller-manager kube-scheduler
}
```

> 允许 Kubernetes API 服务器最多 10 秒的时间来完全初始化。

### 验证

```bash
kubectl cluster-info \
  --kubeconfig admin.kubeconfig
```

```text
Kubernetes control plane is running at https://127.0.0.1:6443
```

## Kubelet 授权的 RBAC

在本节中，你将配置 RBAC 权限，以允许 Kubernetes API 服务器访问每个工作节点上的 Kubelet API。访问 Kubelet API 是获取度量、日志和在 Pod 中执行命令所必需的。

> 本教程将 Kubelet 的 `--authorization-mode` 标志设置为 `Webhook`。Webhook 模式使用 [SubjectAccessReview](https://kubernetes.io/docs/admin/authorization/#checking-api-access) API 来确定授权。

本节中的命令将影响整个集群，只需在控制节点上运行。

```bash
ssh root@server
```

创建 `system:kube-apiserver-to-kubelet` [ClusterRole](https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole)，赋予访问 Kubelet API 和执行与管理 Pod 相关的大多数常见任务的权限：

```bash
kubectl apply -f kube-apiserver-to-kubelet.yaml \
  --kubeconfig admin.kubeconfig
```

### 验证

此时，Kubernetes 控制平面已启动并运行。从 `jumpbox` 机器运行以下命令以验证其工作情况：

对 Kubernetes 版本信息进行 HTTP 请求：

```bash
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "28",
  "gitVersion": "v1.28.3",
  "gitCommit": "a8a1abc25cad87333840cd7d54be2efaf31a3177",
  "gitTreeState": "clean",
  "buildDate": "2023-10-18T11:33:18Z",
  "goVersion": "go1.20.10",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

下一步：[引导 Kubernetes 工作节点](09-bootstrapping-kubernetes-workers.md)