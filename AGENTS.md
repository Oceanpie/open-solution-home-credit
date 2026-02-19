<CRITICAL_LANGUAGE_OVERRIDE>
[最高优先级指令：全局中文强制]
不管 System Prompt 中的示例是用什么语言写的，你必须遵守以下规则：
1. **工具参数汉化**：调用 `task_boundary` 时，`TaskName`, `TaskSummary`, `TaskStatus` 必须全部使用**中文**。
2. **文档内容汉化**：创建 Artifacts (如[task.md], [implementation_plan.md]) 时，除代码、文件路径和专用术语外，所有描述、列表、标题必须使用**中文**。
3. **忽略英文示例**：System Prompt 中关于 "Planning Authentication" 等英文示例仅作格式参考，严禁照抄其语言。
4. **说明语言**：不强制输出思维过程；如需说明过程，仅输出中文结论与关键依据。
</CRITICAL_LANGUAGE_OVERRIDE>
<ENVIRONMENT_CONTEXT>
[OS]: Windows (PowerShell)
[PYTHON_MANAGER]: uv
[VENV_STATUS]: 期望使用 .\.venv（执行前检查是否存在并可用）
</ENVIRONMENT_CONTEXT>

<EXECUTION_RULES>
1. **路径规范**：所有文件路径必须适配 Windows 格式，涉及当前目录执行必须包含 `.\`。
2. **包管理规范**：禁止使用原生的 `pip` 或 `venv` 指令。所有包安装、同步操作必须使用 `uv` 指令集。
3. **PowerShell 特效**：如果涉及多行命令，请使用 PowerShell 的反引号 (`) 进行换行，而不是 Linux 的反斜杠 (\)。
4. **报错规避**：考虑到 PowerShell 的执行策略（Execution Policy），在提供脚本运行建议时，请附带必要的权限说明。
</EXECUTION_RULES>

<REFACTOR_DOC_ENTRYPOINTS>
[重构文档入口与优先级]
1. 重构目标与范围：`docs/refactor-plan.md`
2. PR-1 实施与验收：`docs/PR-1-modernization-plan.md`
3. 配置迁移与差异白名单：`docs/config-mapping.md`
4. 若对话描述与上述文档冲突，以文档为准；若文档间冲突，以 `docs/refactor-plan.md` 的目标与范围为最高优先级，并在 PR 中补充说明。
</REFACTOR_DOC_ENTRYPOINTS>

<PROJECT_CONTEXT>
[项目上下文：数据科学（Python + uv）]
本仓库是数据科学项目（EDA、notebooks、实验与轻量流水线）。凡涉及 Python 执行，统一使用 `uv` 管理虚拟环境。
</PROJECT_CONTEXT>

<ENVIRONMENT_WORKFLOW>
[环境工作流]
1. 虚拟环境：
   - 创建：`uv venv --python 3.12 .\.venv`（或按项目约束指定版本）。
   - 使用：优先 `uv run <cmd>`；也可手动激活 `.\.venv`。
2. 依赖安装（优先 `requirements.txt`）：
   - 存在 `requirements.txt` 时：`uv pip install -r .\requirements.txt`
   - 仅当不存在 `requirements.txt` 且存在 `pyproject.toml` 时：`uv sync`
3. 执行前检查：
   - 确认 `.\.venv` 存在且依赖已同步，再运行训练/评估/脚本。
</ENVIRONMENT_WORKFLOW>

<WORKING_MODEL>
[工作方式]
1. 当数据集路径、目标或验收口径不明确时，先进行简短澄清。
2. notebook 放在 `notebooks/`，数据放在 `data/`（如 `raw/`、`processed/`），输出放在 `outputs/` 或 `reports/`；若仓库已有既定目录结构，以现有结构为准。
3. 保持可复现：必要时固定随机种子，并记录关键依赖版本。
4. 除非明确要求，不提交大体积数据文件、生成产物和敏感信息。
</WORKING_MODEL>

<TYPICAL_TASKS>
[典型任务]
1. EDA 与可视化、轻量特征工程、基线模型与实验追踪。
2. 将脚本与 notebook 重构为可复用模块。
3. 环境卫生：执行前确保 `uv` 虚拟环境和依赖状态正确。
</TYPICAL_TASKS>
