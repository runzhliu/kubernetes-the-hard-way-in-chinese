# Kubernetes The Hard Way in Chinese

Kubernetes The Hard Way in Chinese 是根据 [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) 做的中文版翻译，另外可能会包含一些格式的调整，更方便阅读，大部分的内容会跟原版保持一致。

# Kubernetes The Hard Way

本教程将引导您以较为复杂的方式设置 Kubernetes。此指南并不适合希望使用完全自动化工具启动 Kubernetes 集群的人。Kubernetes The Hard Way 旨在优化学习，这意味着采取较长的路线，以确保您理解引导 Kubernetes 集群所需的每个任务。

> 本教程的结果不应视为生产就绪，可能会受到社区的有限支持，但这并不妨碍您学习！

## 版权

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品根据<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可证</a>授权。

## 目标受众

本教程的目标受众是希望理解 Kubernetes 基础知识以及核心组件如何协同工作的人员。

## 集群详情

Kubernetes The Hard Way 引导您启动一个基本的 Kubernetes 集群，所有控制平面组件都运行在单个节点上，并且有两个工作节点，这足以学习核心概念。

组件版本：

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.28.x
* [containerd](https://github.com/containerd/containerd) v1.7.x
* [cni](https://github.com/containernetworking/cni) v1.3.x
* [etcd](https://github.com/etcd-io/etcd) v3.4.x

## 实验

本教程需要四（4）台基于 ARM64 的虚拟或物理机器，并连接到同一网络。虽然本教程使用的是基于 ARM64 的机器，但所学的知识可以应用于其他平台。

* [先决条件](docs_zh/01-prerequisites.md)
* [设置跳转机](docs_zh/02-jumpbox.md)
* [配置计算资源](docs_zh/03-compute-resources.md)
* [配置 CA 和生成 TLS 证书](docs_zh/04-certificate-authority.md)
* [生成 Kubernetes 配置文件以进行身份验证](docs_zh/05-kubernetes-configuration-files.md)
* [生成数据加密配置和密钥](docs_zh/06-data-encryption-keys.md)
* [引导 etcd 集群](docs_zh/07-bootstrapping-etcd.md)
* [引导 Kubernetes 控制平面](docs_zh/08-bootstrapping-kubernetes-controllers.md)
* [引导 Kubernetes 工作节点](docs_zh/09-bootstrapping-kubernetes-workers.md)
* [配置 kubectl 以进行远程访问](docs_zh/10-configuring-kubectl.md)
* [配置 Pod 网络路由](docs_zh/11-pod-network-routes.md)
* [烟雾测试](docs_zh/12-smoke-test.md)
* [清理工作](docs_zh/13-cleanup.md)