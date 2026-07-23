- **这个技术最核心的设计思想是什么？**
	- 以 DataFrame 为核心，提供高效、易用的数据处理能力。
	- 将数据看作二维表格（Table）进行操作，而不是逐行处理。
	- 基于 NumPy 实现，支持向量化计算（Vectorization），避免大量 Python for 循环，提高数据处理效率。
	- 提供丰富的数据清洗、转换、聚合、统计和时间序列处理能力，使数据分析过程更加简洁。

- **这个技术在 AI Data Engineering 中的典型应用是什么？**
	- 数据读取（CSV、Excel、JSON、Parquet 等）
	- 数据清洗（缺失值、重复值、异常值处理）
	- 数据转换（字段计算、类型转换、格式统一）
	- 特征预处理（Feature Engineering）
	- 文本数据预处理（LLM/RAG 数据准备）
	- 小规模 ETL 数据处理
	- 数据探索分析（EDA）
	- 数据质量检查（Data Validation）

- **这个技术解决了什么问题？**
	- 简化了复杂的数据处理流程，使数据清洗和转换更加高效。
	- 解决了原生 Python 处理结构化数据效率低、代码复杂的问题。
	- 提供统一的数据操作接口，降低不同数据格式之间转换的成本。
	- 解决数据分析过程中频繁进行筛选、聚合、连接、统计计算的问题。
	- 提高数据工程师处理结构化数据和半结构化数据的开发效率。

- **它通常和哪些技术一起使用？**
	- **NumPy**
		- Pandas 底层依赖 NumPy，提供数组计算和向量化能力。
	- **Python**
		- 作为主要开发语言，用于数据处理脚本和 ETL Pipeline。
	- **Parquet**
		- 常作为 Pandas 数据存储格式，用于数据湖场景。
	- **RAG Pipeline**
		- 配合 LangChain、LlamaIndex 处理文档元数据、文本数据清洗和格式转换。

# 1. Core_Concepts

---
## 1.1 Series

Series 是 Pandas 中的一维带标签数组（Labeled One-dimensional Array）。
它由两部分组成： Index、Value

例如：
```python
import pandas as pd

s = pd.Series(
    [100,200,300],
    index=["A","B","C"]
)

A    100
B    200
C    300
```

---
### 1. 带 Index

不同于 NumPy Array：
NumPy：
```
[10,20,30]
```
Series：
```
a 10
b 20
c 30
```

数据拥有标签，可以通过 label 访问。

---
### 2. 支持向量化操作

Series 可以直接进行批量计算： s * 2
底层通过 NumPy 实现高效计算。

---
### 3. 类似数据库中的 Column

在 DataFrame 中：每一列实际上就是一个 Series。

---
### 4. AI Data Engineering Application

常用于：单字段处理 特征计算 数据清洗 字段转换

---
## 1.2 DataFrame

DataFrame 是 Pandas 最核心的数据结构。
它是：一个二维、大小可变、带标签的数据表结构。
类似：
数据库：Table
Excel：Sheet

---
### Core Features

#### 1. 行列索引

- **`df.loc[]`**：**基于标签（Label-based）** 索引。你传入的是**行/列的名称或标签**。
- **`df.iloc[]`**：**基于位置（Integer position-based）** 索引。你传入的是**行/列从0开始的整数序号**。

---
#### 2. 列式操作

Pandas 推荐：df["age"]
而不是： for row in df:
原因：利用向量化计算。

---
#### 3. 数据处理中心

绝大多数 Pandas 操作围绕 DataFrame：
- Filtering
- Cleaning
- Aggregation
- Join
- Transformation

---
### AI Data Engineering Application

典型流程：

```
Raw Data
↓
DataFrame
↓
Cleaning
↓
Transformation
↓
Output Dataset
↓
Embedding / ML
```

---
## 1.3 Index

Index 是 Pandas 中用于标识数据位置的标签结构。
它类似数据库：Primary Key

---
### Core Features

