# Getting Started with the Data Query and Analytics Agent

This guide will help you understand and use the Data Query and Analytics Agent for Sydon.

## What is This Agent?

The Data Query and Analytics Agent is a **data extraction system** that:
- Takes natural language queries from users
- Identifies what analytics data is needed
- Calls MCP (Model Context Protocol) tools to fetch data
- Returns structured JSON responses in predefined formats

**Important**: This agent does NOT create UI. It only returns data.

## The 4 Dashboard Blocks

All agent responses fit into one of these 4 blocks:

### 1. Sales Overview
**When to use**: User asks about sales, revenue, orders, or performance  
**Returns**:
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

### 2. Buy Box Analytics
**When to use**: User asks about buy box, competitors, or pricing  
**Returns**:
```json
{
  "block": "buy_box",
  "data": {
    "win_rate": 78.5,
    "lost_to": "Competitor XYZ",
    "price_difference": 2.00,
    "sku": "SKU-ABC-123"
  }
}
```

### 3. Product Profitability
**When to use**: User asks about profit, margins, or costs  
**Returns**:
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

### 4. Review Sentiment
**When to use**: User asks about reviews, ratings, or customer feedback  
**Returns**:
```json
{
  "block": "reviews",
  "data": {
    "positive": 245,
    "neutral": 38,
    "negative": 17,
    "top_keywords": ["quality", "fast shipping", "durable"]
  }
}
```

## How It Works

### Step 1: User Query
User types a natural language question:
```
"Show me my sales for the last 7 days"
```

### Step 2: Parse Query
Agent identifies:
- **Block type**: sales_overview
- **Date range**: 7 days
- **SKU**: none (account-level)

### Step 3: Call MCP Tool
Agent calls:
```json
{
  "tool": "get_sales_data",
  "params": {
    "date_range": "7_days"
  }
}
```

### Step 4: Receive Data
MCP tool returns:
```json
{
  "total_sales": 50320,
  "total_orders": 672,
  "previous_period_sales": 44750,
  "previous_period_orders": 623
}
```

### Step 5: Calculate & Transform
Agent calculates:
- `avg_order_value = 50320 / 672 = 74.88`
- `trend_percentage = ((50320 - 44750) / 44750) * 100 = 12.45%`
- `trend = "up"` (because positive percentage)

### Step 6: Return JSON
Agent returns:
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

## Common Query Patterns

### Sales Queries
- "Show me my sales"
- "Sales for last 30 days"
- "How many orders this month?"
- "Revenue performance"

### Buy Box Queries
- "Buy box performance for SKU-123"
- "Who am I losing to?"
- "Buy box win rate"

### Profitability Queries
- "Profit margin for SKU-456"
- "Show profitability"
- "Which products are profitable?"

### Review Queries
- "What are customers saying?"
- "Review sentiment for SKU-789"
- "Show customer feedback"

## Date Range Formats

The agent understands:
- "last 7 days" → `7_days`
- "past week" → `7_days`
- "last 30 days" → `30_days`
- "this month" → `30_days`
- "last quarter" → `90_days`

## Available MCP Tools

1. **get_sales_data** - Fetch sales metrics
2. **get_buy_box_data** - Fetch buy box performance (requires SKU)
3. **get_product_profitability** - Fetch profitability (requires SKU)
4. **get_review_sentiment** - Fetch review analysis (requires SKU)
5. **get_all_skus** - List available products

## Quick Start Checklist

To use this agent:

- [ ] Read [AGENT_PROMPT.md](AGENT_PROMPT.md) for complete instructions
- [ ] Review [example-flows.md](examples/example-flows.md) for detailed examples
- [ ] Study [dashboard-blocks.json](schemas/dashboard-blocks.json) for schemas
- [ ] Check [mcp-tools.json](tools/mcp-tools.json) for available tools
- [ ] Follow [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md) to integrate
- [ ] Use [test-cases.md](examples/test-cases.md) to validate

## Example Usage

### Example 1: Simple Sales Query

**Input**: `"Show me my sales"`

**Process**:
1. Identify block: `sales_overview`
2. Default date range: `30_days`
3. Call: `get_sales_data({date_range: "30_days"})`
4. Calculate: avg, trend, percentage
5. Return JSON

**Output**:
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

### Example 2: Product Analysis

**Input**: `"Profitability for SKU-ABC-123"`

**Process**:
1. Identify block: `profitability`
2. Extract SKU: `SKU-ABC-123`
3. Call: `get_product_profitability({sku: "SKU-ABC-123"})`
4. Calculate: cost, profit, margin
5. Return JSON

**Output**:
```json
{
  "block": "profitability",
  "data": {
    "sku": "SKU-ABC-123",
    "revenue": 25000,
    "cost": 18000,
    "profit": 7000,
    "margin": 28.0
  }
}
```

## Important Rules

### ✅ DO:
- Return only structured JSON
- Use exact schema for each block
- Calculate derived fields (avg, margin, etc.)
- Handle errors gracefully
- Validate all outputs

### ❌ DON'T:
- Generate UI components
- Create React code
- Return HTML or CSS
- Add styling
- Deviate from schemas

## Error Handling

When something goes wrong, return:
```json
{
  "error": "Error type",
  "message": "Human-readable description",
  "suggestion": "What user should do next"
}
```

**Example**:
```json
{
  "error": "SKU not found",
  "message": "The SKU 'INVALID-SKU' does not exist in your catalog",
  "suggestion": "Use get_all_skus tool to list available SKUs"
}
```

## Next Steps

1. **Understand the Specs**: Read all documentation in the `agent/` folder
2. **Review Examples**: Study the example flows and test cases
3. **Implement**: Follow the implementation guide
4. **Test**: Use the test cases to validate
5. **Deploy**: Integrate with your MCP client

## Need Help?

- **Agent Behavior**: See [AGENT_PROMPT.md](AGENT_PROMPT.md)
- **Quick Lookup**: See [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
- **Implementation**: See [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md)
- **Examples**: See [example-flows.md](examples/example-flows.md)
- **Testing**: See [test-cases.md](examples/test-cases.md)
- **Schemas**: See [dashboard-blocks.json](schemas/dashboard-blocks.json)
- **Tools**: See [mcp-tools.json](tools/mcp-tools.json)

## Summary

This agent is a **data extraction layer** that sits between user queries and MCP tools. It understands natural language, fetches the right data, calculates insights, and returns structured JSON that matches one of 4 predefined schemas. No UI generation - just pure data.
