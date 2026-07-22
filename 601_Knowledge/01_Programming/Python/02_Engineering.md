Python复习顺序

```
第二阶段：工程能力

★★★★★
面向对象
异常处理
模块包
虚拟环境
logging
上下文管理器

★★★★☆
生成器
装饰器

★★★☆☆
迭代器

```
# 1.Core Concepts
## 1.1 Class ⭐⭐⭐⭐⭐

面向对象是一种积木式的编程思想：通过一个模板（类）实例化出无数对象。类像一张图纸，对象是根据图纸制造出来的实体。数据工程中随处是类的影子——一张数据库表、一个 Airflow DAG、一个 ORM Model，本质上都是类。

### 1.1.1 三大核心概念

| 概念     | 含义                     | 数据工程中的例子                                                  |
| ------ | ---------------------- | --------------------------------------------------------- |
| **封装** | 数据 + 操作打包在一起，对外只暴露必要接口 | ORM Model 封装表的 CRUD，调用者不需要知道 SQL                          |
| **继承** | 子类复用父类的属性和方法，可以覆写或扩展   | 自定义 Exception 继承 `Exception`；多个 DAG 继承同一个 BaseDAG         |
| **多态** | 不同类实现同一个接口，表现出不同行为     | `read_csv()` / `read_parquet()` 返回的都是 DataFrame，但底层解析逻辑不同 |
### 1.1.2 类属性 vs 实例属性

```python
class User:
    db_type = "MySQL"           # 类属性：所有实例共享

    def __init__(self, name):
        self.name = name        # 实例属性：每个对象独有
```

`db_type` 在所有 `User` 对象上一致，改一处全局生效。`name` 每个对象各不相同。

### 1.1.3 方法分类

| 类型   | 声明方式               | 第一个参数        | 用途                  |
| ---- | ------------------ | ------------ | ------------------- |
| 实例方法 | `def method(self)` | `self`（实例自身） | 操作实例数据              |
| 类方法  | `@classmethod`     | `cls`（类自身）   | 工厂方法、替代构造器          |
| 静态方法 | `@staticmethod`    | 无            | 逻辑上归属类但不依赖实例/类的工具函数 |

### 1.1.4 访问控制

Python 没有真正的 private，靠命名约定：

```python
class Config:
    api_key = "xxx"         # 公有
    _retries = 3            # 约定私有（内部使用，外部慎碰）
    __secret = "sk-xxx"     # 名称改写为 _Config__secret（name mangling）
```

单下划线 `_` 是给使用者的信号："这是我的内部实现，不保证兼容"。
双下划线 `__` 触发名称改写，用于避免子类无意覆盖父类的同名属性。

### 1.1.5 魔法方法

Python 通过双下划线方法（dunder methods）让自定义类融入语言内置行为：

| 方法 | 触发时机 | 用途 |
|------|---------|------|
| `__init__` | `obj = Class()` | 初始化对象 |
| `__str__` | `print(obj)` / `str(obj)` | 给人看的描述 |
| `__repr__` | 交互式环境直接输入 `obj` | 给开发者看的，最好能 `eval` 还原 |
| `__len__` | `len(obj)` | 让对象支持 `len()` |
| `__getitem__` | `obj[key]` | 让对象支持索引/切片 |
| `__enter__` / `__exit__` | `with obj:` | 上下文管理器 |

### 1.1.6 数据工程中的典型落地

- **ORM 数据模型**：FastAPI + SQLAlchemy 中，一张表就是一个类，列名就是属性，`Base.metadata.create_all()` 一键建表。
- **Airflow DAG**：`dag = DAG(dag_id="etl_pipeline", ...)` 就是一个对象，每个 `task` 是它挂载的子对象。
- **配置类**：将分散的数据库连接串、API endpoint 收敛到一个 Config 类中，一处修改全局生效。
- **自定义异常**：`class DataQualityError(Exception): pass`，让错误分类更清晰。

### 1.1.7 作用域和命名空间示例
```python
def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)
```

```output
After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam
```

**1. 调用 `scope_test()`**
- Python 在 `scope_test` 的作用域内创建了一个局部变量 `spam`，赋值为 `"test spam"`。

