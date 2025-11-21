# 🔥 Data Query and Analytics Agent for Sydon

**Amazon Seller AI Analytics - Data Extraction & Query Processing**

## Overview

This repository contains the Data Query and Analytics Agent for Sydon, an AI-powered analytics platform for Amazon sellers. The agent extracts analytical data fields from user queries, uses MCP (Model Context Protocol) tools to fetch data, and returns structured JSON responses.

## Key Features

✅ **Pure Data Extraction** - No UI generation, only analytical data  
✅ **MCP Tool Integration** - Leverages Model Context Protocol for data fetching  
✅ **4 Predefined Dashboard Blocks** - Fixed schemas for consistent output  
✅ **Smart Query Processing** - Understands natural language queries  
✅ **Automatic Calculations** - Derives insights from raw data  

## Dashboard Blocks

1. **Sales Overview** - Total sales, orders, trends
2. **Buy Box Analytics** - Win rates, competitor analysis
3. **Product Profitability** - Revenue, costs, margins
4. **Review Sentiment** - Customer feedback analysis

## Quick Start

### 1. View Agent Instructions
See [`agent/AGENT_PROMPT.md`](agent/AGENT_PROMPT.md) for complete agent behavior rules.

### 2. Explore Examples
Check [`agent/examples/example-flows.md`](agent/examples/example-flows.md) for detailed usage examples.

### 3. Implementation Guide
Follow [`agent/IMPLEMENTATION_GUIDE.md`](agent/IMPLEMENTATION_GUIDE.md) to integrate the agent.

## Repository Structure

```
agent/
├── AGENT_PROMPT.md           # Complete agent instructions
├── QUICK_REFERENCE.md        # Quick lookup guide
├── IMPLEMENTATION_GUIDE.md   # Integration guide
├── schemas/
│   └── dashboard-blocks.json # JSON schemas for all blocks
├── tools/
│   └── mcp-tools.json        # MCP tool definitions
└── examples/
    ├── example-flows.md      # Detailed example flows
    └── test-cases.md         # Comprehensive test cases
```

## Example Usage

**User Query:**
```
Show me my last 7 days sales.
```

**Agent Response:**
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

## Documentation

- **[Agent Prompt](agent/AGENT_PROMPT.md)** - Complete agent behavior and rules
- **[Quick Reference](agent/QUICK_REFERENCE.md)** - Query patterns and decision tree
- **[Implementation Guide](agent/IMPLEMENTATION_GUIDE.md)** - Integration instructions
- **[Example Flows](agent/examples/example-flows.md)** - Step-by-step examples
- **[Test Cases](agent/examples/test-cases.md)** - Validation test suite
- **[Block Schemas](agent/schemas/dashboard-blocks.json)** - JSON schema definitions
- **[MCP Tools](agent/tools/mcp-tools.json)** - Available MCP tools

## Design Principles

🎯 **Data-Only Response** - Never generates UI components or styles  
📋 **Schema Compliance** - All responses match predefined block schemas  
🧮 **Smart Calculations** - Automatically derives insights and metrics  
🔧 **MCP Integration** - Uses Model Context Protocol for data access  

## Contributing

This is a specification repository for the Sydon Analytics Agent. For implementation questions or suggestions, please open an issue.

## License

MIT License - See LICENSE file for details

---

**Note**: This repository contains the agent specification and documentation. The actual implementation should follow the guidelines in the [Implementation Guide](agent/IMPLEMENTATION_GUIDE.md).