#### 1. 数据定位

例如：df.loc["user001"]  通过 Index 查找数据。

---
#### 2. Index 与数据分离

Index 不属于数据列，而是数据的标识。

---
#### 3. 自动对齐（重点）

这是 Pandas 非常重要的设计。
例如：
```python
s1:
A 10
B 20

s2:
B 100
A 200
```

相加：
```
A 210
B 120
```

不是按照位置，而是按照 Index 对齐。

---
### Importance

Index 是 Pandas 区别于 NumPy 的核心设计之一。

---
### AI Data Engineering Application

常用于：
- 用户ID
- 时间索引
- 数据Join
- 时间序列处理

---
## 1.4 Vectorization

Vectorization（向量化）：
> 使用底层 C/NumPy 实现批量计算，避免 Python 层循环。

---
Traditional Python
```python
result=[]

for x in data:
    result.append(x*2)
```
问题：
- Python loop 慢
- CPU 利用率低

---
Pandas
```python
df["price"] * 2
```

一次处理整列。

---
### Advantages

减少 Python 循环。
代码更接近业务逻辑。
适合：
- NumPy
- Pandas
- PySpark

---

### AI Data Engineering Application

常见：
- 批量字段转换
- Feature Engineering
- 数据清洗

---

## 1.5 Copy vs View

Pandas 中操作 DataFrame 时：
有两种数据引用方式：
- Copy（复制）
- View（视图）

---
### Copy

```python
创建新的数据对象。
df2 = df.copy()
修改df2，不会影响df
```

---
### View

```Python
共享底层数据。
类似：df --> df_view
修改可能影响原数据。
```

---

### Common Problem

典型：
```python
df[df["age"]>18]["name"]="Tom"

可能产生：Setting With Copy Warning
```

原因：
Pandas 不确定操作的是：Copy 还是 View
- **View** = 原数据的“遥控器”（改了影响原数据）。
- **Copy** = 原数据的“照片”（改了不影响原数据）。
- **链式赋值 `][`** = 你在模糊地带操作，Pandas 不确定你手里拿的是“遥控器”还是“照片”，所以警告你。
- **最佳实践**：永远只用 **`df.loc`** 进行条件赋值，彻底告别这个烦恼。

---
### Importance in Data Engineering

避免：
- 数据被意外修改
- Pipeline 数据污染
- 隐式 Bug

---
# 2. Data_Cleaning
## 2.1 Missing Value

Missing Value（缺失值）是指数据中不存在或不可用的值，在 Pandas 中通常表示为 `NaN`、`None` 或 `pd.NA`。

缺失值会导致：
- 统计结果不准确
- 聚合计算异常
- 模型训练效果下降
- 数据质量降低

Common Methods
- 删除（Drop）
- 填充（Fill）
- 插值（Interpolation）
- 保留并标记缺失

> 处理方式应根据业务含义决定，而不是默认删除。

---
### Best Practice

- 先分析缺失原因，再决定处理方式。
- 区分真正缺失与业务默认值（如 `0`、`Unknown`）。
- 对关键字段（如主键、时间戳）优先保证完整性。
---

### AI Data Engineering Application

- 清洗训练数据
- 构建 RAG 知识库
- ETL 数据预处理
- 数据质量检查
---
## 2.2 Duplicate

Duplicate（重复数据）是指同一条业务记录被重复存储，可能是完全重复，也可能是业务字段重复。
重复数据会导致：
- 指标统计错误
- 数据冗余
- 模型训练偏差
- 存储资源浪费

Common Methods
- 全量去重
- 按业务主键去重
- 按组合字段去重
- 保留最新或最早记录
---
### Best Practice

- 明确业务唯一键，不要仅依赖整行去重。
- 去重策略应符合业务规则，例如订单保留最新状态。
---
### AI Data Engineering Application

- 用户画像构建
- 日志数据去重
- 文档去重（RAG）
- 增量同步数据处理
---
## 2.3 Type Conversion

