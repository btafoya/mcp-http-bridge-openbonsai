# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Critical Context (Read First)
- **Tech Stack**: Node.js 18+ (uses native fetch), JSON-RPC over stdio
- **Main File**: `mcp-http-bridge-openbonsai` (100 lines) - Node.js executable script
- **Core Mechanic**: Bridges stdio JSON-RPC messages to HTTP endpoint for OpenBonsai MCP Server
- **Key Integration**: OpenBonsai MCP Server at configurable HTTP endpoint
- **Platform Support**: Any system with Node.js 18+
- **DO NOT**: Remove API key validation, expose API keys in logs, or break stdio communication

## Quick Reference
**Main Handler**: `handleStdio():21-56` - Processes stdin JSON-RPC messages  
**HTTP Bridge**: `makeHttpRequest():58-95` - Forwards requests to OpenBonsai endpoint  
**Configuration**: `lines 12-13` - MCP_SERVER_URL and MCP_API_KEY environment variables  
**Error Handling**: `lines 72-79, 85-93` - HTTP and connection error responses  

## Architecture
This is a simple stdio-to-HTTP bridge that:
1. Receives JSON-RPC messages via stdin
2. Forwards them to an HTTP endpoint (OpenBonsai MCP Server)
3. Returns responses via stdout
4. Uses console.error for debug logging (doesn't interfere with JSON-RPC communication)

The bridge maintains compatibility with the MCP (Model Context Protocol) specification by preserving JSON-RPC message structure and handling errors according to the protocol.

## Environment Variables
- `MCP_SERVER_URL` - HTTP endpoint for OpenBonsai MCP Server (default: `http://localhost:3988/api/mcp/compliant`)
- `MCP_API_KEY` - Required API key for authentication with OpenBonsai

## Development Commands
```bash
# Run the bridge directly
MCP_API_KEY=your-key node mcp-http-bridge-openbonsai

# Make executable and run
chmod +x mcp-http-bridge-openbonsai
MCP_API_KEY=your-key ./mcp-http-bridge-openbonsai

# Test with custom server URL
MCP_SERVER_URL=https://your-server.com/api/mcp MCP_API_KEY=your-key ./mcp-http-bridge-openbonsai

# Debug mode (logs to stderr)
MCP_API_KEY=your-key node mcp-http-bridge-openbonsai 2>debug.log
```

## Common Development Tasks

### Testing the Bridge
1. Set up environment variables
2. Run the bridge script
3. Send JSON-RPC messages via stdin
4. Monitor stderr for debug output
5. Verify responses on stdout

### Modifying Error Handling
- HTTP errors: Update `lines 72-79`
- Connection errors: Update `lines 85-93`
- JSON parsing errors: Update `lines 45-48`

### Adding Features
- Request transformation: Modify before `line 68`
- Response transformation: Modify after `line 82`
- Additional headers: Update `lines 64-67`

## Anti-Patterns (Avoid These)
❌ **Don't log to stdout** - This breaks JSON-RPC communication (use console.error)  
❌ **Don't remove API key check** - Security requirement for OpenBonsai  
❌ **Don't mix response formats** - Keep all responses as valid JSON-RPC  
❌ **Don't buffer entire response** - Process line-by-line for streaming support  
❌ **Don't expose API keys** - Never log the actual API key value  

## Current State
- [x] Basic stdio-to-HTTP bridge functionality
- [x] JSON-RPC message handling
- [x] Error response formatting
- [x] Environment variable configuration
- [ ] Connection retry logic
- [ ] Request/response logging options
- [ ] Health check endpoint support