https://docs.python.org/zh-cn/3.13/tutorial/index.html

```markdown
## 一句话介绍

Python是一门动态，强类型，解释型(非编译型)通用型语言。

## 为什么学习

AI Data Engineering主要开发语言。

应用：

- 数据处理
- ETL
- 自动化
- AI应用

## 我的总结

Python最大的价值：生态丰富，开发效率高。
```

Python复习顺序

```
第一阶段：必须掌握
基础语法

★★★★★
变量
数据类型
list/dict/set/tuple
条件
循环
函数
异常处理
模块与包

```

---
# 1.Core Concepts

## 1.1 Control Flow ⭐⭐⭐⭐⭐

### `if`

最常用的控制流语句。通过对业务的不同情况进行判断，走向不同的操作或结果。
多种情况使用 `elif` 和 `else`。但不要轻易使用 `else`：如果事前未穷举所有情况，`else` 容易漏掉边界场景。

---
### `for`

`for` 本质是可迭代对象的迭代器循环，不同于 Java/C 的计数循环。遍历 `list`、`range()` 等可迭代对象时，配合 `if` 可对元素做条件筛选和个性化操作。

性能注意事项：
- 纯数值计算的大数据量 → 优先用 NumPy 向量化操作。
- 普通数据转换 → 优先用列表推导式（`[x*2 for x in data]`），底层走 C 循环，比普通 `for` 快不少。

`for` 可通过 `break`、`continue` 控制循环流程，需设置合理的退出条件，避免死循环。

### `for...else`

`else` 在循环未被 `break` 中断时执行。语义是”循环正常完成，未命中 `break`”。不要把 `else` 代码块的内容放到 `break` 之前——两者走的是互斥分支。

### `break` / `continue` / `pass`

- `break`：跳出最近一层循环。
- `continue`：跳过本次迭代，继续下一次循环。
- `pass`：占位符，不执行任何操作。

### `while`

条件为 `True` 时循环执行。适用场景：不知道循环次数，只知道终止条件。

与 `for` 的核心区别：`for` 遍历已知集合，`while` 等待条件达成。数据工程中常用于轮询（如等待文件落盘）、重试逻辑。

`while` 同样支持 `break`、`continue` 和 `else`（与 `for...else` 语义一致）。

---

## 1.2 Function ⭐⭐⭐⭐⭐

### 1.2.1 定义函数

函数是操作的集合，有输入、有输出、有内部逻辑，类比数据管道。部分函数可无参数或无返回值。函数体内可结合 `if`、`for`、`while` 实现任意逻辑。

### 1.2.2 参数

**默认值参数**：为参数提供默认值，调用时可省略该参数，简化调用。但注意：默认值在函数定义时只计算一次，因此应使用不可变类型（`None`、`str`、`int` 等），避免使用 `list`、`dict` 等可变对象——若函数内部修改了该默认值，后续调用会复用被污染的状态。

**关键字参数**：以 `key=value` 形式传参，不依赖参数位置，语义明确，是最不易出错的传参方式。

**位置参数**：按函数定义时的参数顺序依次传入，省略参数名，简洁但需保证顺序正确。

**参数限定符**（Python 3.8+）：`def f(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2)`
- `/` 左侧：必须使用位置传参。
- `/` 与 `*` 之间：位置或关键字均可。
- `*` 右侧：必须使用关键字传参。

### 1.2.3 解包（Unpacking）

解包发生在**函数调用**时。当数据存放在容器中，而函数需要多个独立参数时，使用 `*` 或 `**` 拆开容器。

- `*` 用于可迭代对象（`list`、`tuple` 等）：按位置顺序逐个传入。
- `**` 用于映射对象（`dict` 等）：键值对按参数名（Key）匹配后，以关键字参数形式传入。

> 注：`args = [3, 6]` 只是普通赋值。真正的"打包（Packing）"指函数定义时的 `*args` 和 `**kwargs`。

`*` 可多次使用且位置不限（Python 3.5+），`**` 只能放在最后：

```python
list1 = [1, 2]
list2 = [3]
add(*list1, *list2)  # 等价于 add(1, 2, 3)
```

### 1.2.4 Lambda 表达式

匿名函数，适用于简单逻辑，一行替代多行 `def`。函数体只能是单个表达式（Expression），不能包含语句（Statement）。本质是语法糖，无性能优化。

### 1.2.5 文档字符串（Docstring）

函数的说明注释，用于描述用途和功能，提高可维护性与可读性。

### 1.2.6 `*args` 与 `**kwargs`（打包）