Type Conversion（数据类型转换）是指将字段转换为适合业务处理的数据类型。
常见类型包括：
- Numeric
- String
- Boolean
- Datetime
- Category
---

错误的数据类型会导致：
- 无法计算
- 排序错误
- Join 失败
- 存储效率降低

Common Methods
- 字符串转数值
- 字符串转日期
- 数值转类别
- 布尔类型转换
---
### Best Practice

- 导入数据后优先检查数据类型。
- 时间字段统一转换为 `datetime`。
- 枚举字段可使用 `category` 提高内存利用率。
---
### AI Data Engineering Application

- 时间序列处理
- Feature Engineering
- 数据标准化
- Metadata 统一格式
---

## 2.4 Outlier

Outlier（异常值）是指明显偏离数据整体分布或业务规则的数据。
异常值既可能是真实业务，也可能是数据错误。

异常值可能导致：
- 统计分析失真
- 模型训练偏移
- 数据质量下降
- ETL 流程异常

Common Methods
- IQR（四分位距）
- Z-score
- Percentile（分位数）
- Business Rules（业务规则）

> 在数据工程中，业务规则通常比统计方法更常用。
---
### Best Practice

- 优先结合业务规则判断异常值。
- 删除前确认是否属于真实业务数据。
- 保留异常值处理日志，保证数据可追溯。
---
### AI Data Engineering Application

- 用户行为异常检测
- 日志异常分析
- RAG 文档长度检查
- 模型训练数据质量控制
---
## 总结
```markdown
# Data Cleaning Principles

1. 数据质量优先于数据数量。
2. 不盲目删除数据，应结合业务规则处理。
3. 保留原始数据，避免覆盖源数据。
4. 每一步清洗过程应可追溯、可复现。
5. 清洗后的数据应进行验证（Validation）。
```

---
# 3. Data_Transformation

## 3.1 Filtering（数据筛选）

Filtering 是根据条件筛选符合要求的数据行或数据列。是数据处理过程中最基础、最常用的转换操作。

---
### Why

减少数据规模，仅保留业务需要的数据，提高后续处理效率。

---
### Core Characteristics

- 行筛选（Row Filtering）
- 列筛选（Column Selection）
- 条件组合（AND / OR / NOT）
- 布尔索引（Boolean Indexing）
---

### Best Practice

- 优先使用布尔索引或 `loc`。
- 多条件筛选时保持条件表达式清晰。
- 避免链式索引（Chained Indexing）。
---

### AI Data Engineering Application

- 筛选有效用户
- 筛选训练样本
- 提取指定时间范围数据
- RAG 文档过滤
---
## 3.2 apply（自定义转换）

`apply` 用于将自定义函数应用到 Series 或 DataFrame，实现灵活的数据转换。

---
### Why

当内置向量化函数无法满足业务需求时，使用 `apply` 扩展处理逻辑。

---
### Core Characteristics

- 支持自定义函数
- 可按行或按列执行
- 灵活性高
---
### Best Practice

- 优先使用 Pandas 原生向量化方法。
- `apply` 作为无法向量化时的补充，而不是默认选择。
- 大规模数据处理中应谨慎使用，性能通常低于向量化操作。
---
### AI Data Engineering Application

- 文本清洗
- 字段标准化
- Metadata 生成
- Feature Engineering
---
## 3.3 map（映射转换）

### Concept

`map` 用于根据映射关系，对单个 Series 的每个元素进行值转换。

---
### Why

实现分类编码、标签映射和格式统一。

---

### Core Characteristics

- 作用于单列（Series）
- 支持字典、函数、Series 映射
- 保持数据结构不变，仅修改值
---
### Best Practice

- 一对一字段转换优先使用 `map`。
- DataFrame 整体转换不建议使用 `map`。
---
### AI Data Engineering Application

- 类别编码
- 状态值转换
- 标签标准化
- 枚举字段统一
---
## 3.4 merge（关联）

### Concept

