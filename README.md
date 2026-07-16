# x-hots

An [OpenClaw](https://github.com/openclaw/openclaw) skill that fetches real-time trending topics from **X (Twitter)** and supports interest-domain-based analysis — background, popular opinions, and key controversies for each topic.

The skill is driven by an OpenClaw agent: it analyzes your natural-language input to pick search keywords, runs the bundled Python script to fetch hot tweets via the X API v2, then synthesizes a structured summary of what's trending and why.

[English](./README.md) | [简体中文](./README.zh-CN.md)

---

## ✨ Features

- **Interest-domain analysis** — describe what you care about in plain language ("what's hot in tech?"), and the skill maps it to relevant search keywords (politics, economy, technology, sports, entertainment, society, …).
- **Real-time trending tweets** — pulls recent high-engagement tweets from X using the official API v2.
- **Structured summaries** — for each topic: background, popular opinions (supporters / opponents / neutral), and key controversies.
- **Engagement ranking** — popularity judged by combined likes, retweets, and replies.
- **Clean signal** — filters out retweets and defaults to English tweets.

## 📦 Project Structure

```
x-hots/
├── SKILL.md            # Skill definition + the full agent workflow (the contract)
├── scripts/
│   └── x_hots.py       # Data-fetching implementation (Tweepy / X API v2)
├── LICENSE             # MIT
└── README.md
```

`SKILL.md` is the authoritative workflow specification — parameter contract, output schema, examples, and error handling. `scripts/x_hots.py` is the data-fetching step within that workflow.

## ✅ Prerequisites

- **Python 3.7+**
- **[tweepy](https://docs.tweepy.org/)** library
- An **X API Bearer Token** (app-only OAuth 2.0 — sufficient for `search_recent_tweets`)

Install tweepy:

```bash
pip3 install tweepy
```

Export your bearer token (obtain it from the [X Developer Portal](https://developer.twitter.com/en/portal/dashboard)):

```bash
export X_BEARER_TOKEN="your-token-here"
```

> **Free-tier limits:** tweets are searchable only from the last 7 days, and search endpoints are rate-limited to 10 requests / 15 minutes.

## 🚀 Usage

Run the script with a JSON array of search keywords. Each keyword returns up to 10 related hot tweets.

```bash
# Tech + Economy
python3 scripts/x_hots.py '["AI", "LLM", "stock market", "tariff"]'

# International politics
python3 scripts/x_hots.py '["geopolitics", "diplomacy", "sanctions"]'

# Generic hot topics (no specific domain)
python3 scripts/x_hots.py '["trending", "breaking", "viral", "news"]'
```

### Output Format

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
          "text": "Tweet content...",
          "created_at": "2026-07-09T03:50:00Z",
          "metrics": { "like_count": 100, "retweet_count": 50 },
          "url": "https://twitter.com/twitter/status/1234567890"
        }
      ]
    }
  }
}
```

The agent then turns this raw output into a per-topic summary:

```markdown
## 🔥 [Domain] Hot Topics

### Topic 1: [Title]
**📌 Background** — origin, timeline, key figures/organizations
**💬 Popular Opinions** — supporters / opponents / neutral
**⚔️ Key Controversies** — focal point of the debate
```

## ⚠️ Notes & Limitations

1. **API limits** — free tier searches only the last 7 days of tweets.
2. **Language filter** — defaults to English (`lang:en`); modify the script for other languages.
3. **Deduplication** — retweets are filtered out (`-is:retweet`); only original tweets are returned.
4. **Engagement ranking** — popularity is judged by likes, retweets, and replies combined.
5. **Timeliness** — results are sorted by relevancy, prioritizing high-engagement tweets.

## 🛠 Error Handling

| Scenario | Symptom | Fix |
|---|---|---|
| Missing token | `{"success": false, "error": "Missing environment variable: X_BEARER_TOKEN"}` | Export `X_BEARER_TOKEN` in your shell profile. |
| No results | `{"results": {"AI": {"success": true, "count": 0, "tweets": []}}}` | Try different keywords. |
| Rate limited | `{"success": false, "error": "429 Too Many Requests"}` | Wait 15 minutes, or reduce the number of keywords. |

## 📄 License

[MIT](./LICENSE) © 2026 jiangsier-xyz
