# CLAUDE.md - GoHighLevel MCP Server

This document provides guidance for AI assistants working on this codebase.

## Project Overview

This is a **GoHighLevel MCP (Model Context Protocol) Server** that provides 269+ tools for integrating GoHighLevel CRM functionality with Claude Desktop and ChatGPT. It serves as a bridge between AI systems and GoHighLevel's API, enabling AI-powered CRM automation.

**Key Purpose:**
- Connect GoHighLevel API to AI systems via MCP protocol
- Provide comprehensive CRM automation tools (contacts, conversations, calendars, payments, etc.)
- Support both CLI (stdio) and HTTP (SSE) transport modes

## Architecture

### Directory Structure

```
GoHighLevel-MCP/
├── src/                          # TypeScript source code
│   ├── server.ts                 # CLI MCP server (stdio transport for Claude Desktop)
│   ├── http-server.ts            # HTTP MCP server (SSE transport for web/ChatGPT)
│   ├── clients/
│   │   └── ghl-api-client.ts     # Core GoHighLevel API client (axios-based)
│   ├── tools/                    # MCP tool implementations (one file per domain)
│   │   ├── contact-tools.ts      # 31 contact management tools
│   │   ├── conversation-tools.ts # 20 messaging/conversation tools
│   │   ├── calendar-tools.ts     # 14 calendar/appointment tools
│   │   ├── opportunity-tools.ts  # 10 sales pipeline tools
│   │   ├── invoices-tools.ts     # 39 invoice/billing tools
│   │   ├── payments-tools.ts     # 20 payment tools
│   │   ├── location-tools.ts     # 24 location management tools
│   │   ├── store-tools.ts        # 18 e-commerce tools
│   │   ├── social-media-tools.ts # 17 social media tools
│   │   ├── products-tools.ts     # 10 product tools
│   │   ├── object-tools.ts       # 9 custom objects tools
│   │   ├── association-tools.ts  # 10 association tools
│   │   ├── custom-field-v2-tools.ts # 8 custom field tools
│   │   ├── blog-tools.ts         # 7 blog tools
│   │   ├── email-tools.ts        # 5 email marketing tools
│   │   ├── media-tools.ts        # 3 media library tools
│   │   ├── survey-tools.ts       # 2 survey tools
│   │   ├── workflow-tools.ts     # 1 workflow tool
│   │   └── email-isv-tools.ts    # 1 email verification tool
│   └── types/
│       └── ghl-types.ts          # Comprehensive TypeScript type definitions
├── tests/                        # Jest test suite
│   ├── setup.ts                  # Test environment configuration
│   ├── basic.test.ts             # Basic sanity tests
│   ├── mocks/                    # API client mocks
│   ├── clients/                  # API client tests
│   └── tools/                    # Tool implementation tests
├── api/                          # Vercel API routes
├── dist/                         # Compiled JavaScript output (generated)
└── Configuration files
```

### Core Components

1. **GHLApiClient** (`src/clients/ghl-api-client.ts`)
   - Axios-based HTTP client for GoHighLevel API
   - Handles authentication via Bearer token
   - Manages API versioning and base URL configuration
   - All API calls flow through this client

2. **Tool Classes** (`src/tools/*.ts`)
   - Each domain has its own tool class (e.g., `ContactTools`, `CalendarTools`)
   - Follow a consistent pattern:
     - `getToolDefinitions()` / `getTools()` - Returns MCP tool schemas
     - `executeTool()` / `handleToolCall()` - Routes and executes tool calls
   - Tools wrap GHLApiClient methods with MCP-compatible interfaces

3. **Server Entry Points**
   - `server.ts` - Stdio transport for Claude Desktop integration
   - `http-server.ts` - Express-based HTTP/SSE for web apps (ChatGPT)

## Development Workflow

### Initial Setup

```bash
# Install dependencies
npm install

# Copy environment file and configure
cp .env.example .env
# Edit .env with your GHL_API_KEY, GHL_LOCATION_ID

# Build the project
npm run build
```

### Available Scripts

```bash
npm run build        # Compile TypeScript to dist/
npm run dev          # Development server with hot reload (nodemon + ts-node)
npm start            # Production HTTP server (dist/http-server.js)
npm run start:stdio  # CLI MCP server for Claude Desktop (dist/server.js)
npm run start:http   # HTTP MCP server (same as npm start)
npm test             # Run Jest tests
npm run test:watch   # Watch mode testing
npm run test:coverage # Coverage report
npm run lint         # TypeScript type checking (tsc --noEmit)
```

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GHL_API_KEY` | Yes | Private Integrations API key from GoHighLevel |
| `GHL_LOCATION_ID` | Yes | GoHighLevel location/sub-account ID |
| `GHL_BASE_URL` | No | API base URL (default: `https://services.leadconnectorhq.com`) |
| `PORT` | No | HTTP server port (default: 8000) |
| `NODE_ENV` | No | Environment mode (development/production/test) |

## Code Conventions

### TypeScript Standards

- **Target**: ES2022 with NodeNext module resolution
- **Strict mode**: Enabled
- Types defined in `src/types/ghl-types.ts`
- Use `.js` extensions in imports (NodeNext module resolution)

### Tool Implementation Pattern

When adding new tools, follow this pattern:

