# Linux 安装 KVM 并安装 Windows 系统

> 我使用的发行版是fedora43 其他发行版的安装思路是一样的
>
> 包管理器不一样，最好搜一下对应的软件包，别安装错了

## 1. 安装 KVM 核心组件

使用以下命令安装必要的软件包：

```bash
sudo dnf install qemu-kvm libvirt virt-manager virt-install
```

这条命令会安装：

-   `qemu-kvm`: 用户态的模拟器和 KVM 内核模块接口。
-   `libvirt`: 管理虚拟化的后台服务和 API。
-   `virt-manager`: 图形化管理界面 (你需要这个来装 Windows)。
-   `virt-install`: 命令行创建虚拟机的工具。

安装完成后，请按顺序执行以下配置，确保你有权限且服务正常运行：

### 1.1 启动 libvirt 服务并设为开机自启

```bash
sudo systemctl enable --now libvirtd
```

### 1.2 验证 KVM 内核模块是否加载

```bash
lsmod | grep kvm
```

**预期输出：** 应该看到 `kvm_intel` (如果你是 Intel CPU) 或 `kvm_amd` (如果你是 AMD CPU)。

### 1.3 将你的用户 (`xxx`) 加入 `libvirt` 用户组

这一步是为了让你直接打开 `virt-manager` 而不需要每次都输 `root` 密码，也能正常使用 USB 设备透传。

```bash
sudo usermod -aG libvirt $(whoami)
```

⚠️ **注意**：执行完这步后，你必须注销当前用户并重新登录 (或者重启电脑) 才会生效。

---

执行完了这些步骤，接着准备 Windows 镜像和 VirtIO 驱动。

### 1.4 VirtIO 驱动 (高性能驱动，强烈推荐)

Windows 默认不包含 KVM 的高性能磁盘和网卡驱动。Fedora 提供了打包好的驱动 ISO。
在终端执行：

```bash
sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo -O /etc/yum.repos.d/virtio-win.repo
sudo dnf install virtio-win
```

安装后，驱动镜像文件位于：`/usr/share/virtio-win/virtio-win.iso`。

## 2. 创建虚拟机 (图形化操作)

在应用菜单中打开 "虚拟系统管理器" (Virtual Machine Manager)。

点击左上角的 "创建新虚拟机" 图标（电脑屏幕加个星号的图标）。

### 2.1 步骤 1 (安装介质)

选择“本地安装介质 (ISO映像)”，浏览并选择你的 Windows ISO 文件。

### 2.2 步骤 2 (内存/CPU)

-   **内存**：建议分配 8192 MB (8GB) 或更多（DevEco Studio + 模拟器非常吃内存）。
-   **CPU**：建议分配 4 核 或更多。

### 2.3 步骤 3 (磁盘)

创建一个 60GB 以上的磁盘（鸿蒙 SDK 和镜像很大）。

### 2.4 步骤 4 (关键！)

在点击“完成”之前，务必勾选 `[v] 在安装前自定义配置 (Customize configuration before install)`。

## 3. 关键配置 (自定义界面)

进入自定义界面后，请依次修改以下几项：

### 3.1 CPU 设置 (核心步骤)

1.  点击左侧列表的 `CPUs`。
2.  在右侧，找到 `配置 (Configuration)` 部分。
3.  取消勾选“复制宿主机 CPU 配置” (如果已勾选)。
4.  将 `型号 (Model)` 手动改为：`host-passthrough`。

**解释**：这让 Windows 直接使用你的物理 CPU 特性（包括 VT-x 虚拟化指令）。

### 3.2 挂载驱动 ISO (为了高性能)

1.  点击左下角的 "添加硬件" (Add Hardware)。
2.  选择 `存储 (Storage)` -> 设备类型选 `CDROM`。
3.  点击“管理” -> 浏览并选择 `/usr/share/virtio-win/virtio-win.iso`。

这样你的虚拟机就有两个光驱：一个装 Windows，一个装驱动。

### 3.3 调整磁盘总线 (可选，提升 I/O 性能)

如果你想编译速度更快，点击左侧的 `SATA 磁盘 1`。

将 `磁盘总线 (Disk bus)` 改为 `VirtIO`。

**注意**：如果改为 `VirtIO`，安装 Windows 时会找不到硬盘，需要加载刚才挂载的驱动光盘里的驱动（通常在 `viostor\w10\amd64` 目录下）。如果不熟悉这个操作，保持默认的 SATA 也可以，只是慢一点。

---

配置完成后，点击左上角的 "开始安装" (Begin Installation)。

## 4. 安装 Windows

### 4.1 安装 Windows：正常安装流程。

如果你刚才把磁盘改成了 `VirtIO`：安装界面选硬盘时会是空的。点击“加载驱动” -> 浏览 CD 驱动器 (`virtio-win`) -> 找到 `amd64` -> `w10` (或 `w11`) 文件夹，确认安装驱动后硬盘就会出现。

### 4.2 安装增强工具

进入 Windows 桌面后，打开那张 `virtio-win` 光盘，运行 `virtio-win-gt-x64.exe` 安装包。这将安装所有高性能驱动（网卡、显卡、内存气球等）。
