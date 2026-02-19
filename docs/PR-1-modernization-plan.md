# PR-1 现代化改造方案

## 0) 本文采用的技术文档编写规则

本文遵循 Google Developer Style Guide、Microsoft Learn Style Guide、Diataxis 和 Write the Docs 的主流技术写作规范。

已应用规则：

1. 任务优先结构：先写目标、范围、验收标准，再写实施步骤。
2. 语言直接简洁：短句、主动语态、明确动词。
3. 术语一致：一个概念只使用一个术语（`tracker`、`run`、`metric`、`artifact`）。
4. 操作可扫读：使用编号步骤、明确文件路径、明确输出结果。
5. 避免歧义：明确"做什么"和"不做什么"。
6. 可读性优先：避免口语化和文化特定表达。
7. 文档与改造同步：本文可直接作为 PR 实施清单。

参考来源：
- https://developers.google.com/style
- https://developers.google.com/style/accessibility
- https://learn.microsoft.com/en-us/contribute/content/style-quick-start
- https://learn.microsoft.com/en-us/style-guide/procedures-instructions/formatting-text-in-instructions
- https://diataxis.fr/
- https://www.writethedocs.org/guide/writing/docs-principles.html

## 1) PR-1 目标

将 2018 年项目升级到 Python 3.12 可运行状态：

1. **流水线迁移**：steppy → Hamilton（steppy 不支持 Python 3.12）
2. **移除废弃依赖**：steppy、steppy-toolkit、attrdict、nose
3. **依赖升级**：pandas、numpy、scikit-learn、lightgbm、xgboost、catboost 等
4. **追踪层改造**：Neptune → W&B
5. 保持现有 CLI 和业务逻辑不变

## 2) 范围

### 2.1 阻塞性依赖替换（必须完成）

| 依赖 | 问题 | 替换方案 | 复杂度 |
|------|------|---------|--------|
| **steppy** | 不支持 Python 3.12（仅 3.5-3.7） | Hamilton | 🔴 高 |
| **steppy-toolkit** | 依赖 steppy，已归档 | 自实现 transformers | 🔴 高 |
| **attrdict** | 2019 年归档 | pydantic | 🟡 中 |
| **sklearn.externals.joblib** | sklearn 1.0+ 移除 | 独立 joblib 包 | 🟢 低 |
| **hyperopt** | 不活跃 | Optuna | 🟡 中 |
| **nose** | 已废弃 | pytest（如需测试） | 🟢 低 |

### 2.2 依赖升级

| 依赖 | 当前版本 | 目标版本 | API 变更风险 |
|------|---------|---------|-------------|
| pandas | 0.23.1 | 2.2.x | 中（部分 API 废弃） |
| numpy | 1.22.0 | 2.x | 低 |
| scikit-learn | 0.19.1 | 1.6.x | 中 |
| lightgbm | 2.1.1 | 4.x | 中 |
| xgboost | 0.72.1 | 2.x | 中 |
| catboost | 0.9.1.1 | 1.2.x | 低 |
| category_encoders | 1.2.6 | 2.8.x | 低 |
| click | 6.7 | 8.x | 低 |
| pyyaml | >=4.2b1 | 6.x | 低 |

### 2.3 新增依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| sf-hamilton | 1.x | 流水线框架（替代 steppy） |
| wandb | 0.19.x | 实验追踪（替代 neptune） |
| pydantic | 2.x | 配置管理（替代 attrdict） |
| joblib | 1.4.x | 模型持久化（从 sklearn 分离） |
| optuna | 4.x | 超参调优（替代 hyperopt） |
| ruff | 0.9.x | 代码检查 |

### 2.4 移除的依赖

| 依赖 | 移除原因 |
|------|---------|
| steppy | 不支持 Python 3.12 |
| steppy-toolkit | 依赖 steppy |
| attrdict | 2019 年归档 |
| neptune-cli | 替换为 wandb |
| hyperopt | 不活跃，替换为 optuna |
| nose | 已废弃 |

### 2.5 不包含

- Keras → PyTorch 迁移
- OOF/stacking 算法逻辑变更
- notebook 重写

## 3) steppy → Hamilton 迁移

### 3.1 概念映射

| steppy | Hamilton |
|--------|----------|
| `Step` | 函数（参数名定义依赖） |
| `BaseTransformer` | 纯函数 |
| `Adapter` + `E()` | 函数参数（自动推断） |
| `persist_output=True` | `@materialize` 装饰器 |
| `cache_output=True` | `@cache` 装饰器 |
| `input_steps` | 函数参数名 |

### 3.2 代码迁移示例

