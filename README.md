# macOS Tools MCP Server

[![smithery badge](https://smithery.ai/badge/@tornikegomareli/macos-tools-mcp-server)](https://smithery.ai/server/@tornikegomareli/macos-tools-mcp-server)

A MCP server for macOS that provides advanced system monitoring and file search capabilities.

## Table of Contents

- [Features](#features)
  - [System Performance Monitor](#-system-performance-monitor)
  - [Enhanced File Search](#-enhanced-file-search)
- [Installation](#installation)
  - [Prerequisites](#prerequisites)
  - [Setup](#setup)
  - [Permissions](#permissions)
- [Usage](#usage)
  - [Starting the Server](#starting-the-server)
  - [Claude Desktop Configuration](#claude-desktop-configuration)
- [Available Tools](#available-tools)
  - [system_performance](#system_performance)
  - [enhanced_search](#enhanced_search)
- [Architecture](#architecture)
  - [Project Structure](#project-structure)
  - [Key Features](#key-features)
- [Troubleshooting](#troubleshooting)
- [Testing the Server](#testing-the-server)
  - [Quick Test Prompts](#quick-test-prompts)
  - [Comprehensive Test Suite](#comprehensive-test-suite)
- [Security Considerations](#security-considerations)
- [License](#license)
- [Future Enhancements](#future-enhancements)

## Features

### System Performance Monitor
- **Real-time Monitoring**: Track CPU, memory, disk I/O, and network statistics
- **Process Analysis**: View top resource-consuming processes with detailed metrics
- **Historical Tracking**: Store and analyze performance data over time using SQLite
- **Optimization Suggestions**: Get intelligent recommendations to improve system performance

### Enhanced File Search
- **Deep Content Search**: Search within file contents using regex or plain text
- **Spotlight Integration**: Leverage macOS Spotlight for fast metadata searches
- **Tag Management**: Create, search, and manage custom file tags using extended attributes
- **Advanced Features**: Fuzzy matching, boolean operators, and file type filtering

## Installation

### Installing via Smithery

To install macOS Tools for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@tornikegomareli/macos-tools-mcp-server):

```bash
npx -y @smithery/cli install @tornikegomareli/macos-tools-mcp-server --client claude
```

### Prerequisites
- macOS 10.15 or later
- Node.js 18.0.0 or later
- npm or yarn package manager

### Setup

#### Option 1: NPM Install (Recommended)

The easiest way to use this MCP server is through npm/npx:

```bash
# No installation needed, just add to Claude Desktop config
# See Claude Desktop Configuration section below
```

#### Option 2: From Source

1. Clone the repository:
```bash
git clone https://github.com/tornikegomareli/macos-tools-mcp-server.git
cd macos-tools-mcp-server
```

2. Install dependencies:
```bash
npm install
```

3. Build the project:
```bash
npm run build
```

4. Make the script executable:
```bash
chmod +x dist/index.js
```

### Permissions

For full functionality, the server requires certain permissions:

1. **Full Disk Access** (recommended for file search):
   - System Preferences → Security & Privacy → Privacy → Full Disk Access
   - Add Terminal or your IDE

2. **Developer Tools** (for process monitoring):
   - Install Xcode Command Line Tools if not already installed:
   ```bash
   xcode-select --install
   ```

## Usage

### Starting the Server

Run the MCP server:
```bash
npm start
```

Or use it directly:
```bash
node dist/index.js
```

### Claude Desktop Configuration

To use this MCP server with Claude Desktop:

1. Open your Claude Desktop configuration file:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`

2. Add the macOS Tools server to your configuration:

#### Using NPM (Recommended)
```json
{
  "mcpServers": {
    "macos-tools": {
      "command": "npx",
      "args": [
        "@tgomareli/macos-tools-mcp"
      ],
      "env": {}
    }
  }
}
```

#### Using Local Installation
If you cloned and built from source:
```json
{
  "mcpServers": {
    "macos-tools": {
      "command": "node",
      "args": [
        "/path/to/macos-tools-mcp-server/dist/index.js"
      ],
      "env": {}
    }
  }
}
```

4. If you already have other MCP servers configured, add the macos-tools configuration to the existing mcpServers object:

```json
{
  "mcpServers": {
    "existing-server": {
      // ... existing configuration
    },
    "macos-tools": {
      "command": "node",
      "args": [
        "/Users/tornikegomareli/Development/macos-tools-mcp/dist/index.js"
      ],
      "env": {}
    }
  }
}
```

5. Save the configuration file and restart Claude Desktop.

6. You should now see the macOS Tools server available in Claude Desktop with two tools:
   - `system_performance` - Monitor system resources
   - `enhanced_search` - Advanced file search and tagging

## Available Tools

### system_performance

Monitor and analyze system performance metrics.

**Parameters:**
- `action` (required): "current" | "history" | "processes" | "optimize"
- `timeRange` (optional): Time range for historical data ("1h", "24h", "7d")
- `metric` (optional): Specific metric to analyze ("cpu", "memory", "disk", "network", "all")

**Examples:**

```typescript
await callTool("system_performance", {
  action: "current",
  metric: "all"
});

await callTool("system_performance", {
  action: "history",
  timeRange: "24h",
  metric: "memory"
});

await callTool("system_performance", {
  action: "processes",
  metric: "cpu"
});

await callTool("system_performance", {
  action: "optimize"
});
```

### enhanced_search

Advanced file search with content analysis and tagging capabilities.

**Parameters:**
- `action` (required): "search" | "tag" | "untag"
- `query` (optional): Search query (supports regex)
- `searchType` (optional): "content" | "filename" | "tags" | "regex"
- `fileTypes` (optional): Array of file extensions to include
- `path` (optional): Root directory for search
- `maxResults` (optional): Maximum number of results
- `tags` (optional): Array of tags to search for or apply

**Examples:**

```typescript
// Search for TODO comments in code files
await callTool("enhanced_search", {
  action: "search",
  query: "TODO|FIXME",
  searchType: "regex",
  fileTypes: ["js", "ts", "py"],
  path: "~/Projects"
});

await callTool("enhanced_search", {
  action: "tag",
  path: "~/Documents/important.pdf",
  tags: ["urgent", "project-x"]
});

await callTool("enhanced_search", {
  action: "search",
  searchType: "tags",
  tags: ["urgent"],
  path: "~/Documents"
});

await callTool("enhanced_search", {
  action: "search",
  query: "apiKey",
  searchType: "content",
  fileTypes: ["json", "env"],
  path: "~/Projects",
  maxResults: 50
});
```

## Architecture

### Project Structure
```
macos-tools-mcp/
├── src/
│   ├── index.ts              # MCP server entry point
│   ├── tools/
│   │   ├── performance-monitor.ts
│   │   ├── spotlight-enhanced.ts
│   │   └── types.ts
│   └── utils/
│       ├── system-info.ts    # macOS system calls
│       ├── file-search.ts    # File search utilities
│       └── cache.ts          # Performance caching
├── dist/                     # Compiled JavaScript
├── package.json
├── tsconfig.json
└── README.md
```

### Key Features

1. **Native macOS Integration**
   - Uses native commands (ps, mdfind, xattr) for optimal performance
   - Leverages Spotlight index for fast searches
   - Supports macOS extended attributes for tagging

2. **Performance Optimization**
   - Multi-level caching system
   - Debounced operations
   - Rate limiting for system calls
   - Streaming for large files

3. **Data Persistence**
   - SQLite database for performance history
   - Configurable data retention
   - Efficient time-series queries

## Troubleshooting

### Common Issues

1. **Permission Denied Errors**
   - Ensure the terminal has Full Disk Access
   - Some operations may require sudo (temperature monitoring)

2. **Spotlight Not Finding Files**
   - Check if Spotlight indexing is enabled
   - Verify the search path is not excluded from Spotlight

3. **High CPU Usage**
   - Adjust cache TTL values in cache.ts
   - Limit search depth and file types
   - Use more specific search queries

### Debug Mode

Enable debug logging by setting the environment variable:
```bash
DEBUG=macos-tools-mcp node dist/index.js
```

## Security Considerations

- All shell commands are properly escaped to prevent injection
- File paths are validated and normalized
- Sensitive directories can be excluded via configuration
- Rate limiting prevents resource exhaustion

## License

MIT License - see LICENSE file for details

## Testing the Server

Once you've configured the server in Claude Desktop, you can use these prompts to test all functionality:

### Quick Test Prompts

1. **Basic Performance Check**:
   ```
   Show me my current system performance metrics
   ```

2. **Process Analysis**:
   ```
   What are the top 5 CPU-consuming processes on my system?
   ```

3. **File Search**:
   ```
   Search for TODO comments in JavaScript files in my current directory
   ```

4. **System Optimization**:
   ```
   Analyze my system and suggest performance optimizations
   ```

### Comprehensive Test Suite

Use this comprehensive prompt to test all features:

```
I want to test the macOS Tools MCP server. Please help me:

1. **System Performance Testing**:
   - Show me the current system performance metrics (CPU, memory, disk, network)
   - Display the top 5 processes consuming the most CPU
   - Show me memory usage history for the last hour
   - Analyze my system and provide optimization suggestions
   - Get the top memory-consuming processes

2. **File Search Testing**:
   - Search for all JavaScript and TypeScript files containing "TODO" or "FIXME" comments in my home directory
   - Find all files with "test" in their filename in the current project directory
   - Search for files containing the word "password" in configuration files (.json, .env, .yml)
   - Use regex to find all email addresses in text files
   - Search for files modified in the last 24 hours

3. **Tag Management Testing**:
   - Tag this file with "important" and "reviewed": /Users/tornikegomareli/Development/macos-tools-mcp/README.md
   - Search for all files tagged with "important"
   - Remove the "reviewed" tag from the README file
   - Tag all TypeScript files in the src directory with "source-code"

4. **Combined Operations**:
   - Monitor system performance while performing a large file search
   - Find resource-intensive processes and then search for their log files
   - Show current disk usage and search for large files (> 100MB)

5. **Edge Cases**:
   - Search in a non-existent directory
   - Try to tag a file I don't have permission to modify
   - Request performance data for an invalid time range
   - Search with an invalid regex pattern
```

### Expected Behaviors

- **Performance Monitor**: Should return real-time metrics, process lists, historical data, and optimization suggestions
- **File Search**: Should find files by content, name, or tags, with support for regex patterns
- **Tag Operations**: Should successfully add/remove tags and search by them
- **Error Handling**: Should gracefully handle permission errors, invalid paths, and malformed queries

## Future Enhancements

- [ ] GPU monitoring support
- [ ] Network connection analysis
- [ ] Application-specific performance tracking
- [ ] Integration with Time Machine for file version search
