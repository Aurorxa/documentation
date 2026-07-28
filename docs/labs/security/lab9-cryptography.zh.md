---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - cryptography
  - security
---

# 实验 9: 密码学基础

!!! info

    输入命令 `lab9-cryptography`，启动一个名为 `lab9-cryptography` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将运用在 Rocky Linux 中内置的各种加密工具（保护信息资产的实践）的技能。你将实践公钥和私钥（非对称和对称密钥）的使用。你还将了解公钥基础设施（PKI）的关键方面，因为它涉及证书颁发机构（CA）和 TLS（Transport Layer Security）。

    !!! knowledge "知识要点"

        在当今世界，你无法在不保护信息的情况下完成太多工作。加密技术每天都在被调用和实践。本实验将处理：

        * 使用 GPG 工具和密钥在本地加密文件和签名文件
        * 在传输过程中安全地复制文件
        * 使用 openssl 创建自签名证书

## 目标

1. 学习如何使用 GPG 工具进行文件加密、解密和签名
2. 安全复制文件
3. 使用 openssl 进行基本密钥生成和证书管理

## 先决条件

* 你将需要两台安装了 Rocky Linux 9.x 的机器。
* 管理员（root）或具有 sudo 访问权限的帐户
* 两个节点分别命名为 `Crypto_Server` 和 `Crypto_Client`
* 这两个应该能够相互通信

## 使用 GPG 工具

GNUPG（又称 GPG）是 PGP（Pretty Good Privacy）的一个免费开源版本实现。它是一个允许你加密和解密数据、创建和管理公钥/私钥的工具。

### 为文件加密创建 GPG 密钥

在继续之前，先创建一个示例文件用于加密：

```bash
echo "GPG Test Message." > clear-text.txt
```

#### 基本使用

1. 首先，在 `Crypto_Server` 机器上生成一个新的 GPG 密钥对。使用默认选项。
2. 使用对称密钥加密目标文件。提示：你需要了解适当的对称加密选项。GPG 默认使用 AES。
3. 解密加密的文件。
4. 使用你的非对称密钥对加密文件。
5. 解密加密的文件。

!!! question "问题"

    1. 用于生成 GPG 密钥对的命令是什么？
    2. 如何列出系统密钥环上的密钥？解释输出内容。
    3. 如果只想使用 RSA 算法，如何创建？

#### 共享密钥和文件

6. 将你的公钥导出到一个文件中。使用 ASCII 格式，这样它可以跨平台共享。
7. 将你的公钥文件复制到另一台机器（`Crypto_Client`）。
8. 在 `Crypto_Client` 机器上，导入你的公钥文件，使其可被其他服务使用。
9. 在 `Crypto_Client` 上，尝试使用导入的公钥加密文件，然后解密该文件。

!!! question "问题"

    1. 为什么你可能需要将公钥从一台服务器导出到另一台服务器？
    2. 私钥和公钥都可以用来加密数据，但是否有其他特殊用途限制了各自的使用方式？

## 保护传输中的文件

传输中的文件容易受到攻击和窥探；因此，如果你传输的数据可能包含敏感信息，你应该在传输前对其进行加密。

`scp` 命令是保护网络数据传输的一种常用方式。你将在实验中使用的另一个命令是 `rsync`，用于通过 SSH 复制文件。

让我们使用这些命令来安全地传输文件。

1. 将文件从 `Crypto_Server` 复制到 `Crypto_Client` 机器，使用 `scp` 和 `rsync` 工具进行保护。在目标目录后加上 `/`。

    !!! question "问题"

        与 `scp` 相比，什么时候使用 `rsync` 更合适？

2. 使用 `scp` 和 `rsync` 将整个目录从远程服务器复制到本地服务器。

## 使用 `openssl` 处理证书

数字证书用于在各方之间建立信任，并促进通过 TLS 等协议进行安全通信。证书颁发机构（CA）是验证服务器身份的组织。组织也可以使用自己的 CA 来降低成本和简化管理。这是通过创建和签署自己的证书来实现的。

简言之，自签名证书为你提供了完成 TLS 所需的内容，但缺少由主流浏览器信任的知名证书颁发机构（称为根 CA）验证和验证你的服务器的额外步骤。值得一提的是，你可以使用免费的证书颁发机构，如 Let's Encrypt。

在本实验中，你将首先创建自己的自签名 CA 证书，然后使用它来签署服务器证书。你将首先生成自签名的根 CA 证书，接着使用它签署服务器证书，最后在服务器环境中完成配置。你将创建一个简单的 Web 服务器来展示最终的设置。

### 1. 创建自签名根证书（CA）

在 `Crypto_Server` 上：

```bash
# 创建 CA 私钥
openssl genrsa -out ca.key 2048

# 创建自签名 CA 证书
openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt
```

### 2. 创建服务器证书

在 `Crypto_Server` 上，创建一个服务器证书并使用你自己的简单内部 CA 进行签名。

#### a. 生成服务器私钥

```bash
openssl genrsa -out tls_server.key 2048
```

#### b. 使用你的私钥创建 CSR（Certificate Signing Request）

```bash
openssl req -new -key tls_server.key -out tls_server.csr
```

#### c. 创建 `v3.ext` 文件。在 CSR 之前创建：

```bash
cat > v3.ext <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = <<IP address of your Crypto_Server>>
DNS.1 = <<hostname of your Crypto_Server>>
EOF
```

#### d. 现在使用 CSR 和 v3.ext 文件生成服务器证书：

```bash
openssl x509 -req -in tls_server.csr \
-CA ca.crt \
-CAkey ca.key \
-CAcreateserial \
-out tls_server.crt \
-days 365 \
-sha256 \
-extfile v3.ext
```

### 3. 安装和配置 Web 服务

在 `Crypto_Server` 上安装并配置 Web 服务器（如 `nginx` 或 `httpd`），以使用你创建的服务器证书。

1. 更新防火墙规则，打开目标机器上 Web 服务器的端口。
2. 将必要的服务器证书文件（`tls_server.key` 和 `tls_server.crt`）放置到 Rocky Linux 9.x 上 Web 服务器预期存放证书的适当目录中。
3. 配置 Web 服务器软件以识别和使用你的密钥和证书文件。使用 Web 服务器预期的配置路径和文件名。这可能需要创建配置目录并编辑配置文件。
4. 验证 Web 服务器可以安全地提供内容。从 `Crypto_Client` 使用 `curl` 和浏览器连接到你的 Web 服务器。

### 4. 可选：创建客户端证书

如果你愿意，可以创建客户端证书并进行客户端-服务器 TLS 测试。

!!! question "问题"

    1. 讨论如何使用配置管理和自动化来确保证书在过期之前能够可靠地更新。示例命令：`openssl s_client`。
    2. 查找并解释 `openssl.cnf` 文件的使用。
    3. 信任 SSL 证书的根 CA 位置在哪里？
    4. 你可能希望保护你的私钥（例如 ca.key）。你会使用什么文件权限来保护它们？为什么？

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
