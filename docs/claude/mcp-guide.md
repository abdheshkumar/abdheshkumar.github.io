# MCP (Model Context Protocol) - Complete Guide

## What is MCP?

**MCP (Model Context Protocol)** is a standard protocol that lets Claude Code connect to external services and data sources.

### Simple Analogy

Think of MCP like **USB ports on your computer**:
- Your computer (Claude Code) has USB ports (MCP protocol)
- You can plug in devices (MCP servers) like keyboards, mice, printers
- Each device gives your computer new capabilities

```mermaid
graph LR
    CLAUDE[🤖 Claude Code<br/>Has MCP Protocol]

    subgraph SERVERS[MCP Servers - Like USB Devices]
        DB[🗄️ Database Server<br/>Query data]
        SLACK[💬 Slack Server<br/>Send messages]
        GITHUB[🐙 GitHub Server<br/>Manage repos]
        BROWSER[🌐 Browser Server<br/>Control Chrome]
    end

    CLAUDE <-->|MCP Protocol| DB
    CLAUDE <-->|MCP Protocol| SLACK
    CLAUDE <-->|MCP Protocol| GITHUB
    CLAUDE <-->|MCP Protocol| BROWSER

    style CLAUDE fill:#c8e6c9
    style DB fill:#ffab91
    style SLACK fill:#ffab91
    style GITHUB fill:#ffab91
    style BROWSER fill:#ffab91
```

### Why MCP Exists

**Problem:** Claude Code runs in your terminal and can only access local files and commands.

**Solution:** MCP lets Claude connect to:
- Databases (PostgreSQL, MySQL, MongoDB)
- Communication tools (Slack, Discord, Email)
- Project management (Jira, Linear, GitHub Issues)
- Cloud services (AWS, Google Cloud)
- Web browsers (for testing and automation)
- Custom APIs and services

---

## How MCP Works

### Architecture Overview

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant MCP Server
    participant External Service

    Note over Claude,MCP Server: MCP Server starts when<br/>Claude Code starts

    You->>Claude: "How many users are in the database?"
    Note over Claude: Claude knows it can use<br/>database tools from MCP

    Claude->>MCP Server: query_database("SELECT COUNT(*) FROM users")
    MCP Server->>External Service: Execute query
    External Service->>MCP Server: Result: 1,234 users
    MCP Server->>Claude: 1,234
    Claude->>You: "There are 1,234 users in the database"
```

### Three Parts of MCP

```mermaid
graph TB
    subgraph CLIENT[1. MCP Client - Built into Claude Code]
        C1[Manages connections]
        C2[Sends requests]
        C3[Receives responses]
    end

    subgraph SERVER[2. MCP Server - Program you write or install]
        S1[Implements tools]
        S2[Handles requests]
        S3[Returns results]
    end

    subgraph SERVICE[3. External Service - What you connect to]
        E1[Database]
        E2[API]
        E3[Web service]
    end

    CLIENT <-->|MCP Protocol| SERVER
    SERVER <-->|Service-specific protocol| SERVICE

    style CLIENT fill:#c8e6c9
    style SERVER fill:#ffab91
    style SERVICE fill:#bbdefb
```

---

## MCP Server Capabilities

MCP servers can provide three types of capabilities:

### 1. Tools (Functions Claude Can Call)

```mermaid
graph LR
    CLAUDE[🤖 Claude]

    subgraph TOOLS[Tools Provided by MCP]
        T1[query_database]
        T2[send_slack_message]
        T3[create_jira_ticket]
        T4[search_documents]
    end

    CLAUDE -->|"Can call"| T1
    CLAUDE -->|"Can call"| T2
    CLAUDE -->|"Can call"| T3
    CLAUDE -->|"Can call"| T4

    style CLAUDE fill:#c8e6c9
    style T1 fill:#ce93d8
    style T2 fill:#ce93d8
    style T3 fill:#ce93d8
    style T4 fill:#ce93d8
