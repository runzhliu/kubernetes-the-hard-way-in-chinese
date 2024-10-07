# 配置 CA 和生成 TLS 证书

在本实验中，你将使用 openssl 配置一个 [PKI 基础设施](https://en.wikipedia.org/wiki/Public_key_infrastructure)，以引导一个证书颁发机构，并为以下组件生成 TLS 证书：kube-apiserver、kube-controller-manager、kube-scheduler、kubelet 和 kube-proxy。此部分中的命令应在 `jumpbox` 上运行。

## 证书颁发机构

在本节中，你将配置一个证书颁发机构，可以用于为其他 Kubernetes 组件生成额外的 TLS 证书。使用 `openssl` 设置 CA 和生成证书可能会耗时，特别是第一次进行时。为了简化本实验，我提供了一个 openssl 配置文件 `ca.conf`，该文件定义了生成每个 Kubernetes 组件证书所需的所有详细信息。

花一点时间查看 `ca.conf` 配置文件：

```bash
cat ca.conf
```

你不需要理解 `ca.conf` 文件中的所有内容就能完成本教程，但你应该将其视为学习 `openssl` 和管理证书所需配置的起点。

每个证书颁发机构都从一个私钥和根证书开始。在本节中，我们将创建一个自签名的证书颁发机构，虽然这对于本教程来说已经足够，但这并不应该被视为在真正的生产环境中应做的事情。

生成 CA 配置文件、证书和私钥：

```bash
openssl genrsa -out ca.key 4096
openssl req -x509 -new -sha512 \
  -key ca.key -days 3653 \
  -config ca.conf \
  -out ca.crt
```

结果：

```txt
ca.crt ca.key
```

## 创建客户端和服务器证书

在本节中，你将为每个 Kubernetes 组件生成客户端和服务器证书，以及为 Kubernetes `admin` 用户生成一个客户端证书。

生成证书和私钥：

```bash
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)
```

```bash
for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"
  
  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

运行上述命令的结果将为每个 Kubernetes 组件生成私钥、证书请求和签名的 SSL 证书。你可以使用以下命令列出生成的文件：

```bash
ls -1 *.crt *.key *.csr
```

## 分发客户端和服务器证书

在本节中，你将把各种证书复制到每台机器的一个目录中，以便每个 Kubernetes 组件搜索证书对。在真实的环境中，这些证书应被视为一组敏感秘密，因为它们通常被 Kubernetes 组件用作相互认证的凭据。

将适当的证书和私钥复制到 `node-0` 和 `node-1` 机器：

```bash
for host in node-0 node-1; do
  ssh root@$host mkdir /var/lib/kubelet/
  
  scp ca.crt root@$host:/var/lib/kubelet/
    
  scp $host.crt \
    root@$host:/var/lib/kubelet/kubelet.crt
    
  scp $host.key \
    root@$host:/var/lib/kubelet/kubelet.key
done
```

将适当的证书和私钥复制到 `server` 机器：

```bash
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

> `kube-proxy`、`kube-controller-manager`、`kube-scheduler` 和 `kubelet` 客户端证书将在下一个实验中用于生成客户端认证配置文件。

下一步：[生成 Kubernetes 身份验证配置文件](05-kubernetes-configuration-files.md)