`merge` 用于按照一个或多个键，将多个 DataFrame 进行关联。
对应 SQL 的 **JOIN**。

---
### Why

整合来自不同数据源的数据。

---
### Core Characteristics

- Inner Join
- Left Join
- Right Join
- Outer Join
- 多键关联

---
### Best Practice

- 明确 Join Key。
- 关注关联后的数据量变化。
- 检查重复键导致的数据膨胀。
---
### AI Data Engineering Application

- 用户画像构建
- 特征拼接
- Metadata 合并
- 多数据源 ETL
---

## 3.5 concat（拼接）

### Concept

`concat` 用于按行或按列拼接多个 DataFrame。

---
### Why

整合结构相同或互补的数据集。

---
### Core Characteristics

- Row Concatenation
- Column Concatenation
- 保持原始数据结构
---
### Best Practice

- 行拼接要求字段一致。
- 列拼接要求索引一致。
- 注意索引是否需要重建。
---
### AI Data Engineering Application

- 多批数据合并
- 增量数据拼接
- 多文件汇总
- RAG 文档集合构建
---
## 3.6 groupby（分组聚合）

> **这一节是整个 Pandas 最重要的章节。**

### Concept

`groupby` 根据指定字段进行分组，并对每个分组执行聚合计算。

对应 SQL 的 **GROUP BY**。

---
### Why

实现统计分析和业务指标计算。

---
### Core Characteristics

- Group（分组）
- Aggregate（聚合）
- Transform（组内转换）
- Filter（组过滤）

GroupBy 本质遵循：

> **Split → Apply → Combine**

即：

```text
Data
 ↓
Split（分组）
 ↓
Apply（计算）
 ↓
Combine（合并结果）
```

---
### Best Practice

- 优先理解 **Split–Apply–Combine** 模型，而不是死记 API。
- 熟悉与 SQL `GROUP BY` 的对应关系。
- 区分聚合（Aggregation）与组内转换（Transformation）。

---
### AI Data Engineering Application

- 用户行为统计
- 数据质量分析
- 特征聚合
- 指标计算
- ETL 汇总任务
---
## 3.7 Sort & Ranking（排序与排名）

### Concept

排序（Sorting）用于按照指定字段对数据进行重新排列；排名（Ranking）用于根据排序结果计算数据的相对位置。
排序决定数据的展示和处理顺序，排名用于识别 Top N、Bottom N 及相对名次。

---
### Why

排序与排名能够帮助：
- 提高数据可读性
- 获取 Top N / Bottom N 数据
- 支持业务统计分析
- 为后续聚合、窗口计算和特征工程提供基础
---
### Core Characteristics
#### Sorting（排序）

按照一个或多个字段进行升序或降序排列。
常用方法：
- `sort_values()`
- `sort_index()`
---
#### Ranking（排名）

根据排序结果计算排名。
常用方法：
- `rank()`
- `nlargest()`
- `nsmallest()`
---
#### Stable Sorting（稳定排序）

当多个记录排序值相同时，保持原有相对顺序，有助于保证数据处理结果的一致性。

---
### Best Practice

- 优先使用 `sort_values()` 按业务字段排序。
- Top N 查询优先使用 `nlargest()`、`nsmallest()`，通常比整体排序后再取前几条效率更高。
- 多字段排序时，应明确主排序字段和次排序字段。
- 排序后如需连续索引，可根据业务需要重置索引。
---
### AI Data Engineering Application

- 获取销量、点击量等 Top N 数据
- 用户活跃度排名
- 特征重要性排序
- RAG 检索结果排序
- ETL 数据输出排序
- 数据质量检查与异常数据定位
---
# 4. AI_Data_Engineering

## 4.1 Pandas in ETL

**ETL 中 Pandas 真正承担什么角色**。
### ① Pandas 在 ETL 中的位置

例如：

