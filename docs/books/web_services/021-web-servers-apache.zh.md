---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 2.1 部分 Web 服务器 Apache 
---

## Apache

在本章中，您将学习关于 Apache Web 服务器的知识。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装和配置 Apache

:checkered_flag: **apache**, **http**, **httpd**

**知识**：:star: :star:  
**复杂性**：:star: :star:  

**阅读时间**：30 分钟

****

### 概述

Apache HTTP 服务器是一个志愿者小组的作品：Apache Group。该小组致力于构建一个与商业产品水平相当的 Web 服务器，但作为自由软件（其源代码可用）。

数百名用户加入了最初的团队，贡献了想法、测试和代码行，使 Apache 成为世界上使用最广泛的 Web 服务器。

Apache 的前身是由伊利诺伊大学国家超级计算应用中心（NCSA）开发的可访问服务器。当负责人于 1994 年离开 NCSA 时，这个服务器的演进停止了。用户继续修复 bug 并创建扩展，他们以"patch（补丁）"的形式分发，因此得名"a patchee server"。

Apache 1.0 版本于 1995 年 12 月 1 日发布（距今已 30 多年！）。

开发团队通过邮件列表协调工作，其中发生关于软件提案和更改的讨论。更改在纳入项目之前经过投票。任何人都可以加入开发团队。要成为 Apache Group 的成员，您必须积极为项目做出贡献。

Apache 服务器在互联网上拥有强大的存在，约占所有活跃网站市场份额的 50%。

Apache 经常会市场份额输给它最大的挑战者 Nginx 服务器。后者在提供 Web 页面方面更快，但功能不如 Apache 巨头完整。

### 安装

Apache 是**跨平台**的。它可以在 Linux、Windows、Mac 等系统上使用。

管理员需要在两种安装方法之间做出选择：

* **包安装**：发行版供应商提供**稳定、受支持**（但有时是较旧的）版本

* **从源代码安装**：这涉及管理员自己编译软件，从而可以指定他或她感兴趣的选项，进而优化服务。由于 Apache 具有模块化架构，通常不必要重新编译 Apache 软件来添加或移除附加功能（添加或移除模块）。

强烈推荐基于包的安装方法。有其他仓库可用于在较旧的发行版上安装较新版本的 Apache，但在出现问题时没有人会提供支持。

在 Enterprise Linux 发行版上，`httpd` 包提供了 Apache 服务器。

将来，您可能需要安装一些额外的模块。以下是一些模块及其角色的示例：

* **mod_access**：按主机名、IP 地址或其他特征过滤客户端访问
* **mod_alias**：允许创建别名或虚拟目录
* **mod_auth**：对客户端进行认证
* **mod_cgi**：执行 CGI 脚本
* **mod_info**：提供服务器状态信息
* **mod_mime**：将文件类型与相应的操作关联
* **mod_proxy**：提供代理服务器
* **mod_rewrite**：重写 URL
* 其他

```bash
sudo dnf install httpd
```

Rocky Linux 9 上安装的版本是 2.4。

安装包会创建一个 `apache` 系统用户和一个对应的 `apache` 系统组。

```bash
$ grep apache /etc/passwd
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
$ grep apache /etc/group
apache:x:48:
```

启用并启动服务：

```bash
$ sudo systemctl enable httpd --now
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

您可以检查服务状态：

```bash
$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabl>     Active: active (running) since Fri 2024-06-21 14:22:34 CEST; 8s ago
       Docs: man:httpd.service(8)
   Main PID: 4387 (httpd)
     Status: "Started, listening on: port 80"
      Tasks: 177 (limit: 11110)
     Memory: 24.0M
        CPU: 68ms
     CGroup: /system.slice/httpd.service
             ├─4387 /usr/sbin/httpd -DFOREGROUND
             ├─4389 /usr/sbin/httpd -DFOREGROUND
             ├─4390 /usr/sbin/httpd -DFOREGROUND
             ├─4391 /usr/sbin/httpd -DFOREGROUND
