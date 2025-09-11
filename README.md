# ioBroker MCP Server

> **Built with Vibe Coding** - This project was developed using advanced AI-assisted coding techniques for optimal performance and maintainability.

A comprehensive MCP (Model Context Protocol) server for integrating ioBroker with AI assistants like Claude/Cursor.

## Overview

This project provides a standalone MCP server that communicates with ioBroker via REST API. The server enables AI assistants to gain full access to ioBroker functionalities.

## Features

- **No ioBroker adapter installation required** - Works as standalone server
- **Full ioBroker integration** - Access to states, objects, history, files, and adapter control
- **Intelligent timezone handling** - Automatic local timezone detection and conversion
- **Robust error handling** - Comprehensive error reporting and fallback mechanisms
- **Simple configuration** - Environment-based setup
- **Real-time data access** - History queries with flexible time formats

## Prerequisites

- Node.js 18 or higher
- ioBroker with REST API enabled
- ioBroker History adapter (for historical data queries)
- Network access to ioBroker instance

## Installation

```bash
# Clone or download this folder
cd standalone-mcp-server

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Edit .env with your ioBroker settings
```

## Configuration

Edit `.env` file:

```env
IOBROKER_HOST=192.168.1.19
IOBROKER_PORT=8087
IOBROKER_PROTOCOL=http
```

## Usage

### Direct usage with stdio:

```bash
npm run build
npm start
```

### With MCP Inspector:

```bash
npx @modelcontextprotocol/inspector npm start
```

### With Cursor:

Add to Cursor MCP settings:

```json
{
  "mcpServers": {
    "iobroker": {
      "command": "node",
      "args": ["C:/path/to/standalone-mcp-server/dist/index.js"]
    }
  }
}
```

## MCP Tools Status

### ✅ **Working Tools (9/13)**

#### **Core State Operations**
- **`getState`** ✅ - Read a single state value
- **`setState`** ✅ - Write a state value with acknowledgment
- **`getStates`** ✅ - Read multiple states with pattern matching

#### **Object Management**
- **`getObject`** ✅ - Get detailed object information
- **`getObjects`** ✅ - Get multiple objects with pattern and type filtering

#### **Historical Data**
- **`getHistory`** ✅ - Query historical data with intelligent timezone handling
  - Supports relative time formats: "5m", "2h", "3d", "1w"
  - Supports ISO date strings: "2025-08-16T10:00:00"
  - Automatic local timezone detection and conversion
  - Flexible aggregation options (min, max, avg, sum, count, onchange)

#### **Adapter Control**
- **`sendTo`** ✅ - Send commands to adapters
  - Adapter restart/start/stop functionality
  - General message sending to adapter instances
- **`startScript`** ✅ - Start JavaScript scripts with adapter status checking

#### **File Operations**
- **`writeFile`** ✅ - Write files to ioBroker adapters with base64 encoding

#### **System Operations**
- **`logMessage`** ✅ - Add log entries to ioBroker

### ❌ **Non-working Tools (4/13)**

#### **File Operations**
- **`readFile`** ❌ - "Not exists" (Status: 500)
  - **Problem:** File doesn't exist or path is incorrect
  - **Solution:** Use valid file paths or better error handling

#### **System Operations**
- **`readLogs`** ⚠️ - "no file loggers" (Status: 500) - **IMPROVED**
  - **Problem:** ioBroker not configured for file logging
  - **Solution:** Now provides helpful error message with configuration instructions
  - **Status:** Enhanced error handling with user guidance

#### **Adapter Control**
- **`sendTo` (for other commands)** ❌ - Timeout (30s) for certain commands
  - **Problem:** Certain commands (like `getScripts`) timeout
  - **Solution:** Increase timeout or command-specific handling

#### **Script Management**
- **`startScript`** ❌ - "javascript.0 adapter is not running"
  - **Problem:** JavaScript adapter is not running
  - **Solution:** Start adapter or adapt tool for non-running adapters

## Special Features

### **Intelligent Timezone Handling**
- Automatic detection of local timezone
- Flexible input formats (relative times, ISO dates)
- Consistent output in local timezone
- UTC conversion for API calls

### **Robust Error Handling**
- Comprehensive error reporting
- Fallback mechanisms for critical operations
- User-friendly error messages

### **Performance Optimizations**
- Optimized adapter restart logic
- Efficient history queries
- Reduced API latency

## Examples

### Query recent sensor data:
```
getHistory for tuya.0.bf7200ddef31e20s5bgr46.106, last 35 minutes
```

### Restart an adapter:
```
sendTo shelly.0 with command restart
```

### Read system states:
```
getStates with pattern system.adapter.admin.0.*
```

## Limitations

Compared to the full adapter approach, this standalone server:
- Cannot subscribe to state changes (no real-time events)
- Has higher latency due to HTTP overhead
- Requires REST API to be enabled and accessible
- Some operations depend on ioBroker configuration (file logging, history adapter)

## Known Issues & Solutions

### **Adapter Restarts Take 1-2 Minutes**
- **Cause:** Complex adapters with many connected devices
- **Solution:** Normal behavior for network adapters (Shelly, etc.)

### **"no file loggers" Error**
- **Cause:** ioBroker not configured for file logging
- **Solution:** Configure ioBroker or disable tool

### **"Not exists" with readFile**
- **Cause:** File doesn't exist or adapter not accessible
- **Solution:** Use valid adapter instances (e.g., `admin.0`)

### **Timeout errors**
- **Cause:** Complex operations may take longer than 30 seconds
- **Solution:** Increase timeout or optimize operations

## Performance Notes

- Adapter restarts may take 1-2 minutes for complex adapters
- History queries are optimized for reasonable time ranges
- File operations work best with valid adapter instances

## When to Use

This standalone approach is ideal when:
- You need comprehensive ioBroker access from AI assistants
- You don't want to modify your ioBroker installation
- You need historical data analysis capabilities
- You want to control adapters and scripts remotely
- You're building AI-powered home automation workflows

## Additional Documentation

- **[/standalone-mcp-server/CURSOR_SETUP.md](./standalone-mcp-server/CURSOR_SETUP.md)** - Cursor-specific setup

## License

MIT License - see [LICENSE](LICENSE) file for details. 