```

**Example:** Database MCP server provides `query_database()` tool

### 2. Resources (Data Claude Can Read)

```mermaid
graph LR
    CLAUDE[🤖 Claude]

    subgraph RESOURCES[Resources Provided by MCP]
        R1[database://schema]
        R2[slack://channels]
        R3[jira://projects]
        R4[docs://api-reference]
    end

    CLAUDE -->|"Can read"| R1
    CLAUDE -->|"Can read"| R2
    CLAUDE -->|"Can read"| R3
    CLAUDE -->|"Can read"| R4

    style CLAUDE fill:#c8e6c9
    style R1 fill:#80cbc4
    style R2 fill:#80cbc4
    style R3 fill:#80cbc4
    style R4 fill:#80cbc4
```

**Example:** Database MCP server exposes `database://schema` resource with table information

### 3. Prompts (Pre-built Templates)

```mermaid
graph LR
    CLAUDE[🤖 Claude]

    subgraph PROMPTS[Prompts Provided by MCP]
        P1[sql-query-template]
        P2[bug-report-template]
        P3[code-review-template]
    end

    CLAUDE -->|"Can use"| P1
    CLAUDE -->|"Can use"| P2
    CLAUDE -->|"Can use"| P3

    style CLAUDE fill:#c8e6c9
    style P1 fill:#fff59d
    style P2 fill:#fff59d
    style P3 fill:#fff59d
```

**Example:** SQL template for common query patterns

---

## Creating Your First MCP Server

MCP servers can be written in any language. The most common choices are:
- **Python** (easiest, recommended for beginners)
- **TypeScript/JavaScript** (for Node.js)
- **Go, Rust, Java** (for high-performance needs)

### Example 1: Simple Calculator MCP Server (Python)

Let's create a simple MCP server that provides calculator tools.

#### Step 1: Install MCP SDK

```bash
pip install mcp
```

#### Step 2: Create the Server

**File: `calculator_server.py`**

```python
#!/usr/bin/env python3
"""
Simple Calculator MCP Server
Provides basic math operations to Claude
"""

from mcp.server import Server
from mcp.types import Tool, TextContent
import json

# Create MCP server instance
app = Server("calculator-server")

# Define the tools (functions) Claude can use
@app.list_tools()
async def list_tools() -> list[Tool]:
    """Tell Claude what tools are available"""
    return [
        Tool(
            name="add",
            description="Add two numbers together",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "First number"},
                    "b": {"type": "number", "description": "Second number"}
                },
                "required": ["a", "b"]
            }
        ),
        Tool(
            name="multiply",
            description="Multiply two numbers",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "First number"},
                    "b": {"type": "number", "description": "Second number"}
                },
                "required": ["a", "b"]
            }
        ),
        Tool(
            name="calculate",
            description="Evaluate a mathematical expression",
            inputSchema={
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Math expression like '2 + 2' or '10 * 5'"
                    }
                },
                "required": ["expression"]
            }
        )
    ]

# Implement the tool logic
@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    """Execute the tool Claude requested"""

    if name == "add":
        result = arguments["a"] + arguments["b"]
        return [TextContent(type="text", text=str(result))]

    elif name == "multiply":
        result = arguments["a"] * arguments["b"]
        return [TextContent(type="text", text=str(result))]

    elif name == "calculate":
        try:
            # Safe evaluation of math expression
            result = eval(arguments["expression"], {"__builtins__": {}})
            return [TextContent(type="text", text=str(result))]
        except Exception as e:
            return [TextContent(type="text", text=f"Error: {str(e)}")]

    else:
        return [TextContent(type="text", text=f"Unknown tool: {name}")]

# Run the server
if __name__ == "__main__":
    import asyncio
    from mcp.server.stdio import stdio_server

    asyncio.run(stdio_server(app))
```

#### Step 3: Make it Executable

```bash
chmod +x calculator_server.py
```

#### Step 4: Configure in Claude Code

**File: `~/.config/claude/mcp.json`** (User-level)
Or **`.claude/mcp.json`** (Project-level)

```json
{
  "mcpServers": {
    "calculator": {
      "command": "python3",
      "args": ["/path/to/calculator_server.py"]
    }
  }
}
```

#### Step 5: Use It!

```bash
# Start Claude Code
claude

# Claude now has calculator tools available!
You: "What's 123 * 456?"
Claude: Using calculator tool... The result is 56,088
```

---

### Example 2: Real-World Database MCP Server

Let's create a more practical example - a PostgreSQL database connector.

**File: `database_server.py`**

```python
#!/usr/bin/env python3
"""
PostgreSQL MCP Server
Lets Claude query your database safely
"""

from mcp.server import Server
from mcp.types import Tool, Resource, TextContent
import psycopg2
import os
import json

app = Server("database-server")

# Database connection from environment variable
DB_URL = os.environ.get("DATABASE_URL", "postgresql://localhost/mydb")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="query_database",
            description="Execute a SELECT query on the database (read-only)",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "SQL SELECT query to execute"
                    }
                },
                "required": ["query"]
            }
        ),
        Tool(
            name="list_tables",
            description="List all tables in the database",
            inputSchema={
                "type": "object",
                "properties": {}
            }
        ),
        Tool(
            name="describe_table",
            description="Get the schema of a specific table",
            inputSchema={
                "type": "object",
                "properties": {
                    "table_name": {
                        "type": "string",
                        "description": "Name of the table"
                    }
                },
                "required": ["table_name"]
            }
        )
    ]

@app.list_resources()
async def list_resources() -> list[Resource]:
    """Expose database schema as a resource"""
    return [
        Resource(
            uri="database://schema",
            name="Database Schema",
            mimeType="application/json",
            description="Complete database schema with all tables and columns"
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> str:
    """Provide database schema information"""
    if uri == "database://schema":
        conn = psycopg2.connect(DB_URL)
        cursor = conn.cursor()

        # Get all tables
        cursor.execute("""
            SELECT table_name
            FROM information_schema.tables
            WHERE table_schema = 'public'
        """)
        tables = cursor.fetchall()

        schema = {}
        for (table_name,) in tables:
            cursor.execute(f"""
                SELECT column_name, data_type
                FROM information_schema.columns
                WHERE table_name = '{table_name}'
            """)
            columns = cursor.fetchall()
            schema[table_name] = [
                {"name": col, "type": typ} for col, typ in columns
            ]

        conn.close()
        return json.dumps(schema, indent=2)

    return "{}"

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    conn = psycopg2.connect(DB_URL)
    cursor = conn.cursor()

    try:
        if name == "query_database":
            query = arguments["query"]

            # Security: Only allow SELECT queries
            if not query.strip().upper().startswith("SELECT"):
                return [TextContent(
                    type="text",
                    text="Error: Only SELECT queries are allowed"
                )]

            cursor.execute(query)
            rows = cursor.fetchall()
            columns = [desc[0] for desc in cursor.description]

            # Format as table
            result = {"columns": columns, "rows": rows, "count": len(rows)}
            return [TextContent(type="text", text=json.dumps(result, indent=2))]

        elif name == "list_tables":
            cursor.execute("""
                SELECT table_name
                FROM information_schema.tables
                WHERE table_schema = 'public'
            """)
            tables = [row[0] for row in cursor.fetchall()]
            return [TextContent(type="text", text=json.dumps(tables, indent=2))]

        elif name == "describe_table":
            table_name = arguments["table_name"]
            cursor.execute(f"""
                SELECT column_name, data_type, is_nullable
                FROM information_schema.columns
                WHERE table_name = '{table_name}'
            """)
            columns = cursor.fetchall()
            schema = [
                {
                    "column": col,
                    "type": typ,
                    "nullable": nullable == "YES"
                }
                for col, typ, nullable in columns
            ]
            return [TextContent(type="text", text=json.dumps(schema, indent=2))]

    except Exception as e:
        return [TextContent(type="text", text=f"Error: {str(e)}")]

    finally:
        conn.close()

    return [TextContent(type="text", text="Unknown tool")]

if __name__ == "__main__":
    import asyncio
    from mcp.server.stdio import stdio_server

    asyncio.run(stdio_server(app))
```

#### Configuration

**File: `.claude/mcp.json`**

```json
{
  "mcpServers": {
    "database": {
      "command": "python3",
      "args": ["/path/to/database_server.py"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost/mydb"
      }
    }
  }
}
```

#### Usage

```bash
You: "What tables are in the database?"
Claude: *calls list_tables tool*
        You have 5 tables: users, orders, products, reviews, categories

You: "How many users signed up today?"
Claude: *calls query_database with SELECT query*
        42 users signed up today
```

---

## MCP Configuration

### Configuration Hierarchy

MCP servers can be configured at three levels:

```mermaid
graph TB
    LOCAL[🏠 Local Config<br/>.claude/mcp.json<br/>Highest priority]
    PROJECT[📁 Project Config<br/>project/.claude/mcp.json<br/>Medium priority]
    USER[👤 User Config<br/>~/.config/claude/mcp.json<br/>Lowest priority]

    LOCAL -->|Overrides| PROJECT
    PROJECT -->|Overrides| USER

    style LOCAL fill:#ffab91
    style PROJECT fill:#fff9c4
    style USER fill:#e1f5ff
```

### Configuration Format

**File: `mcp.json`**

```json
{
  "mcpServers": {
    "server-name": {
      "command": "executable",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### Example Configurations

#### Python Server

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python3",
      "args": ["/path/to/server.py"]
    }
  }
}
```

#### Node.js Server

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/server.js"]
    }
  }
}
```

#### With Environment Variables

```json
{
  "mcpServers": {
    "database": {
      "command": "python3",
      "args": ["/path/to/db_server.py"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb",
        "DATABASE_POOL_SIZE": "10"
      }
    }
  }
}
```

#### Multiple Servers

```json
{
  "mcpServers": {
    "database": {
      "command": "python3",
      "args": ["/path/to/db_server.py"]
    },
    "slack": {
      "command": "node",
      "args": ["/path/to/slack_server.js"],
      "env": {
        "SLACK_TOKEN": "xoxb-your-token"
      }
    },
    "jira": {
      "command": "/usr/local/bin/jira-mcp",
      "args": ["--config", "/path/to/jira.conf"]
    }
  }
}
```

---

## MCP Server Development Guide

### Project Structure

```
my-mcp-server/
├── server.py              # Main server code
├── requirements.txt       # Python dependencies
├── README.md             # Documentation
├── tests/                # Tests
│   └── test_server.py
└── examples/             # Example configurations
    └── mcp.json
```

### Basic Template

```python
#!/usr/bin/env python3
from mcp.server import Server
from mcp.types import Tool, Resource, TextContent
import json

# Create server
app = Server("my-server")

# Define tools
@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="my_tool",
            description="What this tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "param": {"type": "string", "description": "Parameter description"}
                },
                "required": ["param"]
            }
        )
    ]