函数**定义**时使用，与调用时的解包方向相反：将多个参数"打包"进一个容器。

- `*args`：接收任意数量的位置参数，打包为元组。
- `**kwargs`：接收任意数量的关键字参数，打包为字典。

```python
def log_event(event_type, *tags, **metadata):
    print(f"[{event_type}]", tags, metadata)

log_event("error", "db", "timeout", user="alfie", retry=3)
# 输出: [error] ('db', 'timeout') {'user': 'alfie', 'retry': 3}
```

命名习惯是 `*args`/`**kwargs`，但 `*tags`/`**meta` 亦可，语法取决于 `*` 和 `**`。

### 1.2.7 `return`

函数通过 `return` 将结果返回给调用者。`return` 后函数立即退出，后续代码不执行。

无 `return` 或 `return` 无值的函数，默认返回 `None`。

Python 可返回多个值，本质是打包成元组：

```python
def stats(data):
    return min(data), max(data), sum(data) / len(data)
# 调用方解包: lo, hi, avg = stats(nums)
```

### 1.2.8 变量作用域（LEGB 规则）

Python 查找变量名时，按 **L → E → G → B** 顺序：

| 层级            | 含义     | 示例                    |
| ------------- | ------ | --------------------- |
| **L**ocal     | 函数内部   | 函数内定义的变量              |
| **E**nclosing | 外层嵌套函数 | 闭包中外层函数的变量            |
| **G**lobal    | 模块全局   | 文件顶层定义的变量             |
| **B**uilt-in  | 内置     | `print`、`len`、`range` |

**常见坑**：在函数内尝试直接修改全局变量或外层变量，Python 会将其当作新的局部变量，导致 `UnboundLocalError`。需用 `global` 或 `nonlocal` 声明。

```python
count = 0

def increment():
    count += 1  # ❌ UnboundLocalError: count 被视为局部变量

def increment_fixed():
    global count
    count += 1  # ✅
```

数据工程中应避免大量使用 `global`——传参和返回值更清晰。但阅读他人代码时需要能理解。

## 1.3 Data Structure ⭐⭐⭐⭐⭐

核心先行：数据结构的选择取决于你要解决的问题——增删改查哪个更频繁、是否需要保持顺序、键值对是否必须唯一。

### 1.3.1 List

可变、有序、异质，底层为动态数组（Dynamic Array）。

**常用操作**：

| 操作    | 方法                 | 说明                     |
| ----- | ------------------ | ---------------------- |
| 末尾添加  | `append(x)`        | O(1) 均摊                |
| 扩展    | `extend(iterable)` | 批量添加可迭代对象              |
| 插入    | `insert(i, x)`     | 指定位置插入，O(n)            |
| 按值删除  | `remove(x)`        | 删除第一个匹配值，O(n)          |
| 按位置删除 | `pop(i)`           | 删除并返回该元素；默认末尾          |
| 清空    | `clear()`          | 移除所有元素                 |
| 查找索引  | `index(x)`         | 返回第一个匹配位置              |
| 计数    | `count(x)`         | 统计出现次数                 |
| 排序    | `sort()`           | 原地排序，支持 `reverse=True` |
| 翻转    | `reverse()`        | 原地翻转                   |
| 浅拷贝   | `copy()`           | 等价于 `lst[:]`           |

**del / pop / remove 区别**：

- `del lst[i]`：语句，直接操作字节码 `DELETE_SUBSCR`，不返回被删值。最快但不灵活。
- `pop(i)`：方法，删除并返回被删值。多一次属性查找和返回值构造，开销微不足道。
- `remove(x)`：方法，先 O(n) 查找第一个匹配值再删除。最慢。

**列表推导式**：比 `for` 快（底层走 C 循环），可读性更强。

```python
[x*2 for x in data]                    # 对每个元素应用操作
[x for x in data if x > 0]             # 筛选符合条件的元素
```

**切片（Slicing）**：`lst[start:stop:step]`，含头不含尾，支持负数索引。

```python
lst[-1]    # 最后一个元素
lst[::-1]  # 翻转
lst[1:5]   # 索引 1~4（不含5）
```

**常见坑**：`*` 复制嵌套列表时，内部元素是浅拷贝（共享引用）。

```python
matrix = [[0] * 3] * 3  # ❌ 三行指向同一个子列表
matrix = [[0] * 3 for _ in range(3)]  # ✅ 每行独立
```

### 1.3.2 Tuple

有序、不可变、异质。单元素元组必须加逗号：`(42,)`。