```
Source  (MySQL/API/CSV)
      │
      ▼
Extract  (read_sql/read_csv)
      │
      ▼
Transform  (Pandas)
      │
      ▼
Load  (to_sql / Parquet / S3)
```

说明：
> Pandas 更多负责中小数据量的数据清洗、转换、聚合，不负责 PB 级计算。

---
### ② ETL 常见 Transformation Pattern

这里只总结最常见的转换。

例如：
```python
列处理
rename()
assign()
drop()
astype()

缺失值处理
isna()
fillna()
dropna()

重复数据
duplicated()
drop_duplicates()

字符串清洗
str.strip()
str.lower()
str.replace()

日期处理
to_datetime()
dt.year
dt.month

聚合统计
groupby()
agg()
pivot_table()
```
---
### ③ ETL Pipeline 示例

例如：
```python
df = (
    pd.read_csv(...)
      .drop_duplicates()
      .assign(
          amount=lambda x: x.amount.fillna(0)
      )
      .query("status == 'SUCCESS'")
      .groupby("user_id")
      .agg(total=("amount","sum"))
)
```

这里强调: 推荐使用 Method Chaining。
这是 Pandas 工程写法。

---
### ④ ETL Best Practice

例如：
不要 for
尽量 Vectorization

不要 iterrows()
尽量 merge()、groupby()、map()、transform()

---
## 4.2 Pandas in RAG Pipeline

实际上 Pandas 在 RAG 里面主要负责 **Metadata Engineering**，而不是 Embedding。

---
### ① RAG Pipeline

```
Documents
↓
Chunk
↓                  ---->           Pandas
Metadata
↓
Embedding
↓
Vector DB
```
---
### ② Metadata 清洗

例如：title、author、date、source、url、language、category
这些字段通常都会放在 DataFrame。
例如：chunk_id、doc_id、page、title、source、text
整个 Metadata 都可以 Pandas 管。

---
### ③ Chunk 管理

例如：chunk_id、document_id、start、end、token、text
这些通常都是 DataFrame。
之后：merge、filter、sort、join
全部 Pandas。

---
### ④ Embedding 前的数据处理

例如
删除：NaN、Empty、Duplicate
过滤：Token Length、Language、Category
统计：Chunk Count、Average Length、Distribution

---
### ⑤ Evaluation

例如
统计：Recall、Hit Rate、Latency、Chunk Length
最终实验数据：
```
Experiment Result
↓
DataFrame
↓
Analysis
```

很多 RAG Evaluation 都是 Pandas。

---
## 4.3 Pandas vs PySpark

|对比|Pandas|PySpark|
|---|---|---|
|数据规模|MB~GB|GB~PB|
|运行方式|单机|分布式|
|内存|全内存|分布式|
|API|Python|Spark|
|性能|小数据快|大数据快|
|AI Pipeline|⭐⭐⭐⭐⭐|⭐⭐⭐|
|ETL|⭐⭐⭐⭐|⭐⭐⭐⭐⭐|

---

下面写一个经验原则：

```
<1GB Pandas
10GB+ PySpark
```

强调：
> AI Data Engineering 中，大量数据准备、特征统计、Embedding Metadata、实验分析依然大量使用 Pandas；真正的大规模离线处理则交给 Spark/PySpark。

---
## 4.4 Pandas vs SQL

### SQL 擅长

```
JOIN、GROUP BY、Window、Aggregation
```

数据库做。

---
### Pandas 擅长

```
EDA(Exploratory Data Analysis)、Cleaning、Feature、Visualization、ML Pipeline
```

Python 做。

---
### 对照关系

|SQL|Pandas|
|---|---|
|SELECT|[]|
|WHERE|query()|
|GROUP BY|groupby()|
|ORDER BY|sort_values()|
|JOIN|merge()|
|COUNT|count()|
|SUM|sum()|
|AVG|mean()|
|DISTINCT|drop_duplicates()|
|CASE WHEN|np.where()|

---

其实 AI Data Engineering 最值得沉淀的是下面两个内容：
## 4.5 Performance Best Practices（性能最佳实践）

