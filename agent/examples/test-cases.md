# Test Cases for Data Query and Analytics Agent

## Test Suite 1: Sales Overview Block

### Test 1.1: Basic Sales Query
**Input**: "Show me my sales for the last 7 days"

**Expected MCP Call**:
```json
{
  "tool": "get_sales_data",
  "params": {
    "date_range": "7_days"
  }
}
```

**Mock MCP Response**:
```json
{
  "total_sales": 10000,
  "total_orders": 100,
  "previous_period_sales": 9000,
  "previous_period_orders": 90
}
```

**Expected Output**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 10000,
    "total_orders": 100,
    "avg_order_value": 100.00,
    "trend": "up",
    "trend_percentage": 11.11
  }
}
```

**Validation**:
- ✓ avg_order_value calculated correctly (10000/100 = 100)
- ✓ trend_percentage calculated correctly ((10000-9000)/9000 * 100 = 11.11)
- ✓ trend is "up" (positive percentage)

---

### Test 1.2: Negative Trend
**Input**: "Sales performance last month"

**Mock MCP Response**:
```json
{
  "total_sales": 8000,
  "total_orders": 120,
  "previous_period_sales": 10000,
  "previous_period_orders": 150
}
```

**Expected Output**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 8000,
    "total_orders": 120,
    "avg_order_value": 66.67,
    "trend": "down",
    "trend_percentage": -20.00
  }
}
```

**Validation**:
- ✓ trend is "down" (negative percentage)
- ✓ avg_order_value rounded to 2 decimals

---

### Test 1.3: Neutral Trend
**Input**: "Show me sales data"

**Mock MCP Response**:
```json
{
  "total_sales": 5000,
  "total_orders": 50,
  "previous_period_sales": 5000,
  "previous_period_orders": 50
}
```

**Expected Output**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 5000,
    "total_orders": 50,
    "avg_order_value": 100.00,
    "trend": "neutral",
    "trend_percentage": 0
  }
}
```

**Validation**:
- ✓ trend is "neutral" (zero percentage)

---

## Test Suite 2: Buy Box Analytics Block

### Test 2.1: Basic Buy Box Query
**Input**: "Buy box performance for SKU-ABC-123"

**Expected MCP Call**:
```json
{
  "tool": "get_buy_box_data",
  "params": {
    "sku": "SKU-ABC-123",
    "date_range": "30_days"
  }
}
```

**Mock MCP Response**:
```json
{
  "sku": "SKU-ABC-123",
  "win_rate": 85.5,
  "competitor_name": "Amazon Warehouse",
  "your_price": 49.99,
  "competitor_price": 47.99
}
```

**Expected Output**:
```json
{
  "block": "buy_box",
  "data": {
    "win_rate": 85.5,
    "lost_to": "Amazon Warehouse",
    "price_difference": 2.00,
    "sku": "SKU-ABC-123"
  }
}
```

**Validation**:
- ✓ price_difference = your_price - competitor_price = 49.99 - 47.99 = 2.00 (positive means you're more expensive)
- ✓ competitor_name mapped to lost_to

---

### Test 2.2: Winning Buy Box
**Input**: "Why am I winning buy box for SKU-XYZ-789?"

**Mock MCP Response**:
```json
{
  "sku": "SKU-XYZ-789",
  "win_rate": 95.0,
  "competitor_name": "Best Seller Co",
  "your_price": 29.99,
  "competitor_price": 32.99
}
```

**Expected Output**:
```json
{
  "block": "buy_box",
  "data": {
    "win_rate": 95.0,
    "lost_to": "Best Seller Co",
    "price_difference": -3.00,
    "sku": "SKU-XYZ-789"
  }
}
```

**Validation**:
- ✓ price_difference = your_price - competitor_price = 29.99 - 32.99 = -3.00 (negative means you're cheaper)
- ✓ high win_rate reflects competitive pricing

---

## Test Suite 3: Product Profitability Block

### Test 3.1: Basic Profitability Query
**Input**: "What's the profit margin for SKU-DEF-456?"

**Expected MCP Call**:
```json
{
  "tool": "get_product_profitability",
  "params": {
    "sku": "SKU-DEF-456",
    "date_range": "30_days"
  }
}
```

**Mock MCP Response**:
```json
{
  "sku": "SKU-DEF-456",
  "revenue": 20000,
  "cogs": 10000,
  "fees": 3000,
  "shipping": 1500,
  "advertising": 500
}
```

**Expected Output**:
```json
{
  "block": "profitability",
  "data": {
    "sku": "SKU-DEF-456",
    "revenue": 20000,
    "cost": 15000,
    "profit": 5000,
    "margin": 25.00
  }
}
```

**Validation**:
- ✓ cost = sum of all costs (10000 + 3000 + 1500 + 500 = 15000)
- ✓ profit = revenue - cost (20000 - 15000 = 5000)
- ✓ margin = (profit / revenue) * 100 = (5000 / 20000) * 100 = 25.00%

---

### Test 3.2: Low Margin Product
**Input**: "Show profitability for SKU-LOW-001"

**Mock MCP Response**:
```json
{
  "sku": "SKU-LOW-001",
  "revenue": 10000,
  "cogs": 7000,
  "fees": 2000,
  "shipping": 800,
  "advertising": 150
}
```

**Expected Output**:
```json
{
  "block": "profitability",
  "data": {
    "sku": "SKU-LOW-001",
    "revenue": 10000,
    "cost": 9950,
    "profit": 50,
    "margin": 0.50
  }
}
```

**Validation**:
- ✓ Very low margin calculated correctly
- ✓ Margin is 0.50% (not 50%)

---

### Test 3.3: Negative Profit
**Input**: "Profitability analysis for SKU-LOSS-999"

**Mock MCP Response**:
```json
{
  "sku": "SKU-LOSS-999",
  "revenue": 5000,
  "cogs": 4000,
  "fees": 1200,
  "shipping": 600,
  "advertising": 400
}
```

**Expected Output**:
```json
{
  "block": "profitability",
  "data": {
    "sku": "SKU-LOSS-999",
    "revenue": 5000,
    "cost": 6200,
    "profit": -1200,
    "margin": -24.00
  }
}
```

**Validation**:
- ✓ Negative profit handled correctly
- ✓ Negative margin calculated properly

---

## Test Suite 4: Review Sentiment Block

### Test 4.1: Basic Review Query
**Input**: "What are customers saying about SKU-REV-123?"

**Expected MCP Call**:
```json
{
  "tool": "get_review_sentiment",
  "params": {
    "sku": "SKU-REV-123",
    "date_range": "all"
  }
}
```

**Mock MCP Response**:
```json
{
  "sku": "SKU-REV-123",
  "positive_count": 150,
  "neutral_count": 20,
  "negative_count": 10,
  "keywords": [
    { "keyword": "excellent", "frequency": 45 },
    { "keyword": "quality", "frequency": 38 },
    { "keyword": "fast delivery", "frequency": 32 },
    { "keyword": "great value", "frequency": 28 },
    { "keyword": "recommended", "frequency": 25 }
  ]
}
```

**Expected Output**:
```json
{
  "block": "reviews",
  "data": {
    "positive": 150,
    "neutral": 20,
    "negative": 10,
    "top_keywords": ["excellent", "quality", "fast delivery", "great value", "recommended"]
  }
}
```

**Validation**:
- ✓ top_keywords is array of strings (not objects)
- ✓ keywords ordered by frequency
- ✓ counts preserved from response

---

### Test 4.2: Mostly Negative Reviews
**Input**: "Review sentiment for SKU-BAD-999"

**Mock MCP Response**:
```json
{
  "sku": "SKU-BAD-999",
  "positive_count": 5,
  "neutral_count": 8,
  "negative_count": 87,
  "keywords": [
    { "keyword": "poor quality", "frequency": 35 },
    { "keyword": "broke", "frequency": 28 },
    { "keyword": "disappointed", "frequency": 22 }
  ]
}
```

**Expected Output**:
```json
{
  "block": "reviews",
  "data": {
    "positive": 5,
    "neutral": 8,
    "negative": 87,
    "top_keywords": ["poor quality", "broke", "disappointed"]
  }
}
```

**Validation**:
- ✓ Negative keywords captured
- ✓ High negative count reflected

---

## Test Suite 5: Edge Cases

### Test 5.1: Missing Previous Period Data
**Input**: "Show sales"

**Mock MCP Response**:
```json
{
  "total_sales": 5000,
  "total_orders": 50,
  "previous_period_sales": null,
  "previous_period_orders": null
}
```

**Expected Output**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 5000,
    "total_orders": 50,
    "avg_order_value": 100.00,
    "trend": "neutral",
    "trend_percentage": 0
  }
}
```

