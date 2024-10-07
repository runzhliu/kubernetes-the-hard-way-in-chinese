# 引导 etcd 集群

Kubernetes 组件是无状态的，并将集群状态存储在 [etcd](https://github.com/etcd-io/etcd) 中。在本实验中，你将引导一个三节点的 etcd 集群，并将其配置为高可用和安全的远程访问。

## 先决条件

将 `etcd` 二进制文件和 systemd 单元文件复制到 `server` 实例：

```bash
scp \
  downloads/etcd-v3.4.27-linux-arm64.tar.gz \
  units/etcd.service \
  root@server:~/
```

本实验中的命令必须在 `server` 机器上运行。使用 `ssh` 命令登录到 `server` 机器。例如：

```bash
ssh root@server
```

## 引导 etcd 集群

### 安装 etcd 二进制文件

提取并安装 `etcd` 服务器和 `etcdctl` 命令行工具：

```bash
{
  tar -xvf etcd-v3.4.27-linux-arm64.tar.gz
  mv etcd-v3.4.27-linux-arm64/etcd* /usr/local/bin/
}
```

### 配置 etcd 服务器

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}
```

每个 etcd 成员在 etcd 集群中必须有一个唯一的名称。将 etcd 名称设置为当前计算实例的主机名：

创建 `etcd.service` systemd 单元文件：

```bash
mv etcd.service /etc/systemd/system/
```

### 启动 etcd 服务器

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## 验证

列出 etcd 集群成员：

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

下一步：[引导 Kubernetes 控制平面](08-bootstrapping-kubernetes-controllers.md)