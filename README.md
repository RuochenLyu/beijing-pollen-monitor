# 北京花粉监测 Agent Skill

让你的 AI Agent 每天自动查询北京花粉数据，生成包含所在区花粉浓度、24 小时趋势和全市概况的简报。

北京共 16 个固定监测站（每区一个），数据来自[北京花粉监测公共服务平台](https://pollenwechat.bjpws.com)。

## 输出示例

```
北京花粉日报（2026-03-30 08:00）

关注站点：海淀区
当前：28（高）
24h趋势：下降（-11），范围 8-42
建议：易引发过敏，加强防护，规范用药

全市概况：平均高
热点：石景山区 52(很高), 通州区 48(很高), 怀柔区 47(很高)
预警：32 项
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
- "每天早上 8 点给我发海淀的花粉简报"
- "对比一下各区花粉浓度"

Agent 会根据 SKILL.md 中的说明自动选择合适的查询模式。

### 定时播报

大多数 Agent 平台支持定时任务。例如在 Claude Code 中：

```
每天早上 8 点，用 beijing-pollen-monitor 查询海淀区的花粉简报，把结果发给我
```

Agent 会自动调用 `report` 模式并返回可读的花粉日报。

### 直接调用脚本（高级用法）

如果你需要在脚本或 CI 中直接调用：

```bash
# 每日简报（文本格式，可直接用于通知推送）
./scripts/beijing-pollen-query.sh query --mode report --district 海淀 --format text | jq -r '.data.text'

# 每日简报（JSON 格式，适合程序处理）
./scripts/beijing-pollen-query.sh query --mode report --district 海淀

# 全市概览
./scripts/beijing-pollen-query.sh query --mode overview

# 单站点 24 小时历史
./scripts/beijing-pollen-query.sh query --mode history --district 朝阳
```

Cron 定时推送示例：

```bash
# crontab -e
0 8 * * * /path/to/beijing-pollen-query.sh query --mode report --district 海淀 --format text | jq -r '.data.text' | your-notify-command
```

## 查询模式

| 模式 | 用途 | 必选参数 |
|---|---|---|
| `report` | 每日简报（推荐） | `--district <区名>` |
| `overview` | 全市概览 + 站点快照 | 无 |
| `stations` | 全部站点实时读数 | 无 |
| `history` | 单站点 24 小时趋势 | `--district <区名>` |

可用区名（支持模糊匹配，如"海淀"或"海淀区"均可）：

> 东城、西城、朝阳、海淀、丰台、石景山、门头沟、房山、通州、顺义、昌平、大兴、怀柔、平谷、密云、延庆

## 花粉等级说明

| 等级 | 浓度范围 | 建议 |
|---|---|---|
| 低 | ≤100 | 可正常户外活动 |
| 较低 | 101-250 | 高敏感人群注意防护 |
| 中等 | 251-400 | 过敏人群加强防护 |
| 高 | 401-800 | 加强防护，规范用药 |
| 很高 | >800 | 减少外出，持续用药 |

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
