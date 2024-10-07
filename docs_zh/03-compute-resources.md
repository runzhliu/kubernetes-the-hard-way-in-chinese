# 配置计算资源

Kubernetes 需要一组机器来托管 Kubernetes 控制平面和运行容器的工作节点。在本实验中，你将配置设置 Kubernetes 集群所需的机器。

## 机器数据库

本教程将利用一个文本文件，作为机器数据库，用于存储在设置 Kubernetes 控制平面和工作节点时所需的各种机器属性。以下模式表示机器数据库中的条目，每行一个条目：

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

每列对应于机器的 IP 地址 `IPV4_ADDRESS`、完全限定域名 `FQDN`、主机名 `HOSTNAME` 和 IP 子网 `POD_SUBNET`。Kubernetes 为每个 `pod` 分配一个 IP 地址，而 `POD_SUBNET` 表示分配给每台机器的唯一 IP 地址范围。

以下是一个示例机器数据库，类似于本教程中使用的机器数据库。请注意，IP 地址已被屏蔽。你的机器可以分配任何 IP 地址，只要每台机器都可以相互访问，并且可以通过 `jumpbox` 访问。

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

现在轮到你创建一个 `machines.txt` 文件，填写你将用于创建 Kubernetes 集群的三台机器的详细信息。使用上面的示例机器数据库，并添加你机器的详细信息。

## 配置 SSH 访问

SSH 将用于配置集群中的机器。验证你是否具有对机器数据库中列出的每台机器的 `root` SSH 访问权限。你可能需要通过更新 sshd_config 文件并重启 SSH 服务器来启用每个节点的 `root` SSH 访问。

### 启用 root SSH 访问

如果已为每台机器启用 `root` SSH 访问，则可以跳过此部分。

默认情况下，新安装的 `debian` 会禁用 `root` 用户的 SSH 访问。这是出于安全原因，因为 `root` 用户是 Linux 系统上的一个众所周知的用户，如果在连接到互联网的机器上使用弱密码，那么只需时间问题，机器就会被他人控制。如前所述，我们将启用 `root` SSH 访问，以简化本教程中的步骤。安全是一种权衡，在这种情况下，我们为便利性进行了优化。在每台机器上，使用你的用户帐户通过 SSH 登录，然后使用 `su` 命令切换到 `root` 用户：

```bash
su - root
```

编辑 `/etc/ssh/sshd_config` SSH 守护进程配置文件，将 `PermitRootLogin` 选项设置为 `yes`：

```bash
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

重启 `sshd` SSH 服务器以加载更新的配置文件：

```bash
systemctl restart sshd
```

### 生成和分发 SSH 密钥

在本节中，你将生成并分发 SSH 密钥对到 `server`、`node-0` 和 `node-1` 机器，这将在整个教程中用于在这些机器上运行命令。从 `jumpbox` 机器运行以下命令。

生成新的 SSH 密钥：

```bash
ssh-keygen
```

```text
生成公钥/私钥 rsa 密钥对。
输入要保存密钥的文件 (/root/.ssh/id_rsa): 
输入密码 (为空表示没有密码): 
再次输入相同的密码: 
你的身份已保存在 /root/.ssh/id_rsa 中
你的公钥已保存在 /root/.ssh/id_rsa.pub 中
```

将 SSH 公钥复制到每台机器：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${IP}
done < machines.txt
```

一旦每个密钥添加完成，验证 SSH 公钥访问是否正常：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh -n root@${IP} uname -o -m
done < machines.txt
```

```text
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
```

## 主机名

在本节中，你将为 `server`、`node-0` 和 `node-1` 机器分配主机名。主机名将在从 `jumpbox` 执行命令到每台机器时使用。主机名在集群中也起着重要作用。Kubernetes 客户端使用主机名而不是 IP 地址来向 Kubernetes API 服务器发出命令，客户端将使用 `server` 主机名。此外，`node-0` 和 `node-1` 工作机器在注册到给定 Kubernetes 集群时也会使用主机名。

在 `jumpbox` 上运行以下命令为每台机器设置主机名：

```bash
while read IP FQDN HOST SUBNET; do 
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

验证每台机器的主机名是否设置：

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## DNS

在本节中，你将生成一个 DNS `hosts` 文件，该文件将附加到 `jumpbox` 本地的 `/etc/hosts` 文件以及本教程中使用的三台机器的 `/etc/hosts` 文件。这将使每台机器都可以通过主机名（如 `server`、`node-0` 或 `node-1`）访问。

创建一个新的 `hosts` 文件并添加一个标题以标识正在添加的机器：

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

为每台机器生成 DNS 条目并将其附加到 `hosts` 文件：

```bash
while read IP FQDN HOST SUBNET; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

查看 `hosts` 文件中的 DNS 条目：

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## 向本地机器添加 DNS 条目

在本节中，你将把 `hosts` 文件中的 DNS 条目附加到 `jumpbox` 机器的 `/etc/hosts` 文件中。

将 `hosts` 中的 DNS 条目附加到 `/etc/hosts`：

```bash
cat hosts >> /etc/hosts
```

验证 `/etc/hosts` 文件已更新：

```bash
cat /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters



# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

此时你应该能够使用主机名 SSH 到 `machines.txt` 文件中列出的每台机器。

```bash
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```text
server aarch64 GNU/Linux
node-0 aarch64 GNU/Linux
node-1 aarch64 GNU/Linux
```

## 向远程机器添加 DNS 条目

在本节中，你将把 `hosts` 文件中的 DNS 条目附加到 `machines.txt` 文本文件中列出的每台机器的 `/etc/hosts`。

将 `hosts` 文件复制到每台机器并将内容附加到 `/etc/hosts`：

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

此时，可以在从 `jumpbox` 机器或 Kubernetes 集群中的三台机器连接时使用主机名。现在，你可以通过主机名（如 `server`、`node-0` 或 `node-1`）连接到机器，而不是使用 IP 地址。

下一步：[配置 CA 和生成 TLS 证书](04-certificate-authority.md)