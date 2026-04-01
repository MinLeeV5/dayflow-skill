# dayflow-skill

一个面向 Codex 的 Dayflow 数据采集 skill，只负责稳定读取本机 Dayflow 时间线数据并导出标准化 JSON，不负责生成工作总结。

## 能力边界

`dayflow-skill` 只做下面这些事：

- 读取 Dayflow 本地 SQLite 数据库
- 支持单日、整月、自定义日期范围
- 输出标准化 JSON，供其他 skill 或脚本继续处理
- 在 Dayflow 运行中的情况下，通过 SQLite 快照方式稳定读取数据
- 按需附带 `detailed_summary` 与 `metadata`

它**不负责**日报、周报、月报的总结生成；这部分会交给上层 skill（例如 `monthly-activity-skill`）。

## 仓库结构

```text
.
├── SKILL.md                    # Skill 主说明
├── README.md                   # 中文仓库说明
├── agents/openai.yaml          # Codex 界面元信息
├── scripts/read_dayflow.py     # Dayflow 数据读取脚本
└── references/dayflow-data.md  # Dayflow 本地数据结构说明
```

## 数据来源

默认读取以下本地路径：

- 数据目录：`~/Library/Application Support/Dayflow/`
- 主数据库：`~/Library/Application Support/Dayflow/chunks.sqlite`
- 录屏/截图目录：`~/Library/Application Support/Dayflow/recordings/`

主要数据表：

- `timeline_cards`：时间线主数据
- `journal_entries`：目标、反思、summary 的补充数据

## 使用方式

### 直接运行脚本

按天读取：

```bash
python3 scripts/read_dayflow.py --date 2026-04-01
```

按月读取：

```bash
python3 scripts/read_dayflow.py --month 2026-03
```

按范围读取并附带详细摘要：

```bash
python3 scripts/read_dayflow.py --from 2026-03-29 --to 2026-03-31 --include-details
```

如需带出 metadata：

```bash
python3 scripts/read_dayflow.py --month 2026-03 --include-details --include-metadata
```

### 作为 Codex skill 使用

把当前仓库复制或软链接到你的 Codex skills 目录，例如：

```bash
ln -s /Users/min/AI/dayflow-skill ~/.codex/skills/dayflow-skill
```

然后在 Codex 中使用：

```text
使用 $dayflow-skill 读取我 2026-03 的 Dayflow 数据
```

## 输出 JSON 结构

脚本输出的顶层字段包括：

- `source`
- `range`
- `generated_at`
- `aggregates`
- `cards`
- `journal_entries`

其中：

- `aggregates.total_hours` 和 `aggregates.total_person_days_8h` 可直接给上游总结脚本使用
- `cards` 保存逐条时间线卡片
- `journal_entries` 适合在需要目标/反思时作为补充证据

## 设计原则

- 优先稳定读取：先快照、再查询
- 优先标准化输出：让其他 skill 可以直接复用 JSON
- 优先事实数据：这里不做任何工作总结结论
- 优先可下钻：需要更多细节时再加 `--include-details` / `--include-metadata`

## 本地校验

```bash
python3 scripts/read_dayflow.py --date 2026-04-01
python3 scripts/read_dayflow.py --month 2026-03 --indent 0
```

如果要校验 skill 结构，可使用 `skill-creator` 自带的 `quick_validate.py`。
