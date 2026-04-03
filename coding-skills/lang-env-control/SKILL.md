---
name: lang-env-control
description: >
  编程语言环境管理守则。当 Agent 涉及任何 Python 或 Node.js 编程任务时，必须强制触发此 Skill。
  适用场景包括但不限于：编写 Python 脚本、新建 Python 项目、安装 Python 依赖包（pip install / uv add）、
  执行 python 命令、编写 Node.js 项目、安装 npm 包、运行 node 脚本。
  只要任务中涉及 Python 或 Node.js 的环境、依赖、包管理，无论用户是否明确提及工具选择，都必须加载此 Skill。
---

# 编程语言环境管理守则

本 Skill 规定 Agent 在 Python 与 Node.js 项目中必须遵守的环境与依赖管理规范。
规则分三级：🔴 **禁止**（永不执行）、🟡 **需确认**（执行前必须询问用户）、🟢 **允许**（可直接执行）。

---

## 一、Python 环境规范

### 工具要求
- **必须使用 `uv`** 管理 Python 依赖和虚拟环境，禁止使用裸 `pip install`（系统级）。

---

### 🔴 禁止

- 直接向系统 Python 安装包：
  - `pip install <package>`（未激活虚拟环境时）
  - `pip3 install <package>`（未激活虚拟环境时）
  - `sudo pip install <package>`
  - `python -m pip install <package>`（未激活虚拟环境时）
- 使用 `conda`、`pipenv`、`poetry` 等其他工具管理 Python 环境（除非用户明确要求）
- 在检测到虚拟环境不存在的情况下，直接执行 `python` / `python3` 命令运行业务脚本

---

### 🟡 需确认（执行前必须询问用户）

**场景 A：执行 python 命令时，当前目录无虚拟环境**
```
检测到当前目录下没有 Python 虚拟环境（.venv/）。
是否需要使用 uv 创建虚拟环境后再执行？
  [Y] 是，使用 `uv venv` 创建并激活虚拟环境
  [N] 否，直接执行（不推荐）
```

**场景 B：需要安装 Python 依赖包，但无虚拟环境**
```
需要安装依赖包 <package>，但当前目录下没有 Python 虚拟环境。
是否创建虚拟环境并安装？
  [Y] 是，使用 `uv venv` 创建虚拟环境，再用 `uv add <package>` 安装
  [N] 否，取消安装
```

> ⚠️ 若用户选择 N（不创建虚拟环境），Agent 必须停止安装操作，不得回退到系统 pip 安装。

---

### 🟢 允许（可直接执行）

- 新建 Python 项目时，使用 `uv init` 初始化项目结构
- 使用 `uv venv` 创建虚拟环境（路径默认为 `.venv/`）
- 使用 `uv add <package>` 在已有虚拟环境中安装依赖
- 使用 `uv run <script>` 在虚拟环境中执行脚本
- 使用 `uv sync` 同步依赖
- 在已激活虚拟环境（`source .venv/bin/activate` 或 `.venv\Scripts\activate`）下执行 `python` 命令

---

### 虚拟环境检测方法

执行任何 Python 相关命令前，Agent 应先检测虚拟环境是否存在：

```bash
# 检测当前目录是否有 .venv
if [ -d ".venv" ]; then
  echo "虚拟环境存在"
else
  echo "虚拟环境不存在，需要确认"
fi

# 或检测是否已激活虚拟环境
if [ -n "$VIRTUAL_ENV" ]; then
  echo "虚拟环境已激活: $VIRTUAL_ENV"
fi
```

---

### 典型工作流

**新建 Python 项目：**
```bash
uv init my-project
cd my-project
uv venv          # 创建 .venv 虚拟环境
uv add requests  # 安装依赖
uv run main.py   # 运行脚本
```

**在已有项目中安装依赖：**
```bash
# 检测 .venv 存在后
uv add pandas numpy
```

**运行脚本：**
```bash
uv run python script.py
# 或激活环境后
source .venv/bin/activate
python script.py
```

---

## 二、Node.js 环境规范

### 工具要求
- **必须使用 `pnpm`** 管理 Node.js 依赖，禁止使用 `npm install` 或 `yarn`（除非用户明确要求）。

---

### 🔴 禁止

- 使用 `npm install <package>` 安装依赖
- 使用 `yarn add <package>` 安装依赖
- 使用 `npm i` 的简写形式

---

### 🟢 允许（可直接执行）

- 使用 `pnpm install` 安装全部依赖
- 使用 `pnpm add <package>` 安装新依赖
- 使用 `pnpm add -D <package>` 安装开发依赖
- 使用 `pnpm remove <package>` 移除依赖
- 使用 `pnpm run <script>` 执行脚本
- 使用 `pnpm dlx <tool>` 执行临时工具（替代 `npx`）
- 新建项目时使用 `pnpm create <template>` 初始化

---

### 典型工作流

**新建 Node.js 项目：**
```bash
pnpm create vite my-app   # 用模板初始化
cd my-app
pnpm install              # 安装依赖
pnpm add axios            # 添加新依赖
pnpm add -D typescript    # 添加开发依赖
pnpm run dev              # 启动开发服务器
```

---

## 三、决策速查表

| 场景 | 操作 | 级别 |
|------|------|------|
| Python：安装包，有 .venv | `uv add <pkg>` | 🟢 允许 |
| Python：安装包，无 .venv | 询问是否创建虚拟环境 | 🟡 确认 |
| Python：`pip install`（系统级） | 永不执行 | 🔴 禁止 |
| Python：执行脚本，无 .venv | 询问是否创建虚拟环境 | 🟡 确认 |
| Python：新建项目 | `uv init` + `uv venv` | 🟢 允许 |
| Node.js：安装依赖 | `pnpm add <pkg>` | 🟢 允许 |
| Node.js：`npm install` | 永不执行 | 🔴 禁止 |
| Node.js：初始化项目 | `pnpm create ...` | 🟢 允许 |