# Skill Example: Twitter Meta Agent

**Real skill from production:** Detects viral Twitter patterns using X's algorithm + Claude.

## Directory Structure

```
skills/twitter-meta-agent/
├── SKILL.md                    # Agent instructions
├── scripts/
│   ├── run_meta_agent.py      # Main detection script
│   ├── track_post.py          # Track individual posts
│   └── learn.py               # Learn from tracked results
├── .venv/                     # Python environment
├── requirements.txt
└── config.example.json
```

## SKILL.md

````markdown
# Twitter Meta Agent Skill

Detect viral Twitter patterns using X's open-source algorithm + Claude.

## Usage

### Run Detection (Heartbeat)

\```bash
cd ~/clawd/skills/twitter-meta-agent
python scripts/run_meta_agent.py
\```

**Frequency:** Daily via HEARTBEAT.md (max 2x/day)

### Track a Post

\```bash
python scripts/track_post.py "https://twitter.com/user/status/123"
\```

### Learn from Results

After tracking 5+ posts:
\```bash
python scripts/learn.py
\```

## Output

**Format:** Markdown report sent to Content Lab Telegram channel (-5286688748)

**Example:**
\```
## Twitter Meta (Mar 2026)

**Current Pattern:** Movie clip quote tweets dominating

**Mechanics:**
- 15-20s emotional reaction clips
- 1-3 lines provocative text
- Quote tweet format (not native)
- Peak engagement: 18:00-22:00 CET

**Example posts:** [3 links with metrics]

**Recommendation:** Test movie clip + hot take format this week
\```

## Cost

- ~$3/day ($2.50 X API + $0.40 Claude)
- 200-300 tweets analyzed per run

## Dependencies

- X API v2 (Bearer token)
- Anthropic API key
- Python 3.11+
- venv at `.venv`

## Installation

\```bash
cd ~/clawd/skills/twitter-meta-agent
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp config.example.json config.json
# Edit config.json with your API keys
\```

## Tracking State

\```json
{
  "last_run": "2026-03-19T14:23:00Z",
  "runs_today": 1,
  "tracked_posts": [
    {
      "url": "https://twitter.com/user/status/123",
      "tracked_at": "2026-03-19T10:00:00Z",
      "metrics": {
        "likes": 1200,
        "retweets": 340,
        "replies": 89
      }
    }
  ]
}
\```

## HEARTBEAT.md Integration

\```markdown
### Twitter Meta Detection

**Frequency:** Daily (max 2x/day, if last scan >12h ago)

**Tracking:** `memory/twitter-meta-tracking.json`

**Execute:**
1. If >12h since last scan → Run NOW
2. Post to Content Lab (-5286688748)
3. Update tracking file
\```

## Learning Loop

1. **Detect** meta pattern
2. **Track** 5-10 example posts
3. **Learn** from engagement data
4. **Adjust** detection criteria
5. Repeat weekly

````

## scripts/run_meta_agent.py (Simplified)

```python
#!/usr/bin/env python3
"""
Detect viral Twitter patterns using X API + Claude
"""
import os
import json
from datetime import datetime, timedelta
import anthropic
import tweepy

def get_tracking_state():
    try:
        with open('memory/twitter-meta-tracking.json') as f:
            return json.load(f)
    except FileNotFoundError:
        return {'last_run': None, 'runs_today': 0}

def should_run(state):
    if not state['last_run']:
        return True
    
    last_run = datetime.fromisoformat(state['last_run'])
    hours_since = (datetime.now() - last_run).total_seconds() / 3600
    
    return hours_since > 12 and state['runs_today'] < 2

def analyze_tweets(client, anthropic_client):
    # Fetch trending tweets
    tweets = client.search_recent_tweets(
        query="lang:en -is:retweet",
        max_results=100,
        tweet_fields=['public_metrics', 'created_at', 'entities']
    )
    
    # Prepare for Claude analysis
    tweet_data = [
        {
            'text': t.text,
            'likes': t.public_metrics['like_count'],
            'retweets': t.public_metrics['retweet_count'],
            'url': f"https://twitter.com/i/status/{t.id}"
        }
        for t in tweets.data
        if t.public_metrics['like_count'] > 1000
    ]
    
    # Ask Claude to detect patterns
    prompt = f"""Analyze these {len(tweet_data)} viral tweets and identify the dominant pattern:

{json.dumps(tweet_data, indent=2)}

What format/structure is working best this week? Be specific:
- Content type (text, video, quote tweet, etc.)
- Length and style
- Common themes
- Engagement mechanics

Provide 3-5 concrete examples with URLs."""

    response = anthropic_client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=2000,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.content[0].text

def main():
    state = get_tracking_state()
    
    if not should_run(state):
        print("⏭️  Skipping (ran recently)")
        return
    
    # Initialize APIs
    twitter_client = tweepy.Client(bearer_token=os.getenv('TWITTER_BEARER_TOKEN'))
    anthropic_client = anthropic.Anthropic(api_key=os.getenv('ANTHROPIC_API_KEY'))
    
    # Analyze
    report = analyze_tweets(twitter_client, anthropic_client)
    
    # Send to Content Lab channel
    os.system(f'''
        openclaw message send \
          --channel telegram \
          --target -5286688748 \
          --message "## Twitter Meta ({datetime.now().strftime('%b %Y')})\\n\\n{report}"
    ''')
    
    # Update tracking
    state['last_run'] = datetime.now().isoformat()
    state['runs_today'] += 1
    
    with open('memory/twitter-meta-tracking.json', 'w') as f:
        json.dump(state, f, indent=2)
    
    print(f"✅ Meta detection complete (${2.90:.2f})")

if __name__ == '__main__':
    main()
```

## Key Patterns

### 1. Rate Limiting
```python
def should_run(state):
    hours_since = (datetime.now() - last_run).total_seconds() / 3600
    return hours_since > 12 and state['runs_today'] < 2
```

Prevents burning API credits unnecessarily.

### 2. State Tracking
```json
{
  "last_run": "2026-03-19T14:23:00Z",
  "runs_today": 1
}
```

Persist across sessions.

### 3. Cost Awareness
Always log cost per run:
```python
print(f"✅ Complete (${cost:.2f})")
```

### 4. HEARTBEAT Integration
Skill is **executed** by HEARTBEAT.md, not manually called.

## Real Results

**Week 1 (Mar 2026):** Detected movie clip meta  
**Week 2:** Tracked 8 posts, avg 2.3k likes  
**Week 3:** Pattern confirmed, Oskar tested format  
**Week 4:** +40% engagement on content using pattern

## Lessons

- Start simple (fetch + analyze + report)
- Add tracking loop later (v2 feature)
- Cost-conscious (cache results, rate limit)
- Integrate with existing workflows (HEARTBEAT.md)
