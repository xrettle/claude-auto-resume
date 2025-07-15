# Claude Auto-Resume

一个能够在 Claude CLI 使用限制解除后自动恢复任务的 Shell 脚本工具，或在等待期间后执行自定义 Shell 命令。它能检测 Claude 使用限制，智能等待，并自动恢复任务执行。

[English](README.md) | 中文

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F11O935)

## ⚠️ 安全警告

**此脚本在执行 Claude 命令时使用 `--dangerously-skip-permissions` 标志，并且可以执行任意 Shell 命令**，这意味着：

- **Claude Code 将在不询问权限的情况下执行任务**
- **自定义 Shell 命令将在不经过用户确认的情况下执行**
- **文件操作、系统命令和代码更改将自动运行**
- **只能在受信任的环境中使用，且仅对受信任的提示/命令使用**
- **运行此脚本前请仔细审查您的提示或命令**

**推荐用法：**
- 在隔离的开发环境中使用
- 避免在生产系统或包含敏感数据的环境中使用
- 提示要具体，以限制操作范围
- 考虑自动执行的潜在影响

## 使用场景

此脚本在以下开发场景中使用 Claude Code 时特别有用：

1. **任务被使用限制中断**：当您的 Claude Code 显示 `Claude usage limit reached.` 但任务尚未完全完成时
2. **自动任务恢复**：在项目根目录中运行 `claude-auto-resume`，当使用限制解除时，脚本将自动让 Claude Code 继续执行您之前未完成的任务
3. **自定义命令执行**：在等待使用限制后执行任何 Shell 命令，适用于重启服务、运行构建或处理数据

## 特性

- 🔄 自动检测 Claude CLI 使用限制
- ⏰ 智能等待，带倒计时显示
- 🚀 自动任务恢复
- 🔧 等待期间后执行自定义命令
- 🛡️ 带取消选项的安全警告
- 🔗 支持复杂命令（管道、重定向、操作符）
- 🧪 内置测试模式，用于开发和验证
- 🖥️ 跨平台支持（Linux/macOS）
- 📦 零外部依赖（仅需要标准 Unix 工具）

## 安装

### 方法 1：使用 Makefile（推荐）

```bash
# 全局安装
sudo make install

# 安装到自定义位置
sudo make install PREFIX=/opt/local

# 卸载
sudo make uninstall
```

### 方法 2：手动安装

```bash
# 复制到系统路径
sudo cp claude-auto-resume.sh /usr/local/bin/claude-auto-resume
sudo chmod +x /usr/local/bin/claude-auto-resume

# 或创建符号链接
sudo ln -s $(pwd)/claude-auto-resume.sh /usr/local/bin/claude-auto-resume
```

### 方法 3：直接使用（无需安装）

```bash
# 让脚本可执行
chmod +x claude-auto-resume.sh

# 直接运行
./claude-auto-resume.sh
```

## 使用方法

### 基本用法

```bash
# 使用默认提示 "continue" 开始新会话
claude-auto-resume

# 使用自定义提示开始新会话
claude-auto-resume "实现用户认证"

# 使用标志指定自定义提示开始新会话
claude-auto-resume -p "编写单元测试"

# 使用自定义提示继续之前的对话
claude-auto-resume -c "请继续之前的任务"

# 使用标志继续之前的对话
claude-auto-resume -c -p "从我们离开的地方继续"

# 等待期间后执行自定义命令
claude-auto-resume -e "npm run dev"

# 使用别名标志执行自定义命令
claude-auto-resume --cmd "python app.py"

# 显示帮助
claude-auto-resume --help
```

### 本地使用（安装前）

```bash
# 确保脚本可执行
chmod +x claude-auto-resume.sh

# 使用默认提示开始新会话
./claude-auto-resume.sh

# 使用自定义提示开始新会话
./claude-auto-resume.sh "创建登录页面"

# 继续之前的对话
./claude-auto-resume.sh -c "继续实现"

# 等待期间后执行自定义命令
./claude-auto-resume.sh -e "make build"
```

## 工作原理

1. **检测限制**：执行 `claude -p 'check'` 命令
2. **解析输出**：寻找 `Claude AI usage limit reached|<timestamp>` 格式的消息
3. **计算等待时间**：基于时间戳计算所需等待时间
4. **显示倒计时**：显示实时剩余等待时间
5. **自动恢复**：自动执行以下之一：
   - `claude --dangerously-skip-permissions -p "<custom-prompt>"`（新会话，默认）
   - `claude -c --dangerously-skip-permissions -p "<custom-prompt>"`（使用 -c 标志继续对话）
   - 使用 `-e/--execute` 或 `--cmd` 标志执行自定义 Shell 命令

## 命令行选项