**为什么用 Tuple？**
- 不可变 → 可哈希 → 可作为 `dict` 的键、`set` 的元素
- 作为函数的多值返回载体
- 相较于 List 更轻量（内存和创建速度略优）

**解包**：`a, b, c = (1, 2, 3)`，数量必须匹配。

### 1.3.3 Set

可变、无序、元素唯一（自动去重），基于哈希表。支持推导式。

创建空集合只能用 `set()`——`{}` 创建的是空字典。

**集合运算（数据工程高频操作）**：

```python
a & b   # 交集
a | b   # 并集
a - b   # 差集（a 有 b 无）
a ^ b   # 对称差集（仅在一边出现的元素）
```

典型场景：两个 ID 列表求共同用户（交集）、已处理去重（差集）。

### 1.3.4 Dict

可变，键值对，键必须可哈希（不可变类型），键唯一。基于哈希表，O(1) 查找。

**核心操作**：

```python
d.get(key, default)      # 安全取值，不存在返回默认值（避免 KeyError）
d.setdefault(key, val)   # 键不存在则设置并返回 val，存在则返回原值
d.items()                # 遍历键值对
d.keys() / d.values()    # 键/值视图
list(d)                  # 返回所有键的列表
```

**字典推导式**：

```python
{x: x**2 for x in (2, 4, 6)}  # {2: 4, 4: 16, 6: 36}
```

**合并字典**（Python 3.9+）：`d1 | d2`，重复键以后者为准。

**常见坑**：若键不存在则 `d[key]` 抛 `KeyError`，优先用 `get()`。

**`collections.Counter`**：统计频率的利器。

```python
from collections import Counter
c = Counter(['a', 'b', 'a', 'c', 'b', 'a'])
c.most_common(2)  # [('a', 3), ('b', 2)]
```

### 1.3.5 `zip` / `enumerate`

`zip(*iterables)` 将多个可迭代对象的元素按位置一一配对，返回迭代器。不等长时按最短截断。

```python
questions = ['name', 'quest', 'favorite color']
answers = ['Lancelot', 'the Holy Grail', 'blue']

for q, a in zip(questions, answers):
    print(f"What is your {q}? It is {a}.")
    
# What is your name? It is Lancelot.
# What is your quest? It is the Holy Grail.
# What is your favorite color? It is blue.
```

`enumerate(iterable, start=0)` 同时获取索引和值：

```python
data = ['a', 'b', 'c']
for i, val in enumerate(data, 1):  # 索引从1开始
    print(f"{i}: {val}")
    
# 1: a
# 2: b
# 3: c
```

## 1.4 Module ⭐⭐⭐⭐⭐

模块是包含 Python 定义和语句的 `.py` 文件。通过导入模块，可复用代码、组织命名空间。

### 1.4.1 导入方式

```python
import pandas                  # 导入整个模块，使用 pandas.DataFrame()
import pandas as pd            # 别名，数据工程惯例
from datetime import datetime  # 按需导入，直接用 datetime.now()
from os.path import join as pjoin  # 导入并重命名
```

数据工程中 `import pandas as pd`、`import numpy as np` 是事实标准。

### 1.4.2 `if __name__ == "__main__"`（必知）

**`__name__` 是 Python 自动给每个模块（`.py` 文件）起的一个“名字”**
当 `.py` 文件被直接执行（`python xxx.py`）时，`__name__` 为 `"__main__"`；被 `import` 时，`__name__` 为模块名。

```python
def main():
    ...

if __name__ == "__main__":
    main()
```

用途：让一个文件既可以作为脚本运行，也可以作为模块被导入而不执行主逻辑。所有脚本都应加这个守卫。

### 1.4.3 包与 `__init__.py`

包是包含 `__init__.py` 的目录。`__init__.py` 可为空文件，也可在其中定义包的公共接口（`__all__`）。

```text
my_package/
    __init__.py
    module_a.py
    module_b.py
```

`__init__.py` 在包首次导入时执行。Python 3.3+ 支持无 `__init__.py` 的命名空间包，但常规包仍建议保留。

### 1.4.4 相对导入 vs 绝对导入

包内部推荐**相对导入**：

```python
from . import module_a        # 同级模块
from .module_a import func    # 同级模块中的函数
from .. import sibling_pkg    # 上级包
```

绝对导入（`from my_package.module_a import func`）从 `sys.path` 根目录开始查找，清晰但依赖包名。

### 1.4.5 `sys.path` —— Python 在哪里找模块

`import` 时 Python 按以下顺序搜索：
1. 当前脚本所在目录
2. `PYTHONPATH` 环境变量
3. 标准库目录
4. `site-packages`（pip 安装的第三方包）

