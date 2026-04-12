---
title: zsh + Starship + Conda + uv + Zellij 配置
index_img: https://images.unsplash.com/photo-1629654297299-c8506221ca97?w=800
mathjax: false
date: 2026-04-12 18:30:00
post_asset_folder: false
password:
summary: 详细介绍在没有 sudo 权限的 Linux 环境下，如何从零开始配置一个美观、功能强大的终端环境，包括 zsh 主题配置、Starship 提示符、Conda 环境管理、uv 加速工具以及 Zellij 终端多路复用器
tags:
  - Linux
  - 终端配置
  - zsh
  - Starship
  - Conda
  - uv
  - Zellij
categories:
  - 工具配置
---

# <b><span style="color:rgba(244,63,94,1)">zsh + Starship + Conda + uv + Zellij 配置</span></b>

# <b><span style="color:rgba(14, 165, 233, 1)">一、前言</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">1.1 背景介绍</span></b>

在使用 Linux 服务器时，很多情况下我们并没有 sudo 权限，无法使用系统的包管理器安装软件。但这并不意味着我们不能拥有一个美观、功能强大的终端环境。本文将详细介绍如何在**没有 sudo 权限**的情况下，从零开始配置一个专业级的终端环境。

## <b><span style="color:rgba(14, 165, 233, 1)">1.2 最终效果</span></b>

配置完成后，你的终端将具备以下特性：

- **美观的提示符**：显示用户名、主机名、当前目录、Git 分支、Python 版本、执行时间等信息
- **智能命令补全**：支持语法高亮和命令建议
- **快速包管理**：使用 uv 加速 Python 包安装
- **高效多会话管理**：使用 Zellij 管理多个终端会话
- **优雅的环境隔离**：使用 Conda 管理不同的 Python 环境

> 本文所有软件都将安装到用户目录下（`~/.local` 或 `~`），无需任何系统权限。

# <b><span style="color:rgba(14, 165, 233, 1)">二、环境准备</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">2.1 检查现有工具</span></b>

首先，检查系统中是否已经安装了 zsh：

```bash
# 检查 zsh 是否已安装
zsh --version

# 检查 git 是否可用（用于克隆仓库）
git --version

# 检查 curl 是否可用（用于下载文件）
curl --version
```

> 如果提示命令不存在也不用担心，我们可以从源码编译安装 zsh。

## <b><span style="color:rgba(14, 165, 233, 1)">2.2 创建必要目录</span></b>

```bash
# 创建本地 bin 目录（用于存放可执行文件）
mkdir -p ~/.local/bin

# 创建配置目录
mkdir -p ~/.config
```

# <b><span style="color:rgba(14, 165, 233, 1)">三、安装并配置 zsh</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">3.1 方式一：系统已有 zsh</span></b>

如果系统已经安装了 zsh，直接跳到安装 Oh My Zsh 部分。

## <b><span style="color:rgba(14, 165, 233, 1)">3.2 方式二：从源码编译安装（无 sudo）</span></b>

如果系统没有 zsh，我们需要从源码编译安装：

```bash
# 下载 zsh 5.9 源码
curl -fsSL https://sourceforge.net/projects/zsh/files/zsh/5.9/zsh-5.9.tar.xz/download -o zsh.tar.xz

# 解压源码
tar -xf zsh.tar.xz
cd zsh-5.9

# 配置（安装到用户目录）
./configure --prefix=$HOME/.local

# 编译（可能需要几分钟）
make -j$(nproc)

# 安装
make install
```

> 编译安装完成后，zsh 可执行文件位于 `~/.local/bin/zsh`。

## <b><span style="color:rgba(14, 165, 233, 1)">3.3 配置 bash 自动启动 zsh</span></b>

由于没有 sudo 权限，我们无法使用 `chsh` 命令更改默认 shell。但可以通过修改 `~/.bashrc`，在 bash 启动时自动切换到 zsh。

编辑 `~/.bashrc` 文件，在文件末尾添加：

