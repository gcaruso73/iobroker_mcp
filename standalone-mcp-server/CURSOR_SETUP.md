# Cursor Integration for ioBroker MCP Server

## Prerequisites

1. ioBroker is running and accessible
2. **ioBroker REST API Adapter installed and configured** ⭐ **IMPORTANT!**
3. Node.js is installed on your Windows PC
4. The Standalone MCP Server is installed and built

### **REST API Installation:**
```bash
# In ioBroker Admin Interface:
# 1. Adapters → Install REST API
# 2. Create instance (e.g., rest-api.0)
# 3. Configure port (default: 8087)
# 4. Enable authentication if desired
# 5. Start instance
# 6. Disable Simple API (important!)
```

**Important**: The MCP server uses exclusively the REST API. The Simple API must be disabled!

## MCP Server Installation

1. Open a terminal in the `standalone-mcp-server` directory
2. Run: `install.bat`
3. Edit the `.env` file with your ioBroker settings:
   ```env
   IOBROKER_HOST=192.168.1.19
   IOBROKER_PORT=8087
   ```

## Cursor Configuration

1. Open Cursor
2. Go to: File → Preferences → Settings
3. Search for "MCP"
4. Click "Edit in settings.json"

5. Add the following configuration:

```json
{
  "mcpServers": {
    "iobroker": {
      "command": "node",
      "args": ["C:\\prog\\iobroker_mcp\\standalone-mcp-server\\dist\\index.js"],
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

**Important**: Adjust the path to your installation!

## Alternative: With MCP-Proxy for Remote Access

If you don't want to run the MCP server locally:

1. Install the MCP-Proxy on the ioBroker system
2. Configure Cursor for remote access:

```json
{
  "mcpServers": {
    "iobroker-remote": {
      "transport": {
        "type": "sse",
        "url": "http://192.168.1.19:3000/sse"
      }
    }
  }
}
```

## Available Commands in Cursor

After configuration, you can use the following commands in Cursor:

### Read states:
```
Show me the value of system.adapter.admin.0.alive
```

### States with special characters:
```
Show me the value of shelly.0.SHPLG-S#9A3649#1.Relay0.Power
```
**Note**: Special characters like `#` are automatically URL-encoded.

### Write states:
```
Set the state hm-rpc.0.ABC123.1.STATE to true
```

### Get multiple states:
```
List all states that start with hm-rpc.0
```

### Get objects:
```
Show me the object system.adapter.admin.0
```

### Script Management (NEW):
```
Start the script javascript.0.Lüfter_Bad_Anbau
Stop the script javascript.0.Lüfter_Bad_Anbau
```

### Log Management (NEW):
```
Show me the log files for localhost
Add a log entry: "Test message" with level "info"
```
**Note:** `readLogs` requires ioBroker to be configured for file logging. If you get "no file loggers" error, enable file logging in ioBroker Admin → Logs tab.

### File Operations (NEW):
```
Read the file vis.0/main/vis-views.json
Write to file vis.0/main/test.json the content: {"test": true}
```

### History Data (NEW):
```
Show me the history data for system.adapter.admin.0.memRss for the last hour
Show me the average values for zigbee.0.00158d008b3d3ece.humidity for the last 24h with 1h steps
Show me the last 10 values for shelly.0.SHPLG-S#9A3649#1.Relay0.Power
```

### Send messages:
```
Send a message to telegram.0 with the text "Hello from Cursor"
```

## Implemented REST API Functions

### ✅ Already implemented (REST API):
- **States**: `getState()`, `setState()`, `getStates()`
- **Objects**: `getObject()`, `getObjects()`
- **Scripts**: `startScript()`, `stopScript()`
- **Logs**: `readLogs()`, `logMessage()`
- **Files**: `readFile()`, `writeFile()`
- **History**: `getHistory()` ⭐
- **Messages**: `sendTo()` (complete)

### ❌ NOT yet implemented:
- **Admin**: `getAdapters()`, `getInstances()`, etc.
- **Binary States**: `getBinaryState()`, `setBinaryState()`
- **Advanced sendTo**: Complex messages with payload

## Troubleshooting

### "Connection refused"
- Check if ioBroker is running
- Check if REST API is enabled
- Check firewall settings

