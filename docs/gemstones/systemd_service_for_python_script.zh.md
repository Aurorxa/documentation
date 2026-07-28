---
title: Systemd 服务 - Python 脚本
author: Antoine Le Morvan
contributors: Steven Spencer
tested_with: 8.6, 9.0
tags:
  - python
  - systemd
  - cron
---

# 用于 Python 脚本的 `systemd` 服务

如果您像许多系统管理员一样，喜欢使用 `* * * * * /I/launch/my/script.sh` 启动 cron 脚本，那么这篇文章应该能让您想到另一种方式，利用 `systemd` 提供的所有功能和便利来完成。

我们将编写一个 Python 脚本，它提供一个持续循环来执行您定义的操作。

我们将了解如何将此脚本作为 `systemd` 服务运行、在 journalctl 中查看日志，以及观察脚本崩溃时会发生什么。

## 前置条件

首先安装脚本使用 journalctl 所需的一些 Python 依赖项：

```bash
shell > sudo dnf install python36-devel systemd-devel
shell > sudo pip3 install systemd
```

## 编写脚本

考虑以下脚本 `my_service.py`：

```python
"""
Sample script to run as script
"""
import time
import logging
import sys
from systemd.journal import JournaldLogHandler

# Get an instance of the logger
LOGGER = logging.getLogger(__name__)

# Instantiate the JournaldLogHandler to hook into systemd
JOURNALD_HANDLER = JournaldLogHandler()
JOURNALD_HANDLER.setFormatter(logging.Formatter(
    '[%(levelname)s] %(message)s'
))

# Add the journald handler to the current logger
LOGGER.addHandler(JOURNALD_HANDLER)
LOGGER.setLevel(logging.INFO)

class Service(): # pylint: disable=too-few-public-methods
    """
    Launch an infinite loop
    """
    def __init__(self):

        duration = 0

        while True:
            time.sleep(60)
            duration += 60
            LOGGER.info("Total duration: %s", str(duration))
            # will failed after 4 minutes
            if duration > 240:
                sys.exit(1)

if __name__ == '__main__':

    LOGGER.info("Starting the service")
    Service()
```

我们首先实例化必要的变量以将日志发送到 journald。然后脚本启动一个无限循环并暂停 60 秒（这是 cron 执行的最小间隔，因此我们可以突破这一限制）。

!!! Note

    我个人以更高级的形式使用此脚本，它持续查询数据库并基于通过 rundeck API 检索的信息执行任务。

## Systemd 集成

现在我们有了一个可以作为您想象力基础的脚本，我们可以将其安装为 systemd 服务。

创建此文件 `my_service.service` 并将其复制到 `/etc/systemd/system/`。

```bash
[Unit]
Description=My Service
After=multi-user.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/python3 my_service.py
WorkingDirectory=/opt/my_service/

StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=my_service

[Install]
WantedBy=multi-user.target
```

如您所见，脚本从 `/opt/my_service/` 启动。记住要调整脚本的路径和 syslog 标识符。

启动并启用新服务：

```bash
shell > sudo systemctl daemon-reload
shell > sudo systemctl enable my_service.service
shell > sudo systemctl start my_service.service
```

## 测试

现在我们可以通过 journalctl 查看日志：

```bash
shell > journalctl -f -u my_service
oct. 14 11:07:48 rocky8 systemd[1]: Started My Service.
oct. 14 11:07:49 rocky8 __main__[270267]: [INFO] Starting the service
oct. 14 11:08:49 rocky8 __main__[270267]: [INFO] Total duration: 60
oct. 14 11:09:49 rocky8 __main__[270267]: [INFO] Total duration: 120
```

现在让我们看看脚本崩溃时会发生什么：

```bash
shell > ps -elf | grep my_service
4 S root      270267       1  0  80   0 - 82385 -      11:07 ?        00:00:00 /usr/bin/python3 my_service.py
shell > sudo kill -9 270267
```

```bash
shell > journalctl -f -u my_service
oct. 14 11:10:49 rocky8 __main__[270267]: [INFO] Total duration: 180
oct. 14 11:11:49 rocky8 __main__[270267]: [INFO] Total duration: 240
oct. 14 11:12:19 rocky8 systemd[1]: my_service.service: Main process exited, code=killed, status=9/KILL
oct. 14 11:12:19 rocky8 systemd[1]: my_service.service: Failed with result 'signal'.
oct. 14 11:12:19 rocky8 systemd[1]: my_service.service: Service RestartSec=100ms expired, scheduling restart.
oct. 14 11:12:19 rocky8 systemd[1]: my_service.service: Scheduled restart job, restart counter is at 1.
oct. 14 11:12:19 rocky8 systemd[1]: Stopped My Service.
oct. 14 11:12:19 rocky8 systemd[1]: Started My Service.
oct. 14 11:12:19 rocky8 __main__[270863]: [INFO] Starting the service
```

我们也可以等待 5 分钟让脚本自行崩溃：（生产环境请移除此部分）

```bash
oct. 14 11:16:02 rocky8 systemd[1]: Started My Service.
oct. 14 11:16:03 rocky8 __main__[271507]: [INFO] Starting the service
oct. 14 11:17:03 rocky8 __main__[271507]: [INFO] Total duration: 60
oct. 14 11:18:03 rocky8 __main__[271507]: [INFO] Total duration: 120
oct. 14 11:19:03 rocky8 __main__[271507]: [INFO] Total duration: 180
oct. 14 11:20:03 rocky8 __main__[271507]: [INFO] Total duration: 240
oct. 14 11:21:03 rocky8 __main__[271507]: [INFO] Total duration: 300
oct. 14 11:21:03 rocky8 systemd[1]: my_service.service: Main process exited, code=exited, status=1/FAILURE
oct. 14 11:21:03 rocky8 systemd[1]: my_service.service: Failed with result 'exit-code'.
oct. 14 11:21:03 rocky8 systemd[1]: my_service.service: Service RestartSec=100ms expired, scheduling restart.
oct. 14 11:21:03 rocky8 systemd[1]: my_service.service: Scheduled restart job, restart counter is at 1.
oct. 14 11:21:03 rocky8 systemd[1]: Stopped My Service.
oct. 14 11:21:03 rocky8 systemd[1]: Started My Service.
oct. 14 11:21:03 rocky8 __main__[271993]: [INFO] Starting the service
```

如您所见，systemd 的 restart 功能非常有用。

## 结论

`systemd` 和 `journald` 为我们提供了工具，使我们能够轻松地创建稳健而强大的脚本，足以替代我们可靠的老 cron 脚本。

希望这个解决方案对您有用。
