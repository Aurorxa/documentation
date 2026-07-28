---
title: 在 Rocky Linux 上部署 Slurm
author: Howard Van Der Wal
contributors: Steven Spencer, Ganna Zhyrnova
ai_contributors: Claude (claude-opus-4-5-20251101)
tested with: 10, 9, 8
tags:
- deploy
- linux
- rocky
- slurm
---

**知识水平**: :star: :star: :star: :star:

**阅读时间**: 30 分钟

## AI 使用声明

人类作者使用 AI 生成了适用于 Rocky Linux 8、9 和 10 的 playbook。人类作者手动测试了这些步骤以确保它们正确运行。作者对本文件的正确性承担全部责任。如果你遇到任何错误，请向 Rocky Linux 文档团队报告。

## 介绍

`slurm` 是 HPC（高性能计算）领域的一项核心技术。它构成了从太空探索到天气预报等各种科学实验的骨干。`slurm` 能够轻松地在由数百甚至数千个节点组成的集群上部署工作负载。

在本指南结尾，你将更好地理解什么是 `slurm`，如何在基本的控制器-计算节点配置上部署 `slurm`，以及如何在计算节点上运行基本作业。

## 背景

`slurm` 是一款用于 Linux 集群的集群管理和作业调度系统。你可以使用它在节点上运行工作负载，从内存使用检查到完整的气动学模拟。

## 前提条件

* 两台 Rocky Linux 虚拟机 / 两台物理 Rocky Linux 节点。

* 用于配置两台虚拟机 / 节点的 Ansible 主机。

## 在 Rocky Linux 8 上设置 Slurm

Rocky Linux 8 仍使用 `cgroups v1`，因此在安装过程中，你必须在主机系统上启用 `cgroups v2`。

部署过程使用 Ansible playbook。

Ansible `hosts` 文件设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/hosts
[rocky-linux8-slurm]
rocky-linux8-slurm-controller-node ansible_ssh_host=192.168.1.120
rocky-linux8-slurm-compute-node ansible_ssh_host=192.168.1.121
EOF
```

Ansible `group_vars` 设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/group_vars/all
---
rocky_linux8_slurm_controller_node_ip: 192.168.1.120
rocky_linux8_slurm_compute_node_ip: 192.168.1.121
EOF
```

Ansible `requirements.yaml`：

```bash
cat << "EOF" | tee ~/requirements.yaml
---
collections:
- name: community.crypto
- name: community.general
- name: community.mysql
- name: ansible.posix
EOF
```

```bash
ansible-galaxy collection install -r requirements.yaml
```

