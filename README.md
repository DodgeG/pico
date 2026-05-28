# pico

`pico` 是一个面向代码仓库的轻量本地 coding agent。它直接跑在终端里，先看当前工作区，再用一组受约束的工具去读文件、改文件、跑命令，并把会话状态保存在本地 `.pico/` 目录里。

它更像一个能在仓库里持续工作的命令行助手，不是纯聊天窗口。你可以拿它做代码排查、测试修复、仓库分析，或者让它在当前项目里执行一次性的工程任务。

## 适合做什么

- 在本地仓库里排查测试失败
- 读取当前代码结构并给出修改建议
- 基于现有文件做小步迭代，而不是脱离仓库空想
- 在会话中保留上下文，支持继续上一次工作

## 主要特性

- 包名是 `pico`
- CLI 命令是 `pico`
- 模块入口是 `python -m pico`
- 会话保存在 `.pico/sessions/`
- 每次运行的工件保存在 `.pico/runs/<run_id>/`
- 支持四类模型后端：
  - Ollama
  - OpenAI 兼容 Responses API
  - Anthropic 兼容 Messages API
  - DeepSeek Anthropic 兼容 API

## 使用截图

CLI 帮助信息：

![pico help](assets/screenshots/pico-help.png)

启动界面：

![pico start](assets/screenshots/pico-start.png)

REPL 内置命令与会话路径：

![pico repl](assets/screenshots/pico-repl.png)

## 安装

需要 Python 3.10+。

如果你用 `uv`，直接安装依赖：

```bash
uv sync
```

如果你已经在自己的 Python 环境里工作，也可以直接装成可编辑模式：

```bash
pip install -e .
```

## 快速开始

在当前仓库里启动交互模式。当前推荐使用 DeepSeek：

```bash
uv run pico --provider deepseek
```

指定另一个工作目录：

```bash
uv run pico --cwd /path/to/repo
```

直接跑一次性任务：

```bash
uv run pico --provider deepseek "inspect the test failures and propose a fix"
```

如果当前环境已经安装过包，也可以直接这样启动：

```bash
python -m pico --provider deepseek
```

## 模型后端

Pico 启动时会读取项目根目录的 `.env`。本地真实 key 放在 `.env`，仓库只保留 `.env.example`。配置优先级是：

```text
显式 CLI 参数 > .env 里的 PICO_* 变量 > 旧环境变量 > 代码默认值
```

本地第一次配置：

```bash
cp .env.example .env
```

然后把要使用的 provider key 填进去。`.env` 已经被 `.gitignore` 忽略，不要提交真实 key。

### Ollama

```bash
ollama serve
ollama pull qwen3.5:4b
uv run pico --provider ollama --model qwen3.5:4b
```

### OpenAI 兼容接口

默认 OpenAI 兼容接口使用 right.codes 的 Codex endpoint：

```bash
PICO_OPENAI_API_BASE="https://www.right.codes/codex/v1"
PICO_OPENAI_API_KEY="your-api-key"
PICO_OPENAI_MODEL="gpt-5.4"
```

也可以改成其他 OpenAI-compatible 服务：

```bash
PICO_OPENAI_API_BASE="https://your-api.example/v1"
PICO_OPENAI_API_KEY="your-api-key"
PICO_OPENAI_MODEL="gpt-5.4"
```

```bash
uv run pico --provider openai
```

### Anthropic 兼容接口

默认 Anthropic 兼容接口使用 right.codes 的 Claude endpoint：

```bash
PICO_ANTHROPIC_API_BASE="https://www.right.codes/claude/v1"
PICO_ANTHROPIC_API_KEY="your-api-key"
PICO_ANTHROPIC_MODEL="claude-sonnet-4-6"
```

```bash
uv run pico --provider anthropic
```

如果你的服务端对多个兼容接口复用了同一套密钥，`pico` 也支持从 `PICO_ANTHROPIC_API_KEY` 回退到 `ANTHROPIC_API_KEY`、`PICO_RIGHT_CODES_API_KEY`、`RIGHT_CODES_API_KEY`、`PICO_OPENAI_API_KEY` 或 `OPENAI_API_KEY`。

### DeepSeek

```bash
PICO_DEEPSEEK_API_KEY="your-api-key"
PICO_DEEPSEEK_MODEL="deepseek-v4-pro"
```

```bash
uv run pico --provider deepseek
```

默认 DeepSeek base URL 是 `https://api.deepseek.com/anthropic`，走 DeepSeek 的 Anthropic 兼容接口。如果需要改到代理服务，可以设置 `PICO_DEEPSEEK_API_BASE` 或启动时传 `--base-url`。

## 常用交互命令

- `/help`：查看内置命令
- `/memory`：查看提炼后的工作记忆
- `/session`：查看当前会话文件路径
- `/reset`：清空当前会话状态
- `/exit` 或 `/quit`：退出 REPL