```bash
# 添加 zsh 到 PATH 环境变量
export PATH="$HOME/.local/bin:$PATH"

# 自动启动 zsh
exec zsh
```

保存后，运行以下命令使配置生效：

```bash
source ~/.bashrc
```

首次启动 zsh 时，会看到 zsh 的配置向导，选择 **0** 退出即可。

# <b><span style="color:rgba(14, 165, 233, 1)">四、安装 Oh My Zsh</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">4.1 Oh My Zsh 简介</span></b>

<b><span style="color:rgba(245,158,11,1)">Oh My Zsh</span></b> 是一个开源的、社区驱动的 zsh 配置管理框架。它提供了丰富的主题、插件系统，可以大大提升终端的使用体验。

## <b><span style="color:rgba(14, 165, 233, 1)">4.2 安装 Oh My Zsh</span></b>

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装脚本会自动检测 zsh 并创建必要的配置文件。安装完成后，你的 zsh 已经可以使用基本的提示符了。

## <b><span style="color:rgba(14, 165, 233, 1)">4.3 安装常用插件</span></b>

Oh My Zsh 强大的插件系统可以显著提升我们的工作效率。下面安装几个最实用的插件：

### <b><span style="color:rgba(217,70,239,1)">zsh-autosuggestions（命令建议）</span></b>

这个插件会根据你的命令历史自动建议补全命令，就像浏览器地址栏的自动补全一样。

```bash
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git \
  ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

### <b><span style="color:rgba(217,70,239,1)">zsh-syntax-highlighting（语法高亮）</span></b>

这个插件会在你输入命令时进行语法高亮，正确命令显示绿色，错误命令显示红色。

```bash
git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

## <b><span style="color:rgba(14, 165, 233, 1)">4.4 配置插件</span></b>

编辑 `~/.zshrc` 文件，找到 `plugins=(git)` 这一行，修改为：

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

保存后，使配置生效：

```bash
source ~/.zshrc
```

> 现在你可以尝试输入之前使用过的命令，会看到灰色的自动建议，按 **→** 键即可补全。

# <b><span style="color:rgba(14, 165, 233, 1)">五、安装 Starship 提示符</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">5.1 为什么选择 Starship</span></b>

<b><span style="color:rgba(245,158,11,1)">Starship</span></b> 是一个用 Rust 编写的跨平台提示符工具，具有以下优势：

- **轻量快速**：用 Rust 编写，启动速度极快
- **高度可定制**：通过简单的配置文件即可自定义提示符样式
- **丰富的集成**：原生支持 Git、Python、Node.js、Docker 等众多工具
- **跨平台**：支持 Linux、macOS、Windows

## <b><span style="color:rgba(14, 165, 233, 1)">5.2 安装 Starship</span></b>

### <b><span style="color:rgba(217,70,239,1)">方法一：官方安装脚本</span></b>

```bash
curl -fsSL https://starship.rs/install.sh | sh -s -- -b ~/.local/bin
```

### <b><span style="color:rgba(217,70,239,1)">方法二：使用 npm 安装</span></b>

如果官方脚本不支持你的系统架构，可以尝试：

```bash
npm install -g starship
```

### <b><span style="color:rgba(217,70,239,1)">方法三：直接下载二进制文件</span></b>

```bash
# 下载最新版本
curl -fsSL https://github.com/starship/starship/releases/latest/download/starship-x86_64-unknown-linux-musl.tar.gz \
  -o starship.tar.gz

# 解压
tar -xzf starship.tar.gz

# 移动到 PATH
mv starship ~/.local/bin/
```

### <b><span style="color:rgba(217,70,239,1)">验证安装</span></b>

```bash
~/.local/bin/starship --version
```

应该看到类似输出：

```
starship 1.24.2
branch:master
commit_hash:33f7077...
build_time:2025-12-30 19:14:09 +00:00
build_env:rustc 1.92.0...
```

## <b><span style="color:rgba(14, 165, 233, 1)">5.3 配置 Starship</span></b>

### <b><span style="color:rgba(217,70,239,1)">初始化 Starship</span></b>

