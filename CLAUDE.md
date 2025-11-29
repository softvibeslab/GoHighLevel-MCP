# CLAUDE.md - GoHighLevel MCP Server

This document provides guidance for AI assistants working with this codebase.

## Project Overview

This is a **Model Context Protocol (MCP) server** that provides 269+ tools for integrating Claude Desktop and ChatGPT with the GoHighLevel CRM platform. The server exposes GoHighLevel's sub-account level APIs through MCP, enabling AI assistants to manage contacts, conversations, calendars, invoices, and more.

### Key Purpose
- Connect GoHighLevel CRM to AI systems via MCP protocol
- Provide both CLI (stdio) and HTTP (SSE) server modes
- Support Claude Desktop and ChatGPT integrations

## Architecture

```
ghl-mcp-server/
├── src/
│   ├── server.ts              # CLI MCP server (stdio transport for Claude Desktop)
│   ├── http-server.ts         # HTTP MCP server (SSE transport for web apps)
│   ├── clients/
│   │   └── ghl-api-client.ts  # Core GoHighLevel API client (axios-based)
│   ├── tools/                 # MCP tool implementations (19 tool classes)
│   │   ├── contact-tools.ts   # 31 contact management tools
│   │   ├── conversation-tools.ts  # 20 messaging tools
│   │   ├── blog-tools.ts      # 7 blog tools
│   │   ├── opportunity-tools.ts   # 10 sales pipeline tools
│   │   ├── calendar-tools.ts  # 14 calendar/appointment tools
│   │   ├── invoices-tools.ts  # 39 invoice/billing tools
│   │   └── ... (14 more tool files)
│   └── types/
│       └── ghl-types.ts       # Comprehensive TypeScript type definitions
├── tests/
│   ├── setup.ts               # Jest test configuration
│   ├── mocks/                 # API client mocks
│   └── tools/                 # Tool unit tests
├── api/
│   └── index.js               # Vercel API route
└── dist/                      # Compiled JavaScript output
```

## Tech Stack

- **Runtime**: Node.js 18+ (LTS)
- **Language**: TypeScript (strict mode, ES2022 target)
- **Module System**: NodeNext (ESM)
- **MCP SDK**: `@modelcontextprotocol/sdk` ^1.12.1
- **HTTP Framework**: Express 5.x
- **HTTP Client**: Axios ^1.9.0
- **Testing**: Jest with ts-jest
- **Build**: TypeScript compiler (tsc)

## Quick Reference Commands

```bash
# Build
npm run build              # Compile TypeScript to dist/

# Development
npm run dev                # Dev server with hot reload (nodemon + ts-node)

# Production
npm start                  # HTTP server (http-server.ts)
npm run start:stdio        # CLI server for Claude Desktop (server.ts)
npm run start:http         # Same as npm start

# Testing
npm test                   # Run all tests
npm run test:watch         # Watch mode
npm run test:coverage      # Generate coverage report

# Linting
npm run lint               # TypeScript type checking (tsc --noEmit)
```

## Environment Variables

Required:
```bash
GHL_API_KEY=<private_integrations_api_key>  # NOT regular API key
GHL_LOCATION_ID=<your_location_id>
GHL_BASE_URL=https://services.leadconnectorhq.com  # Optional, has default
```

Optional:
```bash
PORT=8000                  # HTTP server port
MCP_SERVER_PORT=8000       # Alternative port variable
NODE_ENV=production        # Environment mode
CORS_ORIGINS=*             # CORS configuration
```

## Code Conventions

### Tool Class Pattern

Each tool category follows this pattern (`src/tools/*-tools.ts`):

```typescript
import { Tool } from '@modelcontextprotocol/sdk/types.js';
import { GHLApiClient } from '../clients/ghl-api-client.js';

export class ExampleTools {
  constructor(private ghlClient: GHLApiClient) {}

  // Return MCP tool definitions
  getToolDefinitions(): Tool[] {
    return [
      {
        name: 'example_tool',
        description: 'Tool description for AI understanding',
        inputSchema: {
          type: 'object',
          properties: {
            param1: { type: 'string', description: 'Parameter description' }
          },
          required: ['param1']
        }
      }
    ];
  }

  // Route tool calls to implementations
  async executeTool(name: string, args: any): Promise<any> {
    switch (name) {
      case 'example_tool':
        return this.exampleMethod(args);
      default:
        throw new Error(`Unknown tool: ${name}`);
    }
  }

  // Private implementation methods
  private async exampleMethod(params: ExampleParams): Promise<ExampleResult> {
    const result = await this.ghlClient.someApiMethod(params);
    return {
      success: true,
      data: result.data,
      message: 'Operation completed successfully'
    };
  }
}
```

### API Client Pattern

The `GHLApiClient` class in `src/clients/ghl-api-client.ts`:
- Uses Axios for HTTP requests
- Includes Bearer token authentication
- Handles API versioning via headers
- Provides typed responses using `GHLApiResponse<T>`
- Implements comprehensive error handling

