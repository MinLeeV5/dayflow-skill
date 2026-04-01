---
name: dayflow-work-summary
description: 读取本机 Dayflow 的本地时间线数据（默认来自 `~/Library/Application Support/Dayflow/chunks.sqlite`），按指定日期、月份或自定义时间范围提取活动，并生成结构化中文工作总结。适用于：用户要求总结某天/某周/某月的 Dayflow 工作内容、基于 Dayflow 生成日报周报月报、回看某一天做了什么、或排查如何读取 Dayflow 本地数据。
---

# Dayflow 工作总结

读取 Dayflow 本地 SQLite 时间线，归一化输出为 JSON，并按固定模板生成中文工作总结。优先使用内置脚本而不是临时拼接 SQL，这样在 Dayflow 正在运行时也能保持读取链路稳定。

## 快速开始

1. 选择时间范围：
   - 单日：`--date YYYY-MM-DD`
   - 整月：`--month YYYY-MM`
   - 自定义范围：`--from YYYY-MM-DD --to YYYY-MM-DD`
2. 运行读取脚本：
   ```bash
   python3 scripts/read_dayflow.py --date 2026-04-01
   ```
   当高层卡片信息不够时，加上 `--include-details`。
3. 按 `references/report-format.md` 中的结构输出最终中文总结。
4. 区分事实与推断；如果某个字段不能被 Dayflow 数据直接支撑，要明确写出这一点。

## 工作流

### 1. 提取 Dayflow 数据

- 常规场景一律使用 `scripts/read_dayflow.py`，不要每次手写 SQL。
- 脚本会以只读方式打开活跃数据库，先做 SQLite 快照备份，再查询快照，避免 Dayflow 打开时的 WAL/锁问题。
- 主数据源表：`timeline_cards`
- 次数据源表：`journal_entries`
- 需要更细证据时使用 `--include-details`；只有在需要分心记录、站点线索、上下文切换时才使用 `--include-metadata`

### 2. 解释数据

- 将每一条 timeline card 视为一个连续工作块，包含 `title`、`summary`、时长、分类，以及可选的详细摘要。
- 当 `journal_entries` 中存在 `intentions`、`goals`、`reflections`、`summary` 时，优先用它们支撑“目标/反思”。
- `工时/D` 必须使用脚本统计结果，并始终按 `D = total_hours / 8` 计算。
- 月报场景要先聚合同类主题，再写总结；不要机械罗列每一张卡片。
- 如果请求范围内没有卡片，要明确说明 Dayflow 在该时间段没有记录到时间线活动。

### 3. 撰写总结

- 严格遵循 `references/report-format.md` 中的章节顺序。
- 优先使用保守且可追溯的表达：
  - `根据 Dayflow 记录，...`
  - `从本月多次出现的工作块推测，...`
  - `未从 Dayflow 数据中看到明确延期证据。`
- 不要虚构交付物、指标、延期情况；证据不足时必须标记为推断。
- 某字段缺少来源数据时，保留该章节并说明限制，不要直接省略。

## 示例

- 汇总单日：
  ```bash
  python3 scripts/read_dayflow.py --date 2026-04-01
  ```
- 汇总整月：
  ```bash
  python3 scripts/read_dayflow.py --month 2026-03
  ```
- 拉取更细粒度区间并附带详情：
  ```bash
  python3 scripts/read_dayflow.py --from 2026-03-29 --to 2026-03-31 --include-details --include-metadata
  ```

## 何时继续下钻

- 需要精确到某个短时间窗口的屏幕行为：先用 `--include-details` 重跑；如果还不够，再按 `references/dayflow-data.md` 的说明查看 `observations`
- 需要排查为什么没有数据：先确认数据库路径，再确认目标日期是否存在 `timeline_cards`
- 需要更正式的月报：使用同一份提取 JSON，按主题、交付物、风险聚合输出，而不是按时间戳流水账

## 参考资料

- 在读取或排障前，先看 `references/dayflow-data.md`
- 在生成最终总结前，先看 `references/report-format.md`