编辑 `~/.zshrc` 文件，在文件开头添加：

```bash
# 初始化 Starship
eval "$(starship init zsh)"
```

### <b><span style="color:rgba(217,70,239,1)">创建配置文件</span></b>

创建配置文件：

```bash
mkdir -p ~/.config
nano ~/.config/starship.toml
```

### <b><span style="color:rgba(217,70,239,1)">推荐配置</span></b>

以下是一个美观的配置示例，显示用户名、主机名、目录、Git 分支、Python 版本和命令执行时间：

```toml
# ============================================
# Starship 配置文件
# 路径：~/.config/starship.toml
# ============================================

# 设置提示符格式
format = """
$username$directory$git_branch$python$cmd_duration$line_break$character"""

# ------------------
# 用户名模块
# ------------------
[username]
show_always = true
style_user = "bold green"
style_root = "bold red"

# ------------------
# 目录模块
# ------------------
[directory]
truncation_length = 3
truncate_to_repo = true
format = "[$path]($style)[$read_only]($read_only_style) "
style = "bold cyan"
read_only = " 🔒"
read_only_style = "red"
home_symbol = "~"

# ------------------
# Git 分支模块
# ------------------
[git_branch]
symbol = "  "
format = "on [$symbol$branch]($style) "
style = "bold purple"
truncation_symbol = ""
info_symbol = ""
modified_symbol = ""

# ------------------
# Git 状态模块
# ------------------
[git_status]
format = '([\[$all_status$ahead_behind\]]($style) )'
style = "bold yellow"
conflicted = "⚔️ "
ahead = "⇡${count}"
behind = "⇣${count}"
diverged = "⇕⇡${ahead_count}⇣${behind_count}"
untracked = "?${count}"
stashed = "📦"
modified = "!${count}"
staged = '+${count}'
renamed = "»${count}"
deleted = "-${count}"

# ------------------
# Python 模块
# ------------------
[python]
symbol = " via 🐍 v"
format = "$symbol($environment)($version) "
style = "bold yellow"
pyenv_version_name = false
pyenv_prefix = ""

# ------------------
# 命令执行时间模块
# ------------------
[cmd_duration]
min_time = 500
format = "took [$duration]($style) "
style = "bold yellow"

# ------------------
# 字符模块（命令提示符）
# ------------------
[character]
success_symbol = "[❯](bold green)"
error_symbol = "[❯](bold red)"
vimcmd_symbol = "[❮](bold green)"

# ------------------
# conda 环境模块
# ------------------
[conda]
symbol = " 🏠"
format = "[$symbol$environment]($style) "
style = "bold green"
truncation_length = 1

# ------------------
# 时间模块
# ------------------
[time]
format = "🕐 [$time]($style) "
style = "bold white"
disabled = false
time_format = "%H:%M"
utc_time_offset = "local"

# ------------------
# Docker 相关模块
# ------------------
[docker_context]
symbol = " 🐳 "
format = "[${symbol}${context}]($style) "
style = "bold blue"

# ------------------
# Node.js 模块
# ------------------
[nodejs]
symbol = " ⬢ "
format = "[$symbol($version)]($style) "
style = "bold green"
disabled = true

# ------------------
# 电池模块
# ------------------
[battery]
full = " ⚡-full"
charging = " ⚡charging"
discharging = " ⚡discharging"
unknown = " ⚡?"
empty = " ⚡empty"
display_gone = false
format = "[$symbol$percentage]($style) "
style = "bold red"
threshold = 30
disabled = true

# ------------------
# 包版本模块（显示当前目录的包管理器版本）
# ------------------
[package]
symbol = " 📦 "
format = "[$symbol$version]($style) "
style = "208 bold"
disabled = true
```

保存配置文件后，运行：

```bash
source ~/.zshrc
```

## <b><span style="color:rgba(14, 165, 233, 1)">5.4 使用预设主题</span></b>

Starship 提供了多种预设主题，可以快速应用漂亮的样式：

