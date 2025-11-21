# Implementation Guide for Data Query and Analytics Agent

## Overview

This guide provides instructions for implementing the Data Query and Analytics Agent for Sydon – Amazon Seller AI Analytics.

## Architecture

```
User Query
    ↓
Query Parser (NLP/Pattern Matching)
    ↓
Block Selector (Determines which of 4 blocks)
    ↓
MCP Tool Caller (Fetches data)
    ↓
Data Transformer (Converts to block schema)
    ↓
JSON Response
```

## Integration Steps

### 1. Query Parser Implementation

Create a function that analyzes user input to determine:
- Intent (sales, buy box, profitability, reviews)
- SKU identifier (if present)
- Date range (if specified)

```javascript
function parseQuery(userQuery) {
  const result = {
    block: null,
    sku: null,
    dateRange: '30_days' // default
  };
  
  // Extract SKU pattern
  const skuMatch = userQuery.match(/SKU[- ]?[A-Z0-9]+/i);
  if (skuMatch) {
    result.sku = skuMatch[0];
  }
  
  // Determine block type
  if (/sales|revenue|orders/i.test(userQuery)) {
    result.block = 'sales_overview';
  } else if (/buy box|win rate|competitor/i.test(userQuery)) {
    result.block = 'buy_box';
  } else if (/profit|margin|cost/i.test(userQuery)) {
    result.block = 'profitability';
  } else if (/review|sentiment|feedback/i.test(userQuery)) {
    result.block = 'reviews';
  }
  
  // Extract date range
  if (/7 days|week/i.test(userQuery)) {
    result.dateRange = '7_days';
  } else if (/90 days|quarter/i.test(userQuery)) {
    result.dateRange = '90_days';
  }
  
  return result;
}
```

### 2. MCP Tool Caller Implementation

Create a wrapper for MCP tool calls:

```javascript
async function callMCPTool(toolName, params) {
  // Implementation depends on your MCP client
  // This is a conceptual example
  
  const response = await mcpClient.callTool({
    name: toolName,
    arguments: params
  });
  
  if (response.error) {
    throw new Error(response.error);
  }
  
  return response.data;
}
```

### 3. Data Transformers

Create transformer functions for each block type:

```javascript
function transformToSalesOverview(mcpData) {
  const avgOrderValue = mcpData.total_orders > 0 
    ? mcpData.total_sales / mcpData.total_orders 
    : 0;
  
  let trendPercentage = 0;
  let trend = 'neutral';
  
  if (mcpData.previous_period_sales) {
    trendPercentage = 
      ((mcpData.total_sales - mcpData.previous_period_sales) / 
       mcpData.previous_period_sales) * 100;
    
    trend = trendPercentage > 0 ? 'up' : 
            trendPercentage < 0 ? 'down' : 'neutral';
  }
  
  return {
    block: 'sales_overview',
    data: {
      total_sales: mcpData.total_sales,
      total_orders: mcpData.total_orders,
      avg_order_value: parseFloat(avgOrderValue.toFixed(2)),
      trend: trend,
      trend_percentage: parseFloat(trendPercentage.toFixed(2))
    }
  };
}

function transformToBuyBox(mcpData) {
  const priceDiff = mcpData.competitor_price - mcpData.your_price;
  
  return {
    block: 'buy_box',
    data: {
      win_rate: mcpData.win_rate,
      lost_to: mcpData.competitor_name,
      price_difference: parseFloat(priceDiff.toFixed(2)),
      sku: mcpData.sku
    }
  };
}

function transformToProfitability(mcpData) {
  const cost = mcpData.cogs + mcpData.fees + 
               mcpData.shipping + mcpData.advertising;
  const profit = mcpData.revenue - cost;
  const margin = mcpData.revenue > 0 
    ? (profit / mcpData.revenue) * 100 
    : 0;
  
  return {
    block: 'profitability',
    data: {
      sku: mcpData.sku,
      revenue: mcpData.revenue,
      cost: cost,
      profit: profit,
      margin: parseFloat(margin.toFixed(2))
    }
  };
}

function transformToReviews(mcpData) {
  const topKeywords = mcpData.keywords
    .sort((a, b) => b.frequency - a.frequency)
    .slice(0, 5)
    .map(k => k.keyword);
  
  return {
    block: 'reviews',
    data: {
      positive: mcpData.positive_count,
      neutral: mcpData.neutral_count,
      negative: mcpData.negative_count,
      top_keywords: topKeywords
    }
  };
}
```

