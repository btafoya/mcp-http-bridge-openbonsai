# mcp-http-bridge-openbonsai

HTTP bridge for OpenBonsai MCP Server - enables Claude Code and other MCP clients to communicate with OpenBonsai via HTTP.

## Overview

This package provides a stdio-to-HTTP bridge that allows MCP (Model Context Protocol) clients like Claude Code to communicate with OpenBonsai's HTTP-based MCP server. It translates JSON-RPC messages from stdio to HTTP requests and forwards them to the OpenBonsai endpoint.

## Features

- 🔄 Bidirectional JSON-RPC message translation
- 🔐 Secure API key authentication with validation
- 📝 Enhanced debug logging with timestamp and message tracking
- ⚡ Native Node.js 18+ fetch API (no external dependencies)
- 🛡️ Comprehensive error handling with JSON-RPC 2.0 compliance
- 🔁 Automatic retry logic with exponential backoff for network failures
- ⏱️ Configurable request timeout protection
- ✅ Input validation and sanitization for all requests
- 🚦 Graceful shutdown handling (SIGTERM/SIGINT)
- 🎯 Smart retry strategy (retries on 5xx and network errors, not on 4xx)

## Requirements

- Node.js 18.0.0 or higher
- OpenBonsai API key
- Access to OpenBonsai MCP Server endpoint

## Installation

### Global Installation (Recommended)

```bash
npm install -g mcp-http-bridge-openbonsai
```

### Local Installation

```bash
npm install mcp-http-bridge-openbonsai
```

## Configuration

The bridge accepts the following environment variables, passed directly through the MCP configuration:

### Required
- `MCP_API_KEY`: Your OpenBonsai API key

### Optional
- `MCP_SERVER_URL`: OpenBonsai MCP endpoint URL
  - Default: `http://localhost:3988/api/mcp/compliant`
- `DEBUG`: Enable debug logging (`true` or `1`)
  - Default: `false`
- `MAX_RETRIES`: Maximum number of retry attempts for failed requests
  - Default: `3`
- `RETRY_DELAY`: Initial delay in milliseconds between retries (uses exponential backoff)
  - Default: `1000` (1 second)
- `REQUEST_TIMEOUT`: Request timeout in milliseconds
  - Default: `30000` (30 seconds)

Note: These are passed directly in the MCP configuration, not through `.env` files.

## Usage

### As a Global Command

```bash
# Basic usage
MCP_API_KEY=your-api-key mcp-http-bridge-openbonsai

# With custom server URL
MCP_SERVER_URL=https://your-server.com/api/mcp MCP_API_KEY=your-api-key mcp-http-bridge-openbonsai

# With debug logging
MCP_API_KEY=your-api-key mcp-http-bridge-openbonsai 2>debug.log
```

### In Claude Code Configuration

#### Dynamic Installation (Recommended)

Add this to your Claude Code MCP settings. The package will be automatically downloaded and kept up-to-date:

```json
{
  "mcpServers": {
    "openbonsai": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-http-bridge-openbonsai@latest"
      ],
      "env": {
        "MCP_SERVER_URL": "http://localhost:3988/api/mcp/compliant",
        "MCP_API_KEY": "your-api-key-here",
        "DEBUG": "false",
        "MAX_RETRIES": "3",
        "REQUEST_TIMEOUT": "30000"
      }
    }
  }
}
```

**Key Features:**
- ✅ No manual installation required
- ✅ Always uses the latest version
- ✅ API key and URL passed directly from config
- ✅ The `-y` flag ensures automatic installation without prompts

#### Version Pinning (For Stability)

To use a specific version instead of latest:

```json
{
  "mcpServers": {
    "openbonsai": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-http-bridge-openbonsai@1.0.0"
      ],
      "env": {
        "MCP_SERVER_URL": "https://your-server.com/api/mcp/compliant",
        "MCP_API_KEY": "your-api-key"
      }
    }
  }
}
```

#### Alternative: Pre-installed Global Package

If you prefer to install once globally:

```bash
npm install -g mcp-http-bridge-openbonsai
```

Then use this simpler config:

```json
{
  "mcpServers": {
    "openbonsai": {
      "command": "mcp-http-bridge-openbonsai",
      "env": {
        "MCP_API_KEY": "your-api-key",
        "MCP_SERVER_URL": "http://localhost:3988/api/mcp/compliant"
      }
    }
  }
}
```

### Programmatic Usage

```javascript
const { spawn } = require('child_process');

const bridge = spawn('mcp-http-bridge-openbonsai', [], {
  env: {
    ...process.env,
    MCP_API_KEY: 'your-api-key',
    MCP_SERVER_URL: 'https://your-server.com/api/mcp'
  }
});

// Send JSON-RPC request
bridge.stdin.write(JSON.stringify({
  jsonrpc: '2.0',
  method: 'tools/list',
  id: 1
}) + '\n');

// Receive response
bridge.stdout.on('data', (data) => {
  const response = JSON.parse(data.toString());
  console.log('Response:', response);
});

// Monitor debug output
bridge.stderr.on('data', (data) => {
  console.error('Debug:', data.toString());
});
```

## How It Works

1. **Receives** JSON-RPC messages via stdin
2. **Forwards** requests to OpenBonsai HTTP endpoint with authentication
3. **Returns** responses via stdout
4. **Logs** debug information to stderr

The bridge maintains full JSON-RPC compatibility and handles errors according to the protocol specification.

## Error Handling

The bridge handles various error scenarios:

- Missing API key: Process exits with error message
- HTTP errors: Returns JSON-RPC error response with HTTP status
- Connection failures: Returns JSON-RPC error with connection details
- Invalid JSON: Logs parsing errors to stderr

## Development

### Clone and Setup

```bash
git clone https://github.com/btafoya/mcp-http-bridge-openbonsai.git
cd mcp-http-bridge-openbonsai
npm install
```

### Testing

```bash
# Run with test credentials
MCP_API_KEY=test-key npm start

# Manual testing with echo
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | MCP_API_KEY=test-key node mcp-http-bridge-openbonsai
```

## Troubleshooting

### Bridge not starting
- Ensure Node.js 18+ is installed: `node --version`
- Verify MCP_API_KEY is set: `echo $MCP_API_KEY`

### Connection errors
- Check MCP_SERVER_URL is accessible
- Verify API key is valid
- Check network connectivity to OpenBonsai server

### JSON-RPC errors
- Enable debug logging: `2>debug.log`
- Check stderr output for detailed error messages
- Verify request format matches JSON-RPC 2.0 specification

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues and questions:
- GitHub Issues: [Report a bug](https://github.com/btafoya/mcp-http-bridge-openbonsai/issues)
- OpenBonsai Documentation: [Official Docs](https://openbonsai.com/docs)

## Changelog

### 1.1.0
- Enhanced error handling with proper JSON-RPC 2.0 error codes
- Added automatic retry logic with exponential backoff
- Implemented request timeout protection
- Added comprehensive input validation
- Enhanced debug logging with timestamps and message counting
- Added graceful shutdown handling (SIGTERM/SIGINT)
- Improved error messages with more context
- Added configurable retry and timeout settings
- Smart retry strategy (retry on 5xx and network errors, not on 4xx)

### 1.0.1
- Fixed npm package configuration
- Updated documentation

### 1.0.0
- Initial release
- Basic stdio-to-HTTP bridge functionality
- JSON-RPC 2.0 support
- Environment variable configuration