- **无参数**：使用默认提示 "continue" 开始新会话
- **单个参数**：使用自定义提示开始新会话（例如：`claude-auto-resume "实现功能"`）
- **-p, --prompt**：使用标志指定自定义提示（例如：`claude-auto-resume -p "编写测试"`）
- **-c, --continue**：继续之前的对话（在 claude 命令中添加 -c 标志）
- **-e, --execute**：等待期间后执行自定义 Shell 命令（例如：`claude-auto-resume -e "npm run dev"`）
- **--cmd**：-e/--execute 的别名（例如：`claude-auto-resume --cmd "python app.py"`）
- **--test-mode**：[开发] 使用指定的等待时间（秒）模拟使用限制
- **-h, --help**：显示帮助信息和使用示例
- **-v, --version**：显示版本信息
- **--check**：显示系统检查信息

## 会话类型

### 开始新会话（默认）
使用不带 `-c` 的 `claude` 进行新对话：
```bash
claude-auto-resume                    # 使用 "continue" 的新会话
claude-auto-resume "新功能"            # 使用自定义提示的新会话
claude-auto-resume -p "编写测试"       # 使用标志的新会话
```

### 继续之前的对话
使用 `claude -c` 继续上次对话：
```bash
claude-auto-resume -c "继续进行"           # 使用自定义提示继续
claude-auto-resume -c -p "恢复工作"        # 使用标志继续
```

### 执行自定义命令
等待期间后执行任何 Shell 命令：
```bash
claude-auto-resume -e "npm run dev"                    # 启动开发服务器
claude-auto-resume --cmd "python app.py"               # 运行 Python 应用
claude-auto-resume -e "make build && ./app"            # 带操作符的复杂命令
claude-auto-resume -e "ls -la | grep '.js' | wc -l"    # 管道命令
claude-auto-resume -e "echo '步骤 1'; echo '步骤 2'"     # 多个命令
```

### 开发和测试
使用内置测试模式进行开发和验证：
```bash
claude-auto-resume --test-mode 5 -e "echo '测试命令'"    # 5秒等待测试
claude-auto-resume --test-mode 10 --cmd "npm run test"  # 测试构建过程
```

## 系统要求

- **Claude CLI**：必须已安装并在 PATH 中可用
- **标准 Unix 工具**：`grep`、`date`、`sleep`、`awk`（通常预装）

## 安全考虑

### 权限绕过
此脚本使用 `--dangerously-skip-permissions` 来启用无人值守操作。这意味着：

1. **无交互提示**：Claude 在执行命令前不会要求确认
2. **自动执行**：文件更改、系统命令和其他操作无需用户批准即可运行
3. **信任要求**：您必须信任脚本和您提供的提示

### 最佳实践
- **环境隔离**：仅在开发/测试环境中使用
- **提示审查**：仔细编写提示以限制范围（例如："在 src/auth.js 中继续实现登录功能"）
- **命令审查**：验证自定义命令对您的环境是安全和适当的
- **备份工作**：运行前确保您有版本控制或备份
- **监控执行**：检查输出以了解执行了哪些操作
- **限制范围**：使用具体的提示/命令而非开放式的

## 错误处理

脚本包含全面的错误处理：

- **退出代码 1**：Claude CLI 执行失败
- **退出代码 2**：无法提取有效的恢复时间戳
- **退出代码 4**：恢复命令执行失败

## 测试

```bash
# 语法检查
make test

# 或直接使用 bash
bash -n claude-auto-resume.sh
```

## 项目结构

```
claude-auto-resume/
├── claude-auto-resume.sh    # 主脚本
├── Makefile                 # 安装/卸载脚本
├── docs/                    # 项目文档
│   ├── architecture.md      # 架构文档
│   ├── prd.md              # 产品需求文档
│   └── stories/            # 用户故事
├── CLAUDE.md               # Claude Code 指南
├── README.md               # 项目说明（英文）
└── README.zh.md            # 项目说明（中文）
```

## 路线图

查看我们的[开发路线图](docs/ROADMAP.md)了解计划的功能和改进，包括：

- **第一阶段**：核心稳定性改进（环境验证、错误处理）
- **第二阶段**：功能扩展（自定义命令执行、配置选项）
- **第三阶段**：用户体验优化（增强帮助、更好的时间显示）

## 贡献

1. Fork 此仓库
2. 创建功能分支（`git checkout -b feature/AmazingFeature`）
3. 提交您的更改（`git commit -m 'Add some AmazingFeature'`）
4. 推送到分支（`git push origin feature/AmazingFeature`）
5. 创建 Pull Request

在贡献新功能前，请查看我们的[路线图](docs/ROADMAP.md)以确保与项目目标一致。

## 许可证

此项目基于 MIT 许可证授权 - 详情请参阅 [LICENSE](LICENSE) 文件

## 支持

如果您遇到问题或有建议：

1. 查看现有的[问题](https://github.com/terryso/claude-auto-resume/issues)
2. 创建新的问题描述问题
3. 或提交 Pull Request

## ⭐ Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=terryso/claude-auto-resume&type=Date)](https://www.star-history.com/#terryso/claude-auto-resume&Date)

---

**注意**：此工具依赖于 Claude CLI 输出格式。如果 Claude CLI 更新改变了输出格式，脚本可能需要更新。