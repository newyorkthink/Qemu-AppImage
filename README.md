# QEMU AppImage

## 中文

这是一个非官方 QEMU AppImage 打包仓库。构建流程使用 Arch Linux 官方预编译软件包，不重新编译 QEMU 源码。

### 下载

Release 固定只使用：

- Release：`Latest`
- Tag：`latest`
- 文件：`qemu.AppImage`

固定下载地址：

`https://github.com/newyorkthink/Qemu-AppImage/releases/download/latest/qemu.AppImage`

### 自动构建

GitHub Actions 使用已经验证过的快速打包流程：

- 使用 Arch Linux 官方 `qemu-desktop`、`jack2` 和 `virt-viewer` 软件包。
- 保留 SPICE/virt-viewer 剪贴板支持所需的运行时处理。
- 每次向 `main` 推送都会自动构建。
- 每 6 天自动重新构建一次，以获取软件源中的更新。
- 同一分支出现新的构建时，会取消仍在运行的旧构建，避免重复运行。
- 构建完成后只更新固定的 `latest` Release 和 `qemu.AppImage`。

### 基本使用

添加执行权限：

```bash
chmod +x qemu.AppImage
```

查看 QEMU 版本：

```bash
./qemu.AppImage qemu-system-x86_64 --version
```

创建 QCOW2 磁盘：

```bash
./qemu.AppImage qemu-img create -f qcow2 disk.qcow2 30G
```

启动 x86_64 虚拟机：

```bash
./qemu.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 2 \
  -m 4G \
  -drive file=<磁盘镜像路径>,format=qcow2 \
  -cdrom <ISO路径> \
  -boot d
```

需要 KVM 硬件加速时，主机需要提供 `/dev/kvm` 并允许当前用户访问。

---

## English

This is an unofficial QEMU AppImage packaging repository. The build uses official prebuilt Arch Linux packages and does not rebuild QEMU from source.

### Download

The repository uses one fixed release:

- Release: `Latest`
- Tag: `latest`
- File: `qemu.AppImage`

Stable download URL:

`https://github.com/newyorkthink/Qemu-AppImage/releases/download/latest/qemu.AppImage`

### Automatic builds

GitHub Actions keeps the previously validated fast packaging flow:

- Uses the official Arch Linux `qemu-desktop`, `jack2`, and `virt-viewer` packages.
- Keeps the runtime handling required for SPICE/virt-viewer clipboard support.
- Every push to `main` automatically starts a build.
- A scheduled rebuild runs every 6 days to pick up repository updates.
- A newer build on the same branch cancels an older in-progress build to avoid duplicate runs.
- A successful build only refreshes the fixed `latest` Release and `qemu.AppImage`.

### Basic usage

Make the AppImage executable:

```bash
chmod +x qemu.AppImage
```

Check the QEMU version:

```bash
./qemu.AppImage qemu-system-x86_64 --version
```

Create a QCOW2 disk:

```bash
./qemu.AppImage qemu-img create -f qcow2 disk.qcow2 30G
```

Start an x86_64 virtual machine:

```bash
./qemu.AppImage qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 2 \
  -m 4G \
  -drive file=<disk-image-path>,format=qcow2 \
  -cdrom <iso-path> \
  -boot d
```

For KVM hardware acceleration, the host must provide `/dev/kvm` and allow the current user to access it.
