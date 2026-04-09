# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A minimal, educational Model Context Protocol (MCP) server implementation using only Node.js built-in modules (no Express or other frameworks). Intended for learning the MCP protocol mechanics.

## Commands

```bash
npm run build   # Compile TypeScript → dist/
npm start       # Run compiled server (node dist/index.js)
```

No test suite is configured — test manually with curl (see README.md for full examples):

```bash
# Initialize
curl -s -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'

# List tools
curl -s -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/list","params":{}}'
```

## Architecture

All implementation lives in a single file: `src/index.ts`.

**Request flow:**

```
HTTP POST /mcp → JSON parse → handleJsonRpc() → executeTool() → JSON response
```

**Key sections in `src/index.ts`:**

- `TOOLS` array — static tool definitions (name, description, JSON Schema `inputSchema`)
- `executeTool(name, args)` — dispatches tool calls; add new tool cases here
- `handleJsonRpc()` — routes the 3 MCP methods: `initialize`, `tools/list`, `tools/call`; notifications (no `id`) return HTTP 204
- HTTP server — listens on port 3000, single endpoint `POST /mcp`

**Adding a new tool:**
1. Add an entry to the `TOOLS` array with `name`, `description`, and `inputSchema`
2. Add a `case` in `executeTool()` returning `{ content: [{ type: "text", text: "..." }] }`

## Protocol Notes

This server implements JSON-RPC 2.0 over HTTP as specified by the MCP protocol. See `mcp-intro.md` for a full walkthrough of the protocol lifecycle (initialize → tools/list → tools/call).

## TypeScript Configuration

- ES modules (`"type": "module"` in package.json, `moduleResolution: nodenext`)
- Strict mode enabled; use `import type` for type-only imports (`verbatimModuleSyntax`)
- Source compiled to `dist/` — always run `npm run build` before `npm start`
