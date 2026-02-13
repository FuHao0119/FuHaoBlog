# Fedora 43: Installing and Using Satty for Screenshots

Satty is a powerful screenshot tool that provides interactive selection and editing capabilities. While it's a great utility, you might find that it's not available directly in the official Fedora 43 repositories. This guide will walk you through the process of manually installing Satty using its binary and setting up the necessary dependencies for use in a Niri (Wayland) environment.

## Prerequisites

*   Fedora 43
*   Niri (Wayland) compositor (or another Wayland environment)
*   Basic terminal knowledge

## Installation Steps for Satty

### 1. Download Satty Binary

Since Satty isn't in the Fedora repositories, you'll need to download its pre-compiled binary directly from the official releases page.

Navigate to the [Satty Releases page](https://github.com/witcherylab/satty/releases) on GitHub.

Look for the latest release and download the file named `satty-x86_64-unknown-linux-gnu.tar.gz`.

### 2. Extract and Move the Binary

Once downloaded, open your terminal, navigate to your Downloads directory (or wherever you saved the file), and extract the archive. Then, move the `satty` executable to a directory included in your system's `PATH`, such as `/usr/local/bin/`, so you can run it from anywhere.

```bash
# Navigate to your downloads directory
cd ~/Downloads

# Extract the tar.gz file
tar -xzf satty-*.tar.gz

# Move the executable to /usr/local/bin/
sudo mv satty /usr/local/bin/
```

### 3. Check and Install Dependencies

For Satty to function correctly in a Wayland environment like Niri, it typically relies on two other tools for screen selection and capture: `slurp` and `grim`. These tools are usually available in Fedora's official repositories.

*   **`slurp`**: Used for interactively selecting a region of the screen with your mouse.
*   **`grim`**: Used to capture screen pixels (take the actual screenshot).

Install these dependencies using `dnf`:

```bash
sudo dnf install slurp grim
```

## Using Satty

Once `satty`, `slurp`, and `grim` are installed, you can start using Satty for your screenshot needs.

A typical usage might look like this:

```bash
satty --output-file ~/Pictures/screenshot-$(date +%Y%m%d%H%M%S).png
```

This command will:
1.  Launch `slurp` to allow you to select a screen region.
2.  Use `grim` to capture the selected area.
3.  Open the captured image in Satty for editing.
4.  Save the final image to your `Pictures` directory with a timestamped filename.

## a script about edit pictures after use satty to get screenshot

```
fuhao at fedora in ~/.config/niri/scripts
 ○ pwd            
/home/fuhao/.config/niri/scripts
fuhao at fedora in ~/.config/niri/scripts
 ○ ls
screenshot.sh
fuhao at fedora in ~/.config/niri/scripts
 ○ cat screenshot.sh            
#!/usr/bin/env bash

# 设定触发新逻辑的目标版本号
# 注意：官方 Niri 目前版本是 0.1.x。请确保你的 niri -V 输出格式符合你的预期。
TARGET_VERSION="25.08"

# 获取当前 niri 版本 (例如: "niri 0.1.10" -> "0.1.10")
CURRENT_VERSION=$(niri -V | awk '{print $2}')

# 版本比较函数 (使用 sort -V 处理版本号排序)
# 如果 $1 (当前) >= $2 (目标)，返回 true (0)
version_ge() {
    [ "$(printf '%s\n' "$1" "$2" | sort -V | head -n1)" != "$1" ] || [ "$1" = "$2" ]
}

# === 分支逻辑 ===

if version_ge "$CURRENT_VERSION" "$TARGET_VERSION"; then
    # [逻辑 A] 新版本 Niri: 基于事件流 (Event Stream)
    # 优点: 直接获取文件路径，精准，无需轮询，无剪贴板冲突
    
    # 1. 在后台启动截图交互，不阻塞脚本继续执行
    niri msg action screenshot &

    # 2. 监听事件流，阻塞等待直到捕获到 "Screenshot captured" 这一行
    # grep -m 1: 匹配到一行后立即停止监听，脚本继续向下
    log_output=$(niri msg event-stream | grep -m 1 --line-buffered "Screenshot captured")

    # 3. 提取路径字符串 (删除 "saved to " 及其之前的所有内容)
    # 输出示例: ... saved to /home/user/Pic/1.png -> /home/user/Pic/1.png
    file_path="${log_output##*saved to }"

    # 4. 传递给 satty 编辑
    # 检查路径是否非空（防止用户按 Esc 取消导致 grep 没抓到东西）
    if [ -n "$file_path" ]; then
        satty --filename "$file_path"
    fi

else
# [逻辑 B] 旧版本/回退: 基于剪贴板哈希 (Clipboard Polling)
    
    # 2. 记录基准哈希和开始时间
    CLIP_BASE=$(wl-paste | sha1sum)
    START_TIME=$SECONDS
    TIMEOUT_SEC=3 # 设置超时时间 (秒)

    # 3. 启动截图
    niri msg action screenshot

    # 4. 循环检测: 只要剪贴板哈希没变，就一直等待
    while [ "$(wl-paste | sha1sum)" = "$CLIP_BASE" ]; do
        # 超时判断: 如果耗时超过阈值，静默退出脚本 (防止死循环)
        if (( SECONDS - START_TIME > TIMEOUT_SEC )); then
            exit 0
        fi
        sleep 0.5
    done

    # 5. 剪贴板变化后，管道传递给 satty
    wl-paste | satty -f -
fi
fi
```

then vim the config file in niri and add a hotkey:
```
Mod+Shift+S { spawn "~/.config/niri/scripts/screenshot.sh"; }
```

now you can use Mod+Shift+S to edit screenshot

Now you have a powerful screenshot and editing tool at your fingertips on Fedora 43 with Niri!


