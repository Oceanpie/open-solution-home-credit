# 配置迁移对照表（Config Mapping）

## 1. 用途

本文件用于记录“旧配置键 -> 新配置键”的迁移关系，确保重构过程中配置语义不丢失。

状态说明：

1. `done`：已确定并已落地
2. `planned`：已确定方向，待代码落地
3. `deprecated`：旧键废弃，不再支持
4. `tbd`：待确认

## 2. 迁移总规则

1. 新实现不再依赖 Neptune 注入参数。
2. 新实现以本地配置文件为唯一参数来源。
3. 旧配置文件保留存档，不作为新运行入口。
4. 目标态：新配置采用分层结构（点号分隔）。过渡阶段（PR-1）在现有 YAML 中新增键仍沿用 `xxx__yyy` 风格以保持同文件内一致性，待配置重构 PR 统一切换。

## 3. 全局约束（已确定）

| 项目         | 取值/策略                              | 状态    |
| ------------ | -------------------------------------- | ------- |
| Python       | `3.12`                                 | done    |
| 依赖入口     | `requirements.txt`                     | done    |
| 旧依赖存档   | 新增 `requirements-legacy.txt`（必须） | planned |
| 旧代码目录   | `src/legacy`                           | done    |
| 旧运行策略   | 不再使用（仅存档）                     | done    |
| 实验追踪     | W&B                                    | done    |
| 追踪默认模式 | `tracking.mode=offline`                | done    |
| 在线上传     | `tracking.mode=online` 时允许上传      | done    |
| 代码检查     | `ruff`                                 | done    |
| CI           | 暂置空                                 | done    |
| notebooks    | 先存档，后续重构                       | done    |

## 4. 键映射（已开始填写）

### 4.1 顶层元信息（Neptune 专属）

| 旧键             | 新键                           | 迁移说明                           | 状态    |
| ---------------- | ------------------------------ | ---------------------------------- | ------- |
| `project`        | `tracking.project`             | 原 Neptune 项目名迁移为 W&B 项目名 | planned |
| `name`           | `runtime.run_name`             | 运行名改为运行时配置               | planned |
| `tags`           | `tracking.tags`                | 保留语义                           | planned |
| `metric.channel` | `tracking.primary_metric.name` | 指标定义迁移到 tracking 元信息     | planned |
| `metric.goal`    | `tracking.primary_metric.goal` | 保留 maximize/minimize 语义        | planned |
| `exclude`        | `runtime.artifact_exclude`     | 作为产物上传过滤规则               | planned |

### 4.2 Tracking（新体系）

| 旧键           | 新键               | 迁移说明                  | 状态    |
| -------------- | ------------------ | ------------------------- | ------- |
| 无             | `tracking.backend` | 新增，固定 `wandb`        | done    |
| 无             | `tracking.mode`    | 新增，默认 `offline`      | done    |
| 无             | `tracking.entity`  | 新增，可为空              | planned |
| 无             | `tracking.project` | 新增，项目命名不变        | planned |
| 无             | `tracking.tags`    | 新增列表                  | planned |
| `callbacks_on` | `tracking.enabled` | 统一走 tracking 开关/模式 | planned |

### 4.3 Data 路径

| 旧键（`parameters`）              | 新键                                   | 迁移说明             | 状态    |
| --------------------------------- | -------------------------------------- | -------------------- | ------- |
| `train_filepath`                  | `data.train_filepath`                  | 直接迁移             | planned |
| `test_filepath`                   | `data.test_filepath`                   | 直接迁移             | planned |
| `sample_submission_filepath`      | `data.sample_submission_filepath`      | 直接迁移             | planned |
| `first_level_oof_predictions_dir` | `data.first_level_oof_predictions_dir` | 二级模型输入目录     | planned |
| `experiment_directory`            | `runtime.experiment_directory`         | 运行输出目录         | planned |
| `bureau_filepath`                 | `data.sources.bureau`                  | 来源表路径归组       | planned |
| `bureau_balance_filepath`         | `data.sources.bureau_balance`          | 来源表路径归组       | planned |
| `credit_card_balance_filepath`    | `data.sources.credit_card_balance`     | 来源表路径归组       | planned |
| `installments_payments_filepath`  | `data.sources.installments_payments`   | 来源表路径归组       | planned |
| `POS_CASH_balance_filepath`       | `data.sources.pos_cash_balance`        | 名称规范化为小写蛇形 | planned |
| `previous_application_filepath`   | `data.sources.previous_application`    | 来源表路径归组       | planned |

### 4.4 Kaggle 提交

| 旧键             | 新键                        | 迁移说明         | 状态    |
| ---------------- | --------------------------- | ---------------- | ------- |
| `kaggle_api`     | `submission.kaggle.enabled` | 布尔开关语义保持 | planned |
| `kaggle_message` | `submission.kaggle.message` | 直接迁移         | planned |

