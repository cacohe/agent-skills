# 安全替代方案速查

当用户要求执行危险操作时，优先提供以下安全替代方案。

---

## 文件删除

| 危险做法 | 安全替代 |
|----------|----------|
| `rm -rf ./dist` | 先 `ls ./dist` 确认内容，再执行 |
| Python `shutil.rmtree(path)` | 先打印路径，执行前 `input("确认删除？")` |
| 递归清理日志 | 用 `find . -name "*.log" -print` 预览，再 `-delete` |

**推荐模式（Python）：**
```python
import os, shutil

def safe_delete(path: str, dry_run: bool = True):
    """删除前预览，dry_run=True 时只展示不删除"""
    if not os.path.exists(path):
        print(f"路径不存在: {path}")
        return
    print(f"将删除: {path}（{'预览模式' if dry_run else '实际删除'}）")
    if not dry_run:
        shutil.rmtree(path)
```

---

## Shell 命令执行

| 危险做法 | 安全替代 |
|----------|----------|
| `os.system(cmd)` | `subprocess.run([...], check=True)` |
| `shell=True` + 用户输入 | 列表传参，绝不拼接字符串 |
| `curl URL \| bash` | 先下载到临时文件，审查后再执行 |

**推荐模式（Python）：**
```python
import subprocess

# ✅ 安全：列表形式，无 shell 注入风险
result = subprocess.run(
    ["git", "clone", repo_url, target_dir],
    capture_output=True, text=True, check=True, timeout=60
)
```

---

## 密钥管理与 .env 文件规范

| 危险做法 | 安全替代 |
|----------|----------|
| 硬编码在源码 | `os.environ.get("API_KEY")` |
| 密钥写入 `.env` 并提交 Git | 密钥只写 `.env.local`，`.gitignore` 排除所有 `.env*` |
| 日志打印密钥 | 日志脱敏：`key[:4] + "****"` |
| 明文存储密码 | bcrypt / argon2 哈希存储 |
| 部署时上传 `.env.local` 到云平台 | 在平台控制台逐项手动配置环境变量 |

**项目标准结构：**
```
project/
├── .env.example      # ✅ 提交 Git — 仅含变量名，值为空或占位符
├── .env.local        # 🔴 禁止提交 — 本地真实密钥
└── .gitignore        # 必须包含下方条目
```

**`.env.example` 示例（可提交，无真实值）：**
```bash
# 数据库
DATABASE_URL=

# OpenAI
OPENAI_API_KEY=

# GitHub OAuth
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
```

**`.env.local` 示例（禁止提交，含真实值）：**
```bash
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
GITHUB_CLIENT_ID=Iv1.xxxxxxxx
GITHUB_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**必须包含的 `.gitignore` 条目：**
```gitignore
# 环境变量（.env.example 除外）
.env
.env.local
.env.*.local
.env.development
.env.production
.env.test
.env.staging
*.pem
*.key
secrets/
```

**云平台部署时的正确密钥注入方式（以 Vercel 为例）：**
```
# 不要上传任何 .env 文件！
# 正确做法：在平台控制台手动配置

Vercel Dashboard → 项目 → Settings → Environment Variables
  → 添加 OPENAI_API_KEY = sk-xxx...（仅在控制台填写，不写入任何文件）
  → 添加 DATABASE_URL  = postgresql://...

# 应用代码中读取方式不变，平台运行时自动注入：
process.env.OPENAI_API_KEY   # Node.js
os.environ.get("DATABASE_URL")  # Python
```

---

## 数据库操作

| 危险做法 | 安全替代 |
|----------|----------|
| 字符串拼接 SQL | 参数化查询 / ORM |
| 直连生产库修改数据 | 先在测试库验证，用事务包裹 |
| 无 WHERE 的 DELETE/UPDATE | 加 `LIMIT` + `WHERE` + 事务，先 SELECT 确认行数 |
| 直接 DROP TABLE | 重命名备份：`ALTER TABLE x RENAME TO x_backup_20240101` |

**推荐模式（事务 + 预检）：**
```sql
-- 步骤 1：预检影响行数
SELECT COUNT(*) FROM orders WHERE status = 'cancelled';

-- 步骤 2：事务内执行
BEGIN;
DELETE FROM orders WHERE status = 'cancelled';
-- 确认结果后再 COMMIT，否则 ROLLBACK
COMMIT;
```

---

## 网络请求

| 危险做法 | 安全替代 |
|----------|----------|
| `verify=False` 跳过 TLS | 始终验证证书；若自签名，指定 CA 路径 |
| 无限重试 | 设置最大重试次数 + 指数退避 + 超时 |
| 将用户数据发往第三方 | 脱敏后发送，或仅发送必要字段 |

**推荐模式（Python requests）：**
```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3, backoff_factor=1, status_forcelist=[500, 502, 503])
session.mount("https://", HTTPAdapter(max_retries=retry))

response = session.get(url, timeout=10)  # 始终设置 timeout
response.raise_for_status()
```

---

## 权限最小化

| 场景 | 推荐做法 |
|------|----------|
| 读写 S3 | 仅授予目标 bucket 的 `s3:GetObject` + `s3:PutObject`，禁止 `s3:*` |
| 数据库访问 | 只读任务创建只读账号，写任务限定表级权限 |
| Docker 容器 | 不使用 `--privileged`；必要时仅挂载所需目录 |
| CI/CD Secret | 使用平台级 Secret 存储，不传入环境变量日志 |
