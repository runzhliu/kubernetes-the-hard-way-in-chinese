# 设置跳板机

在本实验中，您将设置四台机器中的一台作为 `jumpbox`。这台机器将用于运行本教程中的命令。虽然使用专用机器可以确保一致性，但这些命令也可以在几乎任何机器上运行，包括您个人的 macOS 或 Linux 工作站。

将 `jumpbox` 视为您在从头开始设置 Kubernetes 集群时使用的管理机器。我们在开始之前需要做的一件事是安装一些命令行工具，并克隆 Kubernetes The Hard Way git 仓库，该仓库包含将在本教程中用于配置各种 Kubernetes 组件的附加配置文件。

登录到 `jumpbox`：

```bash
ssh root@jumpbox
```

所有命令都将在 `root` 用户下运行。这是出于方便考虑，并将有助于减少设置所需的命令数量。

### 安装命令行工具

现在您已作为 `root` 用户登录到 `jumpbox` 机器，您将安装将在本教程中执行各种任务所需的命令行工具。

```bash
apt-get -y install wget curl vim openssl git
```

### 同步 GitHub 仓库

现在是时候下载本教程的副本，该副本包含将用于从头构建 Kubernetes 集群的配置文件和模板。使用 `git` 命令克隆 Kubernetes The Hard Way git 仓库：

```bash
git clone --depth 1 \
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```

切换到 `kubernetes-the-hard-way` 目录：

```bash
cd kubernetes-the-hard-way
```

这将是本教程其余部分的工作目录。如果您迷路，可以运行 `pwd` 命令以验证您在运行 `jumpbox` 上的命令时是否在正确的目录中：

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### 下载二进制文件

在本节中，您将下载各种 Kubernetes 组件的二进制文件。这些二进制文件将存储在 `jumpbox` 上的 `downloads` 目录中，这将减少完成本教程所需的互联网带宽，因为我们避免为 Kubernetes 集群中的每台机器多次下载二进制文件。

从 `kubernetes-the-hard-way` 目录创建一个 `downloads` 目录，使用 `mkdir` 命令：

```bash
mkdir downloads
```

将下载的二进制文件列在 `downloads.txt` 文件中，您可以使用 `cat` 命令查看：

```bash
cat downloads.txt
```

使用 `wget` 命令下载 `downloads.txt` 文件中列出的二进制文件：

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

根据您的互联网连接速度，下载 `584` 兆字节的二进制文件可能需要一段时间，下载完成后，您可以使用 `ls` 命令列出它们：

```bash
ls -loh downloads
```

```text
total 584M
-rw-r--r-- 1 root  41M May  9 13:35 cni-plugins-linux-arm64-v1.3.0.tgz
-rw-r--r-- 1 root  34M Oct 26 15:21 containerd-1.7.8-linux-arm64.tar.gz
-rw-r--r-- 1 root  22M Aug 14 00:19 crictl-v1.28.0-linux-arm.tar.gz
-rw-r--r-- 1 root  15M Jul 11 02:30 etcd-v3.4.27-linux-arm64.tar.gz
-rw-r--r-- 1 root 111M Oct 18 07:34 kube-apiserver
-rw-r--r-- 1 root 107M Oct 18 07:34 kube-controller-manager
-rw-r--r-- 1 root  51M Oct 18 07:34 kube-proxy
-rw-r--r-- 1 root  52M Oct 18 07:34 kube-scheduler
-rw-r--r-- 1 root  46M Oct 18 07:34 kubectl
-rw-r--r-- 1 root 101M Oct 18 07:34 kubelet
-rw-r--r-- 1 root 9.6M Aug 10 18:57 runc.arm64
```

### 安装 kubectl

在本节中，您将安装 `kubectl`，这是官方的 Kubernetes 客户端命令行工具，安装在 `jumpbox` 机器上。`kubectl` 将用于与 Kubernetes 控制交互，一旦您的集群在本教程的后面部分配置完成。

使用 `chmod` 命令使 `kubectl` 二进制文件可执行，并将其移动到 `/usr/local/bin/` 目录：

```bash
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

此时已安装 `kubectl`，可以通过运行 `kubectl` 命令进行验证：

```bash
kubectl version --client
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

此时，`jumpbox` 已设置好所有完成本教程中实验所需的命令行工具和实用程序。

接下来：[配置计算资源](03-compute-resources.md)