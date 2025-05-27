# 🚀 Python MCP Server Template

<div align="center">

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![MCP](https://img.shields.io/badge/MCP-Enabled-brightgreen.svg)
![Platform](https://img.shields.io/badge/Platform-Claude%20Desktop%20|%20Cursor%20|%20Windsurf%20|%20Cline-orange.svg)

**The fastest way to create Python-based MCP servers for Claude Desktop and AI IDEs!**

[📖 Claude Desktop Quick Start](./QUICKSTART_CLAUDE.md) | [🖥️ Full Integration Guide](./CLAUDE_DESKTOP_GUIDE.md)

</div>

## ✨ Why This Template?

- **⚡ Blazing Fast**: Build and run in seconds, not minutes
- **🔄 Multi-instance Friendly**: Run multiple Cursor windows on the same codebase without conflicts
- **🔌 Reliable Connections**: Stable connection handling compared to TypeScript alternatives
- **🛠️ Rapid Development**: Python implementation enables quick iterations and testing

## 🛑 TypeScript Server Challenges

When working with TypeScript-based MCP servers in multi-instance scenarios, numerous issues arise:

- **Port Conflicts**: TypeScript servers frequently conflict when multiple Cursor instances attempt to use the same ports
- **File Locking**: TypeScript build processes often lock files, preventing proper synchronization across instances
- **Resource Consumption**: TypeScript servers consume significantly more resources when running multiple instances
- **Compilation Delays**: Each TypeScript server instance triggers separate compilation processes, causing delays
- **Connection Instability**: Multiple TypeScript server instances compete for resources, leading to connection drops

This Python template eliminates these issues with its lightweight implementation, allowing seamless operation across multiple Cursor windows working on the same codebase simultaneously.

## 🔍 Context Reference

Take advantage of the example files in the `context` folder to enhance your MCP server implementation:

- **Deterministic Context**: Reference implementations that follow best practices
- **Example Patterns**: Common patterns for handling tools and client interactions
- **Ready-to-use Components**: Building blocks for creating robust MCP servers

Studying these context files will significantly accelerate your development process and help you build more reliable MCP servers.

## 🚦 Quick Setup

```bash
# 1. Install uv if you haven't already
pip install uv

# 2. Create virtual environment
uv venv

# 3. Activate virtual environment
# On Windows:
.venv\Scripts\activate
# On Unix/macOS:
source .venv/bin/activate

# 4. Install dependencies
uv pip install -e .

# 5. Run the server
python main.py
```

## ⚙️ Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| `--port` | Port to listen on | 8080 |
| `--transport` | Transport type (sse or stdio) | sse |

## 🖥️ Claude Desktop Integration

### What is MCP?

Model Context Protocol (MCP) is a protocol that enables AI assistants like Claude to interact with external tools and data sources. When you integrate an MCP server with Claude Desktop, you're giving Claude the ability to use custom tools that can:

- Access external APIs and databases
- Process and analyze data
- Interact with local files and systems
- Perform specialized computations
- Integrate with third-party services

### How MCP Works with Claude Desktop

1. **Server Registration**: You configure Claude Desktop to know about your MCP server
2. **Tool Discovery**: When Claude starts, it queries your server for available tools
3. **Tool Invocation**: When you ask Claude to perform a task, it can call your tools
4. **Response Handling**: Your server processes the request and returns results to Claude

### 📋 Step-by-Step Claude Desktop Setup

#### 1. Create Your MCP Server

First, implement your tools in `main.py`. Here's the structure:

```python
@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="your_tool_name",
            description="What this tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "param1": {"type": "string", "description": "Parameter description"}
                },
                "required": ["param1"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "your_tool_name":
        # Your tool logic here
        result = process_something(arguments["param1"])
        return [types.TextContent(type="text", text=str(result))]
```

#### 2. Create a Startup Script

Create `start-mcp.sh` (macOS/Linux):
```bash
#!/bin/bash
cd "$(dirname "$0")"
source .venv/bin/activate
python main.py --transport stdio
```

Create `start-mcp.bat` (Windows):
```batch
@echo off
cd /d "%~dp0"
.venv\Scripts\activate && python main.py --transport stdio
```

Make the script executable:
```bash
chmod +x start-mcp.sh  # macOS/Linux only
```

#### 3. Configure Claude Desktop

Find your Claude Desktop configuration file:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Add your MCP server configuration:

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "/path/to/your/start-mcp.sh",  // macOS/Linux
      // "command": "C:\\path\\to\\your\\start-mcp.bat",  // Windows
    }
  }
}
```

#### 4. Restart Claude Desktop

After saving the configuration, restart Claude Desktop. Your tools will now be available!

### 🔧 Troubleshooting Claude Desktop Integration

#### Common Issues and Solutions

1. **"spawn ENOENT" Error**
   - **Cause**: Path to script is incorrect or script isn't executable
   - **Solution**: Use absolute paths and ensure script has execute permissions

2. **"No module named..." Error**
   - **Cause**: Virtual environment not activated or dependencies not installed
   - **Solution**: Ensure your startup script activates the venv and dependencies are installed

3. **Server Disconnects Immediately**
   - **Cause**: Python errors in your server code
   - **Solution**: Test your server standalone first: `python main.py --transport stdio`

4. **Tools Not Appearing in Claude**
   - **Cause**: Server not properly registering tools
   - **Solution**: Check that `list_tools()` returns valid tool definitions

#### Debugging Tips

1. **Enable Developer Mode**:
   ```bash
   echo '{"allowDevTools": true}' > ~/Library/Application\ Support/Claude/developer_settings.json
   ```
   Then use `Cmd+Option+Shift+I` (macOS) to open DevTools in Claude Desktop.

2. **Check Logs**:
   ```bash
   # macOS
   tail -f ~/Library/Logs/Claude/mcp-server-*.log
   
   # Windows
   type %LOCALAPPDATA%\Claude\logs\mcp-server-*.log
   ```

3. **Test Your Server Standalone**:
   ```bash
   # This should output the MCP initialization sequence
   python main.py --transport stdio
   ```

### 🚀 Example: Building a Weather Tool

Here's a complete example of adding a weather tool to your MCP server:

```python
import aiohttp
from mcp.server import Server
import mcp.types as types