**2. 执行 `do_local()`**
- 调用这个函数，它里面的 `spam = "local spam"` 会在自己的局部作用域里**新建**一个变量，并赋值为 `"local spam"`。
- 函数执行完毕后，这个局部变量被销毁。
- **对外层 `scope_test` 的 `spam` 没有任何影响。**
- ➡️ 所以第一次打印 `After local assignment:` 结果依然是 **`test spam`**。

**3. 执行 `do_nonlocal()`**
- 这里声明了 `nonlocal spam`，意思是："我不要创建新的，我要去**外层最近的那个函数**（即 `scope_test`）里找 `spam`"。
- 它找到了 `scope_test` 中的 `spam`（值为 `"test spam"`），然后把它修改成了 `"nonlocal spam"`。
- ➡️ 所以第二次打印 `After nonlocal assignment:` 结果变成了 **`nonlocal spam`**。

**4. 执行 `do_global()`**
- 这里声明了 `global spam`，意思是："我不要在函数里面找，我要去**模块全局**（文件最顶层）找 `spam`"。
- 但注意！此时**全局作用域里还没有定义过 `spam`**（全局是空的）。
- 既然找不到，`global` 允许直接**在全局作用域里新建一个变量** `spam`，赋值为 `"global spam"`。
- **关键点**：它修改的是"全局的 `spam`"，丝毫不动 `scope_test` 里的 `spam`。
- ➡️ 所以第三次打印 `After global assignment:`，`scope_test` 里的值依然没变，结果依然是 **`nonlocal spam`**。

**5. `scope_test()` 执行完毕，回到全局作用域**
- 最后执行 `print("In global scope:", spam)`。
- 此时全局作用域里，因为刚才 `do_global()` 的执行，已经存在了一个全局变量 `spam`，值为 `"global spam"`。
- ➡️ 所以最后的输出是 **`global spam`**。

### 1.1.8 self

`self` 不是关键字，是**命名约定**。方法的第一个参数代表实例自身：

```python
class User:
    def greet(self):          # self 就是调用这个方法的实例
        print(f"Hello, {self.name}")

u = User()
u.greet()                     # 等价于 User.greet(u)
```

Python 在调用 `u.greet()` 时自动把 `u` 传给 `self`。你可以改成 `this`、`me`，但 PEP 8 强制用 `self`。


### 1.2.9 装饰器（Decorator）

装饰器本质上是一个**接受函数、返回新函数**的高阶函数，用于在不修改原函数的前提下增加额外行为。

```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 耗时: {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def process_data():
    ...  # 自动计时的数据处理函数
```

核心理解：
- `@decorator` 等价于 `process_data = timer(process_data)`
- 装饰器在**定义时**执行，不是在调用时
- 数据工程中常见用途：日志、计时、重试、缓存、权限检查

### 1.1.10 `@property` 装饰器

将方法伪装成属性，实现 getter/setter，不改变外部调用方式：

```python
class Rect:
    def __init__(self, w, h):
        self._w = w
        self._h = h

    @property
    def area(self):           # 像属性一样访问 rect.area
        return self._w * self._h

    @property
    def w(self):
        return self._w

    @w.setter                 # rect.w = 10 触发校验
    def w(self, val):
        if val <= 0:
            raise ValueError("宽度必须为正")
        self._w = val
```

**场景**：升级代码时，把原来的属性改为计算属性，调用方零改动。

### 1.1.11 dataclass

Python 3.7+ 引入，一行装饰器自动生成 `__init__`、`__repr__`、`__eq__`：

```python
from dataclasses import dataclass, field

@dataclass
class Order:
    user_id: int
    amount: float
    status: str = "pending"                                  # 默认值
    tags: list = field(default_factory=list)                 # 可变默认值必须用 field

o = Order(user_id=1, amount=99.9)
print(o)                      # Order(user_id=1, amount=99.9, status='pending', tags=[])
```

**常用参数**：

| 参数 | 效果 |
|------|------|
| `frozen=True` | 不可变实例（类似 namedtuple） |
| `order=True` | 自动生成 `__lt__` 等比较方法 |
| `slots=True`（3.10+） | 启用 `__slots__` |

**数据工程场景**：配置对象、DTO（数据传输对象）、记录行映射。比 namedtuple 灵活，比手写 class 少十倍代码。


### 1.1.12 `super()` 与 MRO

`super()` 调用父类方法，遵循 **MRO**（Method Resolution Order）：