### "Not found: datapoint" with special characters
- **Problem**: Datapoints with `#` are truncated
- **Solution**: The MCP server now automatically uses URL-encoding
- **Example**: `shelly.0.SHPLG-S#9A3649#1.Relay0.Power` is processed correctly

### URL-Encoding for Special Characters
The MCP server automatically handles the following special characters:
- `#` → `%23`
- `%` → `%25`
- `&` → `%26`
- `+` → `%2B`
- `=` → `%3D`
- `@` → `%40`

**Example**:
```
shelly.0.SHPLG-S#9A3649#1.Relay0.Power
```
becomes internally:
```
shelly.0.SHPLG-S%239A3649%231.Relay0.Power
```

### "Function not supported"
- **Problem**: Some REST API functions are not available via Simple API
- **Solution**: Use the full REST API or Admin interface
- **Affected functions**: Complex `sendTo()`, Admin functions

### "Cannot GET /get/..." or "Cannot GET /history"
- **Problem**: REST API adapter is not installed or not started
- **Solution**: 
  1. In ioBroker Admin interface: Adapters → Install REST API
  2. Create instance and configure
  3. Open port 8087
  4. Start instance
  5. **Disable Simple API** (important!)
  6. Restart MCP server

### "Function not supported"
- **Problem**: Some REST API functions are not available via Simple API
- **Solution**: Use the full REST API or Admin interface
- **Affected functions**: Complex `sendTo()`, Admin functions

## History Functions

The `getHistory` function supports various output formats for better LLM readability:

### Available Formats:

1. **`raw`** (default): Complete ioBroker response with all metadata
2. **`simple`**: Simplified table format with [timestamp, value] pairs
3. **`csv`**: CSV-like format for easy processing
4. **`summary`**: Summary with statistics and trend analysis
5. **`hourly`**: Hourly averages
6. **`daily`**: Daily averages

### Example Usage:

```javascript
// Summary of the last hour
getHistory("sensor.id", {start: "1h", end: "now"}, "summary")

// Hourly averages of the last 24 hours
getHistory("sensor.id", {start: "24h", end: "now"}, "hourly")
```

### Format Examples:

**Summary Format:**
```json
{
  "id": "sensor.id",
  "period": "2025-08-16T12:00:00.000Z to 2025-08-16T13:00:00.000Z",
  "statistics": {
    "total_entries": 120,
    "value_range": {"min": 20.5, "max": 25.3},
    "average": 22.8,
    "trend": "increasing",
    "update_frequency": "every_30_seconds"
  },
  "recent_values": [
    ["2025-08-16T13:00:00.000Z", 25.1],
    ["2025-08-16T12:59:30.000Z", 25.0]
  ]
}
```

**Simple Format:**
```json
{
  "id": "sensor.id",
  "data": [
    ["2025-08-16T13:00:00.000Z", 25.1],
    ["2025-08-16T12:59:30.000Z", 25.0]
  ],
  "summary": {
    "count": 120,
    "min": 20.5,
    "max": 25.3,
    "avg": 22.8
  }
}
```

## Chart Viewer Function

The `createChartViewer` function creates interactive HTML charts for multiple sensors:

### Usage:

```javascript
createChartViewer({
  title: "My Sensors",
  data: [
    {
      id: "sensor1.id",
      label: "Temperature",
      color: "#FF0000",
      data: [[timestamp1, value1], [timestamp2, value2], ...]
    },
    {
      id: "sensor2.id", 
      label: "Humidity",
      color: "#00FF00",
      data: [[timestamp1, value1], [timestamp2, value2], ...]
    }
  ],
  width: 1200,
  height: 600
})
```

### Features:

- **Interactive Charts**: Zoom, Pan, Tooltips
- **Multiple Signals**: Up to 10 different sensors simultaneously
- **Time-based X-axis**: Automatic time formatting
- **Responsive Design**: Adapts to screen size
- **Color Coding**: Individual colors per sensor

### Workflow:

1. Call `getHistory()` for each sensor with `format: "simple"`
2. Pass data to `createChartViewer()`
3. Display interactive chart

## Testing the Connection

After configuration, you can test:

```
Show me the value of system.adapter.admin.0.alive
```

Should return `true` if ioBroker is running.

## Performance Tips

1. **Performance**: For better performance, use specific state IDs instead of patterns
2. **Security**: Use HTTPS and authentication in production environments
3. **Logging**: Check Cursor Developer Tools (Help → Toggle Developer Tools) for errors 