**steppy 方式（旧）：**
```python
from steppy.base import Step
from steppy.adapter import Adapter, E

features = Step(
    name='features',
    transformer=FeatureExtractor(),
    input_data=['main_table'],
    adapter=Adapter({'X': E('main_table', 'data')})
)

model = Step(
    name='light_gbm',
    transformer=LightGBM(),
    input_steps=[features],
    adapter=Adapter({'X': E(features.name, 'features')})
)
```

**Hamilton 方式（新）：**
```python
# features.py
import pandas as pd

def features(main_table: dict) -> pd.DataFrame:
    """特征提取"""
    return FeatureExtractor().transform(main_table['data'])

def light_gbm(features: pd.DataFrame, config: dict) -> object:
    """模型训练"""
    return LightGBM(**config).fit(features)
```

### 3.3 steppy-toolkit 替换

需要自实现的类：

| steppy-toolkit | 新实现位置 |
|----------------|-----------|
| `toolkit.keras_transformers.models.ClassifierXY` | `src/models.py:NeuralNetwork`（已有，需重构） |
| `toolkit.sklearn_transformers.models.SklearnClassifier` | `src/models.py:get_sklearn_classifier`（已有，需重构） |

## 4) 文件级改造清单

### 4.1 新增文件

1. **`src/tracking/` 目录**
   - `base.py`：Tracker 抽象接口
   - `wandb_tracker.py`：W&B 实现
   - `noop_tracker.py`：空实现
   - `factory.py`：工厂函数

2. **`src/pipeline/` 目录**（Hamilton 管道）
   - `__init__.py`
   - `features.py`：特征工程函数
   - `models.py`：模型训练函数
   - `preprocessing.py`：预处理函数
   - `config.py`：管道配置

3. **`requirements-legacy.txt`**
   - 存档 2018 年原始依赖版本

### 4.2 重构文件

| 文件 | 变更内容 |
|------|---------|
| `src/pipeline_blocks.py` | steppy Step → Hamilton 函数 |
| `src/pipelines.py` | 使用 Hamilton Driver 执行 |
| `src/models.py` | 移除 steppy BaseTransformer，移除 toolkit 导入 |
| `src/utils.py` | 移除 attrdict，修复 joblib 导入 |
| `src/pipeline_config.py` | 移除 Neptune Context，使用 pydantic 配置类 |
| `src/callbacks.py` | Neptune → tracker |

### 4.3 配置文件

| 文件 | 变更 |
|------|------|
| `configs/*.yaml` | 新增 tracking 配置键 |
| `pyproject.toml` | 填充 dependencies |
| `ruff.toml` | 新增代码检查配置 |

## 5) 实施顺序

### 阶段一：依赖准备
1. 创建 `requirements-legacy.txt` 存档
2. 更新 `pyproject.toml` 依赖
3. 替换 `attrdict` → `pydantic`（定义配置类）
4. 替换 `hyperopt` → `optuna`（如项目使用）
5. 修复 `sklearn.externals.joblib` → `joblib`

### 阶段二：Hamilton 迁移（核心）
1. 创建 `src/pipeline/` 目录结构
2. 将 `pipeline_blocks.py` 中的 Step 转换为函数
3. 将 `BaseTransformer` 子类转换为纯函数
4. 自实现 `ClassifierXY` 和 `SklearnClassifier`
5. 更新 `pipelines.py` 使用 Hamilton Driver

### 阶段三：追踪层改造
1. 新增 `src/tracking/*`
2. 替换 Neptune 调用
3. 接入 W&B

### 阶段四：验收与文档
1. 验证核心命令可运行
2. 更新 README
3. 添加 ruff 配置

## 6) 验收标准

1. **Python 3.12 可运行**：
   ```bash
   uv sync
   python main.py -- train_evaluate_cv --pipeline_name lightGBM -d
   ```

2. **W&B 追踪正常**：
   - `tracking__mode=online` 时可在 W&B 看见指标
   - `tracking__mode=disabled` 时可离线运行

3. **代码质量**：
   - `ruff check .` 通过

4. **行为一致**：
   - 命令行参数不变
   - 模型输出格式不变

## 7) 风险控制

| 风险 | 缓解措施 |
|------|---------|
| Hamilton 学习曲线 | 先实现简单管道，逐步迁移 |
| API 变更导致行为差异 | 保留 `requirements-legacy.txt` 可回滚 |
| 神经网络 Keras 版本 | 保持 Keras 依赖，仅重构管道层 |

## 8) PR-2 预告（不在本 PR）

1. 删除 `src/legacy/` 归档代码
2. Keras → PyTorch 迁移（可选）
3. notebook 重构