```bash
# 查看可用预设
starship preset --list

# 应用预设（以 gruvbox-rainbow 为例）
starship preset gruvbox-rainbow > ~/.config/starship.toml
```

常用的预设主题包括：

- `gruvbox-rainbow`：彩虹色系，适合深色终端
- `Tokyo Night`：东京夜主题，现代化设计
- `One Dark`：VS Code One Dark 风格
- `Dracula`：经典的 Dracula 主题

## <b><span style="color:rgba(14, 165, 233, 1)">5.5 最终效果</span></b>

配置完成后，你的提示符将显示类似以下信息：

```
ljk in ~/projects on  main via 🐍 v3.12.4 ❯
```

或者更丰富的版本：

```
ljk in ~/projects on  main via 🐍 v3.12.4 took 2s ❯
```

# <b><span style="color:rgba(14, 165, 233, 1)">六、安装 Miniconda</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">6.1 为什么需要 Conda</span></b>

<b><span style="color:rgba(245,158,11,1)">Conda</span></b> 是一个开源的包管理和环境管理系统，特别适合数据科学和机器学习工作：

- **环境隔离**：可以为不同项目创建独立的 Python 环境
- **跨平台**：支持 Linux、macOS、Windows
- **丰富生态**：预编译的二进制包，无需编译
- **版本管理**：轻松管理 Python 版本和包依赖

## <b><span style="color:rgba(14, 165, 233, 1)">6.2 安装 Miniconda</span></b>

Miniconda 是 Conda 的最小化版本，只包含必要的组件，比完整版 Anaconda 小得多。

```bash
# 下载 Miniconda 安装脚本
curl -fsSL https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
  -o ~/miniconda.sh

# 安装到用户目录
# -b：批处理模式（无需交互）
# -p：指定安装路径
bash ~/miniconda.sh -b -p ~/.miniconda3

# 清理安装脚本
rm ~/miniconda.sh
```

## <b><span style="color:rgba(14, 165, 233, 1)">6.3 初始化 Conda</span></b>

```bash
# 初始化 conda for zsh
~/.miniconda3/bin/conda init zsh

# 使配置生效
source ~/.zshrc
```

## <b><span style="color:rgba(14, 165, 233, 1)">6.4 基础配置</span></b>

```bash
# 验证安装
conda --version

# 设置不自动激活 base 环境（让终端更干净）
conda config --set auto_activate_base false

# 添加 conda-forge 频道（获取更多包）
conda config --add channels conda-forge

# 设置优先使用 conda-forge
conda config --set channel_priority strict
```

## <b><span style="color:rgba(14, 165, 233, 1)">6.5 常用 conda 命令</span></b>

```bash
# 创建新环境
conda create -n myenv python=3.11

# 激活环境
conda activate myenv

# 退出环境
conda deactivate

# 列出所有环境
conda env list

# 删除环境
conda env remove -n myenv

# 安装包
conda install numpy pandas

# 搜索包
conda search numpy

# 更新包
conda update numpy

# 导出环境配置
conda env export > environment.yaml

# 从配置文件创建环境
conda env create -f environment.yaml
```

## <b><span style="color:rgba(14, 165, 233, 1)">6.6 配置 conda 源（国内用户）</span></b>

如果在国内使用，建议配置国内镜像源加速下载：

```bash
# 添加清华镜像源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2

# 设置搜索时显示的 channel 数量
conda config --set show_channel_urls yes
```

# <b><span style="color:rgba(14, 165, 233, 1)">七、安装 uv 加速工具</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">7.1 为什么选择 uv</span></b>

<b><span style="color:rgba(245,158,11,1)">uv</span></b> 是由 Astral 开发的超快 Python 包安装器，用 Rust 编写，性能比 pip 快 **10-100 倍**：

- **极速安装**：并行下载和安装包
- **兼容 pip**：支持 pip 的所有命令和语法
- **环境管理**：内置虚拟环境管理功能
- **锁文件**：支持依赖锁定，确保环境一致性

## <b><span style="color:rgba(14, 165, 233, 1)">7.2 安装 uv</span></b>