# Implement tool logic
@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "my_tool":
        result = f"You passed: {arguments['param']}"
        return [TextContent(type="text", text=result)]

    return [TextContent(type="text", text="Unknown tool")]

# Optional: Define resources
@app.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="myserver://resource",
            name="My Resource",
            mimeType="text/plain",
            description="Resource description"
        )
    ]

# Optional: Implement resource reading
@app.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "myserver://resource":
        return "Resource content here"
    return ""

# Run server
if __name__ == "__main__":
    import asyncio
    from mcp.server.stdio import stdio_server

    asyncio.run(stdio_server(app))
```

### Tool Input Schema (JSON Schema)

Tools use JSON Schema to define parameters:

```python
# String parameter
{
    "type": "string",
    "description": "User's name",
    "minLength": 1,
    "maxLength": 100
}

# Number parameter
{
    "type": "number",
    "description": "User's age",
    "minimum": 0,
    "maximum": 150
}

# Boolean parameter
{
    "type": "boolean",
    "description": "Is active?"
}

# Enum (choices)
{
    "type": "string",
    "description": "User role",
    "enum": ["admin", "user", "guest"]
}

# Array parameter
{
    "type": "array",
    "description": "List of tags",
    "items": {"type": "string"}
}

# Object parameter
{
    "type": "object",
    "description": "User information",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "number"}
    },
    "required": ["name"]
}
```

---

## Installing Pre-built MCP Servers

Many MCP servers are already available. You don't need to build them from scratch.

### Popular MCP Servers

```mermaid
graph TB
    subgraph OFFICIAL[Official MCP Servers]
        O1[🗄️ SQLite<br/>Local database access]
        O2[🌐 Fetch<br/>Web requests]
        O3[📁 Filesystem<br/>Extended file ops]
        O4[🐙 GitHub<br/>Repository management]
    end

    subgraph COMMUNITY[Community MCP Servers]
        C1[💬 Slack<br/>Team communication]
        C2[📋 Jira<br/>Project management]
        C3[☁️ AWS<br/>Cloud services]
        C4[🐳 Docker<br/>Container management]
    end

    style O1 fill:#bbdefb
    style O2 fill:#bbdefb
    style O3 fill:#bbdefb
    style O4 fill:#bbdefb
    style C1 fill:#ce93d8
    style C2 fill:#ce93d8
    style C3 fill:#ce93d8
    style C4 fill:#ce93d8
