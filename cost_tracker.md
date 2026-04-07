# AI API Cost Tracker

> Quick Start: Copy the table schema and utility function below into your project. Every API call gets logged with model, tokens, cost, and endpoint. Query to find which features are expensive.

## Database Table (SQLAlchemy)

```python
from sqlalchemy import DateTime, Float, Integer, String, func
from sqlalchemy.orm import Mapped, mapped_column
from datetime import datetime

class ApiCostLog(Base):
    __tablename__ = "api_cost_logs"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    request_type: Mapped[str] = mapped_column(String(50), nullable=False, index=True)
    model: Mapped[str] = mapped_column(String(100), nullable=False)
    input_tokens: Mapped[int] = mapped_column(Integer, default=0)
    output_tokens: Mapped[int] = mapped_column(Integer, default=0)
    cache_read_tokens: Mapped[int] = mapped_column(Integer, default=0)
    cache_creation_tokens: Mapped[int] = mapped_column(Integer, default=0)
    estimated_cost_usd: Mapped[float] = mapped_column(Float, default=0.0)
    created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())
```

## SQLite Version (if you're not using SQLAlchemy)

```sql
CREATE TABLE api_cost_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    request_type TEXT NOT NULL,
    model TEXT NOT NULL,
    input_tokens INTEGER DEFAULT 0,
    output_tokens INTEGER DEFAULT 0,
    cache_read_tokens INTEGER DEFAULT 0,
    cache_creation_tokens INTEGER DEFAULT 0,
    estimated_cost_usd REAL DEFAULT 0.0,
    created_at DATETIME DEFAULT (datetime('now'))
);
CREATE INDEX idx_cost_logs_type ON api_cost_logs(request_type);
```

## Pricing + Cost Calculator

```python
# Anthropic pricing per million tokens (USD) — update when pricing changes
PRICING = {
    "claude-sonnet-4-6": {"input": 3.00, "output": 15.00, "cache_read": 0.30, "cache_creation": 3.75},
    "claude-haiku-4-5":  {"input": 0.80, "output": 4.00,  "cache_read": 0.08, "cache_creation": 1.00},
}

def calculate_cost(model, input_tokens, output_tokens, cache_read_tokens=0, cache_creation_tokens=0):
    rates = PRICING.get(model, PRICING["claude-sonnet-4-6"])
    cost = (
        (input_tokens / 1_000_000) * rates["input"]
        + (output_tokens / 1_000_000) * rates["output"]
        + (cache_read_tokens / 1_000_000) * rates["cache_read"]
        + (cache_creation_tokens / 1_000_000) * rates["cache_creation"]
    )
    return round(cost, 6)
```

## Extract Usage from Anthropic Response

```python
def extract_usage(message):
    usage = message.usage
    return {
        "input_tokens": usage.input_tokens,
        "output_tokens": usage.output_tokens,
        "cache_read_tokens": getattr(usage, "cache_read_input_tokens", 0) or 0,
        "cache_creation_tokens": getattr(usage, "cache_creation_input_tokens", 0) or 0,
    }
```

## Log Every Call

```python
# After any Anthropic API call:
message = client.messages.create(...)
usage = extract_usage(message)
cost = calculate_cost(model, **usage)

# Insert into your database:
db.execute(
    "INSERT INTO api_cost_logs (request_type, model, input_tokens, output_tokens, cache_read_tokens, cache_creation_tokens, estimated_cost_usd) VALUES (?, ?, ?, ?, ?, ?, ?)",
    ("generate", model, usage["input_tokens"], usage["output_tokens"], usage["cache_read_tokens"], usage["cache_creation_tokens"], cost)
)
```

## Useful Queries

```sql
-- Cost per endpoint
SELECT request_type, COUNT(*) as calls, ROUND(SUM(estimated_cost_usd), 4) as total_cost, ROUND(AVG(estimated_cost_usd), 6) as avg_cost
FROM api_cost_logs GROUP BY request_type ORDER BY total_cost DESC;

-- Daily spend
SELECT DATE(created_at) as day, ROUND(SUM(estimated_cost_usd), 4) as spend
FROM api_cost_logs GROUP BY day ORDER BY day DESC LIMIT 7;

-- Model breakdown
SELECT model, COUNT(*) as calls, ROUND(SUM(estimated_cost_usd), 4) as total
FROM api_cost_logs GROUP BY model;
```
