# dayflow-skill

一个面向 Codex 的 Dayflow 工作总结技能，用来读取本机 Dayflow 数据，并按固定模板生成中文日报、周报、月报或指定日期范围总结。

## 能力概览

- 读取 Dayflow 本地 SQLite 数据库
- 支持单日、整月、自定义日期范围
- 输出标准化 JSON，便于二次加工
- 固化工作总结模板，覆盖目标、成果、行动、完成情况、质量、工时、人天、难易度、备注
- 支持按需下钻 `detailed_summary` 和 `metadata`

## 仓库结构

```text
.
├── SKILL.md                    # 技能主说明
├── agents/openai.yaml          # Codex 界面元信息
├── scripts/read_dayflow.py     # Dayflow 数据读取脚本
├── references/dayflow-data.md  # Dayflow 本地数据结构说明
└── references/report-format.md # 固定工作总结模板
```

## Dayflow 数据来源

默认读取以下本地路径：

- 数据目录：`~/Library/Application Support/Dayflow/`
- 主数据库：`~/Library/Application Support/Dayflow/chunks.sqlite`
- 录屏/截图目录：`~/Library/Application Support/Dayflow/recordings/`

主表是 `timeline_cards`，补充表是 `journal_entries`。

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

按范围读取并附带详细信息：

```bash
python3 scripts/read_dayflow.py --from 2026-03-29 --to 2026-03-31 --include-details --include-metadata
```

### 作为 Codex 技能使用

把当前仓库复制或软链接到你的 Codex skills 目录，例如：

```bash
ln -s /绝对路径/dayflow-skill ~/.codex/skills/dayflow-work-summary
```

然后在 Codex 中使用：

```text
使用 $dayflow-work-summary 总结我 2026-03 的月度工作内容
```

## 输出模板

固定输出结构见 `references/report-format.md`，包含以下章节：

- 目标（含组织与个人）
- 关键成果（交付物/数据）
- 关键行动举措（对齐组织目标拆解，需体现延期情况）
- 完成情况（成效、问题、风险、措施）
- 工作质量
- 工时/D（具体人天）
- 自评难易（难/中/易）
- 备注

## 设计原则

- 优先稳定读取：脚本会对活跃数据库先做只读快照再查询
- 优先事实：不虚构交付物、指标或延期
- 推断可说，但必须显式标注为推断
- 月报先聚合主题，再写总结，不做流水账

## 本地校验

可直接运行：

```bash
python3 scripts/read_dayflow.py --date 2026-04-01
python3 scripts/read_dayflow.py --month 2026-03 --indent 0
```

如果需要校验技能结构，可使用 `skill-creator` 的校验脚本；若本机缺少 `PyYAML`，先安装后再执行。