```

### Installation Examples

#### 1. SQLite MCP Server

```bash
# Install
npm install -g @modelcontextprotocol/server-sqlite

# Configure
{
  "mcpServers": {
    "sqlite": {
      "command": "mcp-server-sqlite",
      "args": ["/path/to/database.db"]
    }
  }
}
```

#### 2. GitHub MCP Server

```bash
# Install
npm install -g @modelcontextprotocol/server-github

# Configure
{
  "mcpServers": {
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

#### 3. Slack MCP Server

```bash
# Install
npm install -g @slack/mcp-server

# Configure
{
  "mcpServers": {
    "slack": {
      "command": "slack-mcp-server",
      "env": {
        "SLACK_TOKEN": "xoxb-your-token",
        "SLACK_WORKSPACE": "your-workspace"
      }
    }
  }
}
```

---

## Managing MCP Servers

### Check Server Status

```bash
# In Claude Code
/mcp

# Output shows:
# - Connected servers
# - Available tools
# - Token costs
```

### Debugging MCP Servers

#### Enable Debug Logging

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python3",
      "args": ["/path/to/server.py"],
      "env": {
        "LOG_LEVEL": "DEBUG"
      }
    }
  }
}
```

#### Test Server Manually

```bash
# Run server directly to see errors
python3 server.py

