---
name: clean-code-standards
description: 生成符合商用级要求的高质量代码。当用户要求编写代码、创建项目、实现功能模块时使用此skill。确保代码具有清晰的项目目录结构、高可读性、高可扩展性、高可维护性、高可复用性，并符合clean code原则。日志模块必须独立封装。
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: code-generation
---

# Clean Code Standards Skill

## 目标

生成符合**商用级标准**的代码，覆盖以下核心维度：

- 清晰规范的项目目录结构
- Clean Code 原则（命名、函数、注释）
- 高可读性 / 高可扩展性 / 高可维护性 / 高可复用性
- 日志模块独立封装
- 关键代码必须有注释
- 只完成用户指定需求，不随意添加无关代码

---

## 一、项目目录结构规范

### 原则
- 同类职责文件归入同一目录，**避免平铺**
- 每个目录只做一件事，职责单一

---

## 二、Clean Code 命名规范

### 变量与函数命名
- 使用**表意名称**，避免缩写（除公认缩写外如 `id`、`url`）
- 布尔变量以 `is`、`has`、`can`、`should` 开头
- 函数名以**动词**开头，体现行为：`getUserById`、`validateEmail`、`parseConfig`
- 常量全大写 + 下划线：`MAX_RETRY_COUNT`

### 命名示例（正确 vs 错误）
```python
# ❌ 错误：含义不清
def proc(d):
    if d.s == 1:
        return d.n

# ✅ 正确：清晰表意
def get_active_username(user: User) -> str:
    if user.status == UserStatus.ACTIVE:
        return user.name
```

---

## 三、函数设计原则

1. **单一职责**：每个函数只做一件事
2. **短小精悍**：函数体不超过 20–30 行（超出则拆分）
3. **参数控制**：参数不超过 3 个；超过时使用对象/数据类封装
4. **无副作用**：工具函数必须是纯函数，避免修改外部状态
5. **提前返回**：减少嵌套层级，使用 guard clause

```python
# ❌ 深层嵌套，难以阅读
def process_order(order):
    if order:
        if order.items:
            if order.is_paid:
                return fulfill(order)

# ✅ 提前返回，逻辑清晰
def process_order(order):
    if not order:
        return None
    if not order.items:
        return None
    if not order.is_paid:
        return None
    return fulfill(order)
```

---

## 四、注释规范

### 必须注释的场景
- 模块/类/公共函数：使用 docstring（Python）或 JSDoc（JS/TS）
- 非显而易见的业务逻辑（"为什么"而非"是什么"）
- 复杂算法的核心步骤
- TODO / FIXME 标记（需附说明）

### 禁止的注释
- 重复代码本身内容的注释（`i += 1  # i加1`）
- 大段注释掉的废弃代码（应使用版本控制）

```python
def calculate_discount(price: float, user_level: int) -> float:
    """
    根据用户等级计算折扣价格。

    Args:
        price: 原始价格，单位为分
        user_level: 用户等级，1=普通，2=VIP，3=SVIP

    Returns:
        折扣后的价格
    """
    # VIP用户享受9折，SVIP享受8折，依据运营策略2024-Q1
    DISCOUNT_MAP = {1: 1.0, 2: 0.9, 3: 0.8}
    discount_rate = DISCOUNT_MAP.get(user_level, 1.0)
    return price * discount_rate
```

---

## 五、日志模块规范（必须独立封装）

**核心要求：日志模块必须写在独立文件中，不得在业务代码中直接配置日志。**

### Python 示例：`src/logger.py`
```python
"""
日志模块 - 统一配置，全局使用。

所有模块通过 `from src.logger import get_logger` 获取 logger，
禁止在业务模块中直接调用 logging.basicConfig()。
"""

import logging
import sys
from typing import Optional


def get_logger(name: str, level: Optional[int] = None) -> logging.Logger:
    """
    获取指定名称的 logger 实例。

    Args:
        name: logger 名称，建议传入 __name__
        level: 日志级别，默认使用环境变量 LOG_LEVEL 或 INFO

    Returns:
        配置好的 Logger 实例
    """
    logger = logging.getLogger(name)

    # 避免重复添加 handler
    if logger.handlers:
        return logger

    # 统一日志格式：时间 | 级别 | 模块名 | 消息
    formatter = logging.Formatter(
        fmt="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(level or logging.INFO)

    return logger
```