```python
class A:
    def greet(self):
        print("A")

class B(A):
    def greet(self):
        super().greet()       # 调用 A.greet
        print("B")

class C(A):
    def greet(self):
        super().greet()
        print("C")

class D(B, C):                # 菱形继承
    pass

D().greet()

print(D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)

C3 线性化算法（MRO） 强制要求：子类永远排在父类之前。

C3 算法必须同时满足的 3 条硬性约束：
1. 局部优先顺序：在 `class D(B, C)` 中，你写了 `B` 在前，`C` 在后，所以 `B` 必须排在 `C` 前面。
2. 单调性（子类优先）：`C` 继承自 `A`，所以 `C` 必须排在 `A` 前面。
3. 继承传递：`B` 也继承自 `A`，所以 `B` 也必须排在 `A` 前面。

# A
# C
# B
```

Python 3 的 MRO 是 **C3 线性化**（`D.__mro__` 可查看）。菱形继承中，`super()` 不会重复调用同一个父类两次。

**数据工程中少见多继承**，但阅读源码（如 SQLAlchemy、Django REST Framework）时会遇到。重点理解 MRO 而不是自己写。

### 1.1.13 `__slots__` 内存优化

默认每个实例有一个 `__dict__` 字典存储属性。对大量小对象（百万级），这是显著内存开销：

```python
class Point:
    __slots__ = ('x', 'y')    # 禁用 __dict__，只允许 x 和 y
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

| 对比 | 有 `__dict__` | 有 `__slots__` |
|------|--------------|---------------|
| 内存 | 大 | 小（~50% 减少） |
| 属性访问速度 | 慢（字典查找） | 快（直接偏移量） |
| 动态添加属性 | 可以 | 不行 |

代价：不能动态添加属性、不能多继承、不能 pickle 某些场景。

## 1.2 Iterator  ⭐⭐⭐☆ ☆

迭代器是实现了 `__iter__` 和 `__next__` 的对象，一次只产生一个元素，省内存：

```python
nums = iter([1, 2, 3])
next(nums)  # 1
next(nums)  # 2
next(nums)  # 3
next(nums)  # StopIteration
```

`for` 循环本质：`for x in obj` → 调用 `iter(obj)` 获取迭代器 → 反复 `next()` → 遇到 `StopIteration` 结束。

**为什么需要迭代器**：处理无法全部加载到内存的大数据集（GB 级日志文件、流式数据），惰性计算，按需取用。

**自定义迭代器**：

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self                 # 迭代器自身也是可迭代对象

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

for n in Countdown(3):
    print(n)   # 3, 2, 1
```

**判断是否可迭代**：`hasattr(obj, '__iter__')` 或用 `from collections.abc import Iterable`。
**常见坑**：迭代器只能遍历一次，用完即空。需要重复遍历时用 `list` 存下来或重新创建。

## 1.3 Generator ⭐⭐⭐⭐ ☆

生成器是**生成迭代器的函数**，用 `yield` 代替 `return`，自动实现 `__iter__` 和 `__next__`：

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for n in countdown(3):
    print(n)   # 3, 2, 1
```

应用案例：
大文件处理
```Python
def read_file(path):
    with open(path) as f:
        for line in f:
            yield line

```


### 1.3.1 `yield` vs `return`

|      | `return`    | `yield`       |
| ---- | ----------- | ------------- |
| 返回后  | 函数结束，局部变量释放 | 函数**暂停**，状态保留 |
| 再次调用 | 重新执行        | 从上次暂停处继续      |
| 返回值  | 一次性返回全部     | 逐个产生          |

### 1.3.2 生成器表达式

列表推导式的惰性版本，用 `()` 代替 `[]`：

```python
squares = (x**2 for x in range(10**9))  # 不占内存
next(squares)  # 0
```

对比：

```python
sum([x**2 for x in range(10**7)])   # 先创建 1000 万的 list，然后求和 → 内存爆炸
sum(x**2 for x in range(10**7))     # 逐个产生，逐个累加 → 内存恒定
```

### 1.3.3 `yield from`

将子生成器的产出委托给父生成器，避免手写 `for` 循环中转：

```python
def flatten(nested):
    for sublist in nested:
        yield from sublist           # 等价于 for x in sublist: yield x