# Or use MCP inspector
npx @modelcontextprotocol/inspector python3 server.py
```

### Common Issues

```mermaid
graph TD
    ISSUE{MCP Server<br/>Not Working?}

    ISSUE -->|Server won't start| CHECK1[Check command path<br/>and permissions]
    ISSUE -->|Tools not showing| CHECK2[Check list_tools()<br/>implementation]
    ISSUE -->|Connection lost| CHECK3[Check for crashes<br/>in server logs]
    ISSUE -->|Slow responses| CHECK4[Check server<br/>performance]

    style ISSUE fill:#ffab91
    style CHECK1 fill:#fff9c4
    style CHECK2 fill:#fff9c4
    style CHECK3 fill:#fff9c4
    style CHECK4 fill:#fff9c4
```

---

## Best Practices

### 1. Security

```python
# ✅ DO: Validate and sanitize inputs
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "query_database":
        query = arguments["query"].strip()

        # Only allow SELECT queries
        if not query.upper().startswith("SELECT"):
            raise ValueError("Only SELECT queries allowed")

        # Use parameterized queries
        cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])

# ❌ DON'T: Directly execute user input
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")  # SQL injection!
```

### 2. Error Handling

```python
# ✅ DO: Return helpful error messages
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    try:
        result = do_something(arguments)
        return [TextContent(type="text", text=str(result))]
    except ConnectionError as e:
        return [TextContent(type="text", text=f"Connection failed: {e}")]
    except ValueError as e:
        return [TextContent(type="text", text=f"Invalid input: {e}")]
    except Exception as e:
        return [TextContent(type="text", text=f"Error: {e}")]
