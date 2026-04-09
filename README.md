# mcp-sample

Minimal sample implementation of MCP server. For learning purposes.

Start with reading [the MCP overview tutorial](./mcp-intro.md) to understand the basics of MCP and how it works.

## How to build and run

```bash
npm run build
npm start
```

## How to test with curl

```bash
# Step 1: Initialize
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05"}}'

# Step 2: Send initialized notification (expect 204, no body)
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"notifications/initialized"}'

# Step 3: List tools
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'

# Step 4: Call a tool
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"get_weather","arguments":{"city":"Prague"}}}'

# Step 5: Call add_numbers tool
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{"name":"add_numbers","arguments":{"a":5,"b":3}}}'
```
## Integration with GitHub Copilot

GitHub Copilot support for MCP servers is available through:

1. **VS Code with Copilot Extension**:
   - Install the [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension
   - Configure your MCP servers in your workspace settings or global settings

2. **Configuration**:
   Add to `.vscode/mcp.json`:
   ```json
   {
     "servers": {
       "my-mcp-server-7ac00f60": {
         "url": "http://localhost:3000/mcp",
         "type": "http"
       }
     },
     "inputs": []
   }   
   ```

   Make sure your MCP server is running before using Copilot.
   * Cmd+Shift+P → "MCP: List Servers" — check if mcp-sample shows as connected
   * Cmd+Shift+P → "Developer: Reload Window" — forces VS Code to re-initialize MCP connections

## Testing with Your IDE

Once integrated **Ask Copilot** (in chat or inline):
   - "Use the add_numbers tool to add 15 and 20"
   - "Get the weather for London"