list(flatten([[1, 2], [3, 4]]))    # [1, 2, 3, 4]
```

### 1.3.4 `send()` 和双向通信

生成器不仅产出值，还能接收外部传入的值：

```python
def accumulator():
    total = 0
    while True:
        value = yield total          # 产出 total，同时接收外部 value
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)        # 0（启动生成器，执行到 yield）
acc.send(10)     # 10（传入 10，累加后 yield）
acc.send(20)     # 30
```

### 1.3.5 数据工程中的典型场景

- **流式读取大文件**：`for line in open('big.log')` — 文件对象本身就是迭代器，逐行读取不爆内存
- **分页 API**：`yield from` 逐个产出分页结果，调用方只管遍历
- **数据管道**：`read → filter → transform → write` 每步都是生成器，一条数据流过全链路

## 1.4 Errors and Exceptions ⭐⭐⭐⭐⭐

异常机制的三个核心作用：

1. **分离正常逻辑与错误处理**：`try` 走主流程，`except` 兜底异常，代码结构清晰，不混在一起。
2. **阻断错误传播**：未经处理的异常会沿调用栈一路向上抛，直到被捕获或导致程序崩溃。适时捕获可以控制错误范围。
3. **优雅降级，而非直接崩溃**：非致命的步骤（如日志上报、缓存更新）报错不应拖垮整个流程，放到 `except` 里记录后继续执行。

### 基本结构

```python
try:
    result = risky_operation()
except ValueError as e:
    logger.warning(f"数据格式异常: {e}")
    result = default_value       # 降级策略
except (KeyError, IndexError):   # 一次捕获多个类型
    raise                        # 不处理，继续往上抛
else:
    process(result)              # 仅在 try 未抛异常时执行
finally:
    cleanup()                    # 无论是否异常都会执行
```

`else` vs 把代码放 `try` 里：`else` 里的代码不会被 `except` 捕获，语义更精确——"这部分只在成功时运行，出错归出错"。

### 自定义异常

```python
class DataValidationError(Exception):
    """数据校验失败"""
    pass

if not is_valid(row):
    raise DataValidationError(f"row={row}")
```

业务代码中继承 `Exception`（不要继承 `BaseException`），异常类名清晰即可，不需要复杂逻辑。

### `finally` 的三个作用

| 场景     | 例子             |
| ------ | -------------- |
| 释放外部资源 | 关闭文件、网络连接，防止泄漏 |
| 恢复全局状态 | 重置被修改的标志位或配置   |
| 清理临时产物 | 删除临时文件或目录      |

`with` 语句等价于 `try...finally`，推荐优先使用。

### 异常链（Python 3.11+ ExceptionGroup）

```python
raise RuntimeError("服务超时") from original_error  # 保留原始异常上下文
```

### try/except 的开销

- `try` 块本身**几乎无开销**（不触发异常时只是一个 SETUP_FINALLY 字节码）
- 真正的开销在**异常发生时**——栈回溯构造和上下文信息采集
- 工程上：不用为了省性能而不用 `try`，但不要用异常控制正常流程（如循环中用异常判断退出）

### 关键原则

1. **只捕获你可以处理的异常**。裸 `except:` 或 `except Exception` 会吞掉 `KeyboardInterrupt`、`SystemExit`，程序可能无法正常终止。
2. **`except` 块尽量短**，只做恢复和记录，不要在里面写复杂业务逻辑。
3. **不吃异常**。即使捕获了，至少打日志 `logger.exception()`，否则线上排查无迹可寻。

## 1.5 常用库

### Logging

Python 标准库的日志模块，替代 `print()` 实现结构化、可分级、可定向输出的日志记录。

#### 五个级别（由低到高）

| 级别 | 数值 | 用途 |
|---|---|---|
| `DEBUG` | 10 | 开发调试，详细的变量状态 |
| `INFO` | 20 | 正常运行时关键节点（流程开始/结束、配置加载） |
| `WARNING` | 30 | 非预期但程序可继续运行（**默认级别**） |
| `ERROR` | 40 | 功能出错，需关注但程序未崩溃 |
| `CRITICAL` | 50 | 致命错误，程序可能无法继续 |

默认输出级别为 `WARNING`，即 `DEBUG`、`INFO` 默认不输出。

#### 基础配置

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[
        logging.FileHandler("app.log"),
        logging.StreamHandler(),          # 同时输出到控制台
    ],
)

logging.info("Pipeline started")
# 输出: 2026-07-21 14:30:05,123 [INFO] root: Pipeline started
```

