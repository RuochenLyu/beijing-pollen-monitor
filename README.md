# 北京花粉监测 Agent Skill

让你的 AI Agent 每天自动查询北京花粉实时数据和预报，生成包含所在区当前监测、24 小时变化、今天风险和后续趋势的结果。

北京共 16 个固定监测站（每区一个），数据来自[北京花粉监测公共服务平台](https://pollenwechat.bjpws.com)。

## 输出示例

```
朝阳区花粉晨报（2026-04-02 17:13）

当前监测：29（高）
较昨日同一时段：下降（-19），近24h范围 14-48
今天风险：4月2日08时 很高
未来趋势：4月2日08时 很高，4月3日08时 高，4月4日08时 很高
相对当前：今天总量预报等级高于当前站点监测等级。
后续变化：明天总量预报低于今天。
再往后：后天总量预报高于明天。
注：监测值和区级预报不是同一数值体系，只比较等级变化。
分类提示：柏科 柏科花粉浓度指数中等，花粉致敏性中等，一般过敏人群可以无碍出行，重度敏感人群需要做好适当防护
建议：易引发过敏，加强防护，规范用药
```

## 安装

### 一键安装（推荐）

支持 Claude Code、Codex、OpenClaw、Cursor、Cline 等 40+ Agent 平台：

```bash
npx skills add RuochenLyu/beijing-pollen-monitor
```

安装工具会自动识别你的 Agent 并放到正确的位置。也可以指定 Agent：`npx skills add RuochenLyu/beijing-pollen-monitor -a claude-code`

### ClawHub

```bash
npx clawhub@latest install beijing-pollen-monitor
```

### 手动安装

将 `skills/beijing-pollen-monitor` 目录复制到你使用的 Agent 的 skills 路径下：

| Agent | 路径 |
|---|---|
| Claude Code | `~/.claude/skills/` 或项目下 `.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| 其他 Agent | 参考该 Agent 的 skills 目录 |

```bash
# 示例：安装到 Claude Code
cp -r skills/beijing-pollen-monitor ~/.claude/skills/
```

### 运行依赖

需要系统安装 `curl` 和 `jq`：

```bash
# macOS
brew install jq

# Ubuntu / Debian
sudo apt install jq
```

## 使用方式

### 让 Agent 自动调用（推荐）

安装后无需手动操作。直接用自然语言告诉 Agent：

- "查一下朝阳区今天花粉怎么样"
- "看一下海淀区明天花粉预报"
- "每天早上 8 点给我发海淀区今天的花粉情况"
- "对比一下各区花粉浓度"

Agent 会根据 SKILL.md 中的说明自动选择合适的查询模式。

### 定时播报

大多数 Agent 平台支持定时任务。例如在 Claude Code 中：

```
每天早上 8 点，用 beijing-pollen-monitor 查询朝阳区今天的花粉情况，把结果发给我
```

Agent 更适合自动调用 `daily` 模式并返回可读的花粉晨报。

### 直接调用脚本（高级用法）

如果你需要在脚本或 CI 中直接调用：

```bash
# 每日晨报（文本格式，可直接用于通知推送）
./scripts/beijing-pollen-query.sh query --mode daily --district 朝阳 --format text | jq -r '.data.text'

# 每日晨报（JSON 格式，适合程序处理）
./scripts/beijing-pollen-query.sh query --mode daily --district 朝阳

# 即时查询当前监测 + 全市概况
./scripts/beijing-pollen-query.sh query --mode report --district 海淀

# 全市概览
./scripts/beijing-pollen-query.sh query --mode overview

# 单站点 24 小时历史
./scripts/beijing-pollen-query.sh query --mode history --district 朝阳

# 目标区花粉预报
./scripts/beijing-pollen-query.sh query --mode forecast --district 海淀
```

Cron 定时推送示例：

```bash
# crontab -e
0 8 * * * /path/to/beijing-pollen-query.sh query --mode daily --district 朝阳 --format text | jq -r '.data.text' | your-notify-command
```

## 查询模式

| 模式 | 用途 | 必选参数 |
|---|---|---|
| `daily` | 默认推荐：定时晨报 + 今天风险 + 后续趋势 | `--district <区名>` |
| `report` | 即时查询：当前监测 + 24h变化 + 全市概况 | `--district <区名>` |
| `overview` | 全市概览 + 站点快照 | 无 |
| `stations` | 全部站点实时读数 | 无 |
| `history` | 单站点 24 小时趋势 | `--district <区名>` |
| `forecast` | 目标区总量预报 + 分类预报 | `--district <区名>` |

可用区名（支持模糊匹配，如"海淀"或"海淀区"均可）：

> 东城、西城、朝阳、海淀、丰台、石景山、门头沟、房山、通州、顺义、昌平、大兴、怀柔、平谷、密云、延庆

## 数值说明

不同接口的数值不是同一套语义，skill 会统一补充 `value_family` 和 `meaning`：

| value_family | 来源 | 含义 |
|---|---|---|
| `area_realtime` | `weatherPollen/pollens` | 北京区级浓度值，按 `0-100 / 101-250 / 251-400 / 401-800 / >800` 解释 |
| `station_observation` | `latestPollenLevels` / `history24` | 站点观测值，等级以接口返回的 `hfHLv/level` 为准，不直接套用区级阈值 |
| `total_forecast` | `v1/pollen/forecast` | 北京区级总量等级预报，表示风险等级，不是实时浓度值 |
| `classify_forecast` | `v2/pollen/classify/forecast` | 分类花粉预报，优先使用上游 `description` 解释 |

## 项目结构

```
skills/beijing-pollen-monitor/
├── SKILL.md                       # Skill 元数据（Agent 读这个文件来理解如何调用）
├── agents/
│   └── openai.yaml                # OpenAI 兼容接口配置
├── scripts/
│   └── beijing-pollen-query.sh    # 查询脚本（唯一入口）
└── references/
    └── api-contract.md            # 完整 API 约定（JSON 结构、退出码、字段说明）
```

## 许可

MIT