??? note "Rocky Linux 8 Slurm 部署 Playbook"

    ```yaml
    ---
    - name: Slurm setup on a Rocky Linux 8 controller and compute Node
      hosts: rocky-linux8-slurm

      tasks:
      - name: Upgrade all packages on both hosts
        become: true
        ansible.builtin.dnf:
          name:  "*"
          state: latest

      - name: Enable cgroupsv2 in the kernel cmdline parameters
        become: true
        ansible.builtin.shell: grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"

      - name: Reboot the machines
        become: true
        ansible.builtin.reboot:
          reboot_timeout: 3600

      - name: Disable cloud-init management of /etc/hosts so entries persist across reboots
        become: true
        ansible.builtin.lineinfile:
          path: /etc/cloud/cloud.cfg
          regexp: '^\s*manage_etc_hosts'
          line: 'manage_etc_hosts: false'

      - name: Add the IPs and hostnames from each node to /etc/hosts
        become: true
        ansible.builtin.lineinfile:
          path: /etc/hosts
          line: '{{ item }}'
        with_items:
          - '{{ rocky_linux8_slurm_controller_node_ip }} rocky-linux8-slurm-controller-node'
          - '{{ rocky_linux8_slurm_compute_node_ip }} rocky-linux8-slurm-compute-node'

      - name: Install Python 3.8 for compatibility with community.crypto.openssh_keypair
        become: true
        ansible.builtin.dnf:
          name: python38
          state: latest

      - name: Set Python 3.8 as the Ansible interpreter
        ansible.builtin.set_fact:
          ansible_python_interpreter: /usr/bin/python3.8

      - name: Generate an OpenSSH keypair of 4096 rsa so that the keys can be exchanged between the nodes
        become: true
        community.crypto.openssh_keypair:
          path: /root/.ssh/id_rsa

      - name: Fetch the public ssh key from the Controller Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux8_slurm_controller_node_public_key
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Set the Controller Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux8_slurm_controller_node_ssh_public_key: "{{ hostvars['rocky-linux8-slurm-controller-node']['rocky_linux8_slurm_controller_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Controller Node's public key to the Compute Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux8_slurm_controller_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Add the Compute Node to the Controller Node's `known_hosts` file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux8_slurm_compute_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux8_slurm_compute_node_ip) }}"
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Fetch the public ssh key from the Compute Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux8_slurm_compute_node_public_key
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Set the Compute Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux8_slurm_compute_node_ssh_public_key: "{{ hostvars['rocky-linux8-slurm-compute-node']['rocky_linux8_slurm_compute_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Compute Node's public key to the Controller Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux8_slurm_compute_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Add the Controller Node to the Compute Node's known_hosts file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux8_slurm_controller_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux8_slurm_controller_node_ip) }}"
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Pull down the slurm 25.05 tarball
        ansible.builtin.get_url:
          url: https://download.schedmd.com/slurm/slurm-25.05.5.tar.bz2
          dest: /root/slurm-25.05.5.tar.bz2

      - name: Install autofs, munge, nfs and the openmpi utilities
        become: true
        ansible.builtin.dnf:
          name:
            - autofs
            - munge
            - openmpi
            - openmpi-devel
            - nfs-utils
            - rpcbind
          state: latest

      - name: Enable and start the nfs-server service
        become: true
        ansible.builtin.command: systemctl enable --now nfs-server

      - name: Enable and start the rpcbind service
        become: true
        ansible.builtin.command: systemctl enable --now rpcbind

      - name: Add the source IP address from the Controller Node to the Compute Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux8_slurm_controller_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Add the source IP address from the Compute Node to the Controller Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux8_slurm_compute_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Reload the firewall
        become: true
        ansible.builtin.command: firewall-cmd --reload

      - name: Create the nfs directory in the root user's home directory
        ansible.builtin.file:
          path: /root/nfs
          state: directory

      - name: Set up the Controller Node's exports file with the NFS share for the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/exports
          line: /root/nfs {{ rocky_linux8_slurm_compute_node_ip }}(rw,sync,no_subtree_check,no_root_squash)
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Set the NFS share to be mounted on the Compute Node via autofs
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.master
          line: /root/nfs /etc/auto.nfs
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Create the map file for the NFS share on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/auto.nfs
          state: touch
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Add the Controller Node's NFS share to the auto.nfs map on the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.nfs
          line: rocky-linux8-slurm-controller-node:/root/nfs
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Restart the autofs service
        become: true
        ansible.builtin.command: systemctl restart autofs

      - name: Restart the rpcbind service
        become: true
        ansible.builtin.command: systemctl restart rpcbind

      - name: Add the following openmpi PATH lines in .bashrc for the root user
        ansible.builtin.lineinfile:
          path: /root/.bashrc
          line: '{{ item }}'
        with_items:
          - 'PATH=$PATH:/usr/lib64/openmpi/bin'
          - '# LD_LIBRARY_PATH=/usr/lib64/openmpi/lib'

      - name: Source .bashrc
        ansible.builtin.shell: source /root/.bashrc

      - name: Create a machinefile in the root user's NFS directory on the Controller Node
        become: true
        ansible.builtin.file:
          path: /root/nfs/machinefile
          state: touch
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Add the Compute Node's hostname to the Controller Node's machinefile
        ansible.builtin.lineinfile:
          path: /root/nfs/machinefile
          line: rocky-linux8-slurm-compute-node
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Install the mariadb packages on the Controller Node
        become: true
        ansible.builtin.dnf:
          name:
            - mariadb
            - mariadb-devel
            - mariadb-server
            - python38-devel
          state: latest
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Enable and start the mariadb service on the Controller Node
        become: true
        ansible.builtin.command: systemctl enable --now mariadb
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Install the pexpect Python library, so that the mysql_secure_installation script can complete without manual intervention
        become: true
        ansible.builtin.pip:
          name: pexpect
          executable: pip3.8
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Complete the mysql_secure_installation script on the Controller Node
        become: yes
        ansible.builtin.expect:
          command: mysql_secure_installation
          responses:
            'Enter current password for root': ''
            'Set root password': 'n'
            'Remove anonymous users': 'y'
            'Disallow root login remotely': 'y'
            'Remove test database': 'y'
            'Reload privilege tables now': 'y'
          timeout: 1
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Install the python2-PyMySQL package on the Controller Node
        become: yes
        ansible.builtin.dnf:
          name: python2-PyMySQL
          state: latest
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Install the mysqlclient Python library, so that the slurm user can be added to the mariadb database
        become: true
        ansible.builtin.pip:
          name: mysqlclient
          executable: pip3.8
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Create the slurm_acct_db database needed for slurm
        community.mysql.mysql_db:
          name: slurm_acct_db
          state: present
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Create the slurm MySQL user and grant privileges on the Controller Node
        become: yes
        community.mysql.mysql_user:
          name: slurm
          password: '1234'
          host: localhost
          priv: 'slurm_acct_db.*:ALL,GRANT'
          state: present
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Enable the devel repository for access to the munge-devel package
        become: true
        community.general.dnf_config_manager:
          name: devel
          state: enabled

      - name: Install packages to build the slurm RPMs on each node
        become: true
        ansible.builtin.dnf:
          name:
            - autoconf
            - automake
            - dbus-devel
            - libbpf-devel
            - make
            - mariadb-devel
            - munge-devel
            - pam-devel
            - perl-devel
            - perl-ExtUtils-MakeMaker
            - python3
            - readline-devel
            - rpm-build
          state: latest

      - name: Check if the rpmbuild directory already exists
        ansible.builtin.stat:
          path: /root/rpmbuild
        register: rpmbuild_directory

      - name: Build the slurm RPMs on each node
        ansible.builtin.shell: rpmbuild -ta slurm-25.05.5.tar.bz2
        when: not rpmbuild_directory.stat.exists

      - name: Install the slurm RPMs for the Controller Node
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el8.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el8.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmctld-25.05.5-1.el8.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Install the slurm RPMs for the Compute Node
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el8.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el8.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmd-25.05.5-1.el8.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Place the output of the date command from each node and place it into a variable
        ansible.builtin.shell: date
        register: nodes_date_command_output

      - name: Print the output of the variable to std_out to show the nodes are in sync
        ansible.builtin.debug:
          msg: "The current time and date of the node is: {{ nodes_date_command_output }}"

      - name: Create a systemd directory for slurmd on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/systemd/system/slurmd.service.d
          state: directory
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Add Delegate=yes to the slurmd service for cgroups v2 on the Compute Node
        become: true
        ansible.builtin.copy:
          dest: /etc/systemd/system/slurmd.service.d/delegate.conf
          content: |
            [Service]
            Delegate=yes
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Reload systemd daemon
        become: true
        ansible.builtin.command: systemctl daemon-reload
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Create the munge.key using dd (the /usr/sbin/mungekey command is only available for munge version 5.14+) on the Controller Node
        become: true
        ansible.builtin.shell: dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Change the owner of the munge.key to the user munge and the group munge
        become: true
        ansible.builtin.shell: chown munge:munge /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Set only read permissions for the owner (the munge user)
        become: true
        ansible.builtin.shell: chmod 400 /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Copy the munge.key from the Controller Node to the Compute Node
        become: true
        ansible.posix.synchronize:
          src: /etc/munge/munge.key
          dest: /etc/munge/munge.key
          mode: push

        delegate_to: rocky-linux8-slurm-controller-node
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Set the user and group ownership of the munge.key to the munge user on the Compute Node
        become: true
        ansible.builtin.shell: chown munge:munge /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Set only read permissions for the owner (the munge user) on the Compute Node
        become: true
        ansible.builtin.shell: chmod 400 /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Create the /etc/slurm directory on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm
          state: directory

      - name: Create the slurm.conf configuration file on each node
        become: true
        ansible.builtin.copy:
          dest: /etc/slurm/slurm.conf
          content: |
            # slurm.conf file generated by configurator.html.
            # Put this file on all nodes of your cluster.
            # See the slurm.conf man page for more information.
            #
            ClusterName=rocky-linux8-slurm-cluster
            SlurmctldHost=rocky-linux8-slurm-controller-node
            #SlurmctldHost=
            #
            #DisableRootJobs=NO
            #EnforcePartLimits=NO
            #Epilog=
            #EpilogSlurmctld=
            #FirstJobId=1
            #MaxJobId=67043328
            #GresTypes=
            #GroupUpdateForce=0
            #GroupUpdateTime=600
            #JobFileAppend=0
            #JobRequeue=1
            #JobSubmitPlugins=lua
            #KillOnBadExit=0
            #LaunchType=launch/slurm
            #Licenses=foo*4,bar
            #MailProg=/bin/mail
            #MaxJobCount=10000
            #MaxStepCount=40000
            #MaxTasksPerNode=512
            #MpiDefault=
            #MpiParams=ports=#-#
            #PluginDir=
            #PlugStackConfig=
            #PrivateData=jobs
            ProctrackType=proctrack/cgroup
            #Prolog=
            #PrologFlags=
            #PrologSlurmctld=
            #PropagatePrioProcess=0
            #PropagateResourceLimits=
            #PropagateResourceLimitsExcept=
            #RebootProgram=
            ReturnToService=1
            SlurmctldPidFile=/var/run/slurmctld.pid
            SlurmctldPort=6817
            SlurmdPidFile=/var/run/slurmd.pid
            SlurmdPort=6818
            SlurmdSpoolDir=/var/spool/slurmd
            SlurmUser=slurm
            #SlurmdUser=root
            #SrunEpilog=
            #SrunProlog=
            StateSaveLocation=/var/spool/slurmctld
            #SwitchType=
            #TaskEpilog=
            TaskPlugin=task/affinity,task/cgroup
            #TaskProlog=
            #TopologyPlugin=topology/tree
            #TmpFS=/tmp
            #TrackWCKey=no
            #TreeWidth=
            #UnkillableStepProgram=
            #UsePAM=0
            #
            #
            # TIMERS
            #BatchStartTimeout=10
            #CompleteWait=0
            #EpilogMsgTime=2000
            #GetEnvTimeout=2
            #HealthCheckInterval=0
            #HealthCheckProgram=
            InactiveLimit=0
            KillWait=30
            #MessageTimeout=10
            #ResvOverRun=0
            MinJobAge=300
            #OverTimeLimit=0
            SlurmctldTimeout=120
            SlurmdTimeout=300
            #UnkillableStepTimeout=60
            #VSizeFactor=0
            Waittime=0
            #
            #
            # SCHEDULING
            #DefMemPerCPU=0
            #MaxMemPerCPU=0
            #SchedulerTimeSlice=30
            SchedulerType=sched/backfill
            SelectType=select/cons_tres
            #
            #
            # JOB PRIORITY
            #PriorityFlags=
            #PriorityType=priority/multifactor
            #PriorityDecayHalfLife=
            #PriorityCalcPeriod=
            #PriorityFavorSmall=
            #PriorityMaxAge=
            #PriorityUsageResetPeriod=
            #PriorityWeightAge=
            #PriorityWeightFairshare=
            #PriorityWeightJobSize=
            #PriorityWeightPartition=
            #PriorityWeightQOS=
            #
            #
            # LOGGING AND ACCOUNTING
            #AccountingStorageEnforce=0
            #AccountingStorageHost=
            #AccountingStoragePort=
            #AccountingStorageType=
            #AccountingStoreFlags=
            #JobCompHost=
            #JobCompLoc=
            #JobCompParams=
            #JobCompPass=
            #JobCompPort=
            JobCompType=jobcomp/none
            #JobCompUser=
            #JobContainerType=
            JobAcctGatherFrequency=30
            #JobAcctGatherType=
            SlurmctldDebug=info
            SlurmctldLogFile=/var/log/slurmctld.log
            SlurmdDebug=info
            SlurmdLogFile=/var/log/slurmd.log
            #SlurmSchedLogFile=
            #SlurmSchedLogLevel=
            #DebugFlags=
            #
            #
            # POWER SAVE SUPPORT FOR IDLE NODES (optional)
            #SuspendProgram=
            #ResumeProgram=
            #SuspendTimeout=
            #ResumeTimeout=
            #ResumeRate=
            #SuspendExcNodes=
            #SuspendExcParts=
            #SuspendRate=
            #SuspendTime=
            #
            #
            # COMPUTE NODES
            NodeName=rocky-linux8-slurm-compute-node NodeAddr={{ rocky_linux8_slurm_compute_node_ip }} State=UNKNOWN
            PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP

      - name: Add the munge group with a GID of 2001
        ansible.builtin.group:
          name: munge
          gid: 2001
          state: present

      - name: Add the munge user with a UID of 2001
        ansible.builtin.user:
          name: munge
          uid: 2001
          group: munge
          comment: "MUNGE Uid 'N' Gid Emporium"
          home: /var/lib/munge
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Add the slurm group with a GID of 2002
        ansible.builtin.group:
          name: slurm
          gid: 2002
          state: present

      - name: Add the slurm user with a UID of 2002
        ansible.builtin.user:
          name: slurm
          uid: 2002
          group: slurm
          comment: "SLURM Workload Manager"
          home: /var/lib/slurm
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Create the required munge and slurm directories
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          state: directory
        loop:
          - /run/munge
          - /etc/slurm
          - /run/slurm
          - /var/lib/slurm
          - /var/log/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Change ownership of the munge directories to the munge user
        become: true
        ansible.builtin.shell: "chown -R munge: /etc/munge/ /var/log/munge/ /var/lib/munge/ /run/munge/"

      - name: Change ownership of the slurm directories to the slurm user
        become: true
        ansible.builtin.shell: "chown -R slurm: /etc/slurm/ /var/log/slurm/ /var/lib/slurm/ /run/slurm/ /var/spool/slurmd/ /var/spool/slurmctld"

      - name: Set permissions of 0755 on the munge and slurm directories (due to munge version 5.13 being installed)
        become: true
        ansible.builtin.shell: "chmod 0755 /run/munge /etc/slurm/ /var/log/slurm/ /var/lib/slurm/ /run/slurm/ /var/spool/slurmd/ /var/spool/slurmctld"

      - name: Create the /etc/slurm/cgroup.conf file on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm/cgroup.conf
          state: touch

      - name: Set EnableControllers and cgroups v2 in /etc/slurm/cgroup.conf
        become: true
        ansible.builtin.lineinfile:
          path: /etc/slurm/cgroup.conf
          line: "{{ item }}"
        loop:
          - "CgroupPlugin=cgroup/v2"
          - "EnableControllers=yes"

      - name: Enable and start the munge service on both nodes
        become: true
        ansible.builtin.shell: systemctl enable --now munge

      - name: Enable and start the slurmctld service on the Controller Node
        become: true
        ansible.builtin.shell: systemctl enable --now slurmctld
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Enable and start the slurmd service on the Compute Node
        become: true
        ansible.builtin.shell: systemctl enable --now slurmd
        when: inventory_hostname == 'rocky-linux8-slurm-compute-node'

      - name: Run a test srun command on the Controller Node, which should return the hostname of the Compute Node
        ansible.builtin.shell: 'srun -c 1 -n 1 -J crunchy "/bin/hostname"'
        register: srun_hostname_output_from_compute_node
        when: inventory_hostname == 'rocky-linux8-slurm-controller-node'

      - name: Print the output of the variable to std_out
        ansible.builtin.debug:
          msg: "{{ srun_hostname_output_from_compute_node }}"
    ```