**常见坑**：`basicConfig()` 只在首次调用时生效，后续调用忽略。需在程序入口处一次性配置。

#### 最佳实践：模块级 Logger

不要用 `logging.info()` 直接调根 logger。每个模块创建自己的 logger，继承根配置：

```python
logger = logging.getLogger(__name__)   # 以模块路径命名

logger.info("Task started")
logger.error("Connection failed", exc_info=True)   # 附带 traceback
```

`__name__` 作为 logger 名，自动按包层级继承配置。上层包设 `WARNING`，子模块设 `DEBUG` 可按需控制日志粒度。

#### `exception()` vs `error()`

```python
try:
    result = 1 / 0
except ZeroDivisionError:
    logger.error("Calculation failed")      # 只有消息，无堆栈
    logger.exception("Calculation failed")  # 消息 + 完整 traceback（级别=ERROR）
```

线上排障时 `exception()` 是必需品——没有堆栈的 `"Something went wrong"` 几乎无法定位。

#### 常用格式化字段

| 字段 | 含义 |
|---|---|
| `%(asctime)s` | 时间戳 |
| `%(name)s` | Logger 名称（通常=模块路径） |
| `%(levelname)s` | 日志级别 |
| `%(message)s` | 日志消息 |
| `%(filename)s` | 源文件名 |
| `%(lineno)d` | 行号 |

#### 实战模板

```python
import logging

def setup_logging():
    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s [%(levelname)s] %(name)s:%(lineno)d - %(message)s",
        handlers=[
            logging.FileHandler("pipeline.log", encoding="utf-8"),
            logging.StreamHandler(),
        ],
    )

setup_logging()
logger = logging.getLogger(__name__)

def run_pipeline():
    logger.info("Pipeline started")
    try:
        ...  # 主逻辑
    except Exception:
        logger.exception("Pipeline failed")  # 自动包含 traceback
    else:
        logger.info("Pipeline completed")
```

### venv

Python 内置的虚拟环境工具（3.3+），为每个项目创建独立的解释器和包空间，避免依赖冲突。

#### 创建与激活

```bash
# 创建
python -m venv myenv

# 激活
# Windows:
myenv\Scripts\activate
# macOS/Linux:
source myenv/bin/activate

# 退出
deactivate
```

激活后，`pip install` 的所有包仅安装在该环境中，不影响系统 Python。

**常见坑**：不要在项目根目录把虚拟环境命名为 `venv` 后又 `pip install venv`——`venv` 是标准库模块名，不会冲突，但文件夹和库同名会误导。

#### 冻结与还原依赖

```bash
pip freeze > requirements.txt      # 导出当前环境的包列表
pip install -r requirements.txt    # 在新环境一键恢复所有依赖
```

#### `pip freeze` vs `pip list`

- `pip freeze`：输出 `package==version` 格式，供 `requirements.txt` 用。
- `pip list`：人类可读的表格格式。

**常见坑**：`pip freeze` 导出的是当前环境**全部**已装包（含间接依赖），多人协作时建议手写 `requirements.txt` 只列出直接依赖，版本号用 `>=` 而非 `==`。

#### 目录结构惯例

```text
project/
    .venv/              # 虚拟环境（加入 .gitignore）
    requirements.txt    # 依赖声明
    src/                # 源代码
```

虚拟环境目录本身不应提交到 Git——`requirements.txt` 已足够重建。

#### `pip` 常用操作

```bash
pip install pandas==2.0.3      # 指定版本
pip install --upgrade pandas   # 升级
pip uninstall pandas           # 卸载
pip show pandas                # 查看包信息（版本、路径、依赖）
pip list --outdated            # 查看哪些包有新版本
```

#### 常见坑：全局 vs 虚拟环境

激活虚拟环境后，所有 `python` 和 `pip` 命令指向虚拟环境。但如果不小心用了 `python3`，可能调到了系统全局解释器，导致 `ModuleNotFoundError`。激活后用 `which python`（Linux/macOS）或 `where python`（Windows）确认路径。
# 2.Interview

## 1. Class / OOP

### Q1. Python 为什么需要 Class？什么时候使用面向对象？

