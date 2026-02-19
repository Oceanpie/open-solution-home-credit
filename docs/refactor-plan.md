# 重构总计划（Refactor Plan）

## 1. 目标

在不改变项目业务设计思路的前提下，用现代技术栈重构工程实现：

1. Python 版本统一为 `3.12`。
2. 实验追踪使用 W&B（默认 `offline`，可切 `online`）。
3. 旧代码归档到 `src/legacy`，新实现走新代码路径。
4. 使用 `ruff` 做代码质量检查。
5. CI 暂置空，后续单独补齐。

## 2. 范围

包含：

1. 工程结构重组与旧代码归档。
2. 依赖升级与依赖存档。
3. 追踪、流水线、模型实现的现代化替换。
4. 文档与迁移说明完善。

不包含：

1. 以模型分数为目标的重新训练优化。
2. 旧代码删除（本阶段不删）。
3. notebook 立即重写（先存档）。

## 3. 技术栈决策

1. Python: `3.12`
2. 依赖管理: `uv` + `pyproject.toml`
3. 追踪: W&B（默认 `offline`）
4. 流水线: Hamilton（替代已归档的 steppy）
5. 代码质量: `ruff`
6. 目录策略: `src/legacy` 存档旧实现

## 4. 依赖升级策略

### 4.1 PR-1 升级的依赖

| 依赖 | 当前版本 | 目标版本 | 说明 |
|------|---------|---------|------|
| pandas | 0.23.1 | 2.2.x | 核心 DataFrame 库 |
| numpy | 1.22.0 | 2.x | 数值计算基础 |
| scikit-learn | 0.19.1 | 1.6.x | 机器学习框架 |
| lightgbm | 2.1.1 | 4.x | GBDT 模型 |
| xgboost | 0.72.1 | 2.x | GBDT 模型 |
| catboost | 0.9.1.1 | 1.2.x | GBDT 模型 |
| category_encoders | 1.2.6 | 2.8.x | 分类编码 |
| click | 6.7 | 8.x | CLI 框架 |
| pyyaml | >=4.2b1 | 6.x | YAML 解析 |
| wandb | - | 0.19.x | **新增** 实验追踪 |
| ruff | - | 0.9.x | **新增** 代码检查 |

### 4.2 PR-1 移除/替换的依赖

| 依赖 | 当前版本 | 处理方式 | 说明 |
|------|---------|---------|------|
| steppy | 0.1.4 | **移除** | 不支持 Python 3.12，迁移到 Hamilton |
| steppy-toolkit | 0.1.5 | **移除** | 依赖 steppy，自实现替代 |
| attrdict | 2.0.0 | **移除** | 2019 年归档，替换为 pydantic |
| neptune-cli | - | **移除** | 替换为 wandb |
| hyperopt | 0.1 | **移除** | 不活跃，替换为 optuna |
| nose | 1.3.7 | **移除** | 已废弃 |

### 4.3 PR-1 新增的依赖

| 依赖 | 版本 | 说明 |
|------|------|------|
| sf-hamilton | 1.x | 现代 DAG 流水线框架（替代 steppy） |
| pydantic | 2.x | 配置管理（替代 attrdict） |
| joblib | 1.4.x | 模型持久化（从 sklearn 分离） |
| wandb | 0.19.x | 实验追踪（替代 neptune） |
| optuna | 4.x | 超参调优（替代 hyperopt） |
| ruff | 0.9.x | 代码检查 |

### 4.4 依赖存档

- `requirements-legacy.txt`：完整存档 2018 年原始依赖版本
- `pyproject.toml`：新依赖入口

## 5. 分阶段里程碑

1. **PR-1**：Python 3.12 升级 + steppy→Hamilton + 依赖升级 + Neptune→W&B。
2. **PR-2**：目录重组与旧代码归档（`src/legacy`）。
3. **PR-3**：模型层重构（保持现有设计，不做联合优化）。
4. **PR-4**：文档收口与开发流程稳定化（ruff、迁移文档、常见问题）。

## 6. 验收基线（工程行为基线）

不以“重新训练到同分”为验收条件，采用以下基线：

1. 命令基线：核心命令可执行完成（`train/evaluate/predict/train_evaluate_predict_cv`）。
2. 功能基线：支持 `dev_mode`，流程可跑通。
3. 产物基线：关键输出文件结构与字段符合既有约定（例如 `submission` 列结构）。
4. 配置基线：旧配置可映射到新配置，无法映射项需明确说明。
5. 日志基线：默认 `offline` 可运行；切换 `online` 可上报 W&B。
6. 质量基线：`ruff` 通过。

## 7. 风险与约束

1. 旧代码暂不删除，确保可回溯。
2. 先做实现替换，不做算法层过度设计。
3. notebook 先存档，后续再按新代码重构。
