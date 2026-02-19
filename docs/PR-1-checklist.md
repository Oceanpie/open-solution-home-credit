# PR-1 准备与执行清单

## 1) 开始前（必做）

- [ ] 已确定 W&B 组织与项目命名（`entity/project`）。
- [ ] 已确认实验数据可上传到 W&B（合规/隐私/权限通过）。
- [ ] 已完成 `wandb login`（至少一名开发者可用）。
- [ ] 已约定 Python 基线版本（统一 `3.12`）。
- [ ] 已记录工程行为基线（命令、产物结构、日志模式），不以 AUC 对齐为验收目标。
- [ ] 已确认默认 tracking 模式为 `offline`（可切换 `online` / `disabled`）。
- [ ] 已冻结现有 metric 命名，不在 PR-1 重命名。

## 2) 代码改造（PR-1 范围）

- [ ] 新增 `src/tracking/base.py`（Tracker 抽象接口）。
- [ ] 新增 `src/tracking/wandb_tracker.py`（W&B 实现）。
- [ ] 新增 `src/tracking/noop_tracker.py`（禁用/离线空实现）。
- [ ] 新增 `src/tracking/factory.py`（按配置构建 tracker）。
- [ ] 修改 `src/pipeline_config.py`，加入 tracking 配置项。
- [ ] 修改 `src/utils.py`，移除 `read_params()` 对 Neptune `OfflineContextParams` 的依赖，改为直接 YAML 加载。
- [ ] 修改 `src/pipeline_manager.py`，替换 `ctx.channel_send` 到 tracker。
- [ ] 修改 `src/callbacks.py`，替换 Neptune callback 为 tracker callback。
- [ ] 修改 `src/models.py`，接入 tracker callback 工厂。
- [ ] 修改 `main.py`，实现 run 生命周期（init/finish）。
- [ ] 修改 `configs/neptune.yaml`，增加 tracking 配置键。
- [ ] 修改 `configs/neptune_stacking.yaml`，增加 tracking 配置键。
- [ ] 修改 `configs/neptune_selected_features_v1.yaml`，增加 tracking 配置键。
- [ ] 修改 `requirements.txt`，加入 `wandb`（PR-1 可暂留 `neptune-cli`）。
- [ ] 新增 `requirements-legacy.txt`，完整存档旧依赖版本。
- [ ] 修改 `README.md`，补充 W&B 快速开始并标注 Neptune 过时路径。

## 3) 验证与回归

- [ ] `tracking__mode=online` 下能正常创建 W&B run 并看到指标。
- [ ] `tracking__mode=disabled` 下可离线运行且不报错。
- [ ] 关键命令可跑通：`python main.py -- train_evaluate_cv --pipeline_name lightGBM -d`。
- [ ] 指标名与历史保持一致（便于前后实验对比）。
- [ ] 模型输出行为未变化（仅日志后端变化）。

## 4) 提交与合并

- [ ] 提交 1：`src/tracking/*` + 配置接入。
- [ ] 提交 2：`pipeline_manager/models/callbacks/main` 调用替换。
- [ ] 提交 3：`requirements/README/configs` 文档与依赖更新。
- [ ] PR 描述已明确“本 PR 不包含 steppy/Hamilton 与 Keras/PyTorch 迁移”。
- [ ] PR 附上工程行为验收结果（命令可跑通、产物结构正确、日志模式正确）。