当数据和操作需要封装在一起时。
数据工程中：Airflow Operator 封装 DAG 任务、FastAPI Model 定义数据表、LangChain Component 封装 LLM 调用链。
函数处理"做什么"，Class 处理"谁来做" —— 有状态、有生命周期时用类。

---

### Q2. 实例属性和类属性有什么区别？

实例属性每个对象独有（self.name）
类属性所有实例共享（db_type）
改类属性全局生效，改实例属性只影响这一个对象。

---

### Q3. 实例方法、类方法、静态方法有什么区别？

实例方法操作实例数据（self）；
类方法操作用类级别数据（cls），常用于工厂方法；
静态方法不绑定实例或类，只是逻辑上归类的工具函数。
数据工程中 90% 用实例方法。

---

### Q4. self 的作用是什么？

代表当前实例对象的引用。
Python 自动将调用者传给 self，使方法能访问该对象的属性。
不是关键字——是 PEP 8 强制约定。

---

### Q5. 装饰器是什么？为什么工程框架大量使用？

接受函数、返回增强版本的高阶函数。
不修改原代码即可加日志、计时、权限校验。
FastAPI 的 @app.get()、Airflow 的 @task 都是装饰器。
基于 AOP 思想，业务逻辑和横切关注点分离。

---

## 2. Iterator / Generator

### Q6. Iterable(**可迭代对象**) 和 Iterator(**迭代器**) 有什么区别？

Iterable 实现了 __iter__，可以产生迭代器；
Iterator 额外实现了 __next__，逐次返回元素。
list 是 Iterable 但不是 Iterator，for 循环调用 iter() 将其转为 Iterator。

---

### Q7. for 循环底层执行过程是什么？

for x in obj 等价于：获取 iter(obj) 迭代器 -> 反复调用 next() -> 遇到 StopIteration 退出。理解了这两步就理解了迭代器协议的全部。

---

### Q8. 什么是 Generator？为什么数据工程中经常使用？

用 yield 生成迭代器的函数，惰性计算、不一次性占内存。
数据工程中：流式读取 GB 级日志、分页 API 数据拉取、Pipeline 中各阶段用生成器串联——内存恒定，吞吐受限于 I/O 而非 RAM。

---

### Q9. yield 和 return 有什么区别？

return 结束函数并释放局部变量
yield 暂停函数、保留状态，下次调用从暂停处继续
Generator 的本质：一个可以在中途暂停并恢复执行的函数。

---

## 3. Exception

### Q10. Python 异常处理机制是什么？

try/except/finally/else 四件套。
try 走主流程，except 兜底异常，finally 无论是否异常都执行，else 仅在 try 成功时执行。
数据工程中用于捕获 API 超时、DB 连接失败、文件不存在。

---

### Q11. 为什么生产代码需要异常处理？

ETL 任务跑了 6 小时，第 5 个小时某一行数据格式错误直接崩溃——没有异常处理就是线上事故。
try/except + logging + 降级策略 = 任务不中断，脏数据留下记录继续跑。
Airflow DAG 中一子任务失败不该拖垮整个 Pipeline。

---

### Q12. 为什么不能直接捕获 Exception？

裸 except Exception 会吞掉所有意外错误（包括自己 bug 导致的），线上排查无从下手。
只捕获你能处理的异常类型，且至少打日志 logging.exception()。不吃的异常让它往上抛。

---

## 4. Engineering

### Q13. 为什么需要虚拟环境？

隔离项目依赖——A 项目用 pandas 1.x，B 项目用 pandas 2.x，没有 venv 会互相污染。
数据工程每个项目独立 venv 是基本素养。venv + requirements.txt 或更现代的 pyproject.toml。

---

### Q14. 为什么生产环境不用 print，而使用 logging？

print 只能输出到 stdout，无法分级（DEBUG/INFO/WARNING/ERROR）、无法写文件、无法按时间轮转。
logging 支持这些 + 多进程安全 + 可配置格式。生产排查时按级别过滤日志是基本功。

---

### Q15. Python 项目如何组织代码结构？

最小结构：main.py + utils.py + config.py + tests/。
数据工程项目：dag/（Airflow DAG）、etl/（transform/extract/load）、config/（db/API 连接）、tests/、requirements.txt。
核心原则：入口简洁、逻辑分层、配置分离。

---