## 安全与持久化

`pico` 不会默认把所有动作都放开。像 shell 执行、文件写入这类高风险操作，会受审批模式控制：

- `--approval ask`
- `--approval auto`
- `--approval never`

每次运行结束后，都会在 `.pico/runs/<run_id>/` 下写出这些文件：

- `task_state.json`
- `trace.jsonl`
- `report.json`

这些内容默认只保存在本地，不需要跟仓库一起提交。

## 开发

如果装了 Ruff，可以这样检查：

```bash
uv run ruff check .
```

## Git 配置与同步

这个仓库当前建议保留两个远端：

- `origin`：你自己的 GitHub 仓库，用来日常开发和备份
- `upstream`：Gitee 上的原始仓库，用来同步上游更新

第一次在当前电脑配置远端：

```bash
git remote -v
git remote add upstream https://gitee.com/htxoffical/pico.git
git fetch upstream
```

如果 `upstream` 已经存在，但地址不对，可以改成：

```bash
git remote set-url upstream https://gitee.com/htxoffical/pico.git
git fetch upstream
```

查看当前本地、GitHub 和 Gitee 的关系：

```bash
git remote -v
git branch -vv
git status --short --branch
```

### 日常同步上游

先看 Gitee 上游是否有新提交：

```bash
git fetch upstream
git log --oneline --graph --decorate main..upstream/main
git log --oneline --graph --decorate upstream/main..main
```

含义：

- `main..upstream/main`：上游有、你本地没有的提交
- `upstream/main..main`：你本地有、上游没有的提交

如果只是想把 Gitee 的更新合到你当前分支，推荐先用 `merge`：

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

`merge` 的好处是更稳，更适合“我在 GitHub 上有自己的改动，同时还要继续跟 Gitee 上游同步”的情况。

如果你想保留更线性的提交历史，也可以用 `rebase`：

```bash
git checkout main
git fetch upstream
git rebase upstream/main
git push --force-with-lease origin main
```

只有当你明确知道自己在做什么时，才建议用 `rebase` 后强推。

如果上游只改了几个你想单独拿过来的提交，可以用：

```bash
git fetch upstream
git log --oneline upstream/main
git cherry-pick <commit1> <commit2>
git push origin main
```

### 同步前的建议

同步前最好先把工作区清理一下，尤其是本地生成物：

```bash
git status --short --branch
```

如果你只是临时想把未提交改动收起来：

```bash
git stash -u
```

同步完成后再恢复：

```bash
git stash pop
```

如果遇到冲突：

```bash
git status
```

手动解决冲突后：

```bash
git add <冲突文件>
git commit
```

如果你在走 `rebase`，则改为：

```bash
git add <冲突文件>
git rebase --continue
```

## 换电脑后的完整流程

如果你以后换电脑，推荐按下面这套顺序恢复这个项目。

### 1. 拉取 GitHub 仓库

```bash
git clone https://github.com/DodgeG/pico.git
cd pico
```

### 2. 配置 Git 远端

把 Gitee 原仓库配置成 `upstream`，以后可以继续同步上游更新：

```bash
git remote add upstream https://gitee.com/htxoffical/pico.git
git fetch upstream
git remote -v
```

你应该看到类似：

```text
origin    https://github.com/DodgeG/pico.git
upstream  https://gitee.com/htxoffical/pico.git
```

### 3. 准备 Python 环境

项目需要 Python 3.10+。

如果新电脑没有 `uv`，最稳妥的方式是直接用 Python 自带虚拟环境：

```bash
python3.11 -m venv .venv
. .venv/bin/activate
pip install -U pip
pip install -e .
```

如果你已经装了 `uv`，也可以继续用：

```bash
uv sync
```

### 4. 恢复本地 `.env`

```bash
cp .env.example .env
```

然后把你常用 provider 的 key 填进去，例如 DeepSeek：

```bash
PICO_DEEPSEEK_API_BASE=https://api.deepseek.com/anthropic
PICO_DEEPSEEK_API_KEY=your-api-key
PICO_DEEPSEEK_MODEL=deepseek-v4-pro
```

`.env` 不要提交到仓库。

### 5. 启动并验证

交互式启动：

```bash
.venv/bin/pico --provider deepseek
```

或 one-shot 验证：

```bash
.venv/bin/pico --provider deepseek "只回复：启动成功"
```

### 6. 之后的日常工作流

在新电脑上的常规节奏通常是：

```bash
git checkout main
git pull origin main
git fetch upstream
git merge upstream/main
git push origin main
```

如果你准备继续开发自己的改动，建议新建分支：

```bash
git checkout -b my-change
```

开发完成后：

```bash
git add .
git commit -m "your message"
git push origin my-change
```

如果之后再把分支合回 `main`，再继续按上面的 `origin + upstream` 双远端方式维护即可。
