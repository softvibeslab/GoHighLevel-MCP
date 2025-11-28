# CLAUDE.md - GoHighLevel MCP Server

## Project Overview

This is a **GoHighLevel MCP (Model Context Protocol) Server** that provides 269+ tools for integrating GoHighLevel CRM with AI assistants like Claude Desktop and ChatGPT. The server enables AI-powered CRM automation across contacts, messaging, calendars, sales pipelines, invoicing, e-commerce, and more.

## Quick Reference

| Aspect | Details |
|--------|---------|
| **Language** | TypeScript |
| **Runtime** | Node.js 18+ |
| **Package Manager** | npm |
| **Build System** | TypeScript Compiler (tsc) |
| **Testing** | Jest |
| **API Protocol** | MCP (Model Context Protocol) |
| **External API** | GoHighLevel API v2021-07-28 |

## Repository Structure

```
GoHighLevel-MCP/
├── src/
│   ├── server.ts              # CLI MCP server (stdio transport for Claude Desktop)
│   ├── http-server.ts         # HTTP MCP server (SSE transport for web apps)
│   ├── clients/
│   │   └── ghl-api-client.ts  # Core GoHighLevel API client
│   ├── tools/                 # 19 tool modules (269+ tools total)
│   │   ├── contact-tools.ts        # 31 tools - Contact management
│   │   ├── conversation-tools.ts   # 20 tools - Messaging & SMS
│   │   ├── calendar-tools.ts       # 14 tools - Appointments
│   │   ├── opportunity-tools.ts    # 10 tools - Sales pipeline
│   │   ├── invoices-tools.ts       # 39 tools - Billing & estimates
│   │   ├── location-tools.ts       # 24 tools - Sub-account management
│   │   ├── payments-tools.ts       # 20 tools - Payment processing
│   │   ├── store-tools.ts          # 18 tools - E-commerce
│   │   ├── social-media-tools.ts   # 17 tools - Social posting
│   │   ├── products-tools.ts       # 10 tools - Product catalog
│   │   ├── object-tools.ts         # 9 tools - Custom objects
│   │   ├── association-tools.ts    # 10 tools - Data relationships
│   │   ├── custom-field-v2-tools.ts # 8 tools - Custom fields
│   │   ├── blog-tools.ts           # 7 tools - Blog management
│   │   ├── email-tools.ts          # 5 tools - Email campaigns
│   │   ├── media-tools.ts          # 3 tools - Media library
│   │   ├── survey-tools.ts         # 2 tools - Surveys
│   │   ├── workflow-tools.ts       # 1 tool - Workflow discovery
│   │   └── email-isv-tools.ts      # 1 tool - Email verification
│   └── types/
│       └── ghl-types.ts       # Comprehensive TypeScript type definitions
├── tests/
│   ├── basic.test.ts          # Environment setup tests
│   ├── setup.ts               # Test configuration
│   ├── clients/               # API client tests
│   ├── tools/                 # Tool implementation tests
│   └── mocks/                 # Mock data and fixtures
├── api/
│   └── index.js               # Vercel serverless function entry
├── dist/                      # Compiled JavaScript (auto-generated)
├── package.json               # Dependencies and scripts
├── tsconfig.json              # TypeScript configuration
├── jest.config.js             # Testing configuration
├── Dockerfile                 # Container deployment
├── vercel.json                # Vercel deployment config
└── railway.json               # Railway deployment config
```

## Development Commands

```bash
# Install dependencies
npm install

# Development with hot reload
npm run dev

# Build TypeScript
npm run build

# Run tests
npm test                    # Run all tests
npm run test:watch          # Watch mode
npm run test:coverage       # With coverage report

# Type checking (linting)
npm run lint

# Production servers
npm start                   # HTTP server (web apps)
npm run start:stdio         # CLI server (Claude Desktop)
npm run start:http          # HTTP server (explicit)
```

## Environment Configuration

Required environment variables (see `.env.example`):

```bash
GHL_API_KEY=               # Private Integrations API key (NOT regular API key)
GHL_LOCATION_ID=           # Sub-account location ID
GHL_BASE_URL=https://services.leadconnectorhq.com
NODE_ENV=development       # or 'production'
PORT=8000                  # Optional, for HTTP server
```

**Important**: The API key must be from GoHighLevel's **Private Integrations** feature (Settings > Integrations > Private Integrations), not a regular API key.

## Architecture Patterns

### Tool Module Pattern

Each tool module in `src/tools/` follows this structure:

```typescript
import { Tool } from '@modelcontextprotocol/sdk/types.js';
import { GHLApiClient } from '../clients/ghl-api-client.js';

export class ExampleTools {
  constructor(private ghlClient: GHLApiClient) {}

  // Returns MCP tool definitions (JSON Schema)
  getToolDefinitions(): Tool[] { ... }
  // or getTools(): Tool[] { ... }

  // Executes tool by name with arguments
  executeTool(name: string, args: Record<string, unknown>): Promise<any> { ... }
  // or executeExampleTool(...): Promise<any> { ... }
}
```