### Rocky Linux 8 的 Slurm 安装注意事项

任务 `Enable the devel repository for access to the munge-devel package` 需要使用 Devel^5^ 仓库，以便于部署 `munge`。

`munge` 软件包构建完成后，应使用 `dnf config-manager --set-disabled devel` 重新禁用 Devel 仓库。

需要注意的是，从源代码构建 `munge`^4^ 是最佳实践。

任务 `Create the slurm.conf configuration file on each node` 使用了由 Slurm Configurator Tool^6^ 生成的配置。主要需要注意的行包括：

```text
ClusterName=rocky-linux8-slurm-cluster
SlurmctldHost=rocky-linux8-slurm-controller-node
SlurmUser=slurm
NodeName=rocky-linux8-slurm-compute-node NodeAddr={{ rocky_linux8_slurm_compute_node_ip }} State=UNKNOWN
```

根据你的集群需求修改上述各项值。

出于安全^8^原因，通常最好将 `SlurmUser` 保留为 `slurm`，因为你不想以 `root` 身份运行工作负载。

## 在 Rocky Linux 9 上设置 Slurm

Ansible `hosts` 文件设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/hosts
[rocky-linux9-slurm]
rocky-linux9-slurm-controller-node ansible_ssh_host=<CONTROLLER_IP>
rocky-linux9-slurm-compute-node ansible_ssh_host=<COMPUTE_IP>
[rocky-linux9-slurm:vars]
ansible_user=root
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

