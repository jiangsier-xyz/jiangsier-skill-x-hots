---
name: x-hots
version: 1.4.1
description: Fetch trending topics from X (Twitter). Analyze user interests from natural language, retrieve top hot topics per domain, and summarize background, key opinions, and controversies.
---

# X (Twitter) Hot Topics Fetcher

Fetch real-time trending topics from X, filter by user interest domains, and analyze the background, popular opinions, and key controversies of each topic.

## Prerequisites

### Dependencies

- **Python 3.7+**
- **tweepy** library

Install tweepy:
```bash
pip3 install tweepy
```

### Environment Variables

Only the **Bearer Token** is required. The script uses app-only (OAuth 2.0) auth, which is sufficient for the `search_recent_tweets` endpoint and has broader search access than OAuth 1.0a user credentials — so the consumer key/secret and access token/secret are not needed.

```bash
export X_BEARER_TOKEN="***"
```

Obtain this credential from the [X Developer Portal](https://developer.twitter.com/en/portal/dashboard).

**Note**: The Free tier API has limitations:
- Can only search tweets from the last 7 days
- Rate limited to 10 requests per 15 minutes for search endpoints

## When to Use

When the user wants to know what's trending on X/Twitter, or asks about hot topics, public opinion, or viral discussions.

## Workflow

### Step 1: Analyze User Interest Domains

Extract interest domains from the user's natural language input. Common domains include:

- **International Politics**: politics, geopolitics, diplomacy, sanctions, war, NATO, UN
- **Economy & Finance**: economy, stock market, tariff, trade, inflation, crypto, bitcoin
- **Technology**: AI, LLM, GPT, tech, chip, semiconductor, Apple, Google, Microsoft
- **Sports**: football, World Cup, NBA, Olympics, FIFA
- **Entertainment**: movie, music, celebrity, Netflix, Disney
- **Society**: climate, health, education, protest, human rights

**Analysis Rules**:
1. If the user explicitly mentions a domain (e.g., "what's hot in tech?"), use that domain's keywords directly
2. If the user mentions a specific topic (e.g., "what's happening with AI?"), use that topic as the keyword
3. If no specific domain can be identified, use the generic hot topics query (see "Generic Query" below)
4. Select **2-3 core keywords** per domain to avoid excessive queries

### Step 2: Run the Script to Fetch Hot Tweets

```bash
python3 scripts/x_hots.py '<json_queries>'
```

**Parameters**:
- `json_queries`: JSON array of search keywords
- Each keyword returns up to 10 related hot tweets

**Examples**:
```bash
# Tech + Economy domains
python3 scripts/x_hots.py '["AI", "LLM", "stock market", "tariff"]'

# International politics
python3 scripts/x_hots.py '["geopolitics", "diplomacy", "sanctions"]'

# Generic hot topics (when no domain can be identified)
python3 scripts/x_hots.py '["trending", "breaking", "viral"]'
```

**Output Format**:
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
          "metrics": {"like_count": 100, "retweet_count": 50},
          "url": "https://twitter.com/twitter/status/1234567890"
        }
      ]
    }
  }
}
```

### Step 3: Analyze and Present Results

For each domain/keyword, analyze the returned tweets and extract:

1. **Background**: The origin of the hot topic, timeline, key people/organizations involved
2. **Popular Opinions**: What are the mainstream views? Core arguments from supporters and opponents
3. **Key Controversies**: What is the focal point of the debate? Where do different stances diverge?

**Presentation Format**:

```markdown
## 🔥 [Domain Name] Hot Topics

### Topic 1: [Topic Title]

**📌 Background**
[Brief description of the event origin, timeline, key figures/organizations]

**💬 Popular Opinions**
- Supporters: [core argument]
- Opponents: [core argument]
- Neutral: [objective analysis]

**⚔️ Key Controversies**
[Focal point of the debate, divergence between different stances]

---

### Topic 2: [Topic Title]
...
```

## Generic Query (When No Domain Can Be Identified)

When the user's input cannot be mapped to a specific domain (e.g., "what's hot on X?"), use the following strategy:

1. Run the script with generic hot topic queries:
   ```bash
   python3 scripts/x_hots.py '["trending", "breaking", "viral", "news"]'
   ```

2. Identify the **Top 10 highest-engagement topics** from the returned tweets (based on likes, retweets, replies)

3. Present them ranked by engagement, with a brief background for each topic

## Examples

### Example 1: User Mentions a Specific Domain

**User input**: "What's hot in tech?"

**Analysis**: Domain = Tech, Keywords = ["AI", "LLM", "tech"]

**Execute**:
```bash
python3 scripts/x_hots.py '["AI", "LLM", "tech"]'
```

### Example 2: User Mentions a Specific Topic

**User input**: "What's happening with tariff policy?"

**Analysis**: Topic = Tariff, Keywords = ["tariff", "trade war", "tariff policy"]

**Execute**:
```bash
python3 scripts/x_hots.py '["tariff", "trade war", "tariff policy"]'
```

### Example 3: No Domain Can Be Identified

**User input**: "What's hot on X?"

**Analysis**: No specific domain identified, use generic query

**Execute**:
```bash
python3 scripts/x_hots.py '["trending", "breaking", "viral", "news"]'
```

## Notes

1. **API Limits**: Free-tier API can only search tweets from the last 7 days
2. **Language Filter**: Defaults to English tweets (`lang:en`); modify the script for other languages
3. **Deduplication**: Script already filters out retweets (`-is:retweet`), only returns original tweets
4. **Engagement Ranking**: Popularity is judged by likes, retweets, and replies combined
5. **Timeliness**: Results are sorted by relevancy, prioritizing popular/high-engagement tweets

## Error Handling

### API Authentication Failure
```json
{"success": false, "error": "Missing environment variable: X_BEARER_TOKEN"}
```
**Fix**: Ensure the Bearer Token environment variable is set in your shell profile (e.g. `~/.zshrc`):
```bash
export X_BEARER_TOKEN="***"
```
Obtain the credential from the [X Developer Portal](https://developer.twitter.com/en/portal/dashboard).

### No Results Found
```json
{"results": {"AI": {"success": true, "count": 0, "tweets": []}}}
```
**Action**: Inform the user that there are no hot topics in this domain currently; suggest trying different keywords

### Rate Limit Exceeded
```json
{"success": false, "error": "429 Too Many Requests"}
```
**Action**: Wait 15 minutes before retrying, or reduce the number of query keywords
