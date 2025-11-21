# Quick Reference Guide

## Query Pattern Matching

Use this guide to quickly identify which block and tool to use based on user queries:

### Sales Overview Queries
**Trigger Keywords**: sales, revenue, orders, performance, last X days, this month, quarterly

**Example Queries**:
- "Show me my sales"
- "How many orders did I get?"
- "Sales performance last week"
- "What's my revenue this month?"

**Block**: `sales_overview`  
**Tool**: `get_sales_data`

---

### Buy Box Queries
**Trigger Keywords**: buy box, win rate, competitor, pricing, lost to

**Example Queries**:
- "Buy box performance for SKU-123"
- "Who am I losing the buy box to?"
- "Buy box win rate"
- "Why am I not winning buy box?"

**Block**: `buy_box`  
**Tool**: `get_buy_box_data`

---

### Profitability Queries
**Trigger Keywords**: profit, margin, cost, profitability, ROI, earnings

**Example Queries**:
- "What's my profit margin?"
- "Show profitability for SKU-456"
- "Which products are most profitable?"
- "Calculate profit for product X"

**Block**: `profitability`  
**Tool**: `get_product_profitability`

---

### Review Sentiment Queries
**Trigger Keywords**: reviews, ratings, sentiment, feedback, customers saying, opinion

**Example Queries**:
- "What are customers saying?"
- "Review sentiment for SKU-789"
- "Show me customer feedback"
- "Analyze product reviews"

**Block**: `reviews`  
**Tool**: `get_review_sentiment`

---

## Date Range Mapping

| User Input | Normalized Value |
|------------|------------------|
| "last 7 days", "past week" | `7_days` |
| "last 30 days", "past month", "this month" | `30_days` |
| "last 90 days", "past quarter", "quarterly" | `90_days` |
| "from X to Y" | `custom` with start_date and end_date |

---

## Trend Calculation Logic

```
trend_percentage = ((current_value - previous_value) / previous_value) * 100

if trend_percentage > 0:
    trend = "up"
elif trend_percentage < 0:
    trend = "down"
else:
    trend = "neutral"
```

---

## Common Calculations

### Average Order Value
```
avg_order_value = total_sales / total_orders
```

### Profit Margin
```
profit = revenue - cost
margin = (profit / revenue) * 100
```

### Price Difference
```
price_difference = your_price - competitor_price
```
(Negative value means you're more expensive)

---

## Response Format Template

```json
{
  "block": "<block_type>",
  "data": {
    // Block-specific fields
  }
}
```

---

## Agent Decision Tree

```
User Query
    |
    ├── Contains SKU identifier?
    |   ├── Yes → Extract SKU
    |   └── No → Use account-level data
    |
    ├── What type of data?
    |   ├── Sales/Orders → sales_overview
    |   ├── Buy Box/Competitor → buy_box
    |   ├── Profit/Margin → profitability
    |   └── Reviews/Sentiment → reviews
    |
    ├── Date range specified?
    |   ├── Yes → Parse and normalize
    |   └── No → Use default (30_days)
    |
    └── Call MCP Tool → Transform → Return JSON
```

---

## Validation Checklist

Before returning a response, verify:

- [ ] Response matches exact schema for chosen block
- [ ] All required fields are present
- [ ] Numeric values are properly calculated
- [ ] Trend values are "up", "down", or "neutral" (not "increase", "decrease", etc.)
- [ ] No UI components or styling included
- [ ] Response is valid JSON
- [ ] Arrays contain strings (not objects for top_keywords)
- [ ] Numbers are not wrapped in quotes

---

## Error Response Format

When an error occurs:

```json
{
  "error": "<error_type>",
  "message": "<human_readable_message>",
  "suggestion": "<what_user_should_do_next>"
}
```

---

## Multi-Block Responses

For queries requesting multiple items:

```json
[
  {
    "block": "profitability",
    "data": { /* SKU 1 data */ }
  },
  {
    "block": "profitability",
    "data": { /* SKU 2 data */ }
  }
]
```
