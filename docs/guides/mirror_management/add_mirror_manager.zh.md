---
title: 添加 Rocky 镜像站
contributors: Amin Vakil, Steven Spencer, Ganna Zhyrnova
---

# 向 Rocky 镜像管理器添加公共镜像站

## 公共镜像站的最低要求

我们始终欢迎新的公共镜像站。但它们应当维护良好，托管在 24/7 全天候运行的数据中心类环境中。可用带宽至少应为 1 GBit/s。我们更倾向于提供双栈（IPv4 和 IPv6）支持的镜像站。请勿提交使用动态 DNS 配置的镜像站。如果您所在地区只有少量镜像站，我们也会接受较低的速度。

请勿提交托管在 Anycast-CDN（如 Cloudflare 等）上的镜像站，因为这可能导致 `dnf` 在选择最快镜像时性能不佳。

请注意，我们无法接受位于受美国出口管制法规约束国家的公共镜像站。您可以在以下地址找到这些国家的列表：<https://www.bis.doc.gov/index.php/policy-guidance/country-guidance/sanctioned-destinations>

截至撰写本文时（2022 年底），镜像所有当前和历史 Rocky Linux 发行版所需的存储空间约为 2 TB。

我们的主镜像站是 `rsync://msync.rockylinux.org/rocky-linux`。
首次同步请使用离您较近的镜像站。您可以在[此处](https://mirrors.rockylinux.org)找到所有官方镜像站。

请注意，未来我们可能会限制对官方主镜像站的访问，仅允许官方公共镜像站使用。因此，如果您运行的是私有镜像站，请考虑从离您较近的公共镜像站进行 `rsync` 同步。此外，本地镜像站同步速度可能更快。

## 设置您的镜像站

请设置一个 cron 作业定期同步您的镜像站，大约每天运行 6 次。但请确保不整点同步，以帮助分散负载。如果您只检查 `fullfiletimelist-rocky` 的变化并仅在此文件变更时才进行完整同步，则可以每小时同步一次。

以下是一些 crontab 示例：

```bash
#这将在 0:50、4:50、8:50、12:50、16:50、20:50 同步您的镜像站
50 */6  * * * /path/to/your/rocky-rsync-mirror.sh > /dev/null 2>&1

#这将在 2:25、6:25、10:25、14:25、18:25、22:25 同步您的镜像站
25 2,6,10,14,18,22 * * * /path/to/your/rocky-rsync-mirror.sh > /dev/null 2>&1

#这将每小时在整点后 15 分钟同步您的镜像站。
#仅当您使用示例脚本时才使用此项
15 * * * * /path/to/your/rocky-rsync-mirror.sh > /dev/null 2>&1
```

对于简单的同步，可以使用以下 `rsync` 命令：

```bash
rsync -aqH --delete source-mirror destination-dir
```

请考虑使用锁定机制，以避免在我们推送新版本时同时运行多个 `rsync` 作业。

您也可以使用并修改我们的示例脚本，该脚本实现了锁定机制并在需要时进行完整同步。脚本位于 <https://github.com/rocky-linux/rocky-tools/blob/main/mirror/mirrorsync.sh>。

首次完整同步完成后，请检查镜像站是否一切正常。最重要的是检查所有文件和目录是否已同步、cron 作业是否正常运行，以及镜像站是否可从公共互联网访问。请仔细检查防火墙规则！为避免问题，请不要强制 http 到 https 的重定向。

如果您在设置镜像站时有任何疑问，请加入 <https://chat.rockylinux.org/rocky-linux/channels/infrastructure>

完成后，请前往下一节，提交您的镜像站以成为公共镜像！

## 所需条件

- 一个 <https://accounts.rockylinux.org/> 的账户

## 创建站点

Rocky 使用 Fedora 的 Mirror Manager 来组织社区镜像站。

在此访问 Rocky 的 Mirror Manager：<https://mirrors.rockylinux.org/mirrormanager/>

成功登录后，您的个人资料将显示在右上角。选择下拉菜单，然后点击"My sites"。

将加载一个新页面，列出该账户下的所有站点。首次使用时页面为空。点击"Register a new site"。

将加载一个新页面，其中包含一份重要的出口合规声明供您阅读。然后填写以下信息：

- "Site Name"
- "Site Password" — `report_mirrors` 脚本使用，您可自行设置任意密码
- "Organization URL" — 公司/学校/组织的 URL，例如 <https://rockylinux.org/>
- "Private" — 选中此框将隐藏您的镜像站，不对外公开使用。
- "User active" — 取消选中此框将暂时禁用此站点，它将从公开列表中移除。
- "All sites can pull from me?" — 允许所有镜像站点从我这里拉取，无需明确添加到我的列表。
- "Comments for downstream siteadmins. Please include your synchronization source here to avoid dependency loops."

点击"Submit"后，您将返回主镜像页面。

## 配置站点

从主镜像页面选择下拉菜单，然后点击"My sites"。

账户站点页面将加载，站点应该会列出。点击站点进入信息站点页面。

上一节中的所有选项都将再次列出。页面底部有三个新选项：Admins、Hosts 和 Delete site。点击"Hosts [add]"。

## 创建新主机

填写适用于该站点的以下选项：

- "Host name" - 必填：公开终端用户看到的服务器 FQDN
- "User active" - 取消选中此框将暂时禁用此主机，它将从公开列表中移除。
- "Country" - 必填：2 字母 ISO 国家代码
- "Bandwidth" - 必填：整数兆位/秒，该主机可提供的带宽
- "Private" - 例如不向公众提供，为内部私有镜像站
- "Internet2" - 在 Internet2 上
- "Internet2 clients" - 为 Internet2 客户端提供服务，即使是私有镜像站
- "ASN" - 自治系统编号，用于 BGP 路由表。仅当您是 ISP 时才需要。
- "ASN Clients" - 为来自同一 ASN 的所有客户端提供服务。用于 ISP、公司或学校，不适用于个人网络。
- "Robot email" - 电子邮件地址，将收到上游内容更新通知
- "Comment" - 文本，您希望公开终端用户了解的有关镜像站的任何其他信息
- "Max connections" - 每个客户端的最大并行下载连接数，通过 metalinks 建议。

点击"Create"，页面将重定向回该主机的信息站点。

## 更新主机

在信息站点页面底部，"Hosts"选项旁边现在应显示主机标题。点击该名称加载主机页面。上一节中的所有选项都将再次列出。底部还有新的选项。

- "Site-local Netblocks"：Netblock 用于尝试将终端用户引导到站点特定的镜像站。例如，一所大学可能会列出其 Netblock，mirrorlist CGI 将返回大学本地镜像站而非国家级本地镜像站。格式为 18.0.0.0/255.0.0.0、18.0.0.0/8、IPv6 前缀/长度或 DNS 主机名。值必须是公共 IP 地址（不能是 RFC1918 私有空间地址）。仅当您是 ISP 和/或拥有公开可路由的 netblock 时才使用！

- "Peer ASNs"：对等 ASN 用于引导附近网络上的终端用户到我们的镜像站。例如，一所大学可能会列出其对等 ASN，mirrorlist CGI 将返回大学本地镜像站而非国家级本地镜像站。您必须是 MirrorManager 管理员组的成员才能在此处创建新条目。

- "Countries Allowed"：某些镜像站需要限制自己仅为来自其国家的终端用户提供服务。如果您是其中之一，请列出您允许终端用户来自的国家的 2 字母 ISO 代码。mirrorlist CGI 将遵守此设置。

- "Categories Carried"：主机承载的软件类别。示例 Fedora 类别包括 Fedora 和 Fedora Archive。

点击"Categories Carried"下的"[add]"链接。

### 承载的类别

对于类别，选择"Rocky Linux"，然后点击"Create"加载 URL 页面。然后点击"[add]"加载"Add host category URL"页面。有一个选项。对镜像站支持的每种协议重复此操作。

- "URL" - 指向顶级目录的 URL（rsync、https、http）

示例：

- `http://rocky.example.com`
- `https://rocky.example.com`
- `rsync://rocky.example.com`

## 收尾

信息填写完毕后，在下一次镜像刷新时，站点应会出现在镜像列表中。
