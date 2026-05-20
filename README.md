# alpine-main

适用于 **panther-x2 (RK3566)** 的 Alpine Linux 固件构建工具，基于 [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian) rebuild 框架。

## 目录结构

```
.
├── alpine/                          # Alpine 平台脚本（替代 debian/ 目录）
│   ├── alpine-common                # 通用环境变量
│   ├── alpine-release               # 固件版本信息
│   ├── alpine-firstrun              # 首次启动初始化
│   ├── alpine-firstrun-etc          # firstrun 默认配置
│   ├── alpine-hardware-optimization # 硬件优化（I/O调度、IRQ亲和、cpufreq）
│   ├── alpine-ramlog                # RAM 日志服务
│   ├── alpine-ramlog-etc            # ramlog 默认配置
│   ├── alpine-resize-filesystem     # 首次启动自动扩展 rootfs
│   ├── alpine-zram-config           # ZRAM 配置（swap + ramlog 压缩分区）
│   ├── alpine-zram-config-etc       # zram 默认配置
│   ├── alpine-update                # 内核在线升级工具（alpine-update）
│   ├── alpine-rc.local              # rc.local 启动钩子
│   └── alpine-os-release-template   # /etc/os-release 模板（rebuild 兼容）
├── openrc/                          # OpenRC init 脚本（替代 systemd/ 目录）
│   ├── alpine-firstrun              # firstrun 服务
│   ├── alpine-hardware-optimize     # 硬件优化服务
│   ├── alpine-ramlog                # ramlog 服务
│   ├── alpine-resize-filesystem     # 文件系统扩容服务
│   ├── alpine-zram-config           # zram 服务
│   └── local                        # rc.local 执行服务
├── rebuild/                         # ophub rebuild 框架（原样保留，已适配 Alpine）
│   └── build-armbian/
│       └── armbian-files/
│           ├── common-files/        # 通用配置（sysctl、modules、cpufreq 等）
│           └── ...
└── .github/
    └── workflows/
        └── alpine.yml               # GitHub Actions 构建流程
```

## 与 Debian/Ubuntu 版本的主要差异

| 项目 | Debian/Ubuntu | Alpine |
|------|--------------|--------|
| 包管理 | apt / dpkg | apk |
| Init 系统 | systemd | OpenRC |
| 服务目录 | `/lib/systemd/system/` | `/etc/init.d/` |
| 平台脚本目录 | `/usr/lib/debian/` | `/usr/lib/alpine/` |
| 内核更新命令 | `d-u` / `u-u` | `alpine-update` |
| rootfs 引导工具 | debootstrap | apk.static |
| Shell | bash | busybox ash（bash 可选安装）|

## 使用方法

### 在 GitHub Actions 触发构建

1. Fork 本仓库
2. 进入 **Actions → Build Alpine**
3. 点击 **Run workflow**，填写参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `rootfs_url` | 已有 rootfs 的 tar.gz 链接（留空则自动构建）| 空 |
| `alpine_version` | Alpine 版本：`3.21` / `3.20` / `edge` | `3.21` |
| `armbian_kernel` | 内核版本：`6.12.y` / `6.1.y` | `6.12.y` |
| `armbian_fstype` | 文件系统：`ext4` / `btrfs` | `ext4` |
| `root_password` | root 密码 | `1234` |
| `hostname` | 主机名 | `alpine` |
| `timezone` | 时区 | `Asia/Shanghai` |
| `extra_packages` | 额外 apk 包，空格分隔 | 空 |

### 构建模式

**模式 A（自建 rootfs，约 10～15 分钟）**  
不填写 `rootfs_url`，workflow 会用 `apk.static` 引导完整 Alpine 环境，打包 rootfs 和固件一并发布。

**模式 B（复用 rootfs，约 5 分钟）**  
填写上次发布的 `Alpine_*_rootfs.tar.gz` 链接，跳过 rootfs 构建阶段，直接注入内核打包固件。

## 在设备上更新内核

```sh
# 更新到最新稳定版
alpine-update

# 指定版本系列
alpine-update -k 6.1.y

# 指定完整版本并备份旧内核
alpine-update -k 6.1.100 -b yes
```

## 服务管理（OpenRC）

```sh
# 查看服务状态
rc-status

# 启动 / 停止服务
rc-service alpine-ramlog start
rc-service alpine-ramlog stop

# 开机自启 / 禁用
rc-update add alpine-ramlog default
rc-update del alpine-ramlog default
```

## 日志

- 硬件优化日志：`/var/log/alpine-hardware-monitor.log`
- ramlog 同步日志：`/var/log.hdd/alpine-ramlog.log`
- 自定义服务日志：`/tmp/ophub_start_service.log`