常见坑：脚本目录和当前工作目录不同时，相对路径的 `import` 失败。排障第一步：`import sys; print(sys.path)`。

### 1.4.6 循环导入（Circular Import）

A 导入 B，B 又导入 A → `ImportError`。

**修复方案**：
- 将 `import` 移到函数内部（延迟导入）
- 提取共同依赖到第三个模块
- 重构设计，打破循环

## 1.5 I/O & format ⭐⭐⭐⭐⭐

### 1.5.1 文件读写

`open(file, mode)` 打开文件，返回文件对象。**必须**用 `with` 语句包裹——自动关闭文件，即使发生异常也不会泄漏句柄。

```python
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()
```

| mode | 含义 | 注意 |
|---|---|---|
| `'r'` | 只读 | 文件不存在→`FileNotFoundError` |
| `'w'` | 写入（覆盖） | **清空已有内容**，文件不存在则创建 |
| `'a'` | 追加 | 在末尾添加，文件不存在则创建 |
| `'rb'` / `'wb'` | 二进制读写 | 图片、PDF、Parquet 等 |

**常见坑**：`'w'` 模式打开就清空文件，还没写就丢了原数据。不确定时先用 `'a'` 或用临时文件。

**逐行读取大文件**：不要用 `f.readlines()`（一次性加载到内存）。
用迭代器惰性读取：

```python
with open("big_file.txt") as f:
    for line in f:          # ✅ 逐行读取，内存友好
        process(line)
```

### 1.5.2 f-string（格式化字符串字面量）

Python 3.6+，最推荐的格式化方式。

```python
name = "Alfie"; score = 95.6789
f"{name}: {score:.2f}"        # 'Alfie: 95.68'  ——保留小数
f"{10000:,}"                   # '10,000'        ——千位分隔符
f"{name:<10}{score:>8.1f}"    # 'Alfie     95.7'  ——左右对齐+宽度
f"{0.15:.1%}"                  # '15.0%'         ——百分比
f"{255:#x}"                    # '0xff'          ——十六进制
f"{name=}"                     # 'name=Alfie'    ——Python 3.8+ 调试快捷方式
```

### 1.5.3 `str.format()`

f-string 出现**前**的主流方案，适合模板化复用或延迟格式化。现在使用较少。

```python
"{}: {:.2f}".format(name, score)       # 按位置
"{n}: {s:.2f}".format(n=name, s=score) # 按关键字
template = "Hello, {name}"
template.format(name="Alfie")          # 可复用模板
```

### 1.5.4 `print()` 进阶

```python
print(a, b, c, sep=", ")    # 自定义分隔符
print("Loading", end="...")  # 不换行（默认 end="\n"）
print("Error", file=sys.stderr)  # 输出到 stderr
```

### 1.5.5 JSON 读写

数据工程中最常见的数据交换格式。

```python
import json

# 字符串 ↔ Python 对象
data = json.loads('{"name": "Alfie", "age": 27}')
print(data)              # {'name': 'Alfie', 'age': 27}
print(data["name"])      # Alfie

text = json.dumps(data, ensure_ascii=False, indent=2)
print(text)
# {
#   "name": "Alfie",
#   "age": 27
# }

# 文件读写
with open("config.json") as f:
    config = json.load(f)          # 从文件读取
with open("output.json", "w") as f:
    json.dump(data, f, indent=2)   # 写入文件
```

**常见坑**：`json.loads`/`json.dumps`（处理字符串）vs `json.load`/`json.dump`（处理文件），差一个 `s`。

**JSON vs dict**：

|                  | JSON                                    | Python dict             |
| ---------------- | --------------------------------------- | ----------------------- |
| 本质               | 字符串（文本格式）                               | 内存中的数据结构                |
| 类型               | `str`                                   | `dict`                  |
| 键                | 必须双引号                                   | 任意可哈希类型                 |
| 值                | 仅支持：`str/number/object/array/bool/null` | 任意 Python 对象            |
| `True` / `False` | `true` / `false`（小写）                    | `True` / `False`（首字母大写） |
| `None`           | `null`                                  | `None`                  |
| 传输               | 可跨语言、跨网络                                | 仅在 Python 进程内           |

`json.dumps(dict)` → 序列化为字符串；`json.loads(str)` → 反序列化为 dict。

### 1.5.6 CSV 读写