### 4.5 Runtime / CV / 调参

| 旧键                                         | 新键                                                 | 迁移说明              | 状态    |
| -------------------------------------------- | ---------------------------------------------------- | --------------------- | ------- |
| `n_cv_splits`                                | `cv.n_splits`                                        | 直接迁移              | planned |
| `validation_size`                            | `cv.validation_size`                                 | 保留 holdout 比例配置 | planned |
| `stratified_cv`                              | `cv.stratified`                                      | 命名规范化            | planned |
| `shuffle`                                    | `cv.shuffle`                                         | 直接迁移              | planned |
| `clean_experiment_directory_before_training` | `runtime.clean_experiment_directory_before_training` | 运行清理策略          | planned |
| `num_workers`                                | `runtime.num_workers`                                | 线程/并发设置         | planned |
| `verbose`                                    | `runtime.verbose`                                    | 日志级别/输出详细程度 | planned |
| `hyperparameter_search__method`              | `tuning.method`                                      | 前缀转分层            | planned |
| `hyperparameter_search__runs`                | `tuning.runs`                                        | 前缀转分层            | planned |

### 4.6 预处理与特征工程

| 旧键                                               | 新键                                                       | 迁移说明   | 状态    |
| -------------------------------------------------- | ---------------------------------------------------------- | ---------- | ------- |
| `fill_missing`                                     | `preprocessing.fill_missing`                               | 直接迁移   | planned |
| `fill_value`                                       | `preprocessing.fill_value`                                 | 直接迁移   | planned |
| `use_nan_count`                                    | `features.use_nan_count`                                   | 直接迁移   | planned |
| `application_aggregation__use_diffs_only`          | `features.application_aggregation.use_diffs_only`          | 前缀转分层 | planned |
| `installments__last_k_trend_periods`               | `features.installments.last_k_trend_periods`               | 前缀转分层 | planned |
| `installments__last_k_agg_periods`                 | `features.installments.last_k_agg_periods`                 | 前缀转分层 | planned |
| `installments__last_k_agg_period_fractions`        | `features.installments.last_k_agg_period_fractions`        | 前缀转分层 | planned |
| `pos_cash__last_k_trend_periods`                   | `features.pos_cash.last_k_trend_periods`                   | 前缀转分层 | planned |
| `pos_cash__last_k_agg_periods`                     | `features.pos_cash.last_k_agg_periods`                     | 前缀转分层 | planned |
| `pos_cash__last_k_agg_period_fractions`            | `features.pos_cash.last_k_agg_period_fractions`            | 前缀转分层 | planned |
| `bureau__last_k_agg_periods`                       | `features.bureau.last_k_agg_periods`                       | 前缀转分层 | planned |
| `bureau__last_k_agg_period_fractions`              | `features.bureau.last_k_agg_period_fractions`              | 前缀转分层 | planned |
| `bureau_balance__last_k_trend_periods`             | `features.bureau_balance.last_k_trend_periods`             | 前缀转分层 | planned |
| `bureau_balance__last_k_agg_periods`               | `features.bureau_balance.last_k_agg_periods`               | 前缀转分层 | planned |
| `bureau_balance__last_k_agg_period_fractions`      | `features.bureau_balance.last_k_agg_period_fractions`      | 前缀转分层 | planned |
| `previous_applications__last_k_credits`            | `features.previous_applications.last_k_credits`            | 前缀转分层 | planned |
| `application_previous_application__last_k_credits` | `features.application_previous_application.last_k_credits` | 前缀转分层 | planned |

### 4.7 特征选择开关

| 旧键（统一前缀 `use_`）                                     | 新键                                                                          | 迁移说明 | 状态    |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------- | -------- | ------- |
| `use_application`                                           | `feature_selection.use_application`                                           | 直接迁移 | planned |
| `use_bureau`                                                | `feature_selection.use_bureau`                                                | 直接迁移 | planned |
| `use_bureau_balance`                                        | `feature_selection.use_bureau_balance`                                        | 直接迁移 | planned |
| `use_credit_card_balance`                                   | `feature_selection.use_credit_card_balance`                                   | 直接迁移 | planned |
| `use_installments_payments`                                 | `feature_selection.use_installments_payments`                                 | 直接迁移 | planned |
| `use_pos_cash_balance`                                      | `feature_selection.use_pos_cash_balance`                                      | 直接迁移 | planned |
| `use_previous_applications`                                 | `feature_selection.use_previous_applications`                                 | 直接迁移 | planned |
| `use_application_previous_applications`                     | `feature_selection.use_application_previous_applications`                     | 直接迁移 | planned |
| `use_application_aggregations`                              | `feature_selection.use_application_aggregations`                              | 直接迁移 | planned |
| `use_bureau_aggregations`                                   | `feature_selection.use_bureau_aggregations`                                   | 直接迁移 | planned |
| `use_credit_card_balance_aggregations`                      | `feature_selection.use_credit_card_balance_aggregations`                      | 直接迁移 | planned |
| `use_installments_payments_aggregations`                    | `feature_selection.use_installments_payments_aggregations`                    | 直接迁移 | planned |
| `use_pos_cash_balance_aggregations`                         | `feature_selection.use_pos_cash_balance_aggregations`                         | 直接迁移 | planned |
| `use_previous_applications_aggregations`                    | `feature_selection.use_previous_applications_aggregations`                    | 直接迁移 | planned |
| `use_application_categorical_features`                      | `feature_selection.use_application_categorical_features`                      | 直接迁移 | planned |
| `use_previous_application_categorical_features`             | `feature_selection.use_previous_application_categorical_features`             | 直接迁移 | planned |
| `use_application_previous_application_categorical_features` | `feature_selection.use_application_previous_application_categorical_features` | 直接迁移 | planned |