```

### 3. Performance

```python
# ✅ DO: Use connection pooling for databases
from psycopg2 import pool

connection_pool = pool.SimpleConnectionPool(1, 20, DATABASE_URL)

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    conn = connection_pool.getconn()
    try:
        # Use connection
        pass
    finally:
        connection_pool.putconn(conn)
```

### 4. Documentation

```python
# ✅ DO: Write clear descriptions
Tool(
    name="send_email",
    description="Send an email to a user. The email will be sent immediately. "
                "Requires valid email address and non-empty subject and body.",
    inputSchema={
        "type": "object",
        "properties": {
            "to": {
                "type": "string",
                "description": "Recipient email address (e.g., user@example.com)"
            },
            "subject": {
                "type": "string",
                "description": "Email subject line (max 200 characters)"
            },
            "body": {
                "type": "string",
                "description": "Email body content (plain text or HTML)"
            }
        },
        "required": ["to", "subject", "body"]
    }
)
```

---

## Complete Example: Expedia Internal Service MCP

Let's create an MCP server for an Expedia internal service.

**File: `expedia_service_mcp.py`**

```python
#!/usr/bin/env python3
"""
Expedia Internal Service MCP Server
Connects to internal Expedia services
"""

from mcp.server import Server
from mcp.types import Tool, Resource, TextContent
import requests
import os
import json

app = Server("expedia-service")

# Configuration from environment
SERVICE_URL = os.environ.get("EXPEDIA_SERVICE_URL", "https://internal.expedia.com/api")
API_KEY = os.environ.get("EXPEDIA_API_KEY", "")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="get_hotel_inventory",
            description="Get current hotel inventory for a location",
            inputSchema={
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "City or region"},
                    "checkin": {"type": "string", "description": "Check-in date (YYYY-MM-DD)"},
                    "checkout": {"type": "string", "description": "Check-out date (YYYY-MM-DD)"}
                },
                "required": ["location", "checkin", "checkout"]
            }
        ),
        Tool(
            name="get_booking_stats",
            description="Get booking statistics for a date range",
            inputSchema={
                "type": "object",
                "properties": {
                    "start_date": {"type": "string", "description": "Start date (YYYY-MM-DD)"},
                    "end_date": {"type": "string", "description": "End date (YYYY-MM-DD)"},
                    "region": {"type": "string", "description": "Optional region filter"}
                },
                "required": ["start_date", "end_date"]
            }
        ),
        Tool(
            name="search_properties",
            description="Search for properties by name or ID",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"},
                    "limit": {"type": "number", "description": "Max results (default 10)"}
                },
                "required": ["query"]
            }
        )
    ]

