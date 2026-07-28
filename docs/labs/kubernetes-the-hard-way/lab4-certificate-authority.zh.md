---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 4: 配置 CA 并生成 TLS 证书

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将使用 CloudFlare 的 PKI 工具包 `cfssl` 来引导 [Public Key Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure)，然后使用它来引导 Certificate Authority（证书颁发机构），并为以下组件生成 TLS 证书：`etcd`、`kube-apiserver`、`kube-controller-manager`、`kube-scheduler`、`kubelet` 和 `kube-proxy`。

## 证书颁发机构

在本节中，你将为 Kubernetes 集群提供 Certificate Authority（CA）。可以使用 `cfssl` 轻松生成证书，此外，如果需要，还可以轻松地以适合作业的形式生成。

`ca-config.json` 文件设置了 `cfssl` 在后续证书生成步骤中运行的 `profiles`。请注意，`kubernetes` profile 的 `expiry` 设置为 8760 小时（一年）。

```bash
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF
```

创建 `ca-csr.json` 证书签名请求文件：

```bash
cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF
```

生成 CA 证书和私钥：

```bash
  cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

结果：

```text
ca-key.pem
ca.pem
```

## 客户端和服务器证书

在本节中，你将使用之前在本实验中创建的 CA 证书为每个 Kubernetes 组件生成客户端和服务器证书，并为 Kubernetes `admin` 用户生成客户端证书。

### Admin 客户端证书

生成 `admin` 客户端证书和私钥：

```bash
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

结果：

```text
admin-key.pem
admin.pem
```

### Kubelet 客户端证书

Kubernetes 使用[节点专用的授权](https://kubernetes.io/docs/admin/authorization/node/)（[Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/)），特别授权 [Kubelet](https://kubernetes.io/docs/concepts/overview/components/#kubelet) 发出的 API 请求。为了获得 Kubernetes Node Authorizer（节点授权器）的授权，Kubelet 必须使用系统凭据：`system:node:<nodeName>`，并具备 `system:nodes` 组。

为每个满足 Node Authorizer 要求的 worker 节点生成证书和私钥：

```bash
for instance in node-0 node-1; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

EXTERNAL_IP=$(grep ${instance} machines.txt | cut -d " " -f 1)

INTERNAL_IP=${EXTERNAL_IP}

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
```

结果：

```text
node-0-key.pem
node-0.pem
node-1-key.pem
node-1.pem
```

### Controller Manager 客户端证书

生成 `kube-controller-manager` 客户端证书和私钥：

```bash
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

结果：

```text
kube-controller-manager-key.pem
kube-controller-manager.pem
```

### Kube Proxy 客户端证书

生成 `kube-proxy` 客户端证书和私钥：

```bash
{
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

结果：

```text
kube-proxy-key.pem
kube-proxy.pem
```

### Scheduler 客户端证书

生成 `kube-scheduler` 客户端证书和私钥：

```bash
{
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

结果：

```text
kube-scheduler-key.pem
kube-scheduler.pem
```

### Kubernetes API Server 证书

`kubernetes-the-hard-way` 的静态 IP 地址将包含在 Kubernetes API Server 证书的主题替代名称列表中。这将确保远程客户端能够验证证书。

生成 Kubernetes API Server 证书和私钥：

```bash
{
CERT_HOSTNAME=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

KUBERNETES_PUBLIC_ADDRESS=$(grep server machines.txt | cut -d " " -f 1)

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,${CERT_HOSTNAME},${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```

> Kubernetes API 服务器会自动分配 `kubernetes` 内部的 DNS 名称，该名称默认链接到在启动 Kubernetes API 服务器运行时传递的 `--service-cluster-ip-range` 参数指定的范围中的第一个 IP 地址（例如 `10.32.0.1`）。

结果：

```text
kubernetes-key.pem
kubernetes.pem
```

### Service Account 密钥对

Kubernetes 控制器管理器利用密钥对来生成和签名 service account journal，如[管理 Service Account](https://kubernetes.io/docs/admin/service-accounts-admin/) 文档中所述。

生成 `service-account` 证书和私钥：

```bash
{
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

结果：

```text
service-account-key.pem
service-account.pem
```

## 分发客户端和服务器证书

将相应的证书和私钥复制到每台 worker 实例：

```bash
for instance in node-0 node-1; do
  ssh root@${instance} mkdir -p /var/lib/kubelet/

  scp ca.pem ${instance}-key.pem ${instance}.pem \
    root@${instance}:/var/lib/kubelet/
done
```

将相应的证书和私钥复制到 `server` 实例：

```bash
ssh root@server mkdir -p /var/lib/kubernetes/

scp \
    ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem \
    root@server:/var/lib/kubernetes/
```

> `kube-proxy`、`kube-controller-manager`、`kube-scheduler` 和 `kubelet` 客户端证书将在下一实验中用于生成客户端认证配置文件。

下一步: [生成 Kubernetes 配置文件以进行认证](lab5-kubernetes-configuration-files.md)
