# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
npm run build          # TypeScript compile + build CLI (tsc + scripts/build-cli.js)
npm test               # Run all tests with vitest
npm run test:watch     # Run tests in watch mode
npm run test:coverage  # Run tests with coverage report
npm run dev            # Development mode with tsx watch
```

## Architecture Overview

This is an MCP (Model Context Protocol) server that exposes the Notion API as MCP tools. It works by converting an OpenAPI specification into MCP-compatible tools that LLMs can call.

### API Version 2025-09-03

The server implements the Notion API version 2025-09-03 which separates database and data source concepts:
- **Databases** are containers that can hold multiple data sources
- **Data sources** contain the actual schema and data (what was previously called "database")

Available endpoint categories: Databases, Data sources, Pages, Blocks, Users, Search, Comments, File uploads, OAuth

### Core Flow

1. **Entry Point**: `scripts/start-server.ts` - Parses CLI args and initializes the server with either stdio or HTTP transport
2. **Server Initialization**: `src/init-server.ts` - Loads the OpenAPI spec from `scripts/notion-openapi.json` and creates the MCPProxy
3. **MCP Proxy**: `src/openapi-mcp-server/mcp/proxy.ts` - The main server class that:
   - Uses `OpenAPIToMCPConverter` to convert OpenAPI operations into MCP tools
   - Handles `ListToolsRequest` to expose available tools
   - Handles `CallToolRequest` to execute API operations via HttpClient
   - Manages authentication via `NOTION_TOKEN` or `OPENAPI_MCP_HEADERS` env vars

### Key Components

- **OpenAPIToMCPConverter** (`src/openapi-mcp-server/openapi/parser.ts`): Parses the OpenAPI spec and converts each operation into an MCP tool with proper JSON Schema input/output types. Also supports conversion to OpenAI and Anthropic tool formats.

- **HttpClient** (`src/openapi-mcp-server/client/http-client.ts`): Executes API operations using `openapi-client-axios`. Handles parameter separation (path/query vs body), file uploads via multipart/form-data, and error responses.

### Transport Modes

- **stdio** (default): Standard MCP transport for Claude Desktop and similar clients
- **http**: Streamable HTTP transport with bearer token authentication, session management, and a `/health` endpoint

### Authentication

The server supports two authentication methods:
- `NOTION_TOKEN` env var (recommended): Automatically constructs Authorization and Notion-Version headers
- `OPENAPI_MCP_HEADERS` env var: JSON object with custom headers for advanced use cases

## Local Development Testing

To test changes locally with an MCP client:

1. Run `npm link` from repository root
2. Configure your MCP client to use `notion-mcp-server` command with `NOTION_TOKEN` env var
3. Run `npm unlink` when done