```python
import csv

# 读
with open("data.csv") as f:
    reader = csv.DictReader(f)     # 每行作为 dict，列名为 key
    for row in reader:
        print(row["name"], row["age"])
# Alfie 27
# Tom 25

# 写
rows = [{"name": "Alfie", "age": 27}, {"name": "Tom", "age": 25}]
with open("out.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerows(rows)
# out.csv:
# name,age
# Alfie,27
# Tom,25
```

注意：Windows 下 `open` 加 `newline=""` 防止多余空行。
# 2.Interview

# 1. Control Flow

- Python 的 for 和其他语言有什么区别？
    - Python 的 for 是迭代器循环，遍历可迭代对象，不是 Java/C 的计数循环。数据处理中常用于逐行遍历、batch 迭代。
- break、continue、pass 有什么区别？
    - break 跳出最近循环；continue 跳过本次迭代；pass 是占位符。数据管道中 break 用于异常退出、continue 过滤脏数据。
- for...else 的执行机制是什么？
    - else 在循环未被 break 中断时执行。常用于遍历搜索后 fallback 到默认逻辑。

---

# 2. Function

- Python 参数有哪些？
    - 位置参数、默认值参数、*args（打包为元组）、**kwargs（打包为字典）、keyword-only（* 之后）、positional-only（/ 之前，3.8+）。
- Python 是值传递还是引用传递？
    - pass by assignment（按赋值传递） —— 变量是对象引用，赋值改变引用指向，不复制对象。可变对象传进函数可被修改。
    - Python的变量本质上是一个**指向对象的“标签”**。函数调用时，**把标签复制了一份**传给函数。**复制的是标签（引用地址）**，不是对象（非值传递）。**标签指向谁，由赋值号'='决定**，一旦重新赋值，标签就撕下来贴到新对象上，不影响外部的旧标签（非引用传递）。
- Lambda 表达式有什么作用？
    - 匿名函数，一行逻辑。pandas 的 apply、sorted 的 key、filter/map 中常用。不能多行、不能带语句。
- return 返回多个值原理是什么？
    - 本质是打包成元组。a, b = func() 是元组解包的语法糖。
- LEGB 变量查找规则是什么？
    - Local -> Enclosing -> Global -> Built-in。修改全局需 global，修改外层需 nonlocal。数据工程中应避免大量 global——传参更清晰。

---

# 3. Data Structure

- list 和 tuple 的区别？
    - list 可变，tuple 不可变。tuple 可哈希（可作 dict 键），适合函数多值返回。内存上 tuple 略省。
- list 和 set 的区别？
    - list 有序可重复，O(n) 查找；set 无序去重，O(1) 哈希查找。数据工程中 set 用于 ID 去重、交集差集运算。
- dict 底层为什么查询快？
    - 基于哈希表，键通过 hash 映射到槽位，O(1) 定位。键必须是可哈希类型。
- zip 和 enumerate 的作用？
    - zip 将多个可迭代对象按位置配对；enumerate 同时获取索引和值。常用于并行遍历多列、带序号处理记录。
- Python 常见可变对象和不可变对象有哪些？
    - 不可变：int、float、str、tuple、frozenset、bytes；可变：list、dict、set、bytearray。**函数默认参数必须用不可变类型。**

---

# 4. Module

- import 有哪些方式？
    - import module、import module as alias、from module import name、from module import *（不推荐）。import pandas as pd 是数据工程事实标准。
- `if __name__ == "__main__"` 的作用？
    - 脚本直接运行时执行主逻辑，被 import 时不执行。所有脚本都应加这个守卫，防止 import 时产生副作用。
- 什么是循环导入？如何解决？
    - A import B 同时 B import A，形成环形依赖。解决：把 import 移到函数内部延迟加载；重构共用代码到第三个模块；用 import 而非 from import。

---

# 5. I/O

- with open 为什么推荐？
    - with 自动调用 close()，即使中途抛异常也不会泄漏文件句柄。等价于 try/finally 的语法糖。
- read、readline、readlines 区别？
    - read() 一次读入全部（小文件）；readline() 逐行读；readlines() 一次读入所有行到 list。大文件用 for line in file —— 惰性迭代不爆内存。
- JSON 和 dict 的区别？
    - JSON 是字符串格式（跨语言传输），dict 是 Python 内存结构。JSON 键必须双引号，true/false/null vs True/False/None。json.dumps/loads 互转。
- pandas.read_csv() 和 csv 模块有什么区别？
    - csv 模块逐行解析返回 list/dict；pandas.read_csv() 返回 DataFrame，自动推断 dtype、处理缺失值、支持分块。数据工程中 99% 用 pandas。

