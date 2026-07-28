---
title: LibreNMS 监控服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - monitoring
  - network
---

!!! Warning "LibreNMS 文档未更新至 Rocky Linux 8 之后"

    虽然 LibreNMS 项目似乎仍然活跃且运行良好——从其 GitHub 站点的提交和更改来看——但安装和运行的操作说明（此处及 LibreNMS 站点上基本未变的说明）在 Rocky Linux 10 上无法按文档所述工作。目前建议在 Rocky Linux 10 上暂缓安装，直到所有更改得到充分调查。

## 简介

网络和系统管理员几乎总是需要某种形式的监控。这可能包括绘制路由器端点的带宽使用图、监控各种服务器上运行的服务状态等等。有许多监控选项，但一个很好的选项——将大多数（即便不是全部）监控组件汇聚于一个平台之下——就是 LibreNMS。

本文档仅作为 LibreNMS 的入门指南。作者将指引您查看该项目的优秀（且广泛的）文档以获取更多选项。作者曾使用过许多其他监控解决方案——Nagios 和 Cacti 就是其中两个——但 LibreNMS 将它们各自提供的功能整合到了一个平台。

安装过程将紧密遵循[官方安装说明](https://docs.librenms.org/Installation/Install-LibreNMS/)。对此过程的说明和细微修改使得本流程更优于那份优秀的文档。

## 前置条件、假设与约定

- 一台运行 Rocky Linux 的服务器或容器（是的，LibreNMS 可以在容器中运行。如果监控量很大，最佳选择是安装在独立硬件上）。所有命令假设是 Rocky Linux 的全新安装。
- 假设：您能够以 root 身份运行命令，或者可以 *sudo* 提权
- 熟悉命令行工具，包括 *vi* 等文本编辑器
- 假设：使用 SNMP v2。如果要使用 SNMP v3，LibreNMS 支持并且可以运行。您需要切换设备上的 SNMP 配置和选项以匹配 v3。
- 此处包含 SELinux 操作流程。作者在实验室中使用的容器默认不包含它。因此，SELinux 操作流程**未经过**实验室测试。
- 在整个文档中，示例使用 *vi* 编辑器。当文档说明保存更改并退出时，请使用 ++shift+colon+w+q+exclam++。
- 此操作流程需要一些故障排除技能，包括日志监控、Web 测试等

## 安装软件包

以 root 用户身份输入以下命令。在开始之前，请注意此安装流程侧重于 *httpd* 而非 *nginx*。如果您更喜欢后者，请遵循 [LibreNMS 安装说明](https://docs.librenms.org/Installation/Install-LibreNMS/) 和其中的指南。

首先，安装 EPEL（企业 Linux 额外软件包）仓库：

```bash
dnf install -y epel-release
```

当前版本的 LibreNMS 要求 PHP 最低版本为 8.1。Rocky Linux 9.0 的 PHP 版本为 8.0。为此较新版本启用第三方仓库（同样适用于 Rocky Linux 8.6）。

安装仓库的版本取决于您运行的 Rocky Linux 版本。这里假设为版本 9，但请根据您实际运行的版本相应更改：

```bash
dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

然后设置 dnf 使用 remi 软件包而非常规软件包：

```bash
dnf module reset php
dnf module enable php:8.1
```

EPEL 和 REMI 仓库都安装完成后，就该安装软件包了：

```bash
dnf install bash-completion cronie fping git httpd ImageMagick mariadb-server mtr net-snmp net-snmp-utils nmap php-fpm php-cli php-common php-curl php-gd php-gmp php-json php-mbstring php-process php-snmp php-xml php-zip php-mysqlnd python3 python3-PyMySQL python3-redis python3-memcached python3-pip python3-systemd rrdtool unzip wget
```

所有这些软件包都代表 LibreNMS 功能集的某个部分。

## 设置 'librenms' 用户

复制并粘贴（或输入）以下内容：

```bash
useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
```

此命令将用户的默认目录设置为 `/opt/librenms`，但 `-M` 选项表示"不创建该目录"。原因是该目录在 LibreNMS 安装时创建。`-r` 表示将此用户设为系统账户，`-s` 表示设置 shell（此处为 "bash"）。

## 下载 LibreNMS 并设置权限

Git 用于下载。您可能对该流程比较熟悉。首先，切换到 `/opt` 目录：

```bash
cd /opt
```

克隆仓库：

```bash
git clone https://github.com/librenms/librenms.git
```

更改目录权限：

```bash
chown -R librenms:librenms /opt/librenms
chmod 771 /opt/librenms
setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
```

`setfacl` 命令代表"设置文件访问控制列表"，是保护目录和文件的另一种方式。

## 以 `librenms` 用户身份安装 PHP 依赖

LibreNMS 中的 PHP 依赖需要使用 `librenms` 用户安装。为此，运行：

```bash
su - librenms
```

输入以下内容：

```bash
./scripts/composer_wrapper.php install --no-dev
```

退出返回 root：

```text
exit
```

### PHP 依赖安装失败的变通方案

LibreNMS 文档说明，当您位于代理服务器后面时，上述流程可能失败。也可能因其他原因失败。因此，稍后提供安装 Composer 的流程。

## 设置时区

您需要确保系统和 PHP 的设置正确。您可以在[此处](https://php.net/manual/en/timezones.php)找到 PHP 的有效时区设置列表。例如，对于中部时区，常见条目是 "America/Chicago"。首先编辑 `php.ini` 文件：

```bash
vi /etc/php.ini
```

找到 `date.timezone` 行并修改。注意该行已被注释，因此删除行首的 ";" 并在 "=" 后添加您的时区。以中部时区示例为例：

```bash
date.timezone = America/Chicago
```

保存更改并退出 `php.ini` 文件。

您还需要确保系统时区正确。以中部时区示例为例：

```bash
timedatectl set-timezone America/Chicago
```

## MariaDB 设置

在启动 LibreNMS 的数据库需求之前，请先执行 [MariaDB 操作流程](../database/database_mariadb-server.md)，特别是"保护 mariadb-server"部分，然后返回此处进行这些特定设置。更改 `mariadb-server.cnf` 文件：

```bash
vi /etc/my.cnf.d/mariadb-server.cnf
```

将以下行添加到 "[Mysqld]" 部分：

```bash
innodb_file_per_table=1
lower_case_table_names=0
```

然后启用并重启 `mariadb` 服务器：

```bash
systemctl enable mariadb
systemctl restart mariadb
```

以 root 用户身份访问 `mariadb`。请记住使用之前执行"保护 mariadb-server"部分时创建的密码：

```sql
mysql -u root -p
```

为 LibreNMS 进行一些特定更改。使用以下命令时，请记得将密码 "password" 更改为安全的内容，并将其记录在安全的位置。

在 `mysql` 提示符下运行：

```sql
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
```

输入 "exit" 退出 `mariadb`。

## 配置 PHP-FPM

除文件路径外，此部分与官方文档相同。首先，复制 `www.conf`：

更改 `librenms.conf` 文件：

```bash
cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/librenms.conf
vi /etc/php-fpm.d/librenms.conf
```

将 "[www]" 更改为 "[librenms]"

将用户和组更改为 "librenms"：

```bash
user = librenms
group = librenms
```

更改 "listen" 行以反映唯一名称：

```bash
listen = /run/php-fpm-librenms.sock
```

保存更改并退出文件。如果这是本机上运行的唯一 Web 服务，可以删除之前复制的旧 `www.conf` 文件：

```bash
rm -f /etc/php-fpm.d/www.conf
```

## 配置 `httpd`

首先创建此文件：

```bash
vi /etc/httpd/conf.d/librenms.conf
```

在该文件中输入以下内容：

```bash
<VirtualHost *:80>
  DocumentRoot /opt/librenms/html/
  ServerName  librenms.example.com

  AllowEncodedSlashes NoDecode
  <Directory "/opt/librenms/html/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>

  # Enable http authorization headers
  <IfModule setenvif_module>
    SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
  </IfModule>

  <FilesMatch ".+\.php$">
    SetHandler "proxy:unix:/run/php-fpm-librenms.sock|fcgi://localhost"
  </FilesMatch>
</VirtualHost>
```

应移除旧的默认站点 `welcome.conf`：

```bash
rm /etc/httpd/conf.d/welcome.conf
```

启用 `httpd` 和 `php-fpm`：

```bash
systemctl enable --now httpd
systemctl enable --now php-fpm
```

## SELinux

如果您不打算使用 SELinux，请跳到下一节。如果您的 LibreNMS 运行在不支持容器级别 SELinux 的容器上，或者默认不包含 SELinux，这也可能适用于您。

要配置 SELinux 的所有内容，需要安装一个额外的软件包：

```bash
dnf install policycoreutils-python-utils
```

### 配置 LibreNMS 上下文

您需要设置以下上下文，以便 LibreNMS 与 SELinux 正常工作：

```bash
semanage fcontext -a -t httpd_sys_content_t '/opt/librenms/html(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/opt/librenms/(logs|rrd|storage)(/.*)?'
restorecon -RFvv /opt/librenms
setsebool -P httpd_can_sendmail=1
setsebool -P httpd_execmem 1
chcon -t httpd_sys_rw_content_t /opt/librenms/.env
```

### 允许 `fping`

在任意位置创建一个名为 `http_fping.tt` 的文件。位置不重要。安装方式见下文。该文件的内容为：

```bash
module http_fping 1.0;

require {
type httpd_t;
class capability net_raw;
class rawip_socket { getopt create setopt write read };
}

#============= httpd_t ==============
allow httpd_t self:capability net_raw;
allow httpd_t self:rawip_socket { getopt create setopt write read };
```

使用以下命令安装此文件：

```bash
checkmodule -M -m -o http_fping.mod http_fping.tt
semodule_package -o http_fping.pp -m http_fping.mod
semodule -i http_fping.pp
```

如果遇到问题且怀疑可能与 SELinux 问题有关，请运行：

```bash
audit2why < /var/log/audit/audit.log
```

## `firewalld` 配置

`firewalld` 操作说明遵循官方文档。

`firewalld` 允许规则的命令如下：

```bash
firewall-cmd --zone public --add-service http --add-service https
firewall-cmd --permanent --zone public --add-service http --add-service https
```

作者对这种简化的 `firewalld` 规则集存在疑虑。此规则允许您的 Web 服务对全世界开放，但这对监控服务器而言是您想要的吗？

通常情况**并非**如此。如果您希望对 `firewalld` 采用更精细的方法，请查阅[此文档](../security/firewalld.md)，然后相应地更改 `firewalld` 规则。

## 为 `lnms` 命令启用符号链接和 Tab 自动补全

首先，需要为 `lnms` 命令创建一个符号链接，使其可以在任何位置运行：

```bash
ln -s /opt/librenms/lnms /usr/bin/lnms
```

接下来，配置自动补全：

```bash
cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
```

## 配置 `snmpd`

*SNMP* 代表"简单网络管理协议"，许多监控程序使用它来拉取数据。此处使用的版本 2 需要与环境相关的"community string（团体字符串）"。

将此"community string"分配给要监控的网络设备，以便 `snmpd`（此处的 "d" 代表守护进程）能够找到它。如果您的网络不是新建的，您可能已经有正在使用的"community string"。

从 LibreNMS 复制 `snmpd.conf` 文件：

```bash
cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
```

编辑此文件，将社区字符串从 "RANDOMSTRINGGOESHERE" 更改为您正在使用或将使用的任何字符串。示例中为 "LABone"：

```bash
vi /etc/snmp/snmpd.conf
```

更改此行：

```bash
com2sec readonly  default         RANDOMSTRINGGOESHERE
```

为：

```bash
com2sec readonly  default         LABone
```

保存更改并退出。

## 使用 cron 作业自动化

运行以下命令设置 cron 作业：

```bash
cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
```

即使暂时没有可轮询的设备，poller 也必须在运行 Web 设置流程之前至少运行一次。这能为后续验证部分出现 poller 错误时节省故障排除时间。

Poller 以 "librenms" 用户身份运行，虽然可以切换到此用户并运行 cron 文件，但让 poller 自行运行会更好。确保至少经过 5 分钟让 cron 运行，然后继续"Web 设置"部分。

## 日志轮转

LibreNMS 会随着时间推移生成大量日志。需要设置日志轮转以节省磁盘空间。为此，运行此命令：

```bash
cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
```

## 安装 Composer（变通方案）

PHP Composer 是当前安装所必需的（在之前的流程中提到过）。如果您之前运行的安装失败，则需要进行此操作。

在开始之前，需要将当前版本的 `php` 二进制文件链接到 PATH 中的某个位置。此流程使用了 REMI 安装来获取正确版本的 PHP，但它未安装在 PATH 内。

这可以通过符号链接来修复，这将使您在运行剩余步骤时更加轻松：

```bash
ln -s /opt/remi/php81/root/usr/bin/php /usr/bin/php
```

访问 [Composer 网站](https://getcomposer.org/download/) 并确保以下步骤没有变化。然后在机器上的某个位置运行这些命令。完成后将移动 composer：

```php
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

将其移动到 PATH 中的某个位置。使用 `/usr/local/bin/`：

```bash
mv composer.phar /usr/local/bin/composer
```

## Web 设置

所有组件安装和配置完毕后，下一步是通过 Web 完成安装。在实验版本中，没有设置主机名。要完成设置，需要通过 IP 地址访问 Web 服务器。

实验用机器的 IP 是 192.168.1.140。在 Web 浏览器中导航到以下地址以完成安装：

`http://192.168.1.140/librenms`

如果一切正常，将重定向到安装前检查。如果全部显示为绿色，则可以继续。

![LibreNMS 安装前检查](../images/librenms_prechecks.png)

LibreNMS 标志下方有四个按钮。左边第一个按钮用于安装前检查。旁边的按钮用于数据库。您需要输入之前为数据库用户 "librenms" 设置的密码。

如果您一直按照步骤操作，应该已将其保存在安全的位置。点击"Database"按钮。这里只需要"User"和"Password"。填写后，点击"Check Credentials"按钮。

![LibreNMS 数据库](../images/librenms_configure_database.png)

如果返回绿色，点击"Build Database"按钮。

![LibreNMS 数据库状态](../images/librenms_configure_database_status.png)

"Create Admin User"按钮现在将变为可用。点击它。接下来提示输入管理员用户名。实验中为 "admin"。为该用户创建密码。

确保密码安全并将其记录在安全的位置（如密码管理器中）。还需要添加管理用户的电子邮件地址。完成后，点击"Add User"按钮。

![LibreNMS 管理用户](../images/librenms_administrative_user.png)

现在将看到"Finish Install"屏幕。只剩下一个项目要完成安装，即要求您"validate your install"。

点击链接。将重定向到登录页面。使用管理用户和密码登录。

## 添加设备

再次强调，假设之一是您正在使用 SNMP v2。请记住，每个添加的设备必须是您 community string 的成员。作者在此使用两个设备示例：一台 Ubuntu 工作站和一台 CentOS 服务器。

您可能还有托管交换机、路由器和其他设备要添加。根据经验，添加交换机和路由器比添加工作站和服务器更容易。

### Ubuntu 工作站设置

首先，在工作站上安装 `snmpd` 并更新软件包以确保安全：

```bash
sudo update && sudo apt-get upgrade && sudo apt-get install snmpd
```

接下来，需要更改 `snmpd.conf` 文件：

```bash
sudo vi /etc/snmpd/snmpd.conf
```

找到描述工作站的行并将其更改为可识别工作站的信息：

```bash
sysLocation    Desktop
sysContact     Username <user@mydomain.com>
```

在 Ubuntu 上安装 `snmpd` 时，它仅绑定到本地地址。它不会在您的机器 IP 地址上监听。这将导致 LibreNMS 无法连接到它。需要注释掉此行：

```bash
agentaddress  127.0.0.1,[::1]
```

添加新行：（在此示例中，您的工作站 IP 地址为 192.168.1.122，设置的 UDP 端口为 "161"）

```bash
agentAddress udp:127.0.0.1:161,udp:192.168.1.122:161
```

需要指定只读访问社区字符串。找到以下行并按所示注释掉：

```bash
#rocommunity public default -V systemonly
#rocommunity6 public default -V systemonly
```

添加一行：

```bash
rocommunity LABone
```

保存更改并退出。

启用并启动 `snmpd`：

```bash
sudo systemctl enable snmpd
sudo systemctl start snmpd
```

如果您在内部工作站上运行防火墙，则需要更改防火墙以允许来自监控服务器或网络的 UDP 流量。LibreNMS 还希望能够 "ping" 您的设备。确保来自服务器的 ICMP 端口 8 未被过滤。

### CentOS 或 Rocky Linux 服务器设置

假设您是 root 或可以 `sudo` 提权。需要安装一些软件包：

```bash
dnf install net-snmp net-snmp-utils
```

创建 `snmpd.conf` 文件。与其费力浏览包含的文件，不如移动此文件以重命名，然后创建一个全新的空文件：

```bash
mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
```

以及：

```bash
vi /etc/snmp/snmpd.conf
```

将以下内容复制到新文件中：

```bash
# Map 'LABone' community to the 'AllUser'
# sec.name source community
com2sec AllUser default LABone
# Map 'ConfigUser' to 'ConfigGroup' for SNMP Version 2c
# Map 'AllUser' to 'AllGroup' for SNMP Version 2c
# sec.model sec.name
group AllGroup v2c AllUser
# Define 'SystemView', which includes everything under .1.3.6.1.2.1.1 (or .1.3.6.1.2.1.25.1)
# Define 'AllView', which includes everything under .1
# incl/excl subtree
view SystemView included .1.3.6.1.2.1.1
view SystemView included .1.3.6.1.2.1.25.1.1
view AllView included .1
# Give 'ConfigGroup' read access to objects in the view 'SystemView'
# Give 'AllGroup' read access to objects in the view 'AllView'
# context model level prefix read write notify
access AllGroup "" any noauth exact AllView none none
```

CentOS 和 Rocky 使用映射约定来指导事务。上述文件带有注释来定义发生了什么，但不包含原始文件中所有的杂乱内容。

完成更改后，保存并退出文件。

启用并启动 `snmpd`：

```bash
systemctl enable snmpd
systemctl start snmpd
```

#### 防火墙

如果您正在运行服务器，那么您**确实**在运行防火墙，对吗？如果您正在运行 `firewalld`，假设您使用 "trusted" 区域，只想允许来自监控服务器 192.168.1.140 的所有流量：

```bash
firewall-cmd --zone=trusted --add-source=192.168.1.140 --permanent
```

如果 "trusted" 区域不适合您的环境，请根据需要进行更改。在添加规则之前，请考虑其影响。

## 在 LibreNMS 中添加设备

在示例设备配置为接受来自 LibreNMS 服务器的 SNMP 流量后，下一步是将这些设备添加到 LibreNMS。在 LibreNMS 的 Web 界面打开后，点击添加设备：

![LibreNMS 添加设备](../images/librenms_add_device.png)

输入测试设备使用的信息。首先输入 Ubuntu 工作站的 IP。示例中为 192.168.1.122。在 "Community" 字段中添加 "LABone" 社区字符串。

点击"Add Device"按钮。假设一切正确，此过程将成功完成。

如果遇到"添加失败"错误，请检查工作站的 SNMP 设置或其防火墙（如果存在）。对 CentOS 服务器重复"Add Device"过程。

## 获取告警

如开头所述，本文档仅让您入门 LibreNMS。存在大量额外的配置项、广泛的 API（应用程序编程接口）、提供大量传递选项（称为"Transports"）的告警系统等等。

本文档不包含任何告警规则创建流程。取而代之的是编辑预配置的名为"Device Down! Due to no ICMP response"的内置告警规则。对于"Transports"使用"Mail"，即电子邮件。注意，告警不限于此。

邮件必须能够正常发送才能使用电子邮件作为传输方式。使用此 [Postfix 流程](../email/postfix_reporting.md) 使其运行。

### 传输方式

需要一种发送告警的方式。如前所述，LibreNMS 支持大量的传输方式。此处使用定义为"Mail"传输方式的电子邮件告警。设置传输方式：

1. 进入仪表板
2. 鼠标悬停在"Alerts"上
3. 下拉到"Alert Transports"并点击
4. 点击"Create alert transport"按钮（注意"Create transport group"按钮。可以使用它将告警发送给多个人）
5. 在"Transport name:"字段中，输入"Alert By Email"
6. 在"Transport type:"字段中，使用下拉菜单选择"Mail"
7. 确保"Default alert:"字段为"On"
8. 在"Email:"字段中，输入管理员的电子邮件地址

### 将设备组织到组中

设置告警的最佳方式是将设备按逻辑组织。目前，设备中有一台工作站和一台服务器。通常不会像这里一样将两者合并。

此示例也是冗余的，因为存在一个可用于此用途的"All Devices"组。要设置设备组：

1. 进入仪表板
2. 鼠标悬停在"Devices"上
3. 下拉到"Manage Groups"并点击
4. 点击"+ New Device Group"按钮
5. 在"Name"字段中，输入"ICMP Group"
6. 在描述字段中输入您认为有助于描述组的内容
7. 将"Type"字段从"Dynamic"更改为"Static"
8. 将每个设备添加到"Select Devices"字段并保存更改

### 设置告警规则

接下来配置告警规则。默认情况下，LibreNMS 已为您创建了多个告警规则：

1. 进入仪表板
2. 鼠标悬停在"Alerts"上
3. 下拉到"Alert Rules"并点击
4. 此处的顶部活动规则将是"Device Down! Due to no ICMP response."将鼠标移到"Action"（最右侧列），点击铅笔图标编辑规则。
5. 保持顶部所有字段为默认值。在"Match devices, groups and locations list:"字段中，点击字段内部。
6. 从列表中选择"ICMP Group"
7. 确保"All devices except in list:"字段为"Off"
8. 点击"Transports:"字段内部，选择"Mail: Alert By Email"并保存规则。

保存前，您的规则将是：

![LibreNMS 告警规则](../images/librenms_alert_rule.png)

现在，这两个设备若宕机，将通过电子邮件告警，恢复时也会通知。

## 总结

LibreNMS 是一个强大的监控工具，在一个应用中提供完整的功能集。本文档仅触及了其能力的皮毛。一些较简单的界面未在此展示。

当您添加设备时，假设所有 SNMP 属性都设置正确，您将开始在每个设备上接收到带宽、内存利用率和 CPU 利用率图。此实验尚未展示除"Mail"之外的丰富传输方式。

本文档已为您展示了足够的内容，以便您良好地开始监控您的环境。掌握 LibreNMS 的所有元素需要一些时间。您应访问该项目的[优秀文档](https://docs.librenms.org/)以获取更多信息。