app = Server("weather-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="get_weather",
            description="Get current weather for a city",
            inputSchema={
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name (e.g., 'New York')"
                    }
                },
                "required": ["city"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "get_weather":
        city = arguments["city"]
        # Your weather API logic here
        weather_data = await fetch_weather(city)
        return [types.TextContent(
            type="text",
            text=f"Weather in {city}: {weather_data}"
        )]
    else:
        raise ValueError(f"Unknown tool: {name}")
```

### 📊 Real-World Example: SEC-MCP

The SEC-MCP project demonstrates a production-ready MCP server that provides:

- Company search capabilities
- Financial statement retrieval
- Real-time stock price data
- SEC filing downloads

Key implementation patterns from SEC-MCP:

1. **Async Client Management**:
   ```python
   async def get_client(self):
       if self.client is None:
           self.client = YourAPIClient()
           await self.client.__aenter__()
       return self.client
   ```

2. **Structured Response Formatting**:
   ```python
   return [types.TextContent(
       type="text",
       text=f"**{title}**\n\n{formatted_data}"
   )]
   ```

3. **Error Handling**:
   ```python
   try:
       result = await api_call()
       return format_result(result)
   except Exception as e:
       return [types.TextContent(
           type="text",
           text=f"Error: {str(e)}"
       )]
   ```

### 🎯 Best Practices

1. **Tool Naming**: Use clear, descriptive names (e.g., `get_stock_price`, not `gsp`)
2. **Descriptions**: Write detailed descriptions - Claude uses these to decide when to use tools
3. **Input Validation**: Always validate inputs in your tool handlers
4. **Error Messages**: Return helpful error messages that guide the user
5. **Response Formatting**: Use markdown for structured responses
6. **Async Operations**: Use async/await for all I/O operations
7. **Resource Cleanup**: Properly close connections and clean up resources

## 🏗️ Project Structure

```
your-mcp-server/
├── main.py              # Main server implementation
├── pyproject.toml       # Project dependencies
├── start-mcp.sh        # macOS/Linux startup script
├── start-mcp.bat       # Windows startup script
├── context/            # Reference implementations
│   ├── AICONTEXT-TOBUILD.MD
│   └── agent-theory.md
└── .venv/              # Virtual environment (created during setup)
```

## 📚 Additional Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io)
- [SEC-MCP Implementation](https://github.com/LuisRincon23/SEC-MCP) - Production example
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - For TypeScript implementations

## 🤝 Contributing

Contributions are welcome! Feel free to submit issues and pull requests to improve this template.