```

不要忘记开放您的防火墙（参见安全部分）。

您现在可以检查服务的可用性：

* 从任何 Web 浏览器访问，提供您服务器的 IP 地址（例如，<http://192.168.1.100/>）。
* 直接从您的服务器访问。

为此，您必须安装一个文本浏览器，例如 elinks。

```bash
sudo dnf install elinks
```

浏览您的服务器并检查默认页面：

```bash
elinks http://localhost
```

安装 `httpd` 包会生成一个需要完全理解的完整目录树结构：

```text
/etc/httpd/
├── conf
│   ├── httpd.conf
│   └── magic
├── conf.d
│   ├── README
│   ├── autoindex.conf
│   ├── userdir.conf
│   └── welcome.conf
├── conf.modules.d
│   ├── 00-base.conf
│   ├── 00-brotli.conf
│   ├── 00-dav.conf
│   ├── 00-lua.conf
│   ├── 00-mpm.conf
│   ├── 00-optional.conf
│   ├── 00-proxy.conf
│   ├── 00-systemd.conf
│   ├── 01-cgi.conf
│   ├── 10-h2.conf
│   ├── 10-proxy_h2.conf
│   └── README
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
├── run -> /run/httpd
└── state -> ../../var/lib/httpd
/var/log/httpd/
├── access_log
└── error_log
/var/www/
├── cgi-bin
└── html
```

您会注意到 `/etc/httpd/logs` 文件夹是指向 `/var/log/httpd` 目录的符号链接。同样，您会注意到组成默认站点的文件位于 `/var/www/html` 文件夹中。

### 配置

最初，Apache 服务器的配置位于单个 `/etc/httpd/conf/httpd.conf` 文件中。随着时间的推移，这个文件变得越来越庞大，越来越难以阅读。

因此，现代发行版倾向于将 Apache 配置分布在 `/etc/httpd/conf.d` 和 `/etc/httpd/conf.modules.d` 目录中的一系列 `*.conf` 文件里，通过 Include 指令附加到主 `/etc/httpd/conf/httpd.conf` 文件中。

```bash
$ sudo grep "^Include" /etc/httpd/conf/httpd.conf
Include conf.modules.d/*.conf
IncludeOptional conf.d/*.conf
```

`/etc/httpd/conf/httpd.conf` 文件被充分注释。一般情况下，这些注释足以说明管理员的选择。

全局服务器配置位于 `/etc/httpd/conf/httpd.conf` 中。

此文件有三个配置部分：

* **第 1 部分**，全局环境；
* **第 2 部分**，默认站点和默认虚拟站点参数；
* **第 3 部分**，虚拟主机。

**虚拟主机**允许您在同一服务器上**上线多个虚拟站点**。然后根据其域名、IP 地址等来区分站点。

修改第 1 部分或第 2 部分中的值会影响所有托管站点。

在共享环境中，因此修改应在第 3 部分进行。

为了方便未来的更新，强烈建议为每个虚拟站点创建一个第 3 部分的配置文件。

以下是 `httpd.conf` 文件的最小版本：

```file
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/var/www/html"
<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
<Files ".ht*">
    Require all denied
</Files>
ErrorLog "logs/error_log"
LogLevel warn
<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
</IfModule>
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
<IfModule mime_module>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>
AddDefaultCharset UTF-8
<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>
EnableSendfile on
IncludeOptional conf.d/*.conf
```

#### 第 1 部分

第 1 部分中遇到的各种指令如下：

| 选项                   | 信息                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------ |
| `ServerTokens`         | 此指令将在后续章节中介绍。                                      |
| `ServertRoot`          | 指示包含组成 Apache 服务器的所有文件的目录路径。  |
| `Timeout`              | 请求超时前的秒数（传入或传出）。 |
| `KeepAlive`            | 持久连接（每个 TCP 连接多个请求）。                               |
| `MaxKeepAliveRequests` | 最大持久连接数。                                                  |
| `KeepAliveTimeout`     | 在关闭 TCP 连接之前等待下一个客户端请求的秒数。   |
| `Listen`               | 允许 Apache 监听特定的地址或端口。                                     |
| `LoadModule`           | 加载附加模块（模块越少 = 安全性越高）。                                    |
| `Include`              | 包含其他服务器配置文件。                                                  |
| `ExtendedStatus`       | 在服务器状态模块中显示关于服务器的更多信息。                     |
| `User` 和 `Group`     | 允许以不同用户启动 Apache 进程。Apache 始终以 root 启动，然后更改其所有者和组。 |

##### 多进程模块 (MPM)

Apache 服务器设计为强大且灵活，能够在各种平台上运行。

不同的平台和环境通常意味着不同的功能，或者使用其他方法来尽可能高效地实现相同的功能。

Apache 的模块化设计允许管理员通过在编译时或运行时选择加载哪些模块来选择服务器中包含哪些功能。

这种模块化还包括最基本的 Web 服务器功能。

多进程模块（MPM）模块负责与机器的网络端口关联、接受请求并在各个子进程中分发它们。

MPM 模块的配置在 `/etc/httpd/conf.modules.d/00-mpm.conf` 配置文件中：

```file
# Select the MPM module which should be used by uncommenting exactly
# one of the following LoadModule lines.  See the httpd.conf(5) man
# page for more information on changing the MPM.

# prefork MPM: Implements a non-threaded, pre-forking web server
# See: http://httpd.apache.org/docs/2.4/mod/prefork.html
#
# NOTE: If enabling prefork, the httpd_graceful_shutdown SELinux
# boolean should be enabled, to allow graceful stop/shutdown.
#
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

# worker MPM: Multi-Processing Module implementing a hybrid
# multi-threaded multi-process web server
# See: http://httpd.apache.org/docs/2.4/mod/worker.html
#
#LoadModule mpm_worker_module modules/mod_mpm_worker.so

# event MPM: A variant of the worker MPM with the goal of consuming
# threads only for connections with active processing
# See: http://httpd.apache.org/docs/2.4/mod/event.html
#
LoadModule mpm_event_module modules/mod_mpm_event.so
```

如您所见，默认的 MPM 是 `mpm_event`。

Web 服务器的性能和能力在很大程度上取决于 MPM 的选择。

选择一个模块而不是另一个是一项复杂的任务，优化所选的 MPM 模块（客户端数量、查询等）也是如此。

Apache 配置默认假设服务为中等繁忙程度（最多 256 个客户端）。

##### 关于 keepalive 指令

当 `KeepAlive` 指令被禁用时，每个服务器上的资源请求都需要打开一个 TCP 连接，这在网络方面耗时且需要大量系统资源。

当 `KeepAlive` 指令设置为 `On` 时，服务器在 `KeepAlive` 持续期间保持与客户端的连接打开。

这种策略是一种快速见效的策略，因为一个 Web 页面包含多个文件（图片、样式表、JavaScript 等）。

但是，尽可能精确地设置此值很重要：

* 过短的值会对客户不利，
* 过长的值会对服务器资源不利。

每个客户端虚拟主机的 `KeepAlive` 值允许按客户端进行更细粒度的控制。在这种情况下，设置 `KeepAlive` 值直接发生在客户端的 VirtualHost 或代理级别（`ProxyKeepalive` 和 `ProxyKeepaliveTimeout`）。

#### 第 2 部分

第 2 部分设置主服务器使用的值。主服务器响应第 3 部分中任何 Virtualhost 未处理的所有请求。

这些值也用作虚拟站点的默认值。

| 选项                | 信息                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------- |
| `ServerAdmin`       | 指定在某些自动生成的页面（如错误页面）上显示的电子邮件地址。 |
| `ServerName`        | 指定标识服务器的名称。它可以自动发生，但建议显式指定（IP 地址或 DNS 名称）。 |
| `DocumentRoot`      | 指定包含要提供给客户端文件的目录。默认为 /var/www/html/。           |
| `ErrorLog`          | 指定错误文件的路径。                                                               |
| `LogLevel`          | debug, info, notice, warn, error, crit, alert, emerg。                                               |
| `LogFormat`         | 定义一个特定的日志格式。与 CustomLog 指令一起使用。                               |
| `CustomLog`         | 指定访问文件的路径。                                                                        |
| `ServerSignature`   | 在安全部分中介绍。                                                                          |
| `Alias`             | 指定目录树外的一个目录，并通过上下文使其可访问。上下文中最后一个斜杠的存在或不存在很重要。 |
| `Directory`         | 按目录指定行为和访问权限。                                                 |
| `AddDefaultCharset` | 指定发送页面的编码格式（带重音的字符可能被替换为 ?...）。         |
| `ErrorDocument`     | 自定义错误页面。                                                                             |
| `server-status`     | 服务器状态报告。                                                                            |
| `server-info`       | 服务器配置报告。                                                                     |

##### `ErrorLog` 指令

`ErrorLog` 指令定义了要使用的错误日志。

此指令定义了服务器记录其遇到的所有错误的文件名。如果文件路径不是绝对路径，则假定相对于 ServerRoot。

##### `DirectoryIndex` 指令

DirectoryIndex 指令定义了站点的首页。

此指令指定首先加载的文件的名称，该文件将充当站点索引或首页。

语法：

```file
DirectoryIndex display-page
```

不指定完整路径。对文件的搜索在 DocumentRoot 指定的目录中进行。

示例：

```file
DocumentRoot /var/www/html
DirectoryIndex index.php index.htm
```

此指令指定网站索引文件的名称。索引是客户端输入站点 URL（无需输入索引名称）时打开的默认页面。此文件必须在 `DocumentRoot` 指令指定的目录中。

`DirectoryIndex` 指令可以指定多个索引文件名，并用空格分隔。例如，一个默认的索引页面带有动态内容，第二个选择是一个静态页面。

##### `Directory` 指令

Directory 标签用于定义特定于目录的指令。

此标签对一个或多个目录应用权限。目录路径以绝对路径输入。

语法：

```file
<Directory directory-path>
Defining user rights
</Directory>
```

示例：

```file
<Directory /var/www/html/public>
    Require all granted   # 我们允许所有人
</Directory>
```

`Directory` 部分定义了一个指令块，应用于服务器文件系统的一部分。此处的指令仅适用于指定的目录（及子目录）。

此块的语法接受通配符，但最好使用 DirectoryMatch 块。

在以下示例中，无论客户端如何，我们将拒绝对服务器本地硬盘的访问。"/" 目录表示硬盘的根目录。

```file
<Directory />
    Require all denied
</Directory>
```

以下示例显示授权所有客户端访问 `/var/www/html` 发布目录。

```file
<Directory /var/www/html>
    Require all granted
</Directory>
```

当服务器找到 `.htaccess` 文件时，它需要知道放置在该文件中的指令是否有权修改预先存在的配置。`AllowOverride` 指令控制 `Directory` 指令中的授权。当设置为 `none` 时，`.htaccess` 文件被完全忽略。

##### `mod_status`

`mod_status` 显示一个 `/server-status` 或 `/server-info` 页面，总结服务器状态：

```file
<Location /server-status>
    SetHandler server-status
    Require local
</Location>

<Location /server-info>
    SetHandler server-info
    Require local
</Location>
```

请注意，此模块提供的信息不应向您的用户公开。

#### 共享托管（第 3 部分）

通过共享托管，客户以为他们在访问多个服务器。实际上，只有一个服务器和多个虚拟站点。

要设置共享托管，您需要设置虚拟主机：

* 声明多个监听端口
* 声明多个监听 IP 地址（基于 IP 的虚拟托管）
* 声明多个服务器名称（基于名称的虚拟托管）

每个虚拟站点对应不同的目录树结构。

`httpd.conf` 文件的第 3 部分声明这些虚拟主机。

强烈建议您为每个虚拟站点创建一个第 3 部分配置文件，以方便未来的更新。

选择"基于 IP"或"基于名称"的虚拟托管。不建议在生产环境中混合使用这两种解决方案。

* 在独立的配置文件中配置每个虚拟站点
* VirtualHosts 存储在 `/etc/httpd/conf.d/`
* 文件扩展名为 `.conf`

##### `VirtualHost` 指令

`VirtualHost` 指令定义了虚拟主机。

```file
<VirtualHost IP-address[:port]>
    # 如果存在 "NameVirtualHost" 指令
    # 则 "address-IP" 必须与 "NameVirtualHost" 下输入的内容匹配
    # "port" 也是如此。
 ...
 </VirtualHost>
```

如果您使用上面看到的基本指令配置 Apache 服务器，您只能发布一个站点。实际上，您不能使用默认设置发布多个站点：相同的 IP 地址，相同的 TCP 端口，没有主机名或唯一的主机名。

虚拟站点将使我们能够在同一个 Apache 服务器上发布多个网站。您将定义块，每个块描述一个网站。这样，每个站点将有自己的配置。

为方便理解，一个网站通常与一台机器关联。虚拟站点或主机之所以称为虚拟，是因为它们解除了机器和网站之间的关联。

示例 1：

```file
Listen 192.168.0.10:8080
<VirtualHost 192.168.0.10:8080>
  DocumentRoot /var/www/site1/
  ErrorLog /var/log/httpd/site1-error.log
</VirtualHost>

Listen 192.168.0.11:9090
<VirtualHost 192.168.0.11:9090>
  DocumentRoot /var/www/site2/
  ErrorLog /var/log/httpd/site2-error.log
</VirtualHost>
```

基于 IP 的虚拟托管根据接收请求的 IP 地址和端口应用特定的规则。这通常意味着在不同的端口或接口上服务不同的网站。

##### `NameVirtualHost` 指令

`NameVirtualHost` 指令定义了基于名称的虚拟主机。

此指令对于设置基于名称的虚拟主机是必需的。使用此指令，您指定服务器将在其上接收来自基于名称的虚拟主机的请求的 IP 地址。

语法：

```text
NameVirtualHost adresse-IP[:port]
```

示例：

```test
NameVirtualHost 160.210.169.6:80
```

该指令必须位于虚拟站点描述块之前。它指定用于监听客户端对虚拟站点请求的 IP 地址。

要监听服务器所有 IP 地址上的请求，使用 * 字符。

#### 使更改生效

对于每次配置更改，需要使用以下命令重新加载配置：

```bash
sudo systemctl reload httpd
```

#### 手册

一个名为 `httpd-manual` 的包包含一个充当 Apache 用户手册的站点。

```bash
sudo dnf install httpd-manual
sudo systemctl reload httpd
```

安装后，您可以通过 Web 浏览器在 <http://127.0.0.1/manual> 访问手册。

```bash
$ elinks http://127.0.0.1/manual
```

#### `apachectl` 命令

`apachectl` 是 Apache `httpd` 服务器的服务器控制接口。

这是一个非常有用的命令，带有 `-t` 或 `configtest`，它运行配置文件语法测试。

!!! NOTE

    当与 Ansible handler 一起使用时，测试配置非常有用。

### 安全

当使用防火墙（这是一件好事）保护您的服务器时，您可能需要考虑开放它。

```bash
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-service=https
sudo firewall-cmd --reload
```

#### SELinux

默认情况下，如果 SELinux 安全处于活动状态，它会阻止从 `/var/www/` 以外的目录读取站点。

包含站点的目录必须具有安全上下文 `httpd_sys_content_t`。

您可以使用以下命令检查当前上下文：

```bash
* ls -Z /dir
```

使用以下命令添加上下文：

```bash
sudo chcon -vR --type=httpd_sys_content_t /dir
```

它还会阻止打开非标准端口。使用 `semanage` 命令是打开端口的手动操作（默认未安装）。

```bash
sudo semanage port -a -t http_port_t -p tcp 1664
```

#### User 和 Group 指令

`User` 和 `Group` 指令定义了 Apache 管理帐户和组。

历史上，root 运行 Apache，这导致了安全问题。root 始终运行 Apache，但随后其身份被更改。通常是 User `apache` 和 Group `apache`。

永远不要用 ROOT！

Apache 服务器（`httpd` 进程）以 `root` 超级用户帐户启动。每个客户端请求触发一个"子"进程的创建。为了限制风险，这些子进程以权限较低的帐户启动。

User 和 Group 指令声明用于创建子进程的帐户和组。

此帐户和组必须存在于系统中（默认情况下，这在安装过程中发生）。

#### 文件权限

作为一般安全规则，Web 服务器内容不得属于运行服务器的进程。在我们的例子中，文件不应该属于 `apache` 用户和组，因为它对文件夹有写访问权限。

您将内容分配给非特权用户、root 用户和关联的组。顺便也借此机会限制组的访问权限。

```bash
cd /var/www/html
sudo chown -R root:root ./*
sudo find ./ -type d -exec chmod 0755 "{}" \;
sudo find ./ -type f -exec chmod 0644 "{}" \;
```

<!---

### Workshop

#### Task 1 : XXX

#### Task 2 : XXX

#### Task 3 : XXX

#### Task 4 : XXX

### Check your Knowledge

:heavy_check_mark: Simple question? (3 answers)

:heavy_check_mark: Question with multiple answers?

* [ ] Answer 1
* [ ] Answer 2
* [ ] Answer 3
* [ ] Answer 4

## Nginx

In this chapter, you will learn about XXXXXXX.

****

**Objectives**: In this chapter, you will learn how to:

:heavy_check_mark: XXX
:heavy_check_mark: XXX

:checkered_flag: **XXX**, **XXX**

**Knowledge**: :star:
**Complexity**: :star:

**Reading time**: XX minutes

****

### Generalities

### Configuration

### Security

### Workshop

#### Task 1 : XXX

#### Task 2 : XXX

#### Task 3 : XXX

#### Task 4 : XXX

### Check your Knowledge

:heavy_check_mark: Simple question? (3 answers)

:heavy_check_mark: Question with multiple answers?

* [ ] Answer 1
* [ ] Answer 2
* [ ] Answer 3
* [ ] Answer 4

-->