这一章只记录真正影响性能的知识点，例如：

- 向量化（Vectorization）优于循环
- 避免 `iterrows()`，优先 `itertuples()`，更推荐向量化
- 合理设置数据类型（`category`、`int32`、`float32`）
- 使用 `merge`、`map`、`transform` 替代手写循环
- 分块读取（`chunksize`）处理大文件
- Parquet 优于 CSV 作为中间数据格式
- 必要时使用 Polars 或 PySpark 处理超大数据集

---

## 4.6 Best Practices（工程最佳实践）

这一章记录团队开发规范，而不是 API，例如：

- 优先采用 Method Chaining，避免大量中间变量
- 保持 DataFrame 不可变风格，减少副作用
- 列名统一使用 `snake_case`
- 显式指定数据类型，避免隐式类型转换
- 尽早过滤无关数据，减少后续计算量
- 将数据校验（Schema、空值、唯一性）纳入 ETL 流程
- 将 DataFrame 作为数据载体，而不是业务逻辑容器

---
# 5. Interview

## 5.1 Core Concepts

### 1. 什么是 Pandas？为什么需要 Pandas？

**Pandas 是建立在 NumPy 之上的核心数据分析库。**

**为什么需要它，主要有三点：**
1. **数据结构（DataFrame）**：它提供了二维带标签的异构表格，相比 NumPy 的同质数组，更能直观地表达**结构化数据**（如 Excel、数据库表）。
2. **向量化（Vectorization）**：它继承了 NumPy 的向量化特性，将循环下沉到 C 层面执行，避免了 Python 原生慢循环，处理百万级数据极其高效。
3. **专门工具**：它专为数据清洗、转换和聚合而生，提供了比 NumPy 更丰富、更人性化的 API（如 `groupby`、`pivot`、缺失值处理），极大降低了**结构化数据**的处理门槛。
---

### 2. Series 和 DataFrame 有什么区别？

**核心区别在于维度与结构：**
1. **维度不同**：**Series 是一维**（带标签的数组），而 **DataFrame 是二维**（带标签的表格）。
2. **组成关系**：**DataFrame 本质上是多个 Series 的集合**。每一列就是一个独立的 Series，它们共享同一个行索引（Index），从而拼合成一个完整的二维表格。
3. **数据类型**：由于上述结构，Series 通常**仅包含一种数据类型**；而 DataFrame 由多个 Series 构成，所以**每列可以拥有不同的数据类型**（如整型、字符串、浮点型混搭）。
---
### 3. Pandas 和 NumPy 有什么关系？

**Pandas 完全构建在 NumPy 之上**，NumPy 是 Pandas 的底层计算引擎。DataFrame 中的数值型列在底层实际上就是 NumPy 的 `ndarray` 对象，向量化运算最终都下沉到 NumPy 的 C 层面执行。

**两者的核心差异（`ndarray` vs `DataFrame`）：**
1. **数据类型**：`ndarray` 要求所有元素**同质**（必须同一 dtype），而 `DataFrame` 是**异构**的，允许每列拥有不同的数据类型。
2. **索引机制**：`ndarray` 仅支持隐式的整数位置下标；`DataFrame` 则额外提供了显式的行/列标签（Index/Columns），支持基于标签的对齐运算。
3. **功能定位**：NumPy 专注于底层数学运算（线性代数、随机数）；Pandas 在此基础上封装了高级 API，专攻**结构化数据**的清洗、分组、透视和时间序列处理。
---
### 4. Pandas 为什么比 Python for 循环快？

