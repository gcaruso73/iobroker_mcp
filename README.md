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

### ✅ **Working Tools (11/13)**

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
  - Flexible aggregation options (min, max, avg, sum, count, onchange, none)
  - **Fixed boolean value handling** - Now correctly displays true/false instead of null

#### **Adapter Control**
- **`sendTo`** ✅ - Send commands to adapters
  - Adapter restart/start/stop functionality
  - General message sending to adapter instances
- **`startScript`** ✅ - Start JavaScript scripts with adapter status checking

#### **File Operations**
- **`writeFile`** ✅ - Write files to ioBroker adapters with base64 encoding

#### **System Operations**
- **`logMessage`** ✅ - Add log entries to ioBroker

### ❌ **Non-working Tools (2/13)**

#### **File Operations**
- **`readFile`** ❌ - "Not exists" (Status: 500)
  - **Problem:** File doesn't exist or path is incorrect
  - **Solution:** Use valid file paths or better error handling

#### **Log Management**
- **`readLogs`** ✅ - Read log file names and sizes with HTTP download links
  - **Enhanced:** Improved error handling with configuration instructions
  - **Feature:** Automatic IP-to-hostname resolution
  - **Feature:** Returns HTTP download links for log files
- **`readCurrentLog`** ✅ - Automatically find and download current day log file
  - **Feature:** Automatic current log file detection
  - **Feature:** HTML content filtering (removes HTML wrapper)
  - **Feature:** Configurable preview length (default: 100,000 characters)
  - **Feature:** Option to show first or last N characters
  - **Feature:** Automatic IP-to-hostname resolution

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

### Query boolean switch history:
```
getHistory for shelly.0.SHPLG-S#9A3649#1.Relay0.Switch, last 20h with aggregate=none
```

### Read current day log:
```
readCurrentLog with previewLength=50000 and showLast=true
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



## Additional Documentation

- **[/standalone-mcp-server/CURSOR_SETUP.md](./standalone-mcp-server/CURSOR_SETUP.md)** - Cursor-specific setup

## License

MIT License - see [LICENSE](LICENSE) file for details. 