### 4. Main Agent Flow

Orchestrate the complete flow:

```javascript
async function processQuery(userQuery) {
  try {
    // Step 1: Parse query
    const parsed = parseQuery(userQuery);
    
    if (!parsed.block) {
      return {
        error: 'Unable to understand query',
        message: 'Please specify what analytics you need: sales, buy box, profitability, or reviews'
      };
    }
    
    // Step 2: Determine tool and params
    let toolName, toolParams;
    
    switch (parsed.block) {
      case 'sales_overview':
        toolName = 'get_sales_data';
        toolParams = { date_range: parsed.dateRange };
        break;
      
      case 'buy_box':
        if (!parsed.sku) {
          return {
            error: 'SKU required',
            message: 'Buy box analytics require a product SKU'
          };
        }
        toolName = 'get_buy_box_data';
        toolParams = { sku: parsed.sku, date_range: parsed.dateRange };
        break;
      
      case 'profitability':
        if (!parsed.sku) {
          return {
            error: 'SKU required',
            message: 'Profitability analysis requires a product SKU'
          };
        }
        toolName = 'get_product_profitability';
        toolParams = { sku: parsed.sku, date_range: parsed.dateRange };
        break;
      
      case 'reviews':
        if (!parsed.sku) {
          return {
            error: 'SKU required',
            message: 'Review sentiment requires a product SKU'
          };
        }
        toolName = 'get_review_sentiment';
        toolParams = { sku: parsed.sku, date_range: 'all' };
        break;
    }
    
    // Step 3: Call MCP tool
    const mcpResponse = await callMCPTool(toolName, toolParams);
    
    // Step 4: Transform to block schema
    let result;
    switch (parsed.block) {
      case 'sales_overview':
        result = transformToSalesOverview(mcpResponse);
        break;
      case 'buy_box':
        result = transformToBuyBox(mcpResponse);
        break;
      case 'profitability':
        result = transformToProfitability(mcpResponse);
        break;
      case 'reviews':
        result = transformToReviews(mcpResponse);
        break;
    }
    
    // Step 5: Return JSON
    return result;
    
  } catch (error) {
    return {
      error: 'Processing error',
      message: error.message,
      suggestion: 'Please try rephrasing your query or check your MCP connection'
    };
  }
}
```

## Testing

Use the test cases provided in `examples/test-cases.md` to validate your implementation.

### Running Tests

```javascript
// Example test runner
async function runTests() {
  const testCases = [
    {
      input: "Show me my sales for the last 7 days",
      expected: { block: "sales_overview" }
    },
    // ... more test cases
  ];
  
  for (const test of testCases) {
    const result = await processQuery(test.input);
    console.assert(
      result.block === test.expected.block,
      `Test failed: ${test.input}`
    );
  }
}
```

## Error Handling

Always handle these error cases:
1. Invalid/missing SKU
2. MCP tool connection failures
3. Missing or null data in MCP responses
4. Division by zero in calculations
5. Ambiguous user queries

## Performance Considerations

1. **Caching**: Cache MCP responses for frequently requested data
2. **Batch Requests**: For multi-SKU queries, batch MCP calls if supported
3. **Timeout Handling**: Set reasonable timeouts for MCP tool calls
4. **Rate Limiting**: Respect API rate limits

## Deployment Checklist

- [ ] MCP tools configured and tested
- [ ] Schema validation in place
- [ ] Error handling implemented
- [ ] All test cases passing
- [ ] Query parser handles edge cases
- [ ] Data transformers validated
- [ ] Response format matches specifications
- [ ] No UI code in responses
- [ ] Documentation complete

## Monitoring

Track these metrics:
- Query parsing success rate
- Block type distribution
- MCP tool response times
- Error rates by type
- Most common user queries

## Next Steps

1. Implement query parser with NLP library (optional, for better accuracy)
2. Add support for custom date ranges
3. Implement multi-SKU comparison queries
4. Add data caching layer
5. Create dashboard UI (separate from agent) to consume JSON responses
