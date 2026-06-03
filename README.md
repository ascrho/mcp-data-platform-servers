# MCP Data Platform Servers

> Model Context Protocol (MCP) servers that enable AI agents to interact with enterprise data platforms — Pentaho PDI, Power BI, Oracle, and OCR engines.

[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat-square&logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![MCP](https://img.shields.io/badge/MCP-Protocol-000000?style=flat-square&logo=anthropic&logoColor=white)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## What is MCP?

The [Model Context Protocol](https://modelcontextprotocol.io) is an open standard that allows AI assistants (Claude, Copilot, Cursor) to securely interact with external tools and data sources. Each MCP server exposes **tools** that an AI agent can call.

## Servers

### 1. Pentaho PDI Server

Interact with Pentaho Data Integration's repository and scheduler.

| Tool | Description |
|:-----|:------------|
| `list_folders` | Browse the Pentaho repository tree |
| `list_files` | List jobs (.kjb) and transformations (.ktr) |
| `get_file_content` | Download and parse job/transformation XML |
| `list_scheduled_jobs` | View Carte scheduler status |
| `get_full_tree` | Complete repository tree dump |

```bash
# Environment variables
PENTAHO_URL=http://your-pentaho:8080/pentaho
PENTAHO_USER=admin
PENTAHO_PASSWORD=changeme
```

### 2. Power BI Server

Query Power BI REST API through natural language via AI agents.

| Tool | Description |
|:-----|:------------|
| `list_workspaces` | List all Power BI workspaces |
| `list_datasets` | List datasets in a workspace |
| `list_tables` | Show tables in a dataset |
| `execute_dax` | Run DAX queries against a dataset |
| `list_reports` | List reports and dashboards |
| `list_gateways` | Show gateway configurations |
| `refresh_dataset` | Trigger dataset refresh |

```bash
# Environment variables
PBI_USERNAME=your-email@domain.com
PBI_PASSWORD=your-password
PBI_TENANT_ID=your-azure-tenant-id
```

### 3. Oracle Query Server

Execute read-only SQL queries against Oracle databases.

| Tool | Description |
|:-----|:------------|
| `execute_query` | Run SELECT statements with row limits |

```bash
# Environment variables
DB_USER=your-db-user
DB_PASSWORD=your-db-password
DB_TNS=your-tns-connection-string
```

### 4. OCR Server (EasyOCR + RapidOCR)

Extract text from images using two OCR backends.

| Tool | Description |
|:-----|:------------|
| `ocr_extract` | Extract text from image (file, URL, or base64) |
| `ocr_healthcheck` | Verify OCR engine availability |

Supports: `easyocr` (GPU-accelerated) and `rapidocr_onnxruntime` (CPU-optimized)

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    AI Agent                         │
│              (Claude / Cursor / Copilot)            │
└───────┬──────────┬──────────┬──────────┬────────────┘
        │          │          │          │
   ┌────▼───┐ ┌───▼────┐ ┌───▼───┐ ┌───▼────┐
   │Pentaho │ │Power BI│ │Oracle │ │  OCR   │
   │MCP Srv │ │MCP Srv │ │MCP Srv│ │MCP Srv │
   └────┬───┘ └───┬────┘ └───┬───┘ └───┬────┘
        │         │          │         │
   ┌────▼───┐ ┌───▼────┐ ┌───▼───┐ ┌───▼────────┐
   │Pentaho │ │Power BI│ │Oracle │ │EasyOCR /   │
   │REST API│ │REST API│ │  DB   │ │RapidOCR    │
   └────────┘ └────────┘ └───────┘ └────────────┘
```

## Setup

### Prerequisites

- Node.js 18+
- Python 3.10+ (for OCR servers)

### Installation

```bash
git clone https://github.com/ascrho/mcp-data-platform-servers.git
cd mcp-data-platform-servers

# Install Node.js dependencies
npm install

# Install Python OCR dependencies (optional)
pip install easyocr opencv-python-headless numpy requests
# OR
pip install rapidocr-onnxruntime opencv-python-headless numpy requests
```

### Configuration

```bash
cp .env.example .env
# Edit .env with your connection details
```

### Running

Each server can run independently via stdio (for MCP client integration):

```bash
# Pentaho server
node pentaho_server.js

# Power BI server
node powerbi_server.js

# Oracle server
node oracle_server.js

# OCR server (delegates to Python runner)
node ocr_server.js
```

### Cursor/Claude Integration

Add to your MCP configuration:

```json
{
  "mcpServers": {
    "pentaho": {
      "command": "node",
      "args": ["path/to/pentaho_server.js"],
      "env": {
        "PENTAHO_URL": "http://your-pentaho:8080/pentaho",
        "PENTAHO_USER": "admin",
        "PENTAHO_PASSWORD": "changeme"
      }
    }
  }
}
```

## Project Structure

```
mcp-data-platform-servers/
├── pentaho_server.js       # Pentaho PDI MCP server
├── powerbi_server.js       # Power BI MCP server
├── oracle_server.js        # Oracle query MCP server
├── ocr_server.js           # OCR MCP server (EasyOCR)
├── ocr_paddle_server.js    # OCR MCP server (RapidOCR)
├── easyocr_runner.py       # EasyOCR Python backend
├── paddle_ocr_runner.py    # RapidOCR Python backend
├── package.json
├── .env.example
└── README.md
```

## Technologies

| Component | Technology |
|-----------|-----------|
| MCP SDK | `@modelcontextprotocol/sdk` |
| Validation | `zod` |
| Auth (PBI) | `@azure/msal-node` (ROPC flow) |
| Oracle | `oracledb` (Node.js driver) |
| OCR | EasyOCR, RapidOCR (ONNX Runtime) |
| Transport | stdio (MCP standard) |

## License

MIT License — see [LICENSE](LICENSE) for details.

## Author

**Marcos Quintero** — Data Engineer  
[GitHub](https://github.com/MarcosQuintero) | [LinkedIn](https://www.linkedin.com/in/marcosquinterero-dataengineer/)
