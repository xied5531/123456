---
title: sftp服务端配置，lftp连接
date: 2020-7-5 16:02:06
tags: ftp
---

## sftp服务端配置

添加用户和用户组：

```
# groupadd sftp
# useradd backup-om -g sftp -s /bin/false
# passwd backup-om
bck@pass1
```

创建chroot目录：

```
# mkdir -p /sftp/om
# usermod -d /sftp/om backup-om
# chown root:root /sftp/om -R
# chmod 755 /sftp/om
```

创建用户根目录：

```
# mkdir -p /sftp/om/root
# chown backup-om:sftp /sftp/om/root -R
# chmod 755 /sftp/om/root
```

配置sshd：

```
# vi /etc/ssh/sshd_config
Subsystem sftp internal-sftp
Match User backup-om
        ChrootDirectory /sftp/om
        ForceCommand internal-sftp
        AllowTcpForwarding no
		
# systemctl restart sshd.service
# systemctl status sshd.service
```

配置安全策略：

```
# vi /etc/sysconfig/selinux
SELINUX=disabled
```

## lftp连接

```
lftp -u backup-om,'bck@pass1' -e 'ls; exit' sftp://x.x.x.x
```

## 问题定位

客户端lftp打开debug日志

`echo 'debug' >> /etc/lftp.conf`

服务端sshd日志

`tailf /var/log/messages | grep sshd`

```
sshd[]: fatal: bad ownership or modes for chroot directory component "/sftp/" [postauth]
```

> 修正Chroot目录/sftp及其父目录的权限，root和>=750
