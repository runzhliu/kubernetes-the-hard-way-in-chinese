# 生成数据加密配置和密钥

Kubernetes 存储多种数据，包括集群状态、应用配置和秘密。Kubernetes 支持对集群数据进行 [加密](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data)，以保护静态数据。

在本实验中，你将生成一个加密密钥和一个适合加密 Kubernetes 秘密的 [加密配置](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration)。

## 加密密钥

生成加密密钥：

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## 加密配置文件

创建 `encryption-config.yaml` 加密配置文件：

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

将 `encryption-config.yaml` 加密配置文件复制到每个控制实例：

```bash
scp encryption-config.yaml root@server:~/
```

下一步：[引导 etcd 集群](07-bootstrapping-etcd.md)