@app.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="expedia://regions",
            name="Available Regions",
            mimeType="application/json",
            description="List of all supported regions"
        ),
        Resource(
            uri="expedia://property-types",
            name="Property Types",
            mimeType="application/json",
            description="List of property types (hotel, resort, etc.)"
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> str:
    headers = {"Authorization": f"Bearer {API_KEY}"}

    if uri == "expedia://regions":
        response = requests.get(f"{SERVICE_URL}/regions", headers=headers)
        return response.text

    elif uri == "expedia://property-types":
        response = requests.get(f"{SERVICE_URL}/property-types", headers=headers)
        return response.text

    return "{}"

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }

    try:
        if name == "get_hotel_inventory":
            response = requests.post(
                f"{SERVICE_URL}/inventory/hotels",
                json=arguments,
                headers=headers
            )
            response.raise_for_status()
            return [TextContent(type="text", text=response.text)]

        elif name == "get_booking_stats":
            response = requests.post(
                f"{SERVICE_URL}/analytics/bookings",
                json=arguments,
                headers=headers
            )
            response.raise_for_status()
            data = response.json()

            # Format nicely
            summary = f"""
Booking Statistics ({arguments['start_date']} to {arguments['end_date']})
- Total Bookings: {data['total_bookings']}
- Revenue: ${data['total_revenue']:,.2f}
- Average Booking Value: ${data['avg_booking_value']:,.2f}
- Top Region: {data['top_region']}
            """.strip()

            return [TextContent(type="text", text=summary)]

        elif name == "search_properties":
            limit = arguments.get("limit", 10)
            response = requests.get(
                f"{SERVICE_URL}/properties/search",
                params={"q": arguments["query"], "limit": limit},
                headers=headers
            )
            response.raise_for_status()
            return [TextContent(type="text", text=response.text)]

    except requests.HTTPError as e:
        return [TextContent(type="text", text=f"API Error: {e.response.status_code} - {e.response.text}")]
    except Exception as e:
        return [TextContent(type="text", text=f"Error: {str(e)}")]

    return [TextContent(type="text", text="Unknown tool")]

if __name__ == "__main__":
    import asyncio
    from mcp.server.stdio import stdio_server

    asyncio.run(stdio_server(app))
```

**Configuration: `.claude/mcp.json`**

```json
{
  "mcpServers": {
    "expedia": {
      "command": "python3",
      "args": ["/path/to/expedia_service_mcp.py"],
      "env": {
        "EXPEDIA_SERVICE_URL": "https://internal.expedia.com/api",
        "EXPEDIA_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

**Usage:**

```bash
You: "What's the hotel inventory in Seattle for next weekend?"
Claude: *calls get_hotel_inventory*
        Found 234 hotels available in Seattle for those dates...

You: "Show me booking stats for last month"
Claude: *calls get_booking_stats*
        Booking Statistics (2024-01-01 to 2024-01-31)
        - Total Bookings: 45,678
        - Revenue: $12,345,678.90
        ...
```

---

## Resources

### Official Documentation
- **MCP Specification**: https://modelcontextprotocol.io/
- **Python SDK**: https://github.com/modelcontextprotocol/python-sdk
- **TypeScript SDK**: https://github.com/modelcontextprotocol/typescript-sdk

### Example Servers
- **Official Examples**: https://github.com/modelcontextprotocol/servers
- **Community Servers**: https://github.com/topics/mcp-server

### Tools
- **MCP Inspector**: Debug and test MCP servers
  ```bash
  npx @modelcontextprotocol/inspector python3 server.py
  ```

---

## Summary

```mermaid
graph TB
    START[Need to connect Claude<br/>to external service?]

    START --> CHECK{Does an MCP<br/>server exist?}

    CHECK -->|Yes| INSTALL[Install & Configure]
    CHECK -->|No| BUILD[Build Your Own]

    INSTALL --> CONFIGURE[Add to mcp.json]
    BUILD --> CHOOSE{Choose Language}

    CHOOSE -->|Python| PY[Use Python SDK]
    CHOOSE -->|Node.js| JS[Use TypeScript SDK]
    CHOOSE -->|Other| OTHER[Implement protocol]

    PY --> IMPLEMENT[Implement Tools]
    JS --> IMPLEMENT
    OTHER --> IMPLEMENT

    IMPLEMENT --> CONFIGURE
    CONFIGURE --> USE[Use in Claude Code]

    style START fill:#e1f5ff
    style BUILD fill:#ce93d8
    style INSTALL fill:#c8e6c9
    style USE fill:#c8e6c9
```

### Key Takeaways

1. **MCP = Protocol** for connecting Claude to external services
2. **MCP Server = Program** that implements the protocol
3. **Tools** = Functions Claude can call
4. **Resources** = Data Claude can read
5. **Configuration** = mcp.json file tells Claude how to start servers
6. **Python SDK** makes it easy to build servers
7. **Many pre-built servers** exist - check before building

Happy MCP building! 🚀
