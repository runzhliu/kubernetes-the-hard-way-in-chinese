# 配置 kubectl 进行远程访问

在本实验中，你将生成一个用于 `kubectl` 命令行工具的 kubeconfig 文件，该文件基于 `admin` 用户凭据。

> 请在 `jumpbox` 机器上运行本实验中的命令。

## Admin Kubernetes 配置文件

每个 kubeconfig 都需要一个 Kubernetes API 服务器进行连接。

你应该能够基于之前实验中的 `/etc/hosts` DNS 条目 ping `server.kubernetes.local`。

```bash
curl -k --cacert ca.crt \
  https://server.kubernetes.local:6443/version
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

生成一个适合以 `admin` 用户身份进行身份验证的 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

运行上述命令的结果应在 `kubectl` 命令行工具的默认位置 `~/.kube/config` 中创建一个 kubeconfig 文件。这也意味着你可以在不指定配置的情况下运行 `kubectl` 命令。

## 验证

检查远程 Kubernetes 集群的版本：

```bash
kubectl version
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
```

列出远程 Kubernetes 集群中的节点：

```bash
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   30m   v1.28.3
node-1   Ready    <none>   35m   v1.28.3
```

下一步：[配置 Pod 网络路由](11-pod-network-routes.md)