### TypeScript 示例：`src/logger.ts`
```typescript
/**
 * 日志模块 - 统一封装，禁止在业务代码中直接使用 console.log。
 * 使用方式：import { getLogger } from '@/logger';
 */

type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface Logger {
  debug: (msg: string, ...args: unknown[]) => void;
  info:  (msg: string, ...args: unknown[]) => void;
  warn:  (msg: string, ...args: unknown[]) => void;
  error: (msg: string, ...args: unknown[]) => void;
}

/** 格式化日志输出，附加时间戳和模块名 */
function createLogger(module: string): Logger {
  const format = (level: LogLevel, msg: string): string =>
    `${new Date().toISOString()} | ${level.toUpperCase().padEnd(5)} | ${module} | ${msg}`;

  return {
    debug: (msg, ...args) => console.debug(format('debug', msg), ...args),
    info:  (msg, ...args) => console.info(format('info', msg),  ...args),
    warn:  (msg, ...args) => console.warn(format('warn', msg),  ...args),
    error: (msg, ...args) => console.error(format('error', msg), ...args),
  };
}

export { createLogger };
export type { Logger };
```

### 业务模块中的使用方式
```python
# Python 业务模块
from src.logger import get_logger

logger = get_logger(__name__)

def fetch_user(user_id: int):
    logger.info(f"Fetching user: user_id={user_id}")
    # ... 业务逻辑
```

---

## 六、可扩展性设计原则

### 开闭原则（对扩展开放，对修改关闭）
- 使用**策略模式**、**工厂模式**替代大量 `if-elif` 分支
- 新增功能通过添加新类/模块实现，而非修改已有逻辑

```python
# ❌ 违反开闭原则：每次新增支付方式都要修改此函数
def process_payment(method: str, amount: float):
    if method == "alipay":
        alipay_pay(amount)
    elif method == "wechat":
        wechat_pay(amount)

# ✅ 符合开闭原则：通过注册机制扩展
class PaymentProcessor:
    _handlers: dict = {}

    @classmethod
    def register(cls, method: str):
        """注册新支付方式的装饰器"""
        def decorator(func):
            cls._handlers[method] = func
            return func
        return decorator

    @classmethod
    def process(cls, method: str, amount: float):
        handler = cls._handlers.get(method)
        if not handler:
            raise ValueError(f"Unsupported payment method: {method}")
        return handler(amount)
```

---

## 七、可复用性设计原则

- **工具函数**放入 `utils/`，保持纯函数、无状态
- **服务层**封装外部依赖（HTTP、DB），便于 mock 和替换
- **配置与逻辑分离**：硬编码值统一提取到 `config/` 或常量文件
- **类型明确**：所有公共接口必须有类型注解（Python type hints / TypeScript types）

---

## 八、代码生成检查清单

生成代码前，逐项确认：

- [ ] 目录结构是否清晰，职责是否分层
- [ ] 函数命名是否表意，是否以动词开头
- [ ] 函数是否单一职责，长度是否合理
- [ ] 日志是否通过独立 `logger` 模块输出
- [ ] 关键业务逻辑是否有注释（"为什么"）
- [ ] 公共函数/类是否有 docstring / JSDoc
- [ ] 是否存在重复代码（应提取为公共函数）
- [ ] 配置值是否已提取为常量或配置文件
- [ ] 类型注解是否完整
- [ ] 是否只实现了用户要求的功能（无多余代码）

---

## 九、执行规则

1. **严格按需实现**：只完成用户明确要求的功能，不随意添加未被要求的逻辑、工具函数或配置项
2. **日志必须独立**：只要代码中涉及日志打印，日志模块必须写在单独文件（`logger.py` / `logger.ts` 等），业务代码通过导入使用
3. **注释聚焦关键**：只对非显而易见的逻辑添加注释，避免注释噪音
4. **结构先于实现**：生成代码时，先输出目录结构说明，再逐文件实现
5. **完整可运行**：生成的代码必须是完整的、可直接运行的，不留 `# TODO: implement` 占位
