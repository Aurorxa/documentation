---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 6: 生成数据加密密钥

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

Kubernetes 支持[加密静态数据](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)的能力。除了针对 API Server 的各种[命令行选项](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#configuration-and-determining-whether-encryption-at-rest-is-already-enabled)外，它还支持加密密钥。在本实验中，你将生成用于加密 Kubernetes Secrets（Secret）的加密密钥。

## 加密密钥

生成加密密钥：

```bash
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## 加密配置文件

创建名为 `encryption-config.yaml` 的 EncryptionConfiguration 文件：

```bash
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

将 `encryption-config.yaml` 加密配置文件复制到 `server` 实例：

```bash
scp encryption-config.yaml root@server:~/
```

下一步: [引导 etcd 集群](lab7-bootstrapping-etcd.md)
