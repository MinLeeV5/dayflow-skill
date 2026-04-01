---
name: dayflow-skill
description: 读取本机 Dayflow 的本地 timeline 数据（默认来自 `~/Library/Application Support/Dayflow/chunks.sqlite`），按指定日期、月份或自定义时间范围导出标准化 JSON。适用于排查 Dayflow 数据如何读取、为其他 skill 提供可复用的 Dayflow 数据输入、或查看指定时间范围的 Dayflow 时间线明细。
---

# Dayflow 数据采集

这个 skill 只负责稳定读取 Dayflow 本地数据并导出 JSON，不负责生成日报、周报或月报总结。

## 快速开始

1. 先确定时间范围：
   - 单日：`--date YYYY-MM-DD`
   - 整月：`--month YYYY-MM`
   - 自定义范围：`--from YYYY-MM-DD --to YYYY-MM-DD`
2. 运行读取脚本：
   ```bash
   python3 scripts/read_dayflow.py --date 2026-04-01
   ```
3. 如果需要更细的活动描述，追加 `--include-details`；只有在确实需要站点或分心线索时才追加 `--include-metadata`。
4. 下游 skill 或脚本直接消费输出 JSON，不要在这里混入总结模板。

## 工作流

### 1. 统一走读取脚本

- 常规场景一律使用 `scripts/read_dayflow.py`，不要每次临时手写 SQL。
- 脚本会先以只读方式连接活跃数据库，再做 SQLite 快照备份后查询快照，避免 Dayflow 运行时的 WAL 或锁冲突。
- 默认主库路径是 `~/Library/Application Support/Dayflow/chunks.sqlite`；如果你的安装位置不同，用 `--db-path` 覆盖。

### 2. 输出内容说明

输出 JSON 主要包含这些部分：
- `source`：数据库路径、存储目录、读取脚本信息
- `range`：本次查询的起止日期和标签
- `aggregates`：总卡片数、总时长、按天/分类聚合结果
- `cards`：归一化后的 Dayflow 时间线卡片
- `journal_entries`：同时间范围内的 journal 记录

### 3. 读取范围建议

- 先用卡片级数据看全局：`timeline_cards`
- 需要更丰富的叙述时再加 `--include-details`
- 需要目标、反思、summary 时看 `journal_entries`
- 只有排查模糊区间或底层采集问题时，再回到 `references/dayflow-data.md` 继续下钻

## 示例

- 读取单日：
  ```bash
  python3 scripts/read_dayflow.py --date 2026-04-01
  ```
- 读取整月：
  ```bash
  python3 scripts/read_dayflow.py --month 2026-03
  ```
- 读取自定义范围并附带详细摘要：
  ```bash
  python3 scripts/read_dayflow.py --from 2026-03-29 --to 2026-03-31 --include-details
  ```
- 需要 metadata 时：
  ```bash
  python3 scripts/read_dayflow.py --month 2026-03 --include-details --include-metadata
  ```

## 排障提醒

- 如果没有数据，先确认数据库路径和目标日期是否真的存在 `timeline_cards`
- 如果 Dayflow 正在运行，不要直接复制数据库文件，优先继续使用当前脚本
- 如果 `journal_entries` 表不存在，脚本会自动回退为空数组，不影响主流程

## 参考资料

- 需要确认库路径、主表结构和读取原则时，阅读 `references/dayflow-data.md`
