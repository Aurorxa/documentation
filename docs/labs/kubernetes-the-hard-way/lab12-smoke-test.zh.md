---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
  - runc
  - containerd
  - etcd
  - kubectl
---

# 实验 12: 冒烟测试

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将完成一系列任务以确保 Kubernetes 集群正常工作。

## 数据加密

在本节中，你将验证[加密静态 secret 数据](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted)的能力。

创建一个通用 secret：

```bash
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

打印存储在 `etcd` 中的 `kubernetes-the-hard-way` secret 的 hexdump：

```bash
ssh root@server \
    'etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C'
```

```text
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 67 64 d4 64 d9 6c eb  |:v1:key1:gd.d.l.|
00000050  f3 a9 80 8c 82 a0 f3 50  e6 eb 02 cb a6 65 8b 51  |.......P.....e.Q|
00000060  0a 03 2f ef 4a a8 d9 7d  62 f5 68 27 74 1c 6d 88  |../.J..}b.h't.m.|
00000070  ee 89 39 8d c9 e4 c6 2c  9c 3f 55 3b eb 79 2c 55  |..9....,.?U;.y,U|
00000080  d9 47 ec 77 84 0b 6e b1  ab 58 41 22 1e c3 7c 00  |.G.w..n..XA"..|.|
00000090  92 ad ad d0 97 55 f4 d5  b6 1e 6d 57 fb 2f 7c 36  |.....U....mW./|6|
000000a0  f5 6a 45 5e a7 a4 70 52  1d 9d 00 61 14 cd f3 92  |.jE^..pR...a....|
000000b0  9a 05 57 dd cf 41 1c 74  0c a7 2c ac 64 f5 79 51  |..W..A.t..,.d.yQ|
000000c0  e6 5c 16 e1 a9 5e da 00  91 d0 f5 77 e2 32 8f a2  |.\...^.....w.2..|
000000d0  9a c3 26 50 f2 a0 f8 f9  95 09 20 29 f1 7c 7a 7d  |..&P...... ).|z}|
000000e0  4f 82 0b 78 43 28 91 a1  56 ee 66 90 fb ac 26 8c  |O..xC(..V.f...&.|
000000f0  a3 b4 b2 ed 2d 0a d3 54  d1 10 89 a4 c3 dd b6 2d  |....-..T.......-|
00000100  b9 d7 98 bb db c4 0d f2  96 5a 57 8b 01 2e 97 43  |.........ZW....C|
00000110  ea 9c 8b cf cc 9b 80 cd  02 c8 c5 a4 e7 bf 62 73  |..............bs|
00000120  e6 6b e8 c2 cf 34 50 2b  e0 3c 66 a2 29 4f 08 0c  |.k...4P+.<f.)O..|
00000130  65 99 e4 9f 40 4f d9 94  eb 40 bd 3a 01 77 95 2b  |e...@O...@.:.w.+|
00000140  c9 20 ea a8 73 d9 19 2d  d8 00 9b da 3e 55 3e 82  |. ..s..-....>U>.|
00000150  80 3a 39 d5 08 f9 6c de  6b 0a                    |.:9...l.k.|
0000015a
```

需要为 `etcd` 键添加前缀 `k8s:enc:aescbc:v1:key1`，这表示使用 `aescbc` 加密提供程序，配合 `key1` 加密密钥来加密数据。

## Deployments（部署）

在本节中，你将验证创建和管理 [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) 的能力。

为 [nginx](https://nginx.org/en/) Web 服务器创建一个 deployment：

```bash
kubectl create deployment nginx \
  --image=nginx:latest
```

列出 `nginx` deployment 创建的 pod：

```bash
kubectl get pods -l app=nginx
```

```bash
NAME                     READY   STATUS    RESTARTS   AGE
nginx-54c98b4f84-dfwl9   1/1     Running   0          71s
```

### Port Forwarding（端口转发）

在本节中，你将验证使用[端口转发](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)远程访问应用程序的能力。

获取 `nginx` pod 的完整名称：

```bash
POD_NAME=$(kubectl get pods -l app=nginx \
  -o jsonpath="{.items[0].metadata.name}")
```

将本地机器的端口 `8080` 转发到 `nginx` pod 的端口 `80`：

```bash
kubectl port-forward $POD_NAME 8080:80
```

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

在一个新终端中，使用转发地址发起 HTTP 请求：

```bash
curl --head http://127.0.0.1:8080
```

```text
HTTP/1.1 200 OK
Server: nginx/1.27.4
Date: Tue, 04 Mar 2025 01:30:20 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 05 Feb 2025 11:06:32 GMT
Connection: keep-alive
ETag: "67a34638-267"
Accept-Ranges: bytes
```

切换回之前的终端，按 ++ctrl+c++ 停止到 `nginx` pod 的端口转发：

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs（日志）

在本节中，你将验证[获取容器日志](https://kubernetes.io/docs/concepts/cluster-administration/logging/)的能力。

打印 `nginx` pod 日志：

```bash
kubectl logs $POD_NAME
```

```text
...<OUTPUT TRUNCATED>...
127.0.0.1 - - [04/Mar/2025:01:30:20 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.76.1" "-"
```

### Exec（执行命令）

在本节中，你将验证[在容器中执行命令](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container)的能力。

在 `nginx` 容器中执行 `nginx -v` 命令打印 `nginx` 版本：

```bash
kubectl exec -ti $POD_NAME -- nginx -v
```

```text
nginx version: nginx/1.27.4   
```

## Services（服务）

在本节中，你将验证使用 [Service](https://kubernetes.io/docs/concepts/services-networking/service/) 暴露应用程序的能力。

使用 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) 服务暴露 `nginx` deployment：

```bash
kubectl expose deployment nginx \
  --port 80 --type NodePort
```

> 不能使用 LoadBalancer 服务类型，因为你的集群未配置[云提供商集成](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider)。本教程不涵盖云提供商集成的配置。

获取分配给 `nginx` 服务的节点端口：

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
Server: nginx/1.27.4
Date: Tue, 04 Mar 2025 01:40:20 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 05 Feb 2025 11:06:32 GMT
Connection: keep-alive
ETag: "67a34638-267"
Accept-Ranges: bytes
```

下一步: [清理](lab13-cleanup.md)
