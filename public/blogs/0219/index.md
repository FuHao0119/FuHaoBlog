# Niri 桌面美化：给你的 Overview 界面也换上壁纸吧！

如果你正在 Fedora 43 上使用 Niri 窗口管理器，并且用 DMS 来管理系统配置，你可能会发现一个小小的不完美：工作区都有漂亮的壁纸，但切换到多任务概览（Overview）界面时，背景却光秃秃的，有点单调。别急，这篇教程就是来解决这个问题的！跟着我一步步操作，给你的 Niri Overview 也换上心仪的壁纸，让整个桌面体验更上一层楼。

## 需要准备什么？

*   Fedora 43 系统
*   正在使用 Niri 窗口管理器
*   已配置好 DMS 的系统

## 设置壁纸，开干！

### 第一步：安装 `swaybg` 壁纸工具

首先，我们需要安装 `swaybg`，这是一个在 Wayland 环境下很常用的壁纸设置工具。

```bash
sudo dnf install swaybg
```

### 第二步：修改 Niri 配置文件

接下来，咱们得动一动 Niri 的配置文件了。它通常藏在 `~/.config/niri/config.kdl` 这个位置。

```bash
vim ~/.config/niri/config.kdl
```

#### 配置 Overview 选项

打开文件后，确保你有一个 `overview` 配置块。如果没找到，就自己加上一个。建议顺便设置一个 `backdrop-color`，这样可以控制概览界面里窗口背后的背景透明度。

```kdl
overview {
    zoom 0.5
    // 可以设为半透明的黑色，或者用 "transparent" 设置为全透明
    backdrop-color "#00000000"
}
```

#### 添加 `swaybg` 到启动项

现在，加上一个 `spawn-at-startup` 条目，让 Niri 启动时自动运行 `swaybg` 并加载你的壁纸。**记得把 `/xxx/xxx/xxx.jpg` 换成你自己的图片文件的真实路径！**

```kdl
// 记得换成你自己的图片路径
spawn-at-startup "swaybg" "-m" "fill" "-i" "/xxx/xxx/xxx.jpg"
```

*   `-m fill`: 这个选项告诉 `swaybg` 把图片拉伸以填满整个屏幕，它会自动保持宽高比，超出部分会裁剪掉。你也可以试试 `fit`（适应）、`stretch`（拉伸）、`center`（居中）或者 `tile`（平铺）这些选项。
*   `-i /path/to/your/image.jpg`: 用来指定你的壁纸图片路径。

### 第三步：为 `swaybg` 创建图层规则

为了保证 `swaybg` 能在 Niri 里正确地作为背景显示在 Overview 界面，我们还需要添加一条 `layer-rule`（图层规则）。这条规则会把壁纸程序“塞到”背景层去。

```kdl
// 给概览界面的壁纸程序设置规则
layer-rule {
    // 匹配 swaybg 的命名空间
    match namespace="^wallpaper$"

    // 把它放到背景里去
    place-within-backdrop true
    // 在截屏时排除它
    block-out-from "screen-capture"
}
```

*   `match namespace="^wallpaper$"`: 这会匹配到命名空间为 "wallpaper" 的应用，`swaybg` 通常用的就是这个名字。
*   `place-within-backdrop true`: 这条是关键，确保了程序能被放置在所有窗口的后面，真正起到壁纸的作用。
*   `block-out-from "screen-capture"`: 这条规则可能会让截屏工具默认忽略掉壁纸，不过具体效果可能因工具而异。

## 最后一步：重新登录

搞定收工！所有配置都修改完之后，**你需要退出当前用户再重新登录一次**，这样才能让所有设置生效。重新登录后，你就会惊喜地发现，你心爱的壁纸已经骄傲地展示在 Niri 的 Overview 界面啦！

享受你更加个性化、更漂亮的 Niri 桌面吧！
