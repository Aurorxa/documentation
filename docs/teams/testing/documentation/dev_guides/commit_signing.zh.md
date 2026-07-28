---
title: Git 提交签名
author: Al Bowles
contributors: Lukas Magauer
tested_with:
tags:
  - testing
  - git
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver: 8
  level: Final
render_macros: true
---

## 创建主密钥对

1. 启动密钥对生成向导

   ```bash
   gpg --full-generate-key --expert
   ```

2. 选择选项 `(9) ECC and ECC` 作为密钥类型
3. 选择选项 `(1) Curve 25519` 作为椭圆曲线
4. 设置你选择的有效期，最好少于 1 年
5. 指定要与此密钥对关联的真实姓名和电子邮件地址。该电子邮件地址必须与你已验证的 Github 电子邮件地址匹配，或者设置为 `your-github-username@users.noreply.github.com`。
6. 输入密码短语（两次）

## 创建签名密钥对

1. 添加一个签名子密钥

   ```bash
   gpg --expert --edit-key my@email.addr
   gpg> addkey
   ```

2. 选择选项 `(10) ECC (sign only)` 作为密钥类型
3. 选择选项 `(1) Curve 25519` 作为椭圆曲线
4. 设置你选择的有效期，最好少于 1 年
5. 接受提示并输入密码短语（两次）
6. 保存并退出

   ```bash
   gpg> save
   ```

## 创建吊销证书

```bash
gpg --output my_email_addr.gpg-revocation-certificate --gen-revoke my@email.addr
```

## 备份你的密钥对

导出*主密钥对*（将这些与吊销证书一起放在非常安全的地方）

```bash
gpg --export-secret-keys --armor my@email.addr > my_email_addr.private.gpg-key
gpg --export --armor my@email.addr > my_email_addr.public.gpg-key
```

## 从密钥环中移除*主密钥对*

1. 将新密钥对的所有子密钥导出到一个文件

   ```bash
   gpg --export-secret-subkeys my@email.addr > $HOME/.gnupg/subkeys
   ```

2. 从密钥环中删除主密钥——*请务必先备份你的主密钥对！*

   ```bash
   gpg --delete-secret-key my@email.addr
   ```

3. 重新导入之前导出的密钥

   ```bash
   gpg --import $HOME/.gnupg/subkeys
   ```

4. 在输出中查找 `sec#` 而不是 `sec`——井号表示签名子密钥*不在*密钥环中的密钥对里

   ```bash
   gpg --list-secret-keys $HOME/.gnupg/secring.gpg
   ```

## 吊销*签名密钥对*

找到*主密钥对*并导入（最好导入到临时系统如 liveUSB 中）

```bash
 gpg --import /path/to/my_email_addr.public.gpg-key /path/to/my_email_addr.private.gpg-key
 gpg --edit-key my@email.addr
 gpg> revkey
 [ 输入密码短语两次 ]
 gpg> save
```

## 续期已过期或即将过期的密钥对

```bash
 gpg --edit-key my@email.addr
 [ 选择一个密钥 ]
 gpg> expire
 [ 指定一个过期时间 ]
 gpg> save
```

## 创建单个签名 Git 提交

```bash
 git commit -S -m "my awesome signed commit"
```

## 配置 Git 始终使用指定密钥签名提交

```bash
 gpg --list-secret-keys --keyid-format=long # 从 'sec' 行获取指纹
 git config [--global] commit.gpgsign true
 git config [--global] user.signingkey DEADB33FBAD1D3A
```

## 配置 VSCode 签名提交

```bash
 # 用户或工作区设置
 "git.enableCommitSigning": true
```

## 将你的公钥上传到密钥服务器

```bash
 gpg --keyserver pgp.mit.edu --send-keys 0xDEADB33FBAD1D3A
```

## 验证你的密钥是否已发布

```bash
 gpg --keyserver pgp.mit.edu --search-key my@email.addr
```

## 参考资料

- [OpenPGP 最佳实践](https://riseup.net/en/security/message-security/openpgp/best-practices#key-configuration)
- [Github: 签名提交](https://docs.github.com/en/enterprise-server@3.5/authentication/managing-commit-signature-verification/signing-commits)
- [Braincoke's Log: 创建 GPG 密钥](https://blog.braincoke.fr/security/create-a-gpg-key/)
- [创建完美的 GPG 密钥对](https://alexcabal.com/creating-the-perfect-gpg-keypair)
= [Digital Neanderthal: 使用 Curve Ed25519 生成 GPG 密钥](https://web.archive.org/web/20240512060111/https://www.digitalneanderthal.com/post/gpg/)

{% include 'teams/testing/content_bottom.md' %}