### 4.8 模型参数（前缀规则）

| 旧键模式      | 新键模式                  | 迁移说明           | 状态    |
| ------------- | ------------------------- | ------------------ | ------- |
| `lgbm__*`     | `models.lightgbm.*`       | 双下划线前缀转分层 | planned |
| `xgb__*`      | `models.xgboost.*`        | 双下划线前缀转分层 | planned |
| `catboost__*` | `models.catboost.*`       | 双下划线前缀转分层 | planned |
| `rf__*`       | `models.random_forest.*`  | 双下划线前缀转分层 | planned |
| `lr__*`       | `models.log_reg.*`        | 双下划线前缀转分层 | planned |
| `nb__*`       | `models.naive_bayes.*`    | 双下划线前缀转分层 | planned |
| `svc__*`      | `models.svc.*`            | 双下划线前缀转分层 | planned |
| `nn__*`       | `models.neural_network.*` | 双下划线前缀转分层 | planned |

神经网络键差异说明：

1. `neptune.yaml` 与 `neptune_stacking.yaml` 使用 `nn__layers/nn__neurons/...`。
2. `neptune_selected_features_v1.yaml` 使用 `nn__input_shape/nn__learning_rate/nn__epochs`。
3. 新配置统一到 `models.neural_network.*`，并在读取时做向后兼容映射。

配置文件×模型前缀覆盖白名单：

| 模型前缀      | `neptune.yaml` | `neptune_stacking.yaml` | `neptune_selected_features_v1.yaml` | 差异说明                                                         | 状态        |
| ------------- | :------------: | :---------------------: | :---------------------------------: | ---------------------------------------------------------------- | ----------- |
| `lgbm__*`     |       ✅        |            ✅            |                  ✅                  | —                                                                | intentional |
| `catboost__*` |       ✅        |            ✅            |                  ✅                  | —                                                                | intentional |
| `xgb__*`      |       ✅        |            ✅            |                  ✅                  | —                                                                | intentional |
| `nn__*`       |       ✅        |            ✅            |                  ✅                  | 键集不同，见上方说明                                             | intentional |
| `rf__*`       |       ✅        |            ✅            |                  ✅                  | `selected_features_v1` 无 `rf__max_depth`，`max_leaf_nodes=None` | intentional |
| `lr__*`       |       ✅        |            ✅            |                  ✅                  | —                                                                | intentional |
| `nb__*`       |       ✅        |            ❌            |                  ✅                  | stacking 场景不使用 NaiveBayes                                   | intentional |
| `svc__*`      |       ✅        |            ✅            |                  ✅                  | —                                                                | intentional |

不在白名单中的前缀 = 该配置场景不适用，迁移时无需处理。

### 4.9 后处理

| 旧键                 | 新键                                | 迁移说明 | 状态    |
| -------------------- | ----------------------------------- | -------- | ------- |
| `aggregation_method` | `postprocessing.aggregation_method` | 直接迁移 | planned |

## 5. 当前覆盖度

1. 已从 `configs/neptune.yaml`、`configs/neptune_stacking.yaml`、`configs/neptune_selected_features_v1.yaml` 提取键集。
2. 已完成主要键的首版映射（前缀规则 + 核心显式键）。
3. 下一步在对应 PR 中把 `planned` 逐项置为 `done`。

## 6. 后续动作

1. 新增 `config loader` 时同步引用本映射表。
2. 对无法迁移的旧键，明确标记 `deprecated` 并给出警告信息。
3. 每个 PR 更新本文件“状态列”和“变更记录”。

## 7. 变更记录

| 日期       | 变更内容                                        | 负责人 |
| ---------- | ----------------------------------------------- | ------ |
| 2026-02-14 | 初始化模板并填入已确认决策                      | Codex  |
| 2026-02-14 | 基于 `configs/neptune*.yaml` 完成首版键映射填充 | Codex  |
