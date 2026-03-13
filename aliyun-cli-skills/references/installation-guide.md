# 阿里云 CLI 安装、更新与卸载指南

> 官方文档：https://help.aliyun.com/zh/cli/installation-guide/

## 目录

- [Linux 安装](#linux-安装)
- [macOS 安装](#macos-安装)
- [Windows 安装](#windows-安装)
- [更新 CLI](#更新-cli)
- [卸载 CLI](#卸载-cli)
- [验证安装](#验证安装)

---

## Linux 安装

### 方式一：Bash 脚本安装（推荐）

```bash
# 安装最新版本
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)"

# 安装指定版本
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)" -- -V 3.0.277

# 查看帮助
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)" -- -h
```

### 方式二：TGZ 安装包

```bash
# 查看系统架构
uname -m
# arm64 或 aarch64 = ARM64 架构
# 其他 = AMD64 架构

# AMD64 系统
curl https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz -o aliyun-cli-linux-latest.tgz

# ARM64 系统
curl https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-arm64.tgz -o aliyun-cli-linux-latest.tgz

# 解压并安装
tar xzvf aliyun-cli-linux-latest.tgz
sudo mv ./aliyun /usr/local/bin/
```

---

## macOS 安装

### 方式一：PKG 安装包（推荐）

1. 下载：https://aliyuncli.alicdn.com/aliyun-cli-latest.pkg
2. 双击安装包，按提示完成安装

### 方式二：Homebrew

```bash
# 中国内地用户可先设置镜像源（可选）
export HOMEBREW_INSTALL_FROM_API=1
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
brew update

# 安装
brew install aliyun-cli
```

### 方式三：Bash 脚本安装

```bash
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)"
```

### 方式四：TGZ 安装包

```bash
# 下载（通用版本，支持 Intel 和 Apple Silicon）
curl https://aliyuncli.alicdn.com/aliyun-cli-macosx-latest-universal.tgz -o aliyun-cli-macosx-latest-universal.tgz

# 解压并安装
tar xzvf aliyun-cli-macosx-latest-universal.tgz
sudo mv ./aliyun /usr/local/bin/
```

---

## Windows 安装

> **注意**：阿里云 CLI 仅支持 Windows AMD64 架构，暂不支持 32 位及 ARM64 架构。

### 方式一：ZIP 安装包（推荐）

1. 下载：https://aliyuncli.alicdn.com/aliyun-cli-windows-latest-amd64.zip
2. 解压 `aliyun.exe` 到目标目录（如 `C:\AliyunCLI`）
3. 添加环境变量：
   - 按 `Win + S`，搜索"环境变量"
   - 点击"编辑账户的环境变量"
   - 在用户变量中选择 `Path`，点击编辑
   - 新建，输入安装目录路径
   - 确定保存，重启终端

### 方式二：PowerShell 脚本

```powershell
# 下载脚本内容并保存为 Install-CLI-Windows.ps1，然后执行：

# 安装最新版本（默认安装到 %LOCALAPPDATA%\AliyunCLI）
powershell.exe -ExecutionPolicy Bypass -File Install-CLI-Windows.ps1

# 安装指定版本到指定目录
powershell.exe -ExecutionPolicy Bypass -File Install-CLI-Windows.ps1 -Version 3.0.277 -InstallDir "C:\AliyunCLI"
```

---

## 更新 CLI

> **重要**：建议使用与初始安装相同的方式进行更新。

### Linux

```bash
# Bash 脚本更新
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)"

# TGZ 安装包更新（覆盖旧文件）
curl https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz -o aliyun-cli-linux-latest.tgz
tar xzvf aliyun-cli-linux-latest.tgz -C /usr/local/bin/
```

### macOS

```bash
# Homebrew 更新
brew update && brew upgrade aliyun-cli && brew cleanup aliyun-cli

# Bash 脚本更新
/bin/bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)"

# PKG 安装包更新：下载最新 PKG 并重新安装
```

### Windows

```powershell
# 下载最新 ZIP 包，解压覆盖旧的 aliyun.exe
# 或使用 PowerShell 脚本重新安装
```

---

## 卸载 CLI

### Linux/macOS

```bash
# Homebrew 卸载（仅限 Homebrew 安装）
brew uninstall aliyun-cli

# 命令行卸载
sudo sh -c "which aliyun | xargs -r rm -v"

# Bash 脚本卸载（需创建脚本文件）
# 仅卸载可执行文件
bash uninstall.sh

# 卸载并删除配置文件
bash uninstall.sh -C
```

### Windows

1. **图形界面**：删除 `aliyun.exe`，从环境变量 `Path` 中移除安装目录
2. **PowerShell 脚本**：
   ```powershell
   # 仅卸载
   powershell.exe -ExecutionPolicy Bypass -File Uninstall-CLI-Windows.ps1
   
   # 卸载并删除配置
   powershell.exe -ExecutionPolicy Bypass -File Uninstall-CLI-Windows.ps1 -Clean
   ```

### 删除配置文件（可选）

配置文件位置：
- **Windows**: `C:\Users\<USERNAME>\.aliyun`
- **Linux/macOS**: `~/.aliyun`

---

## 验证安装

```bash
aliyun version
# 输出版本号（如 3.0.277）表示安装成功
```

## 配置凭证

安装完成后，执行以下命令配置 AccessKey：

```bash
aliyun configure
```

按提示输入：
- AccessKey ID
- AccessKey Secret
- 默认地域（如 cn-hangzhou）
- 输出格式（json/table/text）
