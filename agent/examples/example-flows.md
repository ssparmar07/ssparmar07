# Example Flows for Data Query and Analytics Agent

## Example 1: Sales Overview Request

### User Query:
```
Show me my last 7 days sales.
```

### Step-by-Step Flow:

1. **Parse Query**: Identify intent = sales data, timeframe = 7 days
2. **Select Block**: `sales_overview`
3. **MCP Tool Call**:
```json
{
  "tool": "get_sales_data",
  "params": {
    "date_range": "7_days"
  }
}
```

4. **Mock MCP Response**:
```json
{
  "total_sales": 50320,
  "total_orders": 672,
  "previous_period_sales": 44750,
  "previous_period_orders": 623
}
```

5. **Transform to Block Schema**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 50320,
    "total_orders": 672,
    "avg_order_value": 74.88,
    "trend": "up",
    "trend_percentage": 12.45
  }
}
```

**Calculations**:
- `avg_order_value = total_sales / total_orders = 50320 / 672 = 74.88`
- `trend_percentage = ((total_sales - previous_period_sales) / previous_period_sales) * 100 = ((50320 - 44750) / 44750) * 100 = 12.45%`
- `trend = "up"` (since trend_percentage > 0)

---

## Example 2: Buy Box Analytics Request

### User Query:
```
What's my buy box performance for SKU-ABC-123?
```

### Step-by-Step Flow:

1. **Parse Query**: Intent = buy box analytics, SKU = "SKU-ABC-123"
2. **Select Block**: `buy_box`
3. **MCP Tool Call**:
```json
{
  "tool": "get_buy_box_data",
  "params": {
    "sku": "SKU-ABC-123",
    "date_range": "30_days"
  }
}
```

4. **Mock MCP Response**:
```json
{
  "sku": "SKU-ABC-123",
  "win_rate": 78.5,
  "competitor_name": "Competitor XYZ",
  "your_price": 29.99,
  "competitor_price": 27.99
}
```

5. **Transform to Block Schema**:
```json
{
  "block": "buy_box",
  "data": {
    "win_rate": 78.5,
    "lost_to": "Competitor XYZ",
    "price_difference": -2.00,
    "sku": "SKU-ABC-123"
  }
}
```

**Calculations**:
- `price_difference = your_price - competitor_price = 29.99 - 27.99 = 2.00` (shown as -2.00 to indicate you're more expensive)

---

## Example 3: Product Profitability Request

### User Query:
```
Show me profit margins for SKU-XYZ-789
```

### Step-by-Step Flow:

1. **Parse Query**: Intent = profitability, SKU = "SKU-XYZ-789"
2. **Select Block**: `profitability`
3. **MCP Tool Call**:
```json
{
  "tool": "get_product_profitability",
  "params": {
    "sku": "SKU-XYZ-789",
    "date_range": "30_days"
  }
}
```

4. **Mock MCP Response**:
```json
{
  "sku": "SKU-XYZ-789",
  "revenue": 15250,
  "cogs": 8500,
  "fees": 2100,
  "shipping": 1200,
  "advertising": 800
}
```

5. **Transform to Block Schema**:
```json
{
  "block": "profitability",
  "data": {
    "sku": "SKU-XYZ-789",
    "revenue": 15250,
    "cost": 12600,
    "profit": 2650,
    "margin": 17.38
  }
}
```

**Calculations**:
- `cost = cogs + fees + shipping + advertising = 8500 + 2100 + 1200 + 800 = 12600`
- `profit = revenue - cost = 15250 - 12600 = 2650`
- `margin = (profit / revenue) * 100 = (2650 / 15250) * 100 = 17.38%`

---

## Example 4: Review Sentiment Request

### User Query:
```
What are customers saying about SKU-DEF-456?
```

### Step-by-Step Flow:

1. **Parse Query**: Intent = review sentiment, SKU = "SKU-DEF-456"
2. **Select Block**: `reviews`
3. **MCP Tool Call**:
```json
{
  "tool": "get_review_sentiment",
  "params": {
    "sku": "SKU-DEF-456",
    "date_range": "all"
  }
}
```

4. **Mock MCP Response**:
```json
{
  "sku": "SKU-DEF-456",
  "positive_count": 245,
  "neutral_count": 38,
  "negative_count": 17,
  "keywords": [
    { "keyword": "quality", "frequency": 89 },
    { "keyword": "fast shipping", "frequency": 67 },
    { "keyword": "durable", "frequency": 54 },
    { "keyword": "value", "frequency": 45 },
    { "keyword": "easy to use", "frequency": 41 }
  ]
}
```

5. **Transform to Block Schema**:
```json
{
  "block": "reviews",
  "data": {
    "positive": 245,
    "neutral": 38,
    "negative": 17,
    "top_keywords": ["quality", "fast shipping", "durable", "value", "easy to use"]
  }
}
```

---

## Example 5: Sales Insights with Calculation

### User Query:
```
Give me insights on my sales performance this month
```

### Step-by-Step Flow:

1. **Parse Query**: Intent = sales insights, timeframe = current month (assume 30 days)
2. **Select Block**: `sales_overview`
3. **MCP Tool Call**:
```json
{
  "tool": "get_sales_data",
  "params": {
    "date_range": "30_days"
  }
}
```

4. **Mock MCP Response**:
```json
{
  "total_sales": 142380,
  "total_orders": 1876,
  "previous_period_sales": 155200,
  "previous_period_orders": 2013
}
```

5. **Transform to Block Schema**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 142380,
    "total_orders": 1876,
    "avg_order_value": 75.90,
    "trend": "down",
    "trend_percentage": -8.26
  }
}
```

**Calculations**:
- `avg_order_value = 142380 / 1876 = 75.90`
- `trend_percentage = ((142380 - 155200) / 155200) * 100 = -8.26%`
- `trend = "down"` (since trend_percentage < 0)

---

## Example 6: Multiple Queries Pattern

### User Query:
```
Compare profitability between SKU-A and SKU-B
```

**Note**: This requires multiple tool calls. The agent should:

1. Call `get_product_profitability` for SKU-A
2. Call `get_product_profitability` for SKU-B
3. Return an array of profitability blocks

**Response**:
```json
[
  {
    "block": "profitability",
    "data": {
      "sku": "SKU-A",
      "revenue": 25000,
      "cost": 18000,
      "profit": 7000,
      "margin": 28.0
    }
  },
  {
    "block": "profitability",
    "data": {
      "sku": "SKU-B",
      "revenue": 18500,
      "cost": 15200,
      "profit": 3300,
      "margin": 17.84
    }
  }
]
```

---

## Edge Cases and Error Handling

### Case 1: Invalid SKU
```json
{
  "error": "SKU not found",
  "message": "The SKU 'INVALID-SKU' does not exist in your catalog",
  "suggestion": "Use get_all_skus tool to list available SKUs"
}
```

### Case 2: Insufficient Data
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 5420,
    "total_orders": 87,
    "avg_order_value": 62.30,
    "trend": "neutral",
    "trend_percentage": 0
  },
  "warning": "Previous period data not available for trend calculation"
}
```

### Case 3: Date Range Too Large
```json
{
  "error": "Invalid date range",
  "message": "Custom date range cannot exceed 365 days",
  "params_received": {
    "start_date": "2023-01-01",
    "end_date": "2025-01-01"
  }
}
```
