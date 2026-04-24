---
name: ylapp-local-bootstrap
description: "Use for: 新电脑拉取 ylapp 后一键完成本地部署与启动。Triggers: 本地部署, 本地启动, 新电脑环境初始化, 启动 ylapp, local bootstrap, run ylapp locally."
compatibility: "Requires git, go (with GOPROXY configured), node (>=18), pnpm, buf, protoc-gen-go, protoc-gen-connect-go. Supports VS Code with run_in_terminal and vscode_askQuestions."
metadata:
  version: 1.1.0
  author: slock-team
  tags: [bootstrap, local-dev, ylapp, setup, startup]
---

# YLAPP Local Bootstrap Skill

在新电脑上从零完成 ylapp 本地运行环境搭建与服务启动。

## 目标

- 检查并验证工具链完整性
- 完成依赖安装与代码生成
- 按需启动 gosvc / fe / demobase / svc
- 所有环境变量均在运行时询问用户输入，绝不持久化

## Source of Truth

启动流程以仓库根目录 `README.md` 的"本地开发部署"章节为准。如本文与 README 冲突，以 README 为准。

## Agent 行为约束

1. **先检查工具链**，不满足时汇总缺失项并停止，不执行后续命令
2. **环境变量必须运行时收集**——通过 `vscode_askQuestions`（VS Code 环境）或交互式提示询问用户
3. **禁止持久化敏感信息**——不写入 `.env`、README、脚本文件，不提交到仓库
4. **仅在当前终端会话设置环境变量**
5. **长时间运行的服务使用异步模式**——`run_in_terminal` 的 `mode=async`（VS Code 环境），或后台运行方式
6. **日志中脱敏**——输出中涉及连接串、token 等敏感值时，仅展示脱敏版本（如 `postgres://user:***@host/db`）

## Step 1: 环境检查

在仓库根目录执行，确认以下命令均可用：

```bash
git --version
go version
node -v        # 要求 >= 18
pnpm -v
buf --version
protoc-gen-go --version
protoc-gen-connect-go --version
```

如缺任一项：**汇总所有缺失项**（不要逐个报错），根据下方安装指引提供建议，等用户安装后再继续。

## Step 1.5: 环境准备（缺失项安装指引）

仅在 Step 1 检测到缺失时执行对应部分。

### Go 语言环境

1. 安装 Go：从 <https://golang.google.cn/dl/> 下载安装
2. 配置国内镜像源：
```bash
go env -w GOPROXY=https://goproxy.cn,direct
```
3. 安装 Go 侧必要依赖（buf、protoc 插件、grpcurl）：
```bash
go install github.com/bufbuild/buf/cmd/buf@latest
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install connectrpc.com/connect/cmd/protoc-gen-connect-go@latest
```

### Node.js / pnpm 环境

1. 安装 Node.js >= 18：从 <https://nodejs.org/> 下载，或使用 nvm
2. 安装 pnpm：
```bash
npm install -g pnpm
```

### 验证

安装完成后，重新执行 Step 1 的所有检查命令，确认全部通过后再继续。

## Step 2: 依赖安装与代码生成

在仓库根目录**按顺序**执行：

```bash
git submodule init
git submodule update
pnpm install
pnpm buf generate
go mod tidy
go mod download
```

每步执行后检查退出码。任一步骤失败时：输出失败命令、错误信息，给出可能原因（网络问题、权限不足、submodule URL 不可达等），停止后续步骤。

## Step 3: 询问要启动的组件

询问用户要启动哪些组件（支持多选）：

| 组件 | 说明 | 工作目录 |
|------|------|----------|
| `gosvc` | Go 后端服务 | 仓库根目录 |
| `fe` | 前端开发服务器 | `packages/fe` |
| `demobase` | Demo 数据库服务 | `packages/demobase` |
| `svc` | Node 后端服务 | `packages/svc` |

## Step 4: 收集环境变量

**仅为用户选择的组件收集所需变量**，不要提前或多余询问。

| 组件 | 所需变量 | 用途说明（告知用户） |
|------|----------|---------------------|
| `gosvc` | `YL_CH_ADDR` | ClickHouse 服务地址 |
| `svc` | `DEMOBASE_DATABASE_URL` | 数据库连接串 |
| `demobase` | `DEMOBASE_DATABASE_URL`（可选） | 数据库连接串（如需数据库功能） |

收集规则：
- 不提供默认值，不猜测
- 用户留空时：明确提示该组件可能无法正常启动，询问是否跳过该组件
- 如 `svc` 和 `demobase` 都选中且都需要 `DEMOBASE_DATABASE_URL`，只问一次

## Step 5: 启动服务

按以下顺序启动用户选择的组件。每个服务使用**独立终端**，异步模式运行。

### gosvc（仓库根目录）

检测 shell 类型后执行：

**PowerShell:**
```powershell
$env:YL_CH_ADDR="<用户输入>"; go run ./gosvc server
```

**Bash/Zsh:**
```bash
YL_CH_ADDR="<用户输入>" go run ./gosvc server
```

### fe（packages/fe）

```bash
pnpm dev
```

### demobase（packages/demobase）

按顺序执行，仅最后一步异步：
```bash
pnpm db:gen
pnpm run build
pnpm dev          # 异步
```

### svc（packages/svc）

**PowerShell:**
```powershell
$env:DEMOBASE_DATABASE_URL="<用户输入>"; pnpm db:gen; pnpm dev
```

**Bash/Zsh:**
```bash
DEMOBASE_DATABASE_URL="<用户输入>" sh -c 'pnpm db:gen && pnpm dev'
```

## Step 6: 启动验证

启动后等待 5-10 秒，检查每个异步终端输出：

- 确认无立即崩溃（exit code 非 0、uncaught exception、panic 等）
- 对失败组件报告：
  - 失败命令
  - 关键报错（前后 5 行上下文）
  - 最可能原因及修复建议：缺环境变量、端口被占用（`EADDRINUSE`）、依赖未安装、数据库不可达

## Step 7: 完成报告

执行完成后输出简要清单：

```
✅ 启动成功：gosvc, fe
⏳ 启动中：demobase (等待编译)
❌ 启动失败：svc — DEMOBASE_DATABASE_URL 未提供，连接被拒绝
📋 下一步：为 svc 提供数据库连接串后重新启动
```

包含：
1. 成功启动的组件及其终端标识
2. 进行中的组件（如编译耗时较长）
3. 失败组件及原因
4. 下一步建议
