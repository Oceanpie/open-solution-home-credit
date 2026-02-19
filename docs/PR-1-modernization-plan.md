# PR-1 现代化改造方案（避免过度设计）

## 0) 本文采用的技术文档编写规则

本文遵循 Google Developer Style Guide、Microsoft Learn Style Guide、Diataxis 和 Write the Docs 的主流技术写作规范。

已应用规则：

1. 任务优先结构：先写目标、范围、验收标准，再写实施步骤。
2. 语言直接简洁：短句、主动语态、明确动词。
3. 术语一致：一个概念只使用一个术语（`tracker`、`run`、`metric`、`artifact`）。
4. 操作可扫读：使用编号步骤、明确文件路径、明确输出结果。
5. 避免歧义：明确“做什么”和“不做什么”。
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

在不改变现有 pipeline 设计和 CLI 使用方式的前提下，将实验追踪从 Neptune 替换为可插拔追踪层，并默认接入 W&B。

## 2) 范围

本 PR 包含：
1. 新增 tracker 抽象与 W&B 实现。
2. 现有指标上报统一走 tracker。
3. 新增 tracking 配置项。
4. 保持命令行为不变。
5. 更新 W&B 所需依赖与文档。

本 PR 不包含：
1. `steppy` 迁移到 Hamilton。
2. Keras 迁移到 PyTorch。
3. OOF/stacking 算法逻辑变更。
4. 调参框架重构。

## 3) 文件级改造清单

### 3.1 新增文件

1. `src/tracking/base.py`
- 定义 `Tracker` 接口：
  - `init_run(run_name: str, config: dict, tags: list[str] | None = None)`
  - `log_param(key: str, value)`
  - `log_params(params: dict)`
  - `log_metric(name: str, value, step: int | None = None)`
  - `log_metrics(metrics: dict, step: int | None = None)`
  - `log_artifact(path: str, name: str | None = None)`
  - `finish()`

2. `src/tracking/wandb_tracker.py`
- 使用 `wandb` 实现 `Tracker`：
  - `wandb.init(...)`
  - `wandb.config.update(...)`
  - `wandb.log(...)`
  - `wandb.log_artifact(...)`
  - `wandb.finish()`

3. `src/tracking/noop_tracker.py`
- 空实现，用于本地离线/CI 禁用上报场景。

4. `src/tracking/factory.py`
- 按配置创建 tracker：
  - backend: `wandb` 或 `noop`
  - mode: `online`、`offline`、`disabled`

### 3.2 修改文件

1. `src/pipeline_config.py`
- 在配置对象中新增 tracking 区块：
  - `tracking.backend`
  - `tracking.project`
  - `tracking.entity`
  - `tracking.mode`
  - `tracking.tags`

2. `src/utils.py`
- `read_params(ctx, fallback_file)` 当前依赖 Neptune 的 `OfflineContextParams` 判断离线/在线模式。
- 改为直接从 YAML 文件加载配置，不再依赖 Neptune Context。
- 移除 `from deepsense import neptune` 相关引用。

3. `src/pipeline_manager.py`
- 将直接 Neptune 上报（`ctx.channel_send`）替换为 tracker 调用。
- 指标名称保持不变（保证实验可比性）。

4. `src/callbacks.py`
- 将 Neptune 专用 callback 改为 tracker 版 callback。
- 保持原有上报语义：
  - 每个 epoch 的训练损失
  - 每个 epoch 的验证损失

5. `src/models.py`
- 将 Neptune callback 创建逻辑替换为 tracker callback 工厂。
- 模型训练/预测逻辑不变。

6. `main.py`
- 每个命令执行周期接入 tracker 生命周期：
  - 命令开始时 init
  - 命令结束（或 finally）时 finish

7. `configs/neptune.yaml`
8. `configs/neptune_stacking.yaml`
9. `configs/neptune_selected_features_v1.yaml`
- 新增配置键：
  - `tracking__backend: wandb`
  - `tracking__project: <project-name>`
  - `tracking__entity: <entity-or-empty>`
  - `tracking__mode: offline`
  - `tracking__tags: []`

10. `requirements.txt`
- 新增 `wandb`。
- PR-1 暂时保留 `neptune-cli`，降低回滚风险。

11. `README.md`
- 增加 W&B 快速开始：
  - `pip install -r requirements.txt`
  - `wandb login`
  - 运行原有命令
- 明确 Neptune 路径为 legacy/deprecated。

## 4) PR-1 内部实施顺序

1. 新增 `src/tracking/*`。
2. 接入配置项。
3. 替换上报调用点（`pipeline_manager`、`callbacks`、`models`）。
4. 在 `main.py` 接入 run 生命周期。
5. 更新配置文件和依赖。
6. 更新 README。

## 5) 验收标准

1. `train_evaluate_cv` 验收需覆盖两条路径（`--model_level` 默认值为 `first`）：
  - `python main.py -- train_evaluate_cv --pipeline_name lightGBM --model_level first -d`
  - `python main.py -- train_evaluate_cv --pipeline_name lightGBM --model_level second -d`
2. 当 `tracking__mode=online` 时，可在 W&B 看见指标。
3. 当 `tracking__mode=disabled` 时，本地可离线运行（不依赖外网）。
4. 现有命令名与主要参数不变。
5. 除日志后端替换外，模型行为不发生变化。

## 6) 风险控制

1. Neptune 依赖仅保留一个 PR 过渡（用于回滚兜底）。
2. PR-1 不重命名任何指标键。
3. PR-1 不改数据切分/CV/OOF 逻辑。
4. 增加最小 smoke test：
  - 初始化 tracker
  - 记录一个 metric
  - 正常 finish

## 7) PR-2 预告（不在本 PR）

1. 删除 Neptune 相关依赖与废弃代码路径。
2. 启动 `steppy` 到 Hamilton 迁移。
3. 启动 Keras 到 PyTorch Lightning 迁移。