**Validation**:
- ✓ Handles null previous period gracefully
- ✓ Defaults to neutral trend

---

### Test 5.2: Zero Orders
**Input**: "Sales data"

**Mock MCP Response**:
```json
{
  "total_sales": 0,
  "total_orders": 0,
  "previous_period_sales": 1000,
  "previous_period_orders": 10
}
```

**Expected Output**:
```json
{
  "block": "sales_overview",
  "data": {
    "total_sales": 0,
    "total_orders": 0,
    "avg_order_value": 0,
    "trend": "down",
    "trend_percentage": -100.00
  }
}
```

**Validation**:
- ✓ Handles division by zero (avg_order_value = 0)
- ✓ Trend shows complete drop

---

### Test 5.3: Invalid SKU
**Input**: "Profitability for INVALID-SKU"

**Expected MCP Response**:
```json
{
  "error": "SKU_NOT_FOUND",
  "message": "SKU 'INVALID-SKU' not found in catalog"
}
```

**Expected Output**:
```json
{
  "error": "SKU not found",
  "message": "The SKU 'INVALID-SKU' does not exist in your catalog",
  "suggestion": "Use get_all_skus tool to list available SKUs"
}
```

**Validation**:
- ✓ Error response format
- ✓ Helpful suggestion provided

---

## Test Suite 6: Multi-Block Responses

### Test 6.1: Compare Multiple Products
**Input**: "Compare profitability for SKU-A and SKU-B"

**Expected Output**:
```json
[
  {
    "block": "profitability",
    "data": {
      "sku": "SKU-A",
      "revenue": 15000,
      "cost": 10000,
      "profit": 5000,
      "margin": 33.33
    }
  },
  {
    "block": "profitability",
    "data": {
      "sku": "SKU-B",
      "revenue": 12000,
      "cost": 9600,
      "profit": 2400,
      "margin": 20.00
    }
  }
]
```

**Validation**:
- ✓ Array of blocks returned
- ✓ Each block follows schema
- ✓ Correct comparison structure

---

## Validation Rules Summary

1. **Numeric Precision**: Round to 2 decimal places
2. **Trend Values**: Only "up", "down", or "neutral"
3. **Array Fields**: Must be arrays of strings (for top_keywords)
4. **Required Fields**: All schema fields must be present
5. **JSON Format**: Response must be valid JSON
6. **No UI Elements**: No React components, styles, or HTML
7. **Block Selection**: Must match one of the 4 predefined blocks
8. **Error Handling**: Use standard error response format
