# x-hots

一个 [OpenClaw](https://github.com/openclaw/openclaw) 技能（Skill），用于从 **X（Twitter）** 获取实时热门话题，并按兴趣领域进行分析——梳理每个话题的背景、主流观点与核心争议。

该技能由 OpenClaw 智能体驱动：它会分析你的自然语言输入，挑选出合适的搜索关键词，运行内置的 Python 脚本通过 X API v2 抓取热门推文，再综合输出一份关于"什么正在热、为什么热"的结构化摘要。

[English](./README.md) | [简体中文](./README.zh-CN.md)

---

## ✨ 功能特性

- **兴趣领域分析** —— 用自然语言描述你关心的方向（例如"科技领域有什么热点？"），技能会将其映射到相关搜索关键词（政治、经济、科技、体育、娱乐、社会等）。
- **实时热门推文** —— 通过官方 X API v2 拉取近期高互动量推文。
- **结构化摘要** —— 针对每个话题输出：背景、主流观点（支持方 / 反对方 / 中立），以及核心争议。
- **互动量排序** —— 综合点赞、转发、回复数评估热度。
- **干净信号** —— 自动过滤转推，默认仅返回英文推文。

## 📦 项目结构

```
x-hots/
├── SKILL.md            # 技能定义 + 完整的智能体工作流（即契约说明）
├── scripts/
│   └── x_hots.py       # 数据抓取实现（基于 Tweepy / X API v2）
├── LICENSE             # MIT
└── README.zh-CN.md
```

`SKILL.md` 是权威的工作流规范——包括参数约定、输出格式、示例与错误处理。`scripts/x_hots.py` 是该工作流中的数据抓取步骤。

## ✅ 前置条件

- **Python 3.7+**
- **[tweepy](https://docs.tweepy.org/)** 库
- 一个 **X API Bearer Token**（应用级 OAuth 2.0，足以调用 `search_recent_tweets`）

安装 tweepy：

```bash
pip3 install tweepy
```

导出你的 Bearer Token（可在 [X 开发者门户](https://developer.twitter.com/en/portal/dashboard) 获取）：

```bash
export X_BEARER_TOKEN="your-token-here"
```

> **免费档限制：** 只能检索最近 7 天内的推文；搜索端点限流为每 15 分钟 10 次请求。

## 🚀 使用方法

运行脚本时传入一个 JSON 数组作为搜索关键词。每个关键词最多返回 10 条相关热门推文。

```bash
# 科技 + 经济
python3 scripts/x_hots.py '["AI", "LLM", "stock market", "tariff"]'

# 国际政治
python3 scripts/x_hots.py '["geopolitics", "diplomacy", "sanctions"]'

# 通用热点（无法识别具体领域时）
python3 scripts/x_hots.py '["trending", "breaking", "viral", "news"]'
```

### 输出格式

```json
{
  "success": true,
  "timestamp": "2026-07-09T03:58:00Z",
  "queries": ["AI", "LLM"],
  "results": {
    "AI": {
      "success": true,
      "count": 10,
      "tweets": [
        {
          "id": "1234567890",
          "text": "推文内容...",
          "created_at": "2026-07-09T03:50:00Z",
          "metrics": { "like_count": 100, "retweet_count": 50 },
          "url": "https://twitter.com/twitter/status/1234567890"
        }
      ]
    }
  }
}
```

随后，智能体会把这份原始输出整理成按话题汇总的摘要：

```markdown
## 🔥 [领域] 热门话题

### 话题 1：[标题]
**📌 背景** —— 事件起源、时间线、关键人物/机构
**💬 主流观点** —— 支持方 / 反对方 / 中立方
**⚔️ 核心争议** —— 争论焦点、不同立场分歧所在
```

## ⚠️ 说明与限制

1. **API 限制** —— 免费档仅能检索最近 7 天的推文。
2. **语言过滤** —— 默认仅返回英文推文（`lang:en`）；如需其他语言请修改脚本。
3. **去重** —— 已过滤转推（`-is:retweet`），仅返回原创推文。
4. **互动量排序** —— 综合点赞、转发、回复数评估热度。
5. **时效性** —— 结果按相关性排序，优先展示高互动量推文。

## 🛠 错误处理

| 场景 | 表现 | 解决办法 |
|---|---|---|
| 缺少 Token | `{"success": false, "error": "Missing environment variable: X_BEARER_TOKEN"}` | 在 shell 配置文件中导出 `X_BEARER_TOKEN`。 |
| 无结果 | `{"results": {"AI": {"success": true, "count": 0, "tweets": []}}}` | 换一组关键词重试。 |
| 触发限流 | `{"success": false, "error": "429 Too Many Requests"}` | 等待 15 分钟，或减少关键词数量。 |

## 📄 许可证

[MIT](./LICENSE) © 2026 jiangsier-xyz