```typescript
// In src/tools/example-tools.ts
import { Tool } from '@modelcontextprotocol/sdk/types.js';
import { GHLApiClient } from '../clients/ghl-api-client.js';

export class ExampleTools {
  constructor(private ghlClient: GHLApiClient) {}

  // Return MCP tool definitions
  getToolDefinitions(): Tool[] {
    return [
      {
        name: 'tool_name',
        description: 'Clear description of what this tool does',
        inputSchema: {
          type: 'object',
          properties: {
            paramName: { type: 'string', description: 'Parameter description' }
          },
          required: ['paramName']
        }
      }
    ];
  }

  // Execute tool calls
  async executeTool(name: string, args: Record<string, unknown>): Promise<any> {
    switch (name) {
      case 'tool_name':
        return this.toolNameMethod(args as SomeParams);
      default:
        throw new Error(`Unknown tool: ${name}`);
    }
  }

  private async toolNameMethod(params: SomeParams): Promise<any> {
    // Call GHL API via this.ghlClient
    return this.ghlClient.someApiMethod(params);
  }
}
```

### Tool Naming Conventions

- Use snake_case for tool names: `create_contact`, `search_opportunities`
- Prefix with `ghl_` for newer tools to avoid conflicts: `ghl_get_workflows`
- Be descriptive: `add_contact_to_workflow` not `add_to_wf`

### API Client Methods

- Methods return raw API responses wrapped in a standard format
- Handle errors at the tool level, not client level
- Use appropriate HTTP methods (GET for reads, POST for creates, PUT/PATCH for updates)

## Testing

### Test Structure

```
tests/
├── setup.ts                    # Global test config and env vars
├── basic.test.ts               # Sanity checks
├── mocks/
│   └── ghl-api-client.mock.ts  # Mock API client
├── clients/
│   └── ghl-api-client.test.ts  # API client tests
└── tools/
    ├── contact-tools.test.ts   # Contact tool tests
    ├── conversation-tools.test.ts
    └── blog-tools.test.ts
```

### Running Tests

```bash
npm test                    # Run all tests
npm run test:watch          # Watch mode
npm run test:coverage       # Generate coverage report
```

### Test Environment

Tests use mock environment variables (defined in `tests/setup.ts`):
- `GHL_API_KEY`: `test_api_key_123`
- `GHL_BASE_URL`: `https://test.leadconnectorhq.com`
- `GHL_LOCATION_ID`: `test_location_123`

### Coverage Thresholds

Configured in `jest.config.js`:
- Branches: 70%
- Functions: 70%
- Lines: 70%
- Statements: 70%

## Adding New Tool Categories

1. **Create tool file**: `src/tools/new-category-tools.ts`
2. **Define types**: Add interfaces to `src/types/ghl-types.ts`
3. **Add to server.ts**:
   - Import the tool class
   - Initialize in constructor
   - Add to `getToolDefinitions()` list
   - Add `isNewCategoryTool()` helper method
   - Add case to `CallToolRequestSchema` handler
4. **Add to http-server.ts**: Same steps as server.ts
5. **Write tests**: `tests/tools/new-category-tools.test.ts`

## API Integration Notes

### GoHighLevel API

- **Base URL**: `https://services.leadconnectorhq.com`
- **Auth**: Bearer token in Authorization header
- **Version Header**: `Version: 2021-07-28` (varies by endpoint)
- **Location Header**: Most endpoints require `location: {locationId}` header

### Rate Limiting

- GoHighLevel has API rate limits
- Implement exponential backoff for 429 responses
- Consider caching for frequently accessed data

### Private Integrations

This project requires a **Private Integrations API key**, NOT a regular API key:
1. Navigate to GoHighLevel Settings > Integrations > Private Integrations
2. Create integration with required scopes
3. Use the generated API key

## Deployment

### Local Development

```bash
npm run dev  # Starts with nodemon for hot reload
```

### Claude Desktop Integration

Add to Claude Desktop's `mcp_settings.json`:
```json
{
  "mcpServers": {
    "ghl-mcp-server": {
      "command": "node",
      "args": ["/path/to/dist/server.js"],
      "env": {
        "GHL_API_KEY": "your_api_key",
        "GHL_LOCATION_ID": "your_location_id"
      }
    }
  }
}
```

### Cloud Deployment

Supported platforms:
- **Vercel**: One-click deploy, uses `api/index.js`
- **Railway**: Uses `Procfile`
- **Docker**: Uses `Dockerfile`

HTTP endpoints:
- `/health` - Health check
- `/tools` - List available tools
- `/sse` - MCP SSE endpoint
- `/capabilities` - Server capabilities

## Common Tasks

### Finding a Tool Implementation

Tools are organized by domain in `src/tools/`:
- Contact operations: `contact-tools.ts`
- Messaging: `conversation-tools.ts`
- Calendar: `calendar-tools.ts`
- Sales pipeline: `opportunity-tools.ts`
- Billing: `invoices-tools.ts`

### Debugging

- Server logs to stderr with `[GHL MCP]` prefix
- HTTP server logs requests to console
- Check API responses for detailed error messages

### Building for Production

```bash
npm run build           # Compile TypeScript
npm run start:stdio     # CLI mode
npm run start:http      # HTTP mode
```

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/server.ts` | Main CLI entry point |
| `src/http-server.ts` | HTTP/SSE entry point |
| `src/clients/ghl-api-client.ts` | GoHighLevel API client |
| `src/types/ghl-types.ts` | All TypeScript interfaces |
| `package.json` | Dependencies and scripts |
| `tsconfig.json` | TypeScript configuration |
| `jest.config.js` | Test configuration |
| `.env.example` | Environment template |

## Important Considerations

1. **Never commit `.env` files** - Contains API credentials
2. **API key security** - Use Private Integrations, not regular API keys
3. **Location ID scope** - All operations are scoped to a single location
4. **Type safety** - Always define types in `ghl-types.ts`
5. **Error handling** - Let MCP framework handle error formatting
