---
name: ubuntu-x11-4k-fix
description: |
  修复 Ubuntu/Linux X11 环境下 4K 显示器的 UI 缩放和窗口拖动卡顿问题。
  当用户提到 Ubuntu、Linux、X11、4K 显示器、拖动窗口卡顿、掉帧、UI 太小、分数缩放性能问题，或者想在不切换到 Wayland 的情况下放大 UI 时，务必使用此 skill。即使用户没有明确要求“修复 X11”，只要他们描述了 Linux 下的 4K 性能问题，就触发此 skill。
---

# Ubuntu X11 4K 缩放修复

此 skill 帮助用户优化 Ubuntu/Linux 在 X11 显示服务器下的 4K 显示器体验，专门解决由分数缩放（Fractional Scaling）引起的严重性能卡顿问题。

## 向用户解释背景
当用户在 X11 下为 4K 显示器开启分数缩放（如 125% 或 150%）时，X11 会强制 GPU 渲染一个巨大的 5K 或 8K 虚拟帧缓冲，然后再将其缩小输出为 4K。这会给 GPU（尤其是 Intel 核显）带来巨大的压力，导致严重的卡顿、掉帧和窗口拖动迟滞。

在 X11 下的解决方案是：关闭分数缩放（保持 100%），转而全局放大系统字体和 Dock 图标。但在 Wayland 下，系统原生支持独立缩放且不会卡顿，因此不需要此妥协方案。

## 执行步骤

1. **检查并打印环境信息**：首先使用 Shell 工具执行 `echo $XDG_SESSION_TYPE` 或 `loginctl show-session $(loginctl | awk '/tty/ {print $1}') -p Type` 检查当前桌面环境，并在最终回复的开头或结尾向用户打印当前检测到的环境（例如：“当前检测到的桌面环境为：X11”）。
2. **根据环境分情况处理**：
   - **如果当前是 X11 环境**：
     - 简要向用户解释上述 X11 分数缩放导致性能问题的原理。
     - 使用 Shell 工具主动执行以下命令，调整全局字体缩放和 Dock 图标大小：
      ```bash
      gsettings set org.gnome.desktop.interface text-scaling-factor 1.5
      gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size 64
      gsettings set org.gnome.desktop.interface cursor-size 32
      ```
       *(注：如果用户需要不同的缩放比例，可相应调整 1.5 这个系数，例如 1.25 或 2.0)*
     - 明确告诉用户，他们**必须**去系统“设置 -> 显示”中，将缩放（Scale）改回 **100%**。
     - 提醒用户这个 X11 妥协方案的主要缺点：如果外接一台 1080p 的副屏，全局字体放大也会应用到副屏上，导致文字偏大。切换到 Wayland 是唯一的出路。
   - **如果当前是 Wayland 环境**：
     - 告诉用户：当前已经是 Wayland 环境，Wayland 原生支持独立分数缩放且不会导致拖动卡顿，**不需要**使用 X11 的妥协方案。
     - 主动执行以下命令，将全局字体缩放还原为默认值，防止界面过大：
       ```bash
       gsettings set org.gnome.desktop.interface text-scaling-factor 1.0
       gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size 48
       gsettings set org.gnome.desktop.interface cursor-size 24
       ```
     - 指导用户：直接去系统“设置 -> 显示”中，为 4K 屏幕开启 150%（或 125%）的缩放即可，1080p 屏幕保持 100%。

## 输出格式
使用中文回复。保持简洁，主动执行命令，清晰列出需要用户手动操作的步骤，并确保明确打印出当前的桌面环境信息。