# Notebook 策略（Notebooks Policy）

## 1. 当前阶段策略

1. 所有现有 notebook 先完整存档，不做功能性改写。
2. notebook 不作为重构阶段的核心验收对象。
3. 先保证主代码路径重构完成，再启动 notebook 重构。

## 2. 存档规则

1. 保持原 notebook 文件名与目录结构。
2. 增加存档说明文件（建议：`notebooks/README-legacy.md`）。
3. 若需移动，建议迁到 `notebooks/legacy/` 并保留索引清单。

## 3. 与新代码的关系

1. 当前阶段不强制 notebook 与新代码立即兼容。
2. notebook 后续重构时，统一改为调用新代码入口。
3. 不在 notebook 中复制核心训练逻辑，核心逻辑必须在 `src/`。

## 4. 后续重构原则（下一阶段）

1. 保留有业务价值的 notebook，淘汰重复或不可复现 notebook。
2. 每个保留 notebook 必须标注：
   - 适用数据版本
   - 依赖环境
   - 对应的新代码入口
3. 输出图表与分析结论应可复现。

## 5. 验收要求（当前阶段）

1. notebook 已完成归档登记。
2. notebook 文件未丢失、未破坏。
3. 主项目重构过程不被 notebook 依赖阻塞。
