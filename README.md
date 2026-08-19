# QEMU AppImage

将 QEMU 打包为可直接运行的 AppImage。该仓库为非官方打包项目，QEMU 本身由 QEMU Project 维护。

## 下载

仓库只保留一个固定的 **Latest Release**：

- [Latest Release](../../releases/latest)
- `QEMU-latest-universal-x86_64.AppImage`：基于 Arch Linux 运行时环境打包。
- `QEMU-latest-jammy-x86_64.AppImage`：在 Ubuntu 22.04 上从 QEMU 官方稳定版源码构建。

Release tag 始终为 `latest`，文件名保持稳定，因此不需要随着 QEMU 版本变化修改下载地址。

## 自动更新

GitHub Actions 每天仅执行一次轻量版本检查，直接读取 **QEMU 官方 GitLab** 的版本 tag：

1. 只接受 `vX.Y.Z` 格式的正式稳定版。
2. 自动排除 `-rc` 等预发布版本。
3. 将 QEMU 官方最新稳定版与当前 `latest` Release 中记录的 `QEMU-Version` 比较。
4. 版本没有变化时立即结束，不执行 AppImage 构建。
5. 发现新稳定版后才构建 Universal 和 Jammy 两个 AppImage。
6. Universal 构建完成后会检查实际打包的 QEMU 版本；如果 Arch 软件包尚未同步到官方最新稳定版，本次发布直接停止，避免发布版本不一致的文件。
7. 两个 AppImage 全部构建成功后才删除旧 Release，并重新创建固定的 `latest` Release。
8. 不使用 GitHub Actions Artifact 作为中间传递或长期存储。

也可以在 **Actions → QEMU AppImage Latest → Run workflow** 手动检查；只有明确启用 `force_build` 时才会在版本未变化的情况下强制重建。

## 基本使用

### 添加执行权限

```bash
chmod +x QEMU-latest-universal-x86_64.AppImage
```

以下示例使用 Universal 文件；如果使用 Jammy，只需要替换 AppImage 文件名。

### 查看 QEMU 版本

```bash
./QEMU-latest-universal-x86_64.AppImage qemu-system-x86_64 --version
```

### 创建 QCOW2 磁盘

```bash
./QEMU-latest-universal-x86_64.AppImage qemu-img create -f qcow2 disk.qcow2 30G
```

### 启动 x86_64 虚拟机

```bash
./QEMU-latest-universal-x86_64.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 2 \
  -m 4G \
  -drive file=disk.qcow2,format=qcow2 \
  -cdrom system.iso \
  -boot d
```

### GTK 图形界面与剪贴板

Jammy AppImage 的启动 wrapper 会在使用 GTK display 时自动补充 `clipboard=on`，无需每次手动追加。

```bash
./QEMU-latest-jammy-x86_64.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 4G \
  -display gtk
```

### 主机与虚拟机共享目录

主机侧示例：

```bash
./QEMU-latest-universal-x86_64.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 4G \
  -drive file=disk.qcow2,format=qcow2 \
  -virtfs local,path=/path/to/share,mount_tag=host0,security_model=mapped,id=host0
```

Linux Guest 内挂载：

```bash
sudo mount -t 9p -o trans=virtio,version=9p2000.L host0 /mnt/share
```

## KVM

需要硬件加速时，主机必须提供 `/dev/kvm` 并允许当前用户访问。无法使用 KVM 时，可以去掉 `-enable-kvm` 和 `-cpu host`，由 QEMU 使用软件模拟。

## 构建结构

- `.github/workflows/qemu-latest.yml`：检查 QEMU 官方稳定版、按需构建并发布固定 `latest` Release。
- `qemu-universal.sh`：Universal AppImage 的现有构建脚本。
- `AppRun`：Jammy AppImage 的原始运行入口。
- `AppRun.wrapper`：Jammy AppImage 的启动 wrapper，并处理 GTK clipboard 参数。
- `files/`：Universal AppImage 所需的 AppRun、desktop、图标和 Arch 配置文件。

## 说明

本仓库只负责 AppImage 打包和自动发布，不是 QEMU 官方仓库。QEMU 的功能、版本和源码以 QEMU Project 官方发布为准。
