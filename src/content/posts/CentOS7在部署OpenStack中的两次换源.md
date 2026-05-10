---
title: CentOS7在部署OpenStack中的两次换源
published: 2026-5-10
---

# CentOS7 在部署 OpenStack 中的两次换源

随着 CentOS7 在 2024 年 6 月 30 日正式宣布停止维护，官方不再提供更新和安全补丁，从而导致其本身的 yum 包管理器已无法使用，进而无法安装所必须的软件包，所以需要进行换源使 yum 源可用。

目前来说如果不换源你可能会出现以下情况：

```bash
[root@server ~]# yum update
Loaded plugins: fastestmirror
https://mirrors.centos.org/centos/7/os/x86_64/repodata/repomd.xml: [Errno 14] HTTPS Error 404 - Not Found
```

这正是 yum 源无法使用导致的问题。

---

## 换源方法

下面两种方法选择其一即可。

### 1. 使用开源脚本换源

社区维护了一个 GNU/Linux 更换系统软件源脚本及 Docker 安装与换源脚本，使其换源更简单，并兼容大部分常见的 Linux 发行版。

使用方法也很简单，你只需要在 root 用户下使用这行命令：

```bash
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

稍作等待后就会弹出 TUI 界面，你可以选择各个不同的镜像源来更换（这里我建议使用阿里源），一路回车即可换源。

### 2. 手动换源

```bash
# 删除原有 yum 源或备份（这里不用原本的了直接删掉就行）
cd /etc/yum.repos.d/
rm -rf /etc/yum.repos.d/*

# 切换 yum 源为阿里云
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

yum makecache
```

完成后即可正常使用 yum 包管理器安装软件。

---

## 第二次换源（部署 OpenStack）

由于 CentOS 官网已不再维护 CentOS7，所以在部署 OpenStack 中我们需要**第二次换源**，在此之前你需要确保第一次换源成功，yum 包管理器处于可用状态。

```bash
# 准备所需软件库
# OpenStack 版本列表
yum -y install centos-release-openstack-train
```

> 在运行完这行命令后你的 yum 包管理器会失效，不用担心，这是正常的。

你需要进入到 yum 包管理器的配置文件目录当中：

```bash
cd /etc/yum.repos.d/
```

使用 `ls -ll` 查看当前目录文件，你会看到：

- `CentOS-OpenStack-train.repo`
- `CentOS-Ceph-Nautilus.repo`
- `CentOS-NFS-Ganesha-28.repo`
- `CentOS-QEMU-EV.repo`

你需要修改这四个文件：

1. 将 `baseurl` 前的 `#` 号删除
2. 将网址改为 `mirrors.aliyun.com`
3. 用 `#` 号注释掉 `mirrorlist` 行的代码

> **注意**：`mirrors.aliyun.com` 链接要修改正确，链接里面的 `mirror` 需要 **+s** 改为 `mirrors`。

### 示例：`CentOS-OpenStack-train.repo`

```ini
[centos-openstack-train]
name=CentOS-7 - OpenStack train
baseurl=http://mirrors.aliyun.com/$contentdir/$releasever/cloud/$basearch/openstack-train/   # 将这一行取消注释并把地址改成阿里云
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=cloud-openstack-train   # 将这一行注释掉
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Cloud
exclude=sip,PyQt4
```

在修改完上述四个文件后：

```bash
# 生成缓存
yum makecache

# 更新（可选）
yum -y update
```

即可正常安装 OpenStack 所需要的软件。
