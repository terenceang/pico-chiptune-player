# KiCad, EasyEDA & JLCPCB AI Toolchain Guide

A complete cross-platform guide to setting up AI-assisted electronics design tools across **Windows**, **macOS**, and **Linux**.

This toolchain gives AI assistants (Claude Desktop, Cursor, Antigravity / Gemini CLI, VS Code Cline) the ability to:
1. **KiCad MCP Server**: Create, edit, route, and run design checks (ERC / DRC) on schematics and PCB layouts.
2. **EasyEDA2KiCad CLI**: Convert any LCSC/EasyEDA part directly into KiCad symbols, footprints, and 3D models from the terminal.
3. **JLCPCB / LCSC Component Sourcing MCP**: Search 450K+ live parts, check stock/pricing/datasheets, and install parts directly into KiCad libraries.

---

## 1. Prerequisites

- **KiCad**: Version 9.0 or 10.0+ (installed with Python scripting support / `pcbnew` enabled).
- **Node.js**: v18.0.0 or higher ([nodejs.org](https://nodejs.org/)).
- **Python**: 3.10+ (KiCad bundles its own Python with `pcbnew`).
- **Package Managers**: `npm`, and optionally `uv` ([astral.sh/uv](https://astral.sh/uv)) or `pip`.

---

## 2. Tool 1: KiCad MCP Server (`mixelpixx/KiCAD-MCP-Server`)

Provides 240+ MCP tools for schematic capture, PCB layout, routing, DRC/ERC validation, and Gerber exports.

### Installation

#### Step 1: Clone the Repository
```bash
git clone --branch stable https://github.com/mixelpixx/KiCAD-MCP-Server.git
cd KiCAD-MCP-Server
```

#### Step 2: Install Node Dependencies & Build
```bash
npm install
npm run build
```
*(This produces the server bundle at `dist/index.js`)*

#### Step 3: Install Python Backend Dependencies
Install the required packages into KiCad's bundled Python environment so `pcbnew` is directly accessible:

- **Windows**:
  ```powershell
  & "C:\Program Files\KiCad\10.0\bin\python.exe" -m pip install -r requirements.txt
  ```
  *(Or run `.\setup-windows.ps1`)*

- **macOS**:
  ```bash
  /Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3 -m pip install -r requirements.txt
  ```

- **Linux**:
  ```bash
  python3 -m pip install -r requirements.txt
  ```

---

## 3. Tool 2: EasyEDA2KiCad CLI

Downloads symbols, footprints, and 3D models for any part using its LCSC Part Number (e.g. `C2040`).

### Installation

Choose either method:

- **Option A: Via `uv` (Recommended - fast and isolated)**:
  ```bash
  uv tool install easyeda2kicad
  ```

- **Option B: Via KiCad Python or Pip**:
  ```bash
  # Windows
  & "C:\Program Files\KiCad\10.0\bin\python.exe" -m pip install easyeda2kicad

  # macOS / Linux
  python3 -m pip install easyeda2kicad
  ```

### Common CLI Commands

```bash
# 1. Download full package (Symbol + Footprint + 3D model)
easyeda2kicad --full --lcsc_id C2040

# 2. Download symbol only into custom library
easyeda2kicad --symbol --lcsc_id C2040 --output my_library.kicad_sym

# 3. Download footprint only
easyeda2kicad --footprint --lcsc_id C2040

# 4. Use project-relative paths for 3D models and cache queries
easyeda2kicad --full --lcsc_id C2040 --project-relative --use-cache

# 5. Add custom BOM fields
easyeda2kicad --full --lcsc_id C2040 --custom-field 'Mfr:TI' 'Package:SOT-23'
```

---

## 4. Tool 3: JLCPCB / LCSC Parts Finder MCP (`@jlcpcb/mcp`)

Connects AI assistants to live JLCPCB and LCSC component catalogs with real-time stock, pricing tiers, and direct KiCad library installation. Also launches an interactive web UI at `http://localhost:3847`.

### Installation

Install globally via npm:
```bash
npm install -g @jlcpcb/mcp
```

### Available Tools Exposed to AI
- `component_search`: Search components by keyword across LCSC and EasyEDA community parts.
- `library_install`: Directly install an LCSC part (e.g., `C2040`) into KiCad libraries (`~/Documents/KiCad/{version}/3rdparty/jlc_mcp/`).
- `library_batch_install`: Install up to 10 parts simultaneously.
- `library_get_component`: Inspect installed component pins, pads, and footprints.
- `library_update`: Regenerate and update local component libraries.
- `library_fix`: Fix pin assignments, swap pins, or add exposed ground pads.

---

## 5. MCP Client Configuration (`mcp_config.json`)

Add the servers to your client configuration file:
- **Claude Desktop (Windows)**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Claude Desktop (macOS)**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Claude Desktop (Linux)**: `~/.config/Claude/claude_desktop_config.json`
- **Antigravity / Gemini CLI**: `~/.gemini/config/mcp_config.json`
- **Cursor**: `~/.cursor/mcp.json` or `.cursor/mcp.json` in workspace

### Windows Configuration Template

```json
{
  "mcpServers": {
    "kicad": {
      "command": "C:\\Program Files\\nodejs\\node.exe",
      "args": [
        "C:\\Users\\<USERNAME>\\source\\repos\\KiCAD-MCP-Server\\dist\\index.js"
      ],
      "env": {
        "KICAD_PYTHON": "C:\\Program Files\\KiCad\\10.0\\bin\\python.exe",
        "PYTHONPATH": "C:\\Program Files\\KiCad\\10.0\\bin\\Lib\\site-packages",
        "NODE_ENV": "production",
        "LOG_LEVEL": "info"
      }
    },
    "jlcpcb": {
      "command": "C:\\Program Files\\nodejs\\node.exe",
      "args": [
        "C:\\Users\\<USERNAME>\\AppData\\Roaming\\npm\\node_modules\\@jlcpcb\\mcp\\dist\\index.js"
      ]
    }
  }
}
```

### macOS Configuration Template

```json
{
  "mcpServers": {
    "kicad": {
      "command": "/usr/local/bin/node",
      "args": [
        "/Users/<USERNAME>/KiCAD-MCP-Server/dist/index.js"
      ],
      "env": {
        "KICAD_PYTHON": "/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3",
        "NODE_ENV": "production",
        "LOG_LEVEL": "info"
      }
    },
    "jlcpcb": {
      "command": "/usr/local/bin/node",
      "args": [
        "/usr/local/lib/node_modules/@jlcpcb/mcp/dist/index.js"
      ]
    }
  }
}
```

*(Note: On Apple Silicon / Homebrew, replace `/usr/local/bin/node` with `/opt/homebrew/bin/node`)*

### Linux Configuration Template

```json
{
  "mcpServers": {
    "kicad": {
      "command": "/usr/bin/node",
      "args": [
        "/home/<USERNAME>/KiCAD-MCP-Server/dist/index.js"
      ],
      "env": {
        "KICAD_PYTHON": "/usr/bin/python3",
        "NODE_ENV": "production",
        "LOG_LEVEL": "info"
      }
    },
    "jlcpcb": {
      "command": "/usr/bin/node",
      "args": [
        "/usr/lib/node_modules/@jlcpcb/mcp/dist/index.js"
      ]
    }
  }
}
```

---

## 6. Verification Checklist

1. **Verify KiCad MCP**:
   Ask the AI:
   > *"What tools are available for KiCad?"*
   >
   > *"Create a new KiCad project named test_board and add a 10k resistor to the schematic."*

2. **Verify Component Search**:
   Ask the AI:
   > *"Find a basic 3.3V LDO regulator on JLCPCB in SOT-23 package with high stock."*

3. **Verify EasyEDA2KiCad CLI**:
   Run in terminal:
   ```bash
   easyeda2kicad --full --lcsc_id C2040
   ```