### Type Definitions

All types are defined in `src/types/ghl-types.ts`:
- Use `GHL` prefix for API types (e.g., `GHLContact`, `GHLOpportunity`)
- Use `MCP` prefix for tool parameter types (e.g., `MCPCreateContactParams`)
- Include JSDoc comments for complex types

### Import Conventions

- Use `.js` extension in imports for NodeNext compatibility
- Import from `@modelcontextprotocol/sdk/types.js` for MCP types
- Group imports: SDK, client, types

## Testing Patterns

Tests use Jest with mocks to avoid real API calls:

```typescript
// tests/tools/example-tools.test.ts
import { ExampleTools } from '../../src/tools/example-tools.js';
import { MockGHLApiClient } from '../mocks/ghl-api-client.mock.js';

describe('ExampleTools', () => {
  let tools: ExampleTools;
  let mockClient: MockGHLApiClient;

  beforeEach(() => {
    mockClient = new MockGHLApiClient();
    tools = new ExampleTools(mockClient as any);
  });

  it('should return correct tool definitions', () => {
    const defs = tools.getToolDefinitions();
    expect(defs).toHaveLength(expectedCount);
  });

  it('should execute tool successfully', async () => {
    const result = await tools.executeTool('tool_name', { param: 'value' });
    expect(result.success).toBe(true);
  });
});
```

Coverage threshold is 70% for branches, functions, lines, and statements.

## Adding New Tools

1. **Create tool file** in `src/tools/`:
   - Follow the tool class pattern above
   - Define tool schemas with clear descriptions
   - Implement error handling

2. **Add types** in `src/types/ghl-types.ts`:
   - API response types (GHL prefix)
   - Tool parameter types (MCP prefix)

3. **Add API methods** in `src/clients/ghl-api-client.ts`:
   - Implement the GoHighLevel API calls
   - Use proper error handling

4. **Register in servers**:
   - Import and instantiate in both `server.ts` and `http-server.ts`
   - Add to tool routing (is*Tool methods and switch statements)

5. **Write tests** in `tests/tools/`:
   - Test tool definitions
   - Test execution with mocks
   - Test error handling

## Tool Categories

| Category | Tools | File |
|----------|-------|------|
| Contacts | 31 | contact-tools.ts |
| Conversations | 20 | conversation-tools.ts |
| Invoices & Billing | 39 | invoices-tools.ts |
| Locations | 24 | location-tools.ts |
| Payments | 20 | payments-tools.ts |
| Store | 18 | store-tools.ts |
| Social Media | 17 | social-media-tools.ts |
| Calendar | 14 | calendar-tools.ts |
| Opportunities | 10 | opportunity-tools.ts |
| Products | 10 | products-tools.ts |
| Associations | 10 | association-tools.ts |
| Custom Objects | 9 | object-tools.ts |
| Custom Fields V2 | 8 | custom-field-v2-tools.ts |
| Blog | 7 | blog-tools.ts |
| Email | 5 | email-tools.ts |
| Media | 3 | media-tools.ts |
| Surveys | 2 | survey-tools.ts |
| Workflows | 1 | workflow-tools.ts |
| Email ISV | 1 | email-isv-tools.ts |

## Deployment

### Vercel (Recommended)
- Uses `vercel.json` configuration
- Builds with `npm run vercel-build`
- Entry point: `dist/http-server.js`

### Railway
- Uses `railway.json` configuration
- Builds with `npm run build`
- Uses `Procfile` for process management

### Docker
- Uses `Dockerfile` with Node.js 18 Alpine
- Exposes port 8000
- Runs `npm start` (HTTP server)

### Claude Desktop
Configure in `mcp_settings.json`:
```json
{
  "mcpServers": {
    "ghl-mcp-server": {
      "command": "node",
      "args": ["path/to/dist/server.js"],
      "env": {
        "GHL_API_KEY": "your_key",
        "GHL_LOCATION_ID": "your_location"
      }
    }
  }
}
```

## Common Issues

### Build Errors
- Run `rm -rf node_modules dist && npm install && npm run build`
- Ensure Node.js 18+ is installed

### Import Errors
- Use `.js` extension in TypeScript imports for NodeNext module resolution
- Example: `import { X } from './file.js'` (not `./file.ts` or `./file`)

### API Authentication
- Must use **Private Integrations API key** (not regular API key)
- Get from: GHL Settings > Integrations > Private Integrations
- Ensure required scopes are enabled

### Test Failures
- Tests require mock setup in `tests/mocks/`
- Environment variables are mocked in `tests/setup.ts`

## API Versioning

The server uses GoHighLevel API versions:
- **Contacts API**: v2021-07-28
- **Conversations API**: v2021-04-15
- Headers: `Version: {api_version}` and `Authorization: Bearer {token}`

## Security Notes

- Never commit API keys (use environment variables)
- Private Integration keys provide FULL sub-account access
- Implement rate limiting for production use
- Monitor API usage to avoid GHL rate limits
- Test in development environments first
