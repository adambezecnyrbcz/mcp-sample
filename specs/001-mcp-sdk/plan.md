# Plan: Alternative MCP Server Using @modelcontextprotocol/sdk

## Background

The current server in `src/index.ts` (~220 lines) manually implements the full MCP protocol:
JSON-RPC routing, `initialize` handshake, `tools/list`, error codes, content block formatting,
and an HTTP server. The official `@modelcontextprotocol/sdk` package handles all of this,
reducing the implementation to business logic only (~30 lines).

The existing `src/index.ts` will be kept untouched as a reference. The SDK-based implementation
will live alongside it in `src/server2.ts`.

## SDK

**Package:** `@modelcontextprotocol/sdk` (official, published by the MCP/Anthropic team)
**Schema validation:** `zod` (replaces raw JSON Schema `inputSchema` objects)

```bash
npm install @modelcontextprotocol/sdk zod
```

## What disappears vs. what remains

| Current manual code | With SDK |
|---|---|
| `handleJsonRpc()` router | Gone — SDK handles it |
| `initialize` method | Gone — SDK handles handshake |
| `tools/list` method | Gone — auto-generated from registrations |
| Error codes (-32600, etc.) | Gone — SDK throws typed errors |
| HTTP server setup | Gone (stdio) or one-liner (HTTP) |
| Tool definitions + dispatch | **Stays** — your actual business logic |

## Target structure: `src/server2.ts`

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "mcp-sample", version: "1.0.0" });

server.registerTool(
  "get_weather",
  {
    description: "Get current weather for a city",
    inputSchema: { city: z.string().describe("City name") },
  },
  async ({ city }) => ({
    content: [{ type: "text", text: `Weather in ${city}: Sunny, 22°C` }],
  })
);

server.registerTool(
  "add_numbers",
  {
    description: "Add two numbers",
    inputSchema: { a: z.number(), b: z.number() },
  },
  async ({ a, b }) => ({
    content: [{ type: "text", text: String(a + b) }],
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Transport choice: stdio vs. HTTP

| Transport | When to use |
|---|---|
| **`StdioServerTransport`** | Claude Desktop, Claude Code, most MCP hosts — they spawn the server as a subprocess |
| **`StreamableHttpServerTransport`** | When HTTP is needed (e.g. Copilot via VS Code, remote deployment) |

For HTTP with the SDK, a minimal native http wrapper around `StreamableHttpServerTransport`
is still far less code than the current manual approach (~20 lines vs. current ~40).

## Implementation steps

1. **Install deps** — `npm install @modelcontextprotocol/sdk zod`
2. **Create `src/server2.ts`** — new file with the SDK-based implementation shown above; `src/index.ts` remains unchanged
3. **Use HTTP as server transport**: wrap `StreamableHttpServerTransport` in a tiny native http handler
4. **Update `package.json`** — add `zod` as a runtime dependency; add a `start2` script pointing to `dist/server2.js`
5. **Update `package.json`** — change  `start` script pointing to `dist/server.js`. Rename src/index.ts to src/server.ts. Change all existing references to `src/index.ts` to `src/server.ts`