### Server Modes

1. **CLI Mode** (`server.ts`): Uses `StdioServerTransport` for Claude Desktop integration
2. **HTTP Mode** (`http-server.ts`): Uses `SSEServerTransport` for web-based MCP clients

### API Client

The `GHLApiClient` (`src/clients/ghl-api-client.ts`) handles all GoHighLevel API communication:
- Bearer token authentication
- Version headers (`Version: 2021-07-28`)
- Location ID injection
- Error handling and logging

## Type System

All types are defined in `src/types/ghl-types.ts`:
- `GHLConfig` - API configuration
- `GHLContact`, `GHLConversation`, etc. - Entity types
- `MCP*Params` - Tool parameter interfaces
- `GHL*Response` - API response types

Types follow the official GoHighLevel OpenAPI specification (v2021-07-28 for Contacts, v2021-04-15 for Conversations).

## Testing Guidelines

- Test files go in `tests/` directory with `.test.ts` extension
- Use mocks from `tests/mocks/` for API client
- Coverage threshold: 70% for branches, functions, lines, statements
- Set test environment variables in test files or `tests/setup.ts`

Example test structure:
```typescript
// Set environment before imports
process.env.GHL_API_KEY = 'test_key';
process.env.GHL_LOCATION_ID = 'test_location';

describe('ToolName', () => {
  it('should do something', () => {
    // Test implementation
  });
});
```

## Code Conventions

### TypeScript
- **Strict mode** enabled
- **ES2022** target with **NodeNext** module resolution
- Use `.js` extensions in imports (required for ESM)
- Prefer explicit types over `any`

### Naming
- Tool names: `snake_case` (e.g., `create_contact`, `send_sms`)
- Tool prefixes: GHL-specific tools use `ghl_` prefix (e.g., `ghl_get_workflows`)
- Classes: `PascalCase` (e.g., `ContactTools`, `GHLApiClient`)
- Files: `kebab-case` (e.g., `contact-tools.ts`)

### Error Handling
- Use `McpError` from SDK for MCP protocol errors
- Log errors to stderr in stdio mode, console in HTTP mode
- Include context in error messages

## Adding New Tools

1. **Create or update tool module** in `src/tools/`:
   ```typescript
   export class NewTools {
     constructor(private ghlClient: GHLApiClient) {}

     getToolDefinitions(): Tool[] {
       return [{
         name: 'new_tool_name',
         description: 'What this tool does',
         inputSchema: {
           type: 'object',
           properties: { /* JSON Schema */ },
           required: ['requiredParam']
         }
       }];
     }

     async executeTool(name: string, args: Record<string, unknown>) {
       switch (name) {
         case 'new_tool_name':
           return this.executeNewTool(args);
         default:
           throw new Error(`Unknown tool: ${name}`);
       }
     }
   }
   ```

2. **Add types** to `src/types/ghl-types.ts`

3. **Register in servers** (`server.ts` and `http-server.ts`):
   - Import the tool class
   - Initialize in constructor
   - Add to tool definitions list
   - Add routing in `CallToolRequestSchema` handler
   - Add `isNewTool()` helper method

4. **Add tests** in `tests/tools/new-tools.test.ts`

## Deployment

### Local Development
```bash
npm run dev  # HTTP server with hot reload
```

### Claude Desktop Integration
Add to `mcp_settings.json`:
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

### Cloud Platforms
- **Vercel**: `vercel --prod` (uses `vercel.json`)
- **Railway**: `railway up` (uses `railway.json`)
- **Docker**: `docker build -t ghl-mcp . && docker run -p 8000:8000 ghl-mcp`

## API Endpoints (HTTP Mode)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Server info and available endpoints |
| `/health` | GET | Health check with tool counts |
| `/capabilities` | GET | MCP server capabilities |
| `/tools` | GET | List all available tools |
| `/sse` | GET/POST | MCP SSE connection endpoint |

## Troubleshooting

### Common Issues

1. **"GHL_API_KEY environment variable is required"**
   - Ensure `.env` file exists with valid Private Integrations API key

2. **API 401/403 errors**
   - Verify using Private Integrations API key (not regular key)
   - Check required scopes are enabled in GHL Private Integration settings

3. **Build errors**
   ```bash
   rm -rf node_modules dist package-lock.json
   npm install
   npm run build
   ```

4. **Test failures**
   - Ensure test environment variables are set before imports
   - Check mock data matches expected types

## Key Dependencies

| Package | Purpose |
|---------|---------|
| `@modelcontextprotocol/sdk` | MCP protocol implementation |
| `axios` | HTTP client for GHL API |
| `express` | HTTP server framework |
| `dotenv` | Environment variable loading |
| `typescript` | Type system and compilation |
| `jest` + `ts-jest` | Testing framework |

## Security Considerations

- Never commit API keys to version control
- Use environment variables for all credentials
- The server has full access to the configured GHL sub-account
- Implement rate limiting for production deployments
- Monitor API usage to avoid GHL rate limits