**核心原因在于 Pandas 将循环操作从 Python 层“下沉”到了 C 层执行。** 
具体有三点：
1. **向量化（Vectorization）**：Pandas 操作（如 `df['col'] + 1`）是对**整个内存块**批量处理，而非在 Python 解释器中逐行遍历。这消除了 Python `for` 循环每次迭代时的类型检查与字节码执行开销。
2. **底层依赖 NumPy**：Pandas 的数值列底层是 NumPy 的 `ndarray`，数据在内存中**连续存储**，利用 CPU 的缓存局部性（Cache Locality）和 SIMD 指令集进行批量运算。
3. **C/C++ 实现**：Pandas 和 NumPy 的核心计算逻辑用 C（或 Cython）编写。`for` 循环跑在 Python 解释器里，而 Pandas 的循环跑在编译好的 C 代码里，两者速度差距通常是 **几十倍到上百倍**。
---

### 5. Index 的作用是什么？

重点：
- Label
- 自动对齐
- Join
- Time Series
---

### 6. loc 和 iloc 有什么区别？

这个是必问。

---

### 7. Copy 和 View 有什么区别？

重点：SettingWithCopyWarning

---

## 5.2 Data Cleaning

### 8. 如何处理缺失值？

重点：
什么时候 Drop
什么时候 Fill
什么时候保留

---

### 9. 如何处理重复数据？

重点：
业务主键
drop_duplicates()

---

### 10. 如何识别异常值？

重点：
IQR
Business Rule

---

### 11. 为什么数据类型转换很重要？

重点：
Join
Memory
Datetime
Category

---

## 5.3 Transformation

### 12. apply、map、transform 有什么区别？

这是高频。

---

### 13. merge 和 concat 有什么区别？

高频。

---

### 14. groupby 的底层思想是什么？

回答：
Split
↓
Apply
↓
Combine

---

### 15. Pandas 如何实现 SQL GROUP BY？

---

### 16. Top N 为什么推荐 nlargest()？

而不是：sort_values().head()

---

## 5.4 AI Data Engineering

### 17. Pandas 在 ETL 中主要负责什么？

重点：
Cleaning
Transformation
Aggregation
Validation

---

### 18. Pandas 在 RAG Pipeline 中承担什么角色？

重点：
Metadata Engineering
Chunk 管理
Evaluation

---

### 19. Pandas 和 SQL 应该如何分工？

重点：
数据库负责：
Filtering
Aggregation
Join

Pandas负责：
Cleaning
Feature
EDA

---

### 20. Pandas 和 PySpark 应该如何选择？

重点：
数据规模
工程复杂度
资源

---
### 21. 为什么不推荐 iterrows()？

---

### 22. Method Chaining 有什么好处？

---

### 23. 如何优化 Pandas 内存？

category
int32
float32
Parquet

---

### 24. Pandas 如何处理百万级数据？

chunksize
PySpark
Polars

---

### 25. AI Data Engineer 为什么必须熟悉 Pandas？

这是你的职业方向。

---

# 3. 毕业标准
## L1 基础

能够理解：
✅ Series
✅ DataFrame
✅ Index
✅ Vectorization
✅ Copy/View

---

能够完成：
- CSV 读取
- DataFrame 创建
- 基本筛选
- 排序
- 简单统计
---

## L2 数据处理

能够独立完成：
- Missing Value
- Duplicate
- Type Conversion
- Outlier
---

熟练使用：
- Filtering
- merge
- concat
- groupby
- sort
---

能够写出：一个完整的数据清洗 Pipeline。

---

## L3 工程实践

能够：
✅ 编写 ETL Pipeline

例如：
```
Read
↓
Cleaning
↓
Transformation
↓
Aggregation
↓
Export
```

---

能够：
避免：
```
iterrows()
for
```
使用：
```
Vectorization
Method Chaining
```

---

理解：
Memory
Performance
Schema
Validation

---

## L4 AI Data Engineering

能够回答：

为什么：Pandas、PySpark、SQL要组合使用。

---

能够：
完成：RAG Metadata Pipeline。
例如：

```
Document
↓
Chunk
↓
Metadata
↓
Cleaning
↓
Embedding
↓
Vector DB
```

---

能够使用 Pandas 统计:
- Recall
- Chunk Length
- Token Distribution
- Dataset Quality

---
