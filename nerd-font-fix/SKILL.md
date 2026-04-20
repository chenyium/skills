---
name: nerd-font-fix
description: >-
  修复终端图标乱码、方块、问号、菱形等显示异常。当用户提到终端乱码、图标不显示、
  powerline/oh-my-zsh/starship/powerlevel10k 符号异常、Nerd Font 图标问题、
  终端字体缺字、zsh 主题图标变方块时使用。即使用户没有提到 Nerd Font，只要描述了
  终端中特殊符号或图标显示不正常，就应该使用此 skill。
---

# 修复终端 Nerd Font 图标乱码

**核心原则：不要更换用户的字体，而是给用户现有字体安装 Nerd Font 版本。**

> 终端必须使用 **Mono** 变体（`XXX Nerd Font Mono`），非 Mono 版本图标字符双宽度，会撑乱终端排版。

## Step 1: 诊断

确定终端模拟器、当前字体和字号：

```bash
echo "Desktop: $XDG_CURRENT_DESKTOP"
dpkg -l 2>/dev/null | grep -iE "gnome-terminal|konsole|xfce4-terminal|alacritty|kitty|wezterm|tilix" | awk '{print $2}'
```

获取当前字体（记为 `CURRENT_FONT`）和字号：

| 终端 | 获取方式 |
|------|---------|
| GNOME Terminal | `dconf dump /org/gnome/terminal/` 找 `font=` |
| Konsole | `~/.local/share/konsole/*.profile` 的 `Font=` |
| Alacritty | `~/.config/alacritty/alacritty.toml` 的 `font.normal.family` |
| Kitty | `~/.config/kitty/kitty.conf` 的 `font_family` |
| WezTerm | `~/.config/wezterm/wezterm.lua` 的 `font` |
| Xfce4 Terminal | `~/.config/xfce4/terminal/terminalrc` 的 `FontName=` |

如果当前字体名已包含 "Nerd Font"，说明字体本身没问题，排查终端配置或 locale 问题。

---

## Step 2: 安装 Nerd Font

先去 https://www.nerdfonts.com/font-downloads （用 WebSearch 或 WebFetch）查找 `CURRENT_FONT` 有无预构建版本。

- 开源字体（FiraCode, JetBrains Mono, Hack, Ubuntu Mono, Meslo 等）→ 通常有
- 专有字体（Consolas, Monaco, SF Mono, Menlo 等）→ 没有，走 2b

### 2a: 有 → 下载安装

release 资源名是去空格驼峰：`JetBrains Mono` → `JetBrainsMono`，`Fira Code` → `FiraCode`，`Source Code Pro` → `SourceCodePro`。不确定时用 WebSearch 确认。

```bash
FONT_NAME="JetBrainsMono"  # 替换为实际名称
LATEST=$(curl -sL https://api.github.com/repos/ryanoasis/nerd-fonts/releases/latest | grep -oP '"tag_name":\s*"\K[^"]+')
curl -fLO "https://github.com/ryanoasis/nerd-fonts/releases/download/${LATEST}/${FONT_NAME}.tar.xz"
mkdir -p ~/.local/share/fonts/${FONT_NAME}
tar -xf ${FONT_NAME}.tar.xz -C ~/.local/share/fonts/${FONT_NAME}/
fc-cache -f ~/.local/share/fonts/${FONT_NAME}/
```

`.tar.xz` 失败则换 `.zip`：`curl -fLO .../${FONT_NAME}.zip && unzip -o ... -d ...`

### 2b: 没有 → Docker 打补丁

```bash
mkdir -p /tmp/nerd-font-patch/{input,output}
# 用 fc-list | grep -i "CURRENT_FONT" 找到字体路径，拷贝全部变体
cp /path/to/原始字体/*.ttf /tmp/nerd-font-patch/input/

docker run --rm \
  -v /tmp/nerd-font-patch/input:/in \
  -v /tmp/nerd-font-patch/output:/out \
  nerdfonts/patcher --mono --complete

mkdir -p ~/.local/share/fonts/patched-nerd-font
cp /tmp/nerd-font-patch/output/*.ttf ~/.local/share/fonts/patched-nerd-font/
fc-cache -f ~/.local/share/fonts/patched-nerd-font/
rm -rf /tmp/nerd-font-patch
```

注意：
- 不要给 docker 命令传字体路径参数，镜像自动扫描 `/in`
- 要拷贝全部变体（Regular/Bold/Italic/BoldItalic）到 input
- Docker 拉取失败参考 `docker-network` skill 配置镜像源
- 没有 Docker 可用 FontForge：`fontforge -script font-patcher --mono --complete font.ttf`

验证：`fc-list | grep -i "Nerd Font Mono"`

---

## Step 3: 配置终端

从 `fc-list` 取准确的 Mono 字体名，**保持用户原字号不变**。

**GNOME Terminal:**
```bash
PROFILE=$(gsettings get org.gnome.Terminal.ProfilesList default | tr -d "'")
# 确保使用自定义字体而非系统字体
dconf write /org/gnome/terminal/legacy/profiles:/:${PROFILE}/use-system-font "false"
dconf write /org/gnome/terminal/legacy/profiles:/:${PROFILE}/font "'字体名 Nerd Font Mono SIZE'"
```

**Alacritty** — `alacritty.toml`: `[font.normal] family = "字体名 Nerd Font Mono"`

**Kitty** — `kitty.conf`: `font_family 字体名 Nerd Font Mono`

**Konsole** — 编辑 `~/.local/share/konsole/*.profile` 的 `Font=` 行

---

## 常见坑

| 现象 | 原因和解决 |
|------|-----------|
| 字符间距过大 | 用了非 Mono 变体，换 `Nerd Font Mono` |
| Docker patcher 报参数错误 | 不要传字体路径，只传 `--mono --complete` |
| fc-cache 后找不到字体 | `fc-cache -fv` 强制刷新 |
| 改了 font 但终端没变化 | GNOME Terminal `use-system-font` 为 true，需设为 false |
