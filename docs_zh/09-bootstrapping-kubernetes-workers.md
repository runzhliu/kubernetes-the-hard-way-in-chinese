# 引导 Kubernetes 工作节点

在本实验中，你将引导两个 Kubernetes 工作节点。将安装以下组件：[runc](https://github.com/opencontainers/runc)、[容器网络插件](https://github.com/containernetworking/cni)、[containerd](https://github.com/containerd/containerd)、[kubelet](https://kubernetes.io/docs/admin/kubelet) 和 [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies)。

## 先决条件

将 Kubernetes 二进制文件和 systemd 单元文件复制到每个工作实例：

```bash
for host in node-0 node-1; do
  SUBNET=$(grep $host machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf 
    
  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml
    
  scp 10-bridge.conf kubelet-config.yaml \
  root@$host:~/
done
```

```bash
for host in node-0 node-1; do
  scp \
    downloads/runc.arm64 \
    downloads/crictl-v1.28.0-linux-arm.tar.gz \
    downloads/cni-plugins-linux-arm64-v1.3.0.tgz \
    downloads/containerd-1.7.8-linux-arm64.tar.gz \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kubelet-config.yaml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@$host:~/
done
```

本实验中的命令必须在每个工作实例 `node-0` 和 `node-1` 上运行。使用 `ssh` 命令登录到工作实例。例如：

```bash
ssh root@node-0
```

## 配置 Kubernetes 工作节点

安装操作系统依赖项：

```bash
{
  apt-get update
  apt-get -y install socat conntrack ipset
}
```

> socat 二进制文件支持 `kubectl port-forward` 命令。

### 禁用交换

默认情况下，如果启用了 [交换](https://help.ubuntu.com/community/SwapFaq)，kubelet 将无法启动。建议禁用交换，以确保 Kubernetes 可以提供适当的资源分配和服务质量。

验证是否启用了交换：

```bash
swapon --show
```

如果输出为空，则未启用交换。如果启用了交换，请立即运行以下命令禁用交换：

```bash
swapoff -a
```

> 为确保在重启后交换保持关闭，请查阅你的 Linux 发行版文档。

创建安装目录：

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

安装工作二进制文件：

```bash
{
  mkdir -p containerd
  tar -xvf crictl-v1.28.0-linux-arm.tar.gz
  tar -xvf containerd-1.7.8-linux-arm64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-arm64-v1.3.0.tgz -C /opt/cni/bin/
  mv runc.arm64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

### 配置 CNI 网络

创建 `bridge` 网络配置文件：

```bash
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

### 配置 containerd

安装 `containerd` 配置文件：

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### 配置 Kubelet

创建 `kubelet-config.yaml` 配置文件：

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### 配置 Kubernetes 代理

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### 启动工作服务

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

## 验证

本教程中创建的计算实例将没有权限完成此部分。请从 `jumpbox` 机器运行以下命令。

列出注册的 Kubernetes 节点：

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   1m     v1.28.3
node-1   Ready    <none>   10s    v1.28.3
```

下一步：[配置 kubectl 进行远程访问](10-configuring-kubectl.md)