### <b><span style="color:rgba(217,70,239,1)">方法一：官方安装脚本（推荐）</span></b>

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

默认安装到 `~/.cargo/bin/uv`，该目录已自动添加到 PATH。

### <b><span style="color:rgba(217,70,239,1)">方法二：使用 pip 安装</span></b>

```bash
pip install uv --user
```

### <b><span style="color:rgba(217,70,239,1)">方法三：使用 conda 安装</span></b>

```bash
conda install -c conda-forge uv
```

## <b><span style="color:rgba(14, 165, 233, 1)">7.3 验证安装</span></b>

```bash
uv --version
```

## <b><span style="color:rgba(14, 165, 233, 1)">7.4 常用 uv 命令</span></b>

```bash
# 安装包
uv pip install numpy pandas

# 从 requirements.txt 安装
uv pip install -r requirements.txt

# 创建虚拟环境
uv venv myproject

# 激活虚拟环境（Linux/macOS）
source myproject/bin/activate

# 激活虚拟环境（Windows）
myproject\Scripts\activate

# 同步环境
uv pip freeze > requirements.txt
uv pip sync requirements.txt

# 卸载包
uv pip uninstall numpy

# 升级包
uv pip install --upgrade numpy

# 显示已安装的包
uv pip list
```

## <b><span style="color:rgba(14, 165, 233, 1)">7.5 uv 与 conda 配合使用</span></b>

uv 和 conda 可以配合使用，各取所长：

```bash
# 使用 conda 管理大型科学计算环境
conda create -n ml python=3.11 pytorch torchvision -c pytorch

# 使用 uv 快速安装轻量级包
uv pip install black isort ruff

# 在 conda 环境中使用 uv
conda activate ml
uv pip install transformers
```

# <b><span style="color:rgba(14, 165, 233, 1)">八、安装 Zellij 终端复用器</span></b>

## <b><span style="color:rgba(14, 165, 233, 1)">8.1 为什么选择 Zellij</span></b>

<b><span style="color:rgba(245,158,11,1)">Zellij</span></b> 是一个现代化的终端多路复用器，用 Rust 编写，比 tmux 更现代，比 screen 更强大：

- **开箱即用**：零配置，智能默认设置
- **现代化 UI**：支持浮动窗格、标签页分隔
- **内置布局**：自动保存和恢复工作区布局
- **插件支持**：可扩展的插件系统
- **Web UI**：可以通过浏览器访问终端会话

## <b><span style="color:rgba(14, 165, 233, 1)">8.2 安装 Zellij</span></b>

```bash
# 使用官方安装脚本，安装到用户目录
curl -fsSL https://raw.githubusercontent.com/zellij-org/zellij/main/install.sh \
  | sh -s -- -d ~/.local/bin
```

## <b><span style="color:rgba(14, 165, 233, 1)">8.3 配置自动启动</span></b>

为了让每次启动 zsh 时自动进入 Zellij，我们可以修改 `~/.bashrc` 文件。

> 注意：这部分修改在 `~/.bashrc` 而不是 `~/.zshrc` 中，因为 Zellij 应该在 zsh 启动后立即启动。

编辑 `~/.bashrc`，找到我们之前添加的 zsh 启动行，修改为：

```bash
# 添加 zsh 到 PATH 环境变量
export PATH="$HOME/.local/bin:$PATH"

# 自动启动 zellij（如果已存在会话则附加，否则创建新会话）
exec zellij attach -c -d always 2>/dev/null || exec zsh
```

保存后，运行：

```bash
source ~/.bashrc
```

## <b><span style="color:rgba(14, 165, 233, 1)">8.4 配置 Zellij</span></b>

创建配置文件：

```bash
mkdir -p ~/.config/zellij
nano ~/.config/zellij/config.yaml
```

### <b><span style="color:rgba(217,70,239,1)">推荐配置</span></b>

以下是一个功能丰富的配置示例：

```yaml
# ============================================
# Zellij 配置文件
# 路径：~/.config/zellij/config.yaml
# ============================================

# 主题设置
theme: gruvbox

# UI 设置
ui:
  # 窗格边框样式
  pane_frames:
    # 隐藏窗格边框（更简洁）
    omitted: true
    sessions_dir: "~/.local/share/zellij/sessions"

  # 状态栏设置
  statusbar:
    enabled: true
    left:
      - session_name
      - mode
      - sync_panes
    right:
      - cpu
      - ram
      - time

  # 选项卡栏设置
  tab_bar:
    enabled: true
    show_index: true
    show_aliases: true

# 鼠标模式
mouse_mode: true

#  scrollbar（滚动条）
scrollbar:
  enabled: true
  symbol: " ▐"

# 默认 shell
default_shell:
  command: "/usr/bin/zsh"
  args: ["-l"]

# 键绑定
keybinds:
  # 解除所有默认绑定
  unbind: true

  # 基础导航
  bind:
    # 切换窗格焦点
    "Alt-h": MoveFocus Left
    "Alt-j": MoveFocus Down
    "Alt-k": MoveFocus Up
    "Alt-l": MoveFocus Right

    # 调整窗格大小
    "Alt-Left": Resize Left
    "Alt-Down": Resize Down
    "Alt-Up": Resize Up
    "Alt-Right": Resize Right

    # 窗格操作
    "Alt-n": NewPane
    "Alt-d": NewPane Right
    "Alt-S-d": NewPane Down
    "Alt-w": ClosePane
    "Alt-v": TogglePaneFullscreen
    "Alt-f": TogglePaneFrames

    # 标签页操作
    "Alt-t": NewTab
    "Alt-q": GoToTab 0
    "Alt-1": GoToTab 1
    "Alt-2": GoToTab 2
    "Alt-3": GoToTab 3
    "Alt-4": GoToTab 4

    # 复制和滚动
    "Alt-c": CopyToClipboard
    "Alt-v": PasteFromClipboard

    # 会话操作
    "Alt-a": {
      Toggle: uiPlusZellijMode
    }

    # 分离会话
    "Alt-a d": Detach

    # 其他快捷键
    "Alt-Left-tab": PreviousTab
    "Alt-Right-tab": NextTab
    "Alt--": ToggleActiveSyncTab
    "Alt-s": ToggleStatus

    # 搜索
    "/": {
      Toggle: Search
    }

    # 清屏
    "Alt-l": ClearScreen

# 插件配置（可选）
plugins:
  # 主题选择器
  # - url: https://github.com/zellij-org/zellij/wiki/Example-Zellij-Plugins#tab-bar
```

保存配置文件后，运行：

```bash
source ~/.bashrc
```

## <b><span style="color:rgba(14, 165, 233, 1)">8.5 Zellij 常用快捷键</span></b>

### <b><span style="color:rgba(217,70,239,1)">基础操作</span></b>

| 快捷键 | 功能 |
|--------|------|
| `Alt-n` | 新建窗格 |
| `Alt-d` | 水平分屏（新建右窗格） |
| `Alt-S-d` | 垂直分屏（新建下窗格） |
| `Alt-w` | 关闭当前窗格 |
| `Alt-v` | 切换窗格全屏 |
| `Alt-q` | 退出 Zellij |

### <b><span style="color:rgba(217,70,239,1)">导航操作</span></b>

| 快捷键 | 功能 |
|--------|------|
| `Alt-h` | 聚焦左侧窗格 |
| `Alt-j` | 聚焦下方窗格 |
| `Alt-k` | 聚焦上方窗格 |
| `Alt-l` | 聚焦右侧窗格 |
| `Alt-t` | 新建标签页 |
| `Alt-1~4` | 切换到指定标签页 |

### <b><span style="color:rgba(217,70,239,1)">会话管理</span></b>

| 快捷键 | 功能 |
|--------|------|
| `Ctrl-a` | 进入 Zellij 命令模式 |
| `Ctrl-a d` | 分离当前会话 |
| `Alt-a` | 打开快捷键菜单 |

