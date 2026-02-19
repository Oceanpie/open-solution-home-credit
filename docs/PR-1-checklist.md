# PR-1 准备与执行清单

## 1) 开始前（必做）

- [ ] 已确定 W&B 组织与项目命名（`entity/project`）。
- [ ] 已确认实验数据可上传到 W&B（合规/隐私/权限通过）。
- [ ] 已完成 `wandb login`（至少一名开发者可用）。
- [ ] 已约定 Python 基线版本（统一 `3.12`）。
- [ ] 已阅读 Hamilton 文档，了解基本概念。

## 2) 阻塞性依赖替换

### 2.1 steppy → Hamilton 迁移

- [ ] 安装 `sf-hamilton` 并验证可用。
- [ ] 创建 `src/pipeline/` 目录结构。
- [ ] 将 `src/pipeline_blocks.py` 中的 Step 转换为 Hamilton 函数。
- [ ] 将 `src/models.py` 中 `BaseTransformer` 子类重构为纯函数或类。
- [ ] 自实现 `ClassifierXY`（原 `toolkit.keras_transformers`）。
- [ ] 自实现 `SklearnClassifier`（原 `toolkit.sklearn_transformers`）。
- [ ] 更新 `src/pipelines.py` 使用 Hamilton Driver。
- [ ] 移除所有 `from steppy.*` 导入。
- [ ] 移除所有 `from toolkit.*` 导入。

### 2.2 attrdict → pydantic 替换

- [ ] 安装 `pydantic`。
- [ ] 定义 pydantic 配置类（替代 AttrDict）。
- [ ] 替换 `src/utils.py` 中的 `AttrDict` → pydantic 模型。
- [ ] 替换 `src/pipeline_config.py` 中的 `AttrDict` → pydantic 模型。
- [ ] 替换 `src/models.py` 中的 `AttrDict` → pydantic 模型。
- [ ] 移除所有 `from attrdict import AttrDict` 导入。

### 2.3 hyperopt → optuna 替换

- [ ] 安装 `optuna`。
- [ ] 替换 `src/hyperparameter_tuning.py` 中的 hyperopt 调用（如有使用）。
- [ ] 移除 `hyperopt` 依赖。

### 2.4 sklearn.externals.joblib 修复

- [ ] 安装独立 `joblib` 包。
- [ ] 替换 `from sklearn.externals import joblib` → `import joblib`。

## 3) 依赖升级

- [ ] 创建 `requirements-legacy.txt`，完整存档 2018 年原始依赖版本。
- [ ] 更新 `requirements.txt`：
  - [ ] pandas 0.23.1 → 2.2.x
  - [ ] numpy 1.22.0 → 2.x
  - [ ] scikit-learn 0.19.1 → 1.6.x
  - [ ] lightgbm 2.1.1 → 4.x
  - [ ] xgboost 0.72.1 → 2.x
  - [ ] catboost 0.9.1.1 → 1.2.x
  - [ ] category_encoders 1.2.6 → 2.8.x
  - [ ] click 6.7 → 8.x
  - [ ] pyyaml >=4.2b1 → 6.x
- [ ] 新增依赖：`sf-hamilton`、`wandb`、`pydantic`、`joblib`、`optuna`、`ruff`。
- [ ] 移除依赖：`steppy`、`steppy-toolkit`、`attrdict`、`neptune-cli`、`hyperopt`、`nose`。
- [ ] 更新 `pyproject.toml`，填充 `dependencies` 字段。
- [ ] 适配 pandas 2.x API 变更（如 `append()` → `pd.concat()`）。
- [ ] 适配 scikit-learn 1.x API 变更。

## 4) 追踪层改造

- [ ] 新增 `src/tracking/base.py`（Tracker 抽象接口）。
- [ ] 新增 `src/tracking/wandb_tracker.py`（W&B 实现）。
- [ ] 新增 `src/tracking/noop_tracker.py`（禁用/离线空实现）。
- [ ] 新增 `src/tracking/factory.py`（按配置构建 tracker）。
- [ ] 修改 `src/pipeline_config.py`，加入 tracking 配置项，移除 Neptune Context。
- [ ] 修改 `src/utils.py`，移除 `read_params()` 对 Neptune 的依赖。
- [ ] 修改 `src/pipeline_manager.py`，替换 `ctx.channel_send` 到 tracker。
- [ ] 修改 `src/callbacks.py`，替换 Neptune callback 为 tracker callback。
- [ ] 修改 `src/models.py`，接入 tracker callback 工厂。
- [ ] 修改 `main.py`，实现 run 生命周期（init/finish）。
- [ ] 修改 `configs/*.yaml`，增加 tracking 配置键。

## 5) 配置与文档

- [ ] 重命名配置文件（可选）：`neptune.yaml` → `config.yaml`。
- [ ] 修改 `README.md`，补充 W&B 快速开始。
- [ ] 添加 `ruff.toml` 或 `pyproject.toml` 中的 ruff 配置。
- [ ] 更新 `AGENTS.md` 中的 Python 版本和依赖说明。

## 6) 验证与回归

- [ ] `uv sync` 成功安装所有依赖。
- [ ] `python main.py -- train_evaluate_cv --pipeline_name lightGBM -d` 可运行。
- [ ] `tracking__mode=online` 下能正常创建 W&B run 并看到指标。
- [ ] `tracking__mode=disabled` 下可离线运行且不报错。
- [ ] `ruff check .` 通过。

## 7) 提交与合并

- [ ] 提交 1：`requirements-legacy.txt` + 依赖升级。
- [ ] 提交 2：steppy → Hamilton 迁移（`src/pipeline/`、`src/pipeline_blocks.py`、`src/pipelines.py`）。
- [ ] 提交 3：attrdict → pydantic + hyperopt → optuna + joblib 修复。
- [ ] 提交 4：追踪层改造（`src/tracking/*` + 调用替换）。
- [ ] 提交 5：配置与文档更新。
- [ ] PR 描述已明确"本 PR 包含：Python 3.12 升级、steppy→Hamilton、依赖升级、Neptune→W&B"。
- [ ] PR 附上工程行为验收结果。
