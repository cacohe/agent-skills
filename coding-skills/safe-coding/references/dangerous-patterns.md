# 高危代码模式速查表

Agent 在生成或执行代码前，应扫描是否匹配以下模式。

---

## Shell 命令

| 模式 | 风险 | 处理 |
|------|------|------|
| `rm -rf /` `rm -rf ~` `rm -rf *` | 不可逆删除 | 🔴 禁止 |
| `curl \| bash` `wget \| sh` | 远程代码执行 | 🔴 禁止 |
| `:(){ :\|:& };:` | Fork Bomb | 🔴 禁止 |
| `sudo chmod 777 /` | 权限破坏 | 🔴 禁止 |
| `> /etc/passwd` | 系统文件覆盖 | 🔴 禁止 |
| `nohup ... &` | 后台进程 | 🟡 确认 |
| `crontab -e` | 定时任务修改 | 🔴 禁止 |
| `dd if=/dev/zero of=/dev/sda` | 磁盘擦除 | 🔴 禁止 |

---

## Python

```python
# 🔴 禁止 - 系统命令注入风险
os.system(user_input)
subprocess.call(user_input, shell=True)
eval(user_input)
exec(user_input)

# 🔴 禁止 - 危险的文件操作
shutil.rmtree("/")
open("/etc/shadow").read()

# 🔴 禁止 - 不安全的反序列化
pickle.loads(untrusted_data)
yaml.load(data)               # 应使用 yaml.safe_load

# 🔴 禁止 - 硬编码密钥
API_KEY = "sk-abc123..."

# 🟡 需确认 - 递归删除用户目录
shutil.rmtree(user_specified_path)  # 执行前展示路径

# ✅ 安全替代
subprocess.run(["ls", "-la"], check=True)  # 列表形式，无 shell=True
yaml.safe_load(data)
os.environ.get("API_KEY")              # 从环境变量读取
```

---

## JavaScript / Node.js

```javascript
// 🔴 禁止 - 代码注入
eval(userInput)
new Function(userInput)()
child_process.exec(userInput)           // 字符串形式

// 🔴 禁止 - 路径穿越
fs.readFile("../../" + userInput)       // 未校验路径

// 🔴 禁止 - 硬编码密钥
const SECRET = "my-secret-key-12345"

// 🟡 需确认 - 删除操作
fs.rmSync(path, { recursive: true })    // 执行前确认路径

// ✅ 安全替代
child_process.execFile('/bin/ls', ['-la'])  // 数组形式
path.resolve(baseDir, userInput).startsWith(baseDir)  // 路径校验
process.env.SECRET                          // 环境变量
```

---

## SQL

```sql
-- 🔴 禁止（无条件）
DROP DATABASE production;
DROP TABLE users;
TRUNCATE TABLE orders;

-- 🔴 禁止 - SQL 注入风险
"SELECT * FROM users WHERE id = " + userId  -- 字符串拼接

-- 🟡 需确认 - 批量删除
DELETE FROM logs WHERE created_at < '2024-01-01';  -- 确认行数

-- ✅ 安全替代
SELECT * FROM users WHERE id = ?  -- 参数化查询
BEGIN; ... ROLLBACK;              -- 事务包裹，先验证再提交
```

---

## 正则快速检测（可在代码审查中使用）

```
危险 shell 模式：  rm\s+-rf\s+[/~*]
远程执行管道：    (curl|wget).*(bash|sh)
Fork Bomb：       :\(\)\s*\{
sudo 提权：       sudo\s+(chmod|chown|rm|dd)
硬编码密钥：      (api_key|secret|password|token)\s*=\s*["'][^"']{8,}
eval 注入：       eval\s*\(|exec\s*\(
不安全 pickle：   pickle\.loads\(
```
