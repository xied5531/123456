---
title: NFS配置和使用
date: 2020-7-5 16:42:26
tags: nfs
---

## 安装NFS和rpc

```
yum install -y nfs-utils
yum install -y rpcbind
```

## 启动服务和设置开机启动

```
systemctl start rpcbind #启动
systemctl enable rpcbind #开机启动
systemctl start nfs-server nfs-secure-server #启动nfs和nfs安全传输服务
systemctl enable nfs-server nfs-secure-server #开机启动
firewall-cmd --permanent --add-service=nfs #防火墙放通nfs
firewall-cmd --reload #重载防火墙配置
```

## NFS配置和使用

```
mkdir -p /data/share
chmod 755 /data/share
```

```
# vim /etc/exports
/data/share 192.168.0.1/16(rw,sync,insecure,no_subtree_check,no_root_squash)
```

```
# systemctl restart nfs
# showmount -e localhost
Export list for localhost:
/NFS 192.168.0.1/16
```

参数说明：共享目录+空格+允许挂载的客户端的网段范围(各类权限)

```
ro					只读访问
rw					读写访问
sync				所有数据在请求时写入共享
async				nfs 在写入数据前可以响应请求
secure				nfs 通过 1024 以下的安全 TCP/IP 端口发送
insecure			nfs 通过 1024 以上的端口发送
wdelay				如果多个用户要写入 nfs 目录，则归组写入（默认）
no_wdelay			如果多个用户要写入 nfs 目录，则立即写入，当使用 async 时，无需此设置
hide				在 nfs 共享目录中不共享其子目录
no_hide				共享 nfs 目录的子目录
subtree_check		如果共享 /usr/bin 之类的子目录时，强制 nfs 检查父目录的权限（默认）
no_subtree_check	不检查父目录权限
all_squash			共享文件的 UID 和 GID 映射匿名用户 anonymous，适合公用目录
no_all_squash		保留共享文件的 UID 和 GID（默认）
root_squash			root 用户的所有请求映射成如 anonymous 用户一样的权限（默认）
no_root_squash		root 用户具有根目录的完全管理访问权限
anonuid=xxx			指定 nfs 服务器 /etc/passwd 文件中匿名用户的 UID
anongid=xxx			指定 nfs 服务器 /etc/passwd 文件中匿名用户的 GID
```
