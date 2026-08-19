# QEMU AppImage

将 QEMU 官方最新稳定版打包为可直接运行的 AppImage。该仓库为非官方打包项目，QEMU 本身由 QEMU Project 维护。

## 下载

仓库只保留一个固定的 **Latest Release**：

- [Latest Release](../../releases/latest)
- `qemu.AppImage`：从 QEMU 官方最新稳定版源码构建。

Release 标题固定为 `Latest`，Release tag 固定为 `latest`，文件名固定为 `qemu.AppImage`，因此 QEMU 更新后下载地址无需变化。

## 自动更新

GitHub Actions 会自动检查 **QEMU 官方 GitLab** 的版本 tag：

1. 只接受 `vX.Y.Z` 格式的正式稳定版。
2. 自动排除 `-rc` 等预发布版本。
3. 将 QEMU 官方最新稳定版与当前 `latest` Release 中记录的 `QEMU-Version` 比较。
4. 定时检查时，版本没有变化会直接结束，不重复构建。
5. 发现新稳定版后，从对应 QEMU 官方稳定 tag 构建 `qemu.AppImage`。
6. 修改构建相关文件并推送到 `main` 时会自动重新构建。
7. 构建成功后才删除旧 Release，并重新创建固定的 `latest` Release。
8. 旧的 `QEMU-git-*` 等资产会随旧 Release 一并删除，不再单独发布 Git 分支构建版本。
9. 不使用 GitHub Actions Artifact 作为中间传递或长期存储。

也可以在 **Actions → QEMU AppImage Latest → Run workflow** 手动检查；只有明确启用 `force_build` 时才会在版本未变化的情况下强制重建。

## 基本使用

### 添加执行权限

```bash
chmod +x qemu.AppImage
```

### 查看 QEMU 版本

```bash
./qemu.AppImage qemu-system-x86_64 --version
```

### 创建 QCOW2 磁盘

```bash
./qemu.AppImage qemu-img create -f qcow2 disk.qcow2 30G
```

### 启动 x86_64 虚拟机

```bash
./qemu.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 2 \
  -m 4G \
  -drive file=disk.qcow2,format=qcow2 \
  -cdrom system.iso \
  -boot d
```

### GTK 图形界面与剪贴板

启动 wrapper 会在使用 GTK display 时自动补充 `clipboard=on`，无需每次手动追加。

```bash
./qemu.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 4G \
  -display gtk
```

### 主机与虚拟机共享目录

主机侧示例：

```bash
./qemu.AppImage qemu-system-x86_64 \
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
- `AppRun`：AppImage 的原始运行入口。
- `AppRun.wrapper`：AppImage 启动 wrapper，并处理 GTK clipboard 参数。
- `qemu.desktop`、`qemu_logo_no_text.svg`、`libunionpreload.so`：AppImage 打包所需文件。

## 说明

本仓库只负责 AppImage 打包和自动发布，不是 QEMU 官方仓库。QEMU 的功能、版本和源码以 QEMU Project 官方发布为准。