Ansible `group_vars` 设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/group_vars/all
---
rocky_linux9_slurm_controller_node_ip: <CONTROLLER_IP>
rocky_linux9_slurm_compute_node_ip: <COMPUTE_IP>
EOF
```

Ansible `requirements.yaml`：

```bash
cat << "EOF" | tee ~/requirements.yaml
---
collections:
- name: community.crypto
- name: community.general
- name: community.mysql
- name: ansible.posix
EOF
```

??? note "Rocky Linux 9 Slurm 部署 Playbook"

    ```yaml
    ---
    # Slurm 25.05 Deployment Playbook for Rocky Linux 9.7
    # Controller-Compute Node Configuration

    - name: Slurm setup on a Rocky Linux 9 controller and compute Node
      hosts: rocky-linux9-slurm

      tasks:
      - name: Upgrade all packages on both hosts
        become: true
        ansible.builtin.dnf:
          name: "*"
          state: latest

      - name: Enable cgroupsv2 in the kernel cmdline parameters
        become: true
        ansible.builtin.shell: grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"

      - name: Reboot the machines
        become: true
        ansible.builtin.reboot:
          reboot_timeout: 3600

      - name: Disable cloud-init management of /etc/hosts so entries persist across reboots
        become: true
        ansible.builtin.lineinfile:
          path: /etc/cloud/cloud.cfg
          regexp: '^\s*manage_etc_hosts'
          line: 'manage_etc_hosts: false'

      - name: Add the IPs and hostnames from each node to /etc/hosts
        become: true
        ansible.builtin.lineinfile:
          path: /etc/hosts
          line: '{{ item }}'
        with_items:
          - '{{ rocky_linux9_slurm_controller_node_ip }} rocky-linux9-slurm-controller-node'
          - '{{ rocky_linux9_slurm_compute_node_ip }} rocky-linux9-slurm-compute-node'

      - name: Generate an OpenSSH keypair so that the keys can be exchanged between the nodes
        become: true
        community.crypto.openssh_keypair:
          path: /root/.ssh/id_rsa

      - name: Fetch the public ssh key from the Controller Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux9_slurm_controller_node_public_key
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Set the Controller Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux9_slurm_controller_node_ssh_public_key: "{{ hostvars['rocky-linux9-slurm-controller-node']['rocky_linux9_slurm_controller_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Controller Node's public key to the Compute Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux9_slurm_controller_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Add the Compute Node to the Controller Node's known_hosts file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux9_slurm_compute_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux9_slurm_compute_node_ip) }}"
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Fetch the public ssh key from the Compute Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux9_slurm_compute_node_public_key
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Set the Compute Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux9_slurm_compute_node_ssh_public_key: "{{ hostvars['rocky-linux9-slurm-compute-node']['rocky_linux9_slurm_compute_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Compute Node's public key to the Controller Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux9_slurm_compute_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Add the Controller Node to the Compute Node's known_hosts file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux9_slurm_controller_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux9_slurm_controller_node_ip) }}"
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Pull down the slurm 25.05 tarball
        ansible.builtin.get_url:
          url: https://download.schedmd.com/slurm/slurm-25.05.5.tar.bz2
          dest: /root/slurm-25.05.5.tar.bz2

      - name: Install autofs, munge, nfs and the openmpi utilities
        become: true
        ansible.builtin.dnf:
          name:
            - autofs
            - munge
            - openmpi
            - openmpi-devel
            - nfs-utils
            - rpcbind
          state: latest

      - name: Enable and start the nfs-server service
        become: true
        ansible.builtin.systemd:
          name: nfs-server
          enabled: yes
          state: started

      - name: Enable and start the rpcbind service
        become: true
        ansible.builtin.systemd:
          name: rpcbind
          enabled: yes
          state: started

      - name: Add the source IP address from the Controller Node to the Compute Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux9_slurm_controller_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Add the source IP address from the Compute Node to the Controller Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux9_slurm_compute_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Reload the firewall
        become: true
        ansible.builtin.command: firewall-cmd --reload

      - name: Create the nfs directory in the root user's home directory
        ansible.builtin.file:
          path: /root/nfs
          state: directory

      - name: Set up the Controller Node's exports file with the NFS share for the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/exports
          line: /root/nfs {{ rocky_linux9_slurm_compute_node_ip }}(rw,sync,no_subtree_check,no_root_squash)
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Export the NFS shares on the Controller Node
        become: true
        ansible.builtin.command: exportfs -a
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Set the NFS share to be mounted on the Compute Node via autofs
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.master
          line: /root/nfs /etc/auto.nfs
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Create the map file for the NFS share on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/auto.nfs
          state: touch
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Add the Controller Node's NFS share to the auto.nfs map on the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.nfs
          line: rocky-linux9-slurm-controller-node:/root/nfs
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Restart the autofs service
        become: true
        ansible.builtin.systemd:
          name: autofs
          state: restarted

      - name: Restart the rpcbind service
        become: true
        ansible.builtin.systemd:
          name: rpcbind
          state: restarted

      - name: Add the following openmpi PATH lines in .bashrc for the root user
        ansible.builtin.lineinfile:
          path: /root/.bashrc
          line: '{{ item }}'
        with_items:
          - 'PATH=$PATH:/usr/lib64/openmpi/bin'
          - '# LD_LIBRARY_PATH=/usr/lib64/openmpi/lib'

      - name: Source .bashrc
        ansible.builtin.shell: source /root/.bashrc

      - name: Create a machinefile in the root user's NFS directory on the Controller Node
        become: true
        ansible.builtin.file:
          path: /root/nfs/machinefile
          state: touch
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Add the Compute Node's hostname to the Controller Node's machinefile
        ansible.builtin.lineinfile:
          path: /root/nfs/machinefile
          line: rocky-linux9-slurm-compute-node
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Reset the mariadb module stream to resolve modular filtering
        become: true
        ansible.builtin.command: dnf module reset mariadb -y
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'
        ignore_errors: yes

      - name: Enable the mariadb module stream
        become: true
        ansible.builtin.command: dnf module enable mariadb -y
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'
        ignore_errors: yes

      - name: Install the mariadb packages on the Controller Node
        become: true
        ansible.builtin.dnf:
          name:
            - mariadb
            - mariadb-devel
            - mariadb-server
            - python3-devel
          state: latest
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Enable and start the mariadb service on the Controller Node
        become: true
        ansible.builtin.systemd:
          name: mariadb
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Install the pexpect Python library for mysql_secure_installation automation
        become: true
        ansible.builtin.pip:
          name: pexpect
          executable: pip3
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Complete the mysql_secure_installation script on the Controller Node
        become: yes
        ansible.builtin.expect:
          command: mysql_secure_installation
          responses:
            'Enter current password for root': ''
            'Switch to unix_socket authentication': 'n'
            'Change the root password': 'n'
            'Set root password': 'n'
            'Remove anonymous users': 'y'
            'Disallow root login remotely': 'y'
            'Remove test database': 'y'
            'Reload privilege tables now': 'y'
          timeout: 30
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'
        ignore_errors: yes

      - name: Install the python3-PyMySQL package on the Controller Node
        become: yes
        ansible.builtin.dnf:
          name: python3-PyMySQL
          state: latest
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Create the slurm_acct_db database needed for slurm
        become: true
        community.mysql.mysql_db:
          name: slurm_acct_db
          state: present
          login_unix_socket: /var/lib/mysql/mysql.sock
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Create the slurm MySQL user and grant privileges on the Controller Node
        become: yes
        community.mysql.mysql_user:
          name: slurm
          password: '1234'
          host: localhost
          priv: 'slurm_acct_db.*:ALL,GRANT'
          state: present
          login_unix_socket: /var/lib/mysql/mysql.sock
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Enable the CRB repository for access to the munge-devel package
        become: true
        ansible.builtin.command: dnf config-manager --set-enabled crb

      - name: Install packages to build the slurm RPMs on each node
        become: true
        ansible.builtin.dnf:
          name:
            - autoconf
            - automake
            - dbus-devel
            - libbpf-devel
            - make
            - mariadb-devel
            - munge-devel
            - pam-devel
            - perl-devel
            - perl-ExtUtils-MakeMaker
            - python3
            - readline-devel
            - rpm-build
          state: latest

      - name: Check if the rpmbuild directory already exists
        ansible.builtin.stat:
          path: /root/rpmbuild
        register: rpmbuild_directory

      - name: Build the slurm RPMs on each node
        ansible.builtin.shell: rpmbuild -ta slurm-25.05.5.tar.bz2
        args:
          chdir: /root
        when: not rpmbuild_directory.stat.exists

      - name: Install the slurm RPMs for the Controller Node
        become: true
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el9.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el9.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmctld-25.05.5-1.el9.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Install the slurm RPMs for the Compute Node
        become: true
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el9.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el9.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmd-25.05.5-1.el9.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Place the output of the date command from each node and place it into a variable
        ansible.builtin.shell: date
        register: nodes_date_command_output

      - name: Print the output of the variable to std_out to show the nodes are in sync
        ansible.builtin.debug:
          msg: "The current time and date of the node is: {{ nodes_date_command_output.stdout }}"

      - name: Create a systemd directory for slurmd on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/systemd/system/slurmd.service.d
          state: directory
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Add Delegate=yes to the slurmd service for cgroups v2 on the Compute Node
        become: true
        ansible.builtin.copy:
          dest: /etc/systemd/system/slurmd.service.d/delegate.conf
          content: |
            [Service]
            Delegate=yes
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Reload systemd daemon
        become: true
        ansible.builtin.command: systemctl daemon-reload
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Create the munge.key using dd on the Controller Node
        become: true
        ansible.builtin.shell: dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Change the owner of the munge.key to the user munge and the group munge
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          owner: munge
          group: munge
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Set only read permissions for the owner (the munge user)
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          mode: '0400'
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Copy the munge.key from the Controller Node to the Compute Node
        become: true
        ansible.posix.synchronize:
          src: /etc/munge/munge.key
          dest: /etc/munge/munge.key
          mode: push
        delegate_to: rocky-linux9-slurm-controller-node
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Set the user and group ownership of the munge.key to the munge user on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          owner: munge
          group: munge
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Set only read permissions for the owner (the munge user) on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          mode: '0400'
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Create the /etc/slurm directory on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm
          state: directory

      - name: Create the slurm.conf configuration file on each node
        become: true
        ansible.builtin.copy:
          dest: /etc/slurm/slurm.conf
          content: |
            # slurm.conf file generated by configurator.html.
            # Put this file on all nodes of your cluster.
            # See the slurm.conf man page for more information.
            #
            ClusterName=rocky-linux9-slurm-cluster
            SlurmctldHost=rocky-linux9-slurm-controller-node
            #SlurmctldHost=
            #
            #DisableRootJobs=NO
            #EnforcePartLimits=NO
            #Epilog=
            #EpilogSlurmctld=
            #FirstJobId=1
            #MaxJobId=67043328
            #GresTypes=
            #GroupUpdateForce=0
            #GroupUpdateTime=600
            #JobFileAppend=0
            #JobRequeue=1
            #JobSubmitPlugins=lua
            #KillOnBadExit=0
            #LaunchType=launch/slurm
            #Licenses=foo*4,bar
            #MailProg=/bin/mail
            #MaxJobCount=10000
            #MaxStepCount=40000
            #MaxTasksPerNode=512
            #MpiDefault=
            #MpiParams=ports=#-#
            #PluginDir=
            #PlugStackConfig=
            #PrivateData=jobs
            ProctrackType=proctrack/cgroup
            #Prolog=
            #PrologFlags=
            #PrologSlurmctld=
            #PropagatePrioProcess=0
            #PropagateResourceLimits=
            #PropagateResourceLimitsExcept=
            #RebootProgram=
            ReturnToService=1
            SlurmctldPidFile=/var/run/slurmctld.pid
            SlurmctldPort=6817
            SlurmdPidFile=/var/run/slurmd.pid
            SlurmdPort=6818
            SlurmdSpoolDir=/var/spool/slurmd
            SlurmUser=slurm
            #SlurmdUser=root
            #SrunEpilog=
            #SrunProlog=
            StateSaveLocation=/var/spool/slurmctld
            #SwitchType=
            #TaskEpilog=
            TaskPlugin=task/affinity,task/cgroup
            #TaskProlog=
            #TopologyPlugin=topology/tree
            #TmpFS=/tmp
            #TrackWCKey=no
            #TreeWidth=
            #UnkillableStepProgram=
            #UsePAM=0
            #
            #
            # TIMERS
            #BatchStartTimeout=10
            #CompleteWait=0
            #EpilogMsgTime=2000
            #GetEnvTimeout=2
            #HealthCheckInterval=0
            #HealthCheckProgram=
            InactiveLimit=0
            KillWait=30
            #MessageTimeout=10
            #ResvOverRun=0
            MinJobAge=300
            #OverTimeLimit=0
            SlurmctldTimeout=120
            SlurmdTimeout=300
            #UnkillableStepTimeout=60
            #VSizeFactor=0
            Waittime=0
            #
            #
            # SCHEDULING
            #DefMemPerCPU=0
            #MaxMemPerCPU=0
            #SchedulerTimeSlice=30
            SchedulerType=sched/backfill
            SelectType=select/cons_tres
            #
            #
            # JOB PRIORITY
            #PriorityFlags=
            #PriorityType=priority/multifactor
            #PriorityDecayHalfLife=
            #PriorityCalcPeriod=
            #PriorityFavorSmall=
            #PriorityMaxAge=
            #PriorityUsageResetPeriod=
            #PriorityWeightAge=
            #PriorityWeightFairshare=
            #PriorityWeightJobSize=
            #PriorityWeightPartition=
            #PriorityWeightQOS=
            #
            #
            # LOGGING AND ACCOUNTING
            #AccountingStorageEnforce=0
            #AccountingStorageHost=
            #AccountingStoragePort=
            #AccountingStorageType=
            #AccountingStoreFlags=
            #JobCompHost=
            #JobCompLoc=
            #JobCompParams=
            #JobCompPass=
            #JobCompPort=
            JobCompType=jobcomp/none
            #JobCompUser=
            #JobContainerType=
            JobAcctGatherFrequency=30
            #JobAcctGatherType=
            SlurmctldDebug=info
            SlurmctldLogFile=/var/log/slurmctld.log
            SlurmdDebug=info
            SlurmdLogFile=/var/log/slurmd.log
            #SlurmSchedLogFile=
            #SlurmSchedLogLevel=
            #DebugFlags=
            #
            #
            # POWER SAVE SUPPORT FOR IDLE NODES (optional)
            #SuspendProgram=
            #ResumeProgram=
            #SuspendTimeout=
            #ResumeTimeout=
            #ResumeRate=
            #SuspendExcNodes=
            #SuspendExcParts=
            #SuspendRate=
            #SuspendTime=
            #
            #
            # COMPUTE NODES
            NodeName=rocky-linux9-slurm-compute-node NodeAddr={{ rocky_linux9_slurm_compute_node_ip }} State=UNKNOWN
            PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP

      - name: Add the munge group with a GID of 2001
        become: true
        ansible.builtin.group:
          name: munge
          gid: 2001
          state: present

      - name: Add the munge user with a UID of 2001
        become: true
        ansible.builtin.user:
          name: munge
          uid: 2001
          group: munge
          comment: "MUNGE Uid 'N' Gid Emporium"
          home: /var/lib/munge
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Add the slurm group with a GID of 2002
        become: true
        ansible.builtin.group:
          name: slurm
          gid: 2002
          state: present

      - name: Add the slurm user with a UID of 2002
        become: true
        ansible.builtin.user:
          name: slurm
          uid: 2002
          group: slurm
          comment: "SLURM Workload Manager"
          home: /var/lib/slurm
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Create the required munge and slurm directories
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          state: directory
        loop:
          - /run/munge
          - /etc/slurm
          - /run/slurm
          - /var/lib/slurm
          - /var/log/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Change ownership of the munge directories to the munge user
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          owner: munge
          group: munge
          recurse: yes
        loop:
          - /etc/munge
          - /var/log/munge
          - /var/lib/munge
          - /run/munge

      - name: Change ownership of the slurm directories to the slurm user
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          owner: slurm
          group: slurm
          recurse: yes
        loop:
          - /etc/slurm
          - /var/log/slurm
          - /var/lib/slurm
          - /run/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Set permissions of 0755 on the munge and slurm directories
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          mode: '0755'
        loop:
          - /run/munge
          - /etc/slurm
          - /var/log/slurm
          - /var/lib/slurm
          - /run/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Create the /etc/slurm/cgroup.conf file on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm/cgroup.conf
          state: touch

      - name: Set EnableControllers and cgroups v2 in /etc/slurm/cgroup.conf
        become: true
        ansible.builtin.lineinfile:
          path: /etc/slurm/cgroup.conf
          line: "{{ item }}"
        loop:
          - "CgroupPlugin=cgroup/v2"
          - "EnableControllers=yes"

      - name: Enable and start the munge service on both nodes
        become: true
        ansible.builtin.systemd:
          name: munge
          enabled: yes
          state: started

      - name: Enable and start the slurmctld service on the Controller Node
        become: true
        ansible.builtin.systemd:
          name: slurmctld
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Enable and start the slurmd service on the Compute Node
        become: true
        ansible.builtin.systemd:
          name: slurmd
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux9-slurm-compute-node'

      - name: Wait for slurmd to register with controller
        ansible.builtin.pause:
          seconds: 10
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Run a test srun command on the Controller Node, which should return the hostname of the Compute Node
        ansible.builtin.shell: 'srun -c 1 -n 1 -J crunchy "/bin/hostname"'
        register: srun_hostname_output_from_compute_node
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'

      - name: Print the output of the variable to std_out
        ansible.builtin.debug:
          msg: "{{ srun_hostname_output_from_compute_node }}"
        when: inventory_hostname == 'rocky-linux9-slurm-controller-node'
    ```


## 在 Rocky Linux 10 上设置 Slurm

Ansible `hosts` 文件设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/hosts
[rocky-linux10-slurm]
rocky-linux10-slurm-controller-node ansible_ssh_host=<CONTROLLER_IP>
rocky-linux10-slurm-compute-node ansible_ssh_host=<COMPUTE_IP>
[rocky-linux10-slurm:vars]
ansible_user=root
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

Ansible `group_vars` 设置：

```bash
cat << "EOF" | sudo tee /etc/ansible/group_vars/all
---
rocky_linux10_slurm_controller_node_ip: <CONTROLLER_IP>
rocky_linux10_slurm_compute_node_ip: <COMPUTE_IP>
EOF
```

Ansible `requirements.yaml`：

```bash
cat << "EOF" | tee ~/requirements.yaml
---
collections:
- name: community.crypto
- name: community.general
- name: community.mysql
- name: ansible.posix
EOF
```

??? note "Rocky Linux 10 Slurm 部署 Playbook"

    ```yaml
    ---
    # Slurm 25.05 Deployment Playbook for Rocky Linux 10.1
    # Controller-Compute Node Configuration
    # Note: Rocky Linux 10 uses cgroups v2 by default, no kernel parameter modification required

    - name: Slurm setup on a Rocky Linux 10 controller and compute Node
      hosts: rocky-linux10-slurm

      tasks:
      - name: Upgrade all packages on both hosts
        become: true
        ansible.builtin.dnf:
          name: "*"
          state: latest

      # Note: cgroups v2 is the default on Rocky Linux 10, no grubby command needed

      - name: Reboot the machines to ensure all updates are applied
        become: true
        ansible.builtin.reboot:
          reboot_timeout: 3600

      - name: Disable cloud-init management of /etc/hosts so entries persist across reboots
        become: true
        ansible.builtin.lineinfile:
          path: /etc/cloud/cloud.cfg
          regexp: '^\s*manage_etc_hosts'
          line: 'manage_etc_hosts: false'

      - name: Add the IPs and hostnames from each node to /etc/hosts
        become: true
        ansible.builtin.lineinfile:
          path: /etc/hosts
          line: '{{ item }}'
        with_items:
          - '{{ rocky_linux10_slurm_controller_node_ip }} rocky-linux10-slurm-controller-node'
          - '{{ rocky_linux10_slurm_compute_node_ip }} rocky-linux10-slurm-compute-node'

      - name: Generate an OpenSSH keypair so that the keys can be exchanged between the nodes
        become: true
        community.crypto.openssh_keypair:
          path: /root/.ssh/id_rsa

      - name: Fetch the public ssh key from the Controller Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux10_slurm_controller_node_public_key
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Set the Controller Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux10_slurm_controller_node_ssh_public_key: "{{ hostvars['rocky-linux10-slurm-controller-node']['rocky_linux10_slurm_controller_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Controller Node's public key to the Compute Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux10_slurm_controller_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Add the Compute Node to the Controller Node's known_hosts file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux10_slurm_compute_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux10_slurm_compute_node_ip) }}"
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Fetch the public ssh key from the Compute Node
        become: true
        ansible.builtin.slurp:
          src: /root/.ssh/id_rsa.pub
        register: rocky_linux10_slurm_compute_node_public_key
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Set the Compute Node's public key as a fact for all hosts
        ansible.builtin.set_fact:
          rocky_linux10_slurm_compute_node_ssh_public_key: "{{ hostvars['rocky-linux10-slurm-compute-node']['rocky_linux10_slurm_compute_node_public_key']['content'] | b64decode | trim }}"

      - name: Add the Compute Node's public key to the Controller Node's authorized_keys file
        become: true
        ansible.posix.authorized_key:
          user: root
          state: present
          key: "{{ rocky_linux10_slurm_compute_node_ssh_public_key }}"
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Add the Controller Node to the Compute Node's known_hosts file
        become: true
        ansible.builtin.known_hosts:
          path: /root/.ssh/known_hosts
          name: "{{ rocky_linux10_slurm_controller_node_ip }}"
          key: "{{ lookup('pipe', 'ssh-keyscan ' + rocky_linux10_slurm_controller_node_ip) }}"
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Pull down the slurm 25.05 tarball
        ansible.builtin.get_url:
          url: https://download.schedmd.com/slurm/slurm-25.05.5.tar.bz2
          dest: /root/slurm-25.05.5.tar.bz2

      - name: Install autofs, munge, nfs and the openmpi utilities
        become: true
        ansible.builtin.dnf:
          name:
            - autofs
            - munge
            - openmpi
            - openmpi-devel
            - nfs-utils
            - rpcbind
          state: latest

      - name: Enable and start the nfs-server service
        become: true
        ansible.builtin.systemd:
          name: nfs-server
          enabled: yes
          state: started

      - name: Enable and start the rpcbind service
        become: true
        ansible.builtin.systemd:
          name: rpcbind
          enabled: yes
          state: started

      - name: Add the source IP address from the Controller Node to the Compute Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux10_slurm_controller_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Add the source IP address from the Compute Node to the Controller Node's firewall
        become: true
        ansible.builtin.command: firewall-cmd --add-rich-rule='rule family="ipv4" source address="{{ rocky_linux10_slurm_compute_node_ip }}" accept' --permanent
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Reload the firewall
        become: true
        ansible.builtin.command: firewall-cmd --reload

      - name: Create the nfs directory in the root user's home directory
        ansible.builtin.file:
          path: /root/nfs
          state: directory

      - name: Set up the Controller Node's exports file with the NFS share for the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/exports
          line: /root/nfs {{ rocky_linux10_slurm_compute_node_ip }}(rw,sync,no_subtree_check,no_root_squash)
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Export the NFS shares on the Controller Node
        become: true
        ansible.builtin.command: exportfs -a
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Set the NFS share to be mounted on the Compute Node via autofs
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.master
          line: /root/nfs /etc/auto.nfs
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Create the map file for the NFS share on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/auto.nfs
          state: touch
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Add the Controller Node's NFS share to the auto.nfs map on the Compute Node
        become: true
        ansible.builtin.lineinfile:
          path: /etc/auto.nfs
          line: rocky-linux10-slurm-controller-node:/root/nfs
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Restart the autofs service
        become: true
        ansible.builtin.systemd:
          name: autofs
          state: restarted

      - name: Restart the rpcbind service
        become: true
        ansible.builtin.systemd:
          name: rpcbind
          state: restarted

      - name: Add the following openmpi PATH lines in .bashrc for the root user
        ansible.builtin.lineinfile:
          path: /root/.bashrc
          line: '{{ item }}'
        with_items:
          - 'PATH=$PATH:/usr/lib64/openmpi/bin'
          - '# LD_LIBRARY_PATH=/usr/lib64/openmpi/lib'

      - name: Source .bashrc
        ansible.builtin.shell: source /root/.bashrc

      - name: Create a machinefile in the root user's NFS directory on the Controller Node
        become: true
        ansible.builtin.file:
          path: /root/nfs/machinefile
          state: touch
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Add the Compute Node's hostname to the Controller Node's machinefile
        ansible.builtin.lineinfile:
          path: /root/nfs/machinefile
          line: rocky-linux10-slurm-compute-node
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Install the mariadb packages on the Controller Node
        become: true
        ansible.builtin.dnf:
          name:
            - mariadb
            - mariadb-devel
            - mariadb-server
            - python3-devel
          state: latest
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Enable and start the mariadb service on the Controller Node
        become: true
        ansible.builtin.systemd:
          name: mariadb
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Install the pexpect Python library for mysql_secure_installation automation
        become: true
        ansible.builtin.pip:
          name: pexpect
          executable: pip3
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Complete the mysql_secure_installation script on the Controller Node
        become: yes
        ansible.builtin.expect:
          command: mysql_secure_installation
          responses:
            'Enter current password for root': ''
            'Switch to unix_socket authentication': 'n'
            'Change the root password': 'n'
            'Set root password': 'n'
            'Remove anonymous users': 'y'
            'Disallow root login remotely': 'y'
            'Remove test database': 'y'
            'Reload privilege tables now': 'y'
          timeout: 30
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'
        ignore_errors: yes

      - name: Install the python3-PyMySQL package on the Controller Node
        become: yes
        ansible.builtin.dnf:
          name: python3-PyMySQL
          state: latest
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Create the slurm_acct_db database needed for slurm
        become: true
        community.mysql.mysql_db:
          name: slurm_acct_db
          state: present
          login_unix_socket: /var/lib/mysql/mysql.sock
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Create the slurm MySQL user and grant privileges on the Controller Node
        become: yes
        community.mysql.mysql_user:
          name: slurm
          password: '1234'
          host: localhost
          priv: 'slurm_acct_db.*:ALL,GRANT'
          state: present
          login_unix_socket: /var/lib/mysql/mysql.sock
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Enable the CRB repository for access to the munge-devel package
        become: true
        ansible.builtin.command: dnf config-manager --set-enabled crb

      - name: Install packages to build the slurm RPMs on each node
        become: true
        ansible.builtin.dnf:
          name:
            - autoconf
            - automake
            - dbus-devel
            - libbpf-devel
            - make
            - mariadb-devel
            - munge-devel
            - pam-devel
            - perl-devel
            - perl-ExtUtils-MakeMaker
            - python3
            - readline-devel
            - rpm-build
          state: latest

      - name: Check if the rpmbuild directory already exists
        ansible.builtin.stat:
          path: /root/rpmbuild
        register: rpmbuild_directory

      - name: Build the slurm RPMs on each node
        ansible.builtin.shell: rpmbuild -ta slurm-25.05.5.tar.bz2
        args:
          chdir: /root
        when: not rpmbuild_directory.stat.exists

      - name: Install the slurm RPMs for the Controller Node
        become: true
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el10.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el10.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmctld-25.05.5-1.el10.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Install the slurm RPMs for the Compute Node
        become: true
        ansible.builtin.dnf:
          name:
            - /root/rpmbuild/RPMS/x86_64/slurm-25.05.5-1.el10.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-perlapi-25.05.5-1.el10.x86_64.rpm
            - /root/rpmbuild/RPMS/x86_64/slurm-slurmd-25.05.5-1.el10.x86_64.rpm
          state: present
          disable_gpg_check: yes
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Place the output of the date command from each node and place it into a variable
        ansible.builtin.shell: date
        register: nodes_date_command_output

      - name: Print the output of the variable to std_out to show the nodes are in sync
        ansible.builtin.debug:
          msg: "The current time and date of the node is: {{ nodes_date_command_output.stdout }}"

      - name: Create a systemd directory for slurmd on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/systemd/system/slurmd.service.d
          state: directory
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Add Delegate=yes to the slurmd service for cgroups v2 on the Compute Node
        become: true
        ansible.builtin.copy:
          dest: /etc/systemd/system/slurmd.service.d/delegate.conf
          content: |
            [Service]
            Delegate=yes
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Reload systemd daemon
        become: true
        ansible.builtin.command: systemctl daemon-reload
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Create the munge.key using dd on the Controller Node
        become: true
        ansible.builtin.shell: dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Change the owner of the munge.key to the user munge and the group munge
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          owner: munge
          group: munge
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Set only read permissions for the owner (the munge user)
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          mode: '0400'
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Copy the munge.key from the Controller Node to the Compute Node
        become: true
        ansible.posix.synchronize:
          src: /etc/munge/munge.key
          dest: /etc/munge/munge.key
          mode: push
        delegate_to: rocky-linux10-slurm-controller-node
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Set the user and group ownership of the munge.key to the munge user on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          owner: munge
          group: munge
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Set only read permissions for the owner (the munge user) on the Compute Node
        become: true
        ansible.builtin.file:
          path: /etc/munge/munge.key
          mode: '0400'
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Create the /etc/slurm directory on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm
          state: directory

      - name: Create the slurm.conf configuration file on each node
        become: true
        ansible.builtin.copy:
          dest: /etc/slurm/slurm.conf
          content: |
            # slurm.conf file generated by configurator.html.
            # Put this file on all nodes of your cluster.
            # See the slurm.conf man page for more information.
            #
            ClusterName=rocky-linux10-slurm-cluster
            SlurmctldHost=rocky-linux10-slurm-controller-node
            #SlurmctldHost=
            #
            #DisableRootJobs=NO
            #EnforcePartLimits=NO
            #Epilog=
            #EpilogSlurmctld=
            #FirstJobId=1
            #MaxJobId=67043328
            #GresTypes=
            #GroupUpdateForce=0
            #GroupUpdateTime=600
            #JobFileAppend=0
            #JobRequeue=1
            #JobSubmitPlugins=lua
            #KillOnBadExit=0
            #LaunchType=launch/slurm
            #Licenses=foo*4,bar
            #MailProg=/bin/mail
            #MaxJobCount=10000
            #MaxStepCount=40000
            #MaxTasksPerNode=512
            #MpiDefault=
            #MpiParams=ports=#-#
            #PluginDir=
            #PlugStackConfig=
            #PrivateData=jobs
            ProctrackType=proctrack/cgroup
            #Prolog=
            #PrologFlags=
            #PrologSlurmctld=
            #PropagatePrioProcess=0
            #PropagateResourceLimits=
            #PropagateResourceLimitsExcept=
            #RebootProgram=
            ReturnToService=1
            SlurmctldPidFile=/var/run/slurmctld.pid
            SlurmctldPort=6817
            SlurmdPidFile=/var/run/slurmd.pid
            SlurmdPort=6818
            SlurmdSpoolDir=/var/spool/slurmd
            SlurmUser=slurm
            #SlurmdUser=root
            #SrunEpilog=
            #SrunProlog=
            StateSaveLocation=/var/spool/slurmctld
            #SwitchType=
            #TaskEpilog=
            TaskPlugin=task/affinity,task/cgroup
            #TaskProlog=
            #TopologyPlugin=topology/tree
            #TmpFS=/tmp
            #TrackWCKey=no
            #TreeWidth=
            #UnkillableStepProgram=
            #UsePAM=0
            #
            #
            # TIMERS
            #BatchStartTimeout=10
            #CompleteWait=0
            #EpilogMsgTime=2000
            #GetEnvTimeout=2
            #HealthCheckInterval=0
            #HealthCheckProgram=
            InactiveLimit=0
            KillWait=30
            #MessageTimeout=10
            #ResvOverRun=0
            MinJobAge=300
            #OverTimeLimit=0
            SlurmctldTimeout=120
            SlurmdTimeout=300
            #UnkillableStepTimeout=60
            #VSizeFactor=0
            Waittime=0
            #
            #
            # SCHEDULING
            #DefMemPerCPU=0
            #MaxMemPerCPU=0
            #SchedulerTimeSlice=30
            SchedulerType=sched/backfill
            SelectType=select/cons_tres
            #
            #
            # JOB PRIORITY
            #PriorityFlags=
            #PriorityType=priority/multifactor
            #PriorityDecayHalfLife=
            #PriorityCalcPeriod=
            #PriorityFavorSmall=
            #PriorityMaxAge=
            #PriorityUsageResetPeriod=
            #PriorityWeightAge=
            #PriorityWeightFairshare=
            #PriorityWeightJobSize=
            #PriorityWeightPartition=
            #PriorityWeightQOS=
            #
            #
            # LOGGING AND ACCOUNTING
            #AccountingStorageEnforce=0
            #AccountingStorageHost=
            #AccountingStoragePort=
            #AccountingStorageType=
            #AccountingStoreFlags=
            #JobCompHost=
            #JobCompLoc=
            #JobCompParams=
            #JobCompPass=
            #JobCompPort=
            JobCompType=jobcomp/none
            #JobCompUser=
            #JobContainerType=
            JobAcctGatherFrequency=30
            #JobAcctGatherType=
            SlurmctldDebug=info
            SlurmctldLogFile=/var/log/slurmctld.log
            SlurmdDebug=info
            SlurmdLogFile=/var/log/slurmd.log
            #SlurmSchedLogFile=
            #SlurmSchedLogLevel=
            #DebugFlags=
            #
            #
            # POWER SAVE SUPPORT FOR IDLE NODES (optional)
            #SuspendProgram=
            #ResumeProgram=
            #SuspendTimeout=
            #ResumeTimeout=
            #ResumeRate=
            #SuspendExcNodes=
            #SuspendExcParts=
            #SuspendRate=
            #SuspendTime=
            #
            #
            # COMPUTE NODES
            NodeName=rocky-linux10-slurm-compute-node NodeAddr={{ rocky_linux10_slurm_compute_node_ip }} State=UNKNOWN
            PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP

      - name: Add the munge group with a GID of 2001
        become: true
        ansible.builtin.group:
          name: munge
          gid: 2001
          state: present

      - name: Add the munge user with a UID of 2001
        become: true
        ansible.builtin.user:
          name: munge
          uid: 2001
          group: munge
          comment: "MUNGE Uid 'N' Gid Emporium"
          home: /var/lib/munge
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Add the slurm group with a GID of 2002
        become: true
        ansible.builtin.group:
          name: slurm
          gid: 2002
          state: present

      - name: Add the slurm user with a UID of 2002
        become: true
        ansible.builtin.user:
          name: slurm
          uid: 2002
          group: slurm
          comment: "SLURM Workload Manager"
          home: /var/lib/slurm
          shell: /sbin/nologin
          create_home: yes
          state: present

      - name: Create the required munge and slurm directories
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          state: directory
        loop:
          - /run/munge
          - /etc/slurm
          - /run/slurm
          - /var/lib/slurm
          - /var/log/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Change ownership of the munge directories to the munge user
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          owner: munge
          group: munge
          recurse: yes
        loop:
          - /etc/munge
          - /var/log/munge
          - /var/lib/munge
          - /run/munge

      - name: Change ownership of the slurm directories to the slurm user
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          owner: slurm
          group: slurm
          recurse: yes
        loop:
          - /etc/slurm
          - /var/log/slurm
          - /var/lib/slurm
          - /run/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Set permissions of 0755 on the munge and slurm directories
        become: true
        ansible.builtin.file:
          path: "{{ item }}"
          mode: '0755'
        loop:
          - /run/munge
          - /etc/slurm
          - /var/log/slurm
          - /var/lib/slurm
          - /run/slurm
          - /var/spool/slurmd
          - /var/spool/slurmctld

      - name: Create the /etc/slurm/cgroup.conf file on both nodes
        become: true
        ansible.builtin.file:
          path: /etc/slurm/cgroup.conf
          state: touch

      - name: Set EnableControllers and cgroups v2 in /etc/slurm/cgroup.conf
        become: true
        ansible.builtin.lineinfile:
          path: /etc/slurm/cgroup.conf
          line: "{{ item }}"
        loop:
          - "CgroupPlugin=cgroup/v2"
          - "EnableControllers=yes"

      - name: Enable and start the munge service on both nodes
        become: true
        ansible.builtin.systemd:
          name: munge
          enabled: yes
          state: started

      - name: Enable and start the slurmctld service on the Controller Node
        become: true
        ansible.builtin.systemd:
          name: slurmctld
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Enable and start the slurmd service on the Compute Node
        become: true
        ansible.builtin.systemd:
          name: slurmd
          enabled: yes
          state: started
        when: inventory_hostname == 'rocky-linux10-slurm-compute-node'

      - name: Wait for slurmd to register with controller
        ansible.builtin.pause:
          seconds: 10
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Run a test srun command on the Controller Node, which should return the hostname of the Compute Node
        ansible.builtin.shell: 'srun -c 1 -n 1 -J crunchy "/bin/hostname"'
        register: srun_hostname_output_from_compute_node
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'

      - name: Print the output of the variable to std_out
        ansible.builtin.debug:
          msg: "{{ srun_hostname_output_from_compute_node }}"
        when: inventory_hostname == 'rocky-linux10-slurm-controller-node'
    ```


## 结论

现在你已经在 Rocky Linux 上运行了 `slurm` 集群，接下来的方向几乎是无限的。以下是一些用于家庭实验室的实用选项：

* 设置 `slurm` 记账功能（accounting）^1^，以便跟踪每次作业的提交情况和消耗的资源。

* 将 Grafana^3^ 部署到你的 `slurm` 集群上，以便直观地确认 GPU 利用率、内存使用情况、正在运行的作业等信息。

* 尝试 Apptainer^2^，使用它拉取容器镜像并提交 `slurm` 作业，运行容器化的应用程序。

* 尝试更高的挑战，在你的 `slurm` 集群上运行 Minecraft 或类似的游戏服务器。

可做的事情数不胜数，而 `slurm` 在 HPC 和数据科学领域被广泛使用。

## 参考文献

1. SchedMD 的 "Accounting and Resource Limits" [https://slurm.schedmd.com/accounting.html](https://slurm.schedmd.com/accounting.html)
2. Apptainer Team 的 "Batch Scheduler / Slurm" [https://apptainer.org/docs/user/main/mpi.html#batch-scheduler-slurm](https://apptainer.org/docs/user/main/mpi.html#batch-scheduler-slurm)
3. Sean Smith 的 "GPU Monitoring with Grafana" [https://swsmith.cc/posts/grafana-slurm.html](https://swsmith.cc/posts/grafana-slurm.html)
4. Chris Dunlap (Dun) 的 "munge Installation Guide" [https://github.com/dun/munge/wiki/Installation-Guide](https://github.com/dun/munge/wiki/Installation-Guide)
5. Rocky Linux Team 的 "Notes on: Devel" [https://wiki.rockylinux.org/rocky/repo/#notes-on-devel](https://wiki.rockylinux.org/rocky/repo/#notes-on-devel)
6. SchedMD 的 "Slurm Configuration Tool" [https://slurm.schedmd.com/configurator.html](https://slurm.schedmd.com/configurator.html)
7. SchedMD 的 "Slurm Documentation" [https://slurm.schedmd.com/documentation.html](https://slurm.schedmd.com/documentation.html)
8. SchedMD 邮件列表中的 "[slurm-users] Why SlurmUser is set to slurm by default?" [https://lists.schedmd.com/pipermail/slurm-users/2018-May/001443.html](https://lists.schedmd.com/pipermail/slurm-users/2018-May/001443.html)
