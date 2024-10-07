# 烟雾测试

在本实验中，你将完成一系列任务，以确保你的 Kubernetes 集群正常运行。

## 数据加密

在这一部分，你将验证加密 [静态秘密数据](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted) 的能力。

创建一个通用的秘密：

```bash
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

打印存储在 etcd 中的 `kubernetes-the-hard-way` 秘密的十六进制转储：

```bash
ssh root@server \
    'etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C'
```

```text
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 9b 79 a5 b9 49 a2 77  |:v1:key1:.y..I.w|
00000050  c0 6a c9 12 7c b4 c7 c4  64 41 37 97 4a 83 a9 c1  |.j..|...dA7.J...|
00000060  4f 14 ae 73 ab b8 38 26  11 14 0a 40 b8 f3 0e 0a  |O..s..8&...@....|
00000070  f5 a7 a2 2c b6 35 b1 83  22 15 aa d0 dd 25 11 3e  |...,.5.."....%.>|
00000080  c4 e9 69 1c 10 7a 9d f7  dc 22 28 89 2c 83 dd 0b  |..i..z..."(.,...|
00000090  a4 5f 3a 93 0f ff 1f f8  bc 97 43 0e e5 05 5d f9  |._:.......C...].|
000000a0  ef 88 02 80 49 81 f1 58  b0 48 39 19 14 e1 b1 34  |....I..X.H9....4|
000000b0  f6 b0 9b 0a 9c 53 27 2b  23 b9 e6 52 b4 96 81 70  |.....S'+#..R...p|
000000c0  a7 b6 7b 4f 44 d4 9c 07  51 a3 1b 22 96 4c 24 6c  |..{OD...Q..".L$l|
000000d0  44 6c db 53 f5 31 e6 3f  15 7b 4c 23 06 c1 37 73  |Dl.S.1.?.{L#..7s|
000000e0  e1 97 8e 4e 1a 2e 2c 1a  da 85 c3 ff 42 92 d0 f1  |...N..,.....B...|
000000f0  87 b8 39 89 e8 46 2e b3  56 68 41 b8 1e 29 3d ba  |..9..F..VhA..)=.|
00000100  dd d8 27 4c 7f d5 fe 97  3c a3 92 e9 3d ae 47 ee  |..'L....<...=.G.|
00000110  24 6a 0b 7c ac b8 28 e6  25 a6 ce 04 80 ee c2 eb  |$j.|..(.%.......|
00000120  4c 86 fa 70 66 13 63 59  03 c2 70 57 8b fb a1 d6  |L..pf.cY..pW....|
00000130  f2 58 08 84 43 f3 70 7f  ad d8 30 63 3e ef ff b6  |.X..C.p...0c>...|
00000140  b2 06 c3 45 c5 d8 89 d3  47 4a 72 ca 20 9b cf b5  |...E....GJr. ...|
00000150  4b 3d 6d b4 58 ae 42 4b  7f 0a                    |K=m.X.BK..|
0000015a
```

etcd 键应以 `k8s:enc:aescbc:v1:key1` 为前缀，这表示使用了 `aescbc` 提供程序来加密数据，使用的加密密钥为 `key1`。

## 部署

在这一部分，你将验证创建和管理 [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) 的能力。

为 [nginx](https://nginx.org/en/) Web 服务器创建一个部署：

```bash
kubectl create deployment nginx \
  --image=nginx:latest
```

列出由 `nginx` 部署创建的 Pod：

```bash
kubectl get pods -l app=nginx
```

```bash
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56fcf95486-c8dnx   1/1     Running   0          8s
```

### 端口转发

在这一部分，你将验证使用 [端口转发](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) 远程访问应用程序的能力。

获取 `nginx` Pod 的完整名称：

```bash
POD_NAME=$(kubectl get pods -l app=nginx \
  -o jsonpath="{.items[0].metadata.name}")
```

将本地机器的端口 `8080` 转发到 `nginx` Pod 的端口 `80`：

```bash
kubectl port-forward $POD_NAME 8080:80
```

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

在新终端中使用转发地址发起 HTTP 请求：

```bash
curl --head http://127.0.0.1:8080
```

```text
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 01:44:32 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes

```

切换回之前的终端并停止对 `nginx` Pod 的端口转发：

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### 日志

在这一部分，你将验证 [检索容器日志](https://kubernetes.io/docs/concepts/cluster-administration/logging/) 的能力。

打印 `nginx` Pod 的日志：

```bash
kubectl logs $POD_NAME
```

```text
...
127.0.0.1 - - [01/Nov/2023:06:10:17 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```

### 执行

在这一部分，你将验证 [在容器中执行命令](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container) 的能力。

通过在 `nginx` 容器中执行 `nginx -v` 命令打印 nginx 版本：

```bash
kubectl exec -ti $POD_NAME -- nginx -v
```

```text
nginx version: nginx/1.25.3
```

## 服务

在这一部分，你将验证使用 [Service](https://kubernetes.io/docs/concepts/services-networking/service/) 暴露应用程序的能力。

使用 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) 服务暴露 `nginx` 部署：

```bash
kubectl expose deployment nginx \
  --port 80 --type NodePort
```

> 由于你的集群未配置 [云提供商集成](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider)，因此无法使用 LoadBalancer 服务类型。设置云提供商集成超出了本教程的范围。

检索分配给 `nginx` 服务的节点端口：

```bash
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

使用 IP 地址和 `nginx` 节点端口发起 HTTP 请求：

```bash
curl -I http://node-0:${NODE_PORT}
```

```text
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 05:11:15 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes
```

下一步：[清理